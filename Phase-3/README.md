# Configure Jenkins and CICD Pipeline
1. Configure Jenkins
2. Create CICD Pipeline
3. CICD + Mail Notification
---
### Step-1: Browse and login to the Jenkins Server: <jenkins_instance_ip:8080>
---

---
### Step-2: Install all necessary Plugins in Jenkins
---
##### Go to Jenkins dashboard -> Manage Jenkins -> Manage Plugins -> Available tab.

Search for each plugin by name.
Select and install each plugin without restarting.

- list of the all the necessary plugins:
1. Eclipse Temurin Installer: Automatically install and configure the Eclipse Temurin JDK.
2. Pipeline Maven Integration: Provides Maven support for Jenkins Pipeline.
3. File Provider: Manage configuration files centrally in Jenkins.
4. SonarQube Scanner: Integrate Jenkins with SonarQube for code analysis.
5. Kubernetes CLI: Interact with Kubernetes clusters using kubectl.
6. Kubernetes: Run Jenkins agents as pods within a Kubernetes cluster.
7. Docker: Integrate Jenkins with Docker for builds and registry interactions.
7. Docker Pipeline Step: Extend Jenkins Pipeline with Docker build, publish, and run steps.

After installing, configure the plugins as needed in Jenkins global configuration or job configurations.

You're right, there's no need to repeat those steps for each type of credential. Here’s a simplified and more concise version:

---
### Step-3: Configuration of the System
---
##### Go to Dashboard > Manage Jenkins > System Configuration: System (Configure global settings and paths.)

1. Jenkins Location
2. SonarQube servers

---
### Step-4: Configuration of the Tools
---
##### Go to Dashboard > Manage Jenkins > System Configuration: Tools (Configure tools, their locations and automatic installers.)

1. jdk installations
2. SonarQube Scanner installations
3. Maven installations
4. Docker installations

---
### Step-4: Configure Credentials
---
##### Go to Dashboard > Manage Jenkins > Security: Credentials -> (global) -> Add Credentials

1. github
```
1. Choose "Username with password" or "Secret text" based on your preference.
2. Enter your GitHub username and token/password and set id to git-cred
3. Click "OK".
```
2. docker
```
1. Choose "Username with password".
2. Enter your Docker registry username and password. Set id to docker-cred
3. Click "OK".
```
3. sonarqube token
   
Token:
```
1. Generate a token in your SonarQube account. Set id to sonar-cred
2. Choose "Secret text".
3. Enter the generated SonarQube token.
4. Click "OK".
```
Webhook
```
Go to Administration > Configuration > Webhooks and create a webhook for jenkins
add : http://jenkins_ip:webhook/
```
4. kubernetes
   1. Create a Kubernetes secret using the following YAML configuration:
```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: myserviceaccount
```
   2. Save this as service-account-token.yml, then run:
   ```
   kubectl -n YOUR_NAMESPACE create -f service-account-token.yml
   ```
   3. Retrieve the token from the created secret:
   ```
   kubectl -n YOUR_NAMESPACE describe secret mysecretname
   ```
Look for the token field in the output and copy its value.

1. In Jenkins, choose "Secret text".
2. Paste the token value retrieved from the Kubernetes secret. Set id as k8s-cred
3. Click "OK"

### Step-5: Start writing the Pipeline
1. Goto Jenkins Dashboard.
2. Click "+ New item"
    1. Name: Project_CICD_bishal
    2. Select options as "Pipeline"
3. Ok
4. Click "Discard old builds"
    1. Max # of builds to keep: 3
5.  In Pipeline Tab:
    1. Script: "Hello World"
##### @Note: After we select "Hello World" , we can start modifying if by firstly creating multiple stages and so on.
6.  Click "Apply"

No. of stages that we are going to build on this project are as follows:
```
1. Git CheckOut
2. mvn compile
3. mvn test
4. Trivy File system scan
5. SonarQube Analaysis
6. Quality Gate
7. mvn package Build
8. Publish to Nexus
9. Build and Tag Docker Image
10. Docker Image Scan
11. Push Docker Image
12. Deploy to k8's
13. Verify the Deployment
```
##### Final Pipeline that we are going to use for the Project:
```
pipeline {
   agent any

   tools {
      jdk 'jdk17'
      maven 'maven3'
   }

   environment {
      SCANNER_HOME= tool 'sonar-scanner'
   }

   stages {
      stage('Git Checkout') {
         steps {
            git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/bishaldml/springSchoolManagement'
         }
      }

      stage('Compile') {
         steps {
            sh "mvn compile"
         }
      }

      stage('Tests') {
         steps {
            sh "mvn test"
         }
      }

      stage('File System Scan') {
         steps {
            sh "trivy fs --format table -o trivy-fs-report.html ."
         }
      }

      stage('SonarQuebe Analysis') {
         steps {
            withSonarQubeEnv("sonar") {
               sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=schoolManagement -Dsonar.projectKey=schoolManagement \
                            -Dsonar.java.binaries=. '''
            }
         }
      }

      stage('Quality Gate') {
         steps {
            script {
               waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
         }
      }


      stage('Build') {
         steps {
            sh "mvn package"
         }
      }


      stage('Publish to nexus ') {
         steps {
            withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
               sh "mvn deploy"
            }
         }
      }

      stage('Build & Tag Docker Image') {
         steps {
            script {
               withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                  sh "docker build -t bishaldml/school_management:latest ."
               }
            }
         }
      }

      stage('Docker Image Scan') {
         steps {
            sh "trivy image --format table -o trivy-fs-report.html bishaldml/school_management:latest"
         }
      }

      stage('Push Docker Image') {
         steps {
            script {
               withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                  sh "docker push bishaldml/school_management:latest"
               }
            }
         }
      }


      stage('Deploy to Kubernetes') {
         steps {
            withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.55.142:6443') {
               sh "kubectl apply -f ./k8s/app-deployment.yaml"
            }
         }
      }


      stage('Verify the Deployment') {
         steps {
            withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.55.142:6443') {
               sh "kubectl get pods -n webapps"
               sh "kubectl get svc -n webapps"
            }
         }
      }


   }
   post {
      always {
         script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'bishaldhimal321@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
            )
         }
      }
   }
}
```
