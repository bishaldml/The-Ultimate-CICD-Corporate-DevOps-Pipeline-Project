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
2. docker
3. sonarqube token
4. kubernetes

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

### Step

### Step
