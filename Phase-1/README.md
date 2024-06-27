vm -----> Jenkins

vm -----> SonarQube

vm -----> Nexus

K8's -----> EKS

### Step-1: Create 3 vm's instances:
1. Login to your AWS Account
2. Launch vm with below configurations
   
     1. AMI: ```Ubuntu latest version```
     2. Instance_Type: ```t2.large``` # 8 RAM and 2 CPU (for jenkis server only for other vm's use ```t2.medium```)
     3. NSG: ```Project-Demo-NSG-bishal```
     ```
     Type -----> Port range
     SMTP -----> 25
     Custom TCP -----> 3000-10000
     HTTP -----> 80
     HTTPS -----> 445
     SSH -----> 22
     Custom TCP -----> 6443
     Custom TCP -----> 30000-32767
     SMTPS -----> 465
     Customs TCP -----> 27017
     ```
     4. Storage: ```25 GiB```.
 

### Step-2: Create a new user for creating EKS.
1. Goto IAM.
2. Select 'Users'
3. Create User
4. User name: eks-create-cluster-bishal
##### @ Note: Once user is created, assign the policy so that user can create EKS.

1. Goto your created user
2. select 'Add Permissions'
3. Permissions options: Attach policies directly
4. Search the following policies and add:
  1. AmazonEC2FullAccess
  2. AmazonEKS_CNI_Policy
  3. AmazonEKSClusterPolicy
  4. AmazonEKSWorkerNodePolicy
  5. AWSCloudFormationFullAccess
  6. IAMFullAccess

##### @ Note: Once all the policies are added and we need to add the inline policy also. For that:
1. Again Select "Add Permission"
2. On drop-down menu: Select 'Create inline policy'
3. Select 'JSON' on rightside.
4. Select all and delete it.
5. Paste the below content/code:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```
6. Provide Policy name: EKS-inline-policy-bishal
##### @Note: After completion, again In IAM Dashboard.
1. Select Users.
2. Click 'your created user'
3. Click on "Security Credentials"
4. On Access Tab:
5. Create access key.
6. In 'Use Case'
7. Select 'CLI'
8. Click on "I understand ...."
9. Click on "Next".

Its going to create user_name and passwords which we can download in CSV format.

##### @Note: we have created a user that is responsible for creating EKS Cluster. Its not best practice to use root account to create service or resouce.

##### Download portable version of MobaXterm Home Edition and Install it.

### Step-3: Creating a EKS Cluster.
##### @Note: For creating a EKS Cluster, we can use a separate vm or use a Jenkins_Server Instance. In this lab, we will use Jenkins Instance to create EKS.

1. Open MobaXterm
2. click on "Session"
3. Click on "SSH"
4. On "Remote host: ```<IP_Addr_Jenkins_Server>```"
5. Username: ubuntu
6. Advanced SSH Setting:
7. Use Private Key: ```<.pem key_file location>
8. Ok
---
1. On 'User Sessions'
2. Click on your machine and rename it to Jenkins_Server
3. Click on 'Customize tab color and gowith red'
4. Then close your terminal tab.
5. Again open it form user session.
---
ssh to Jenkins_Server instance:
```
sudo apt update && apt upgrade -y
```
Now, we will install AWSCLI, KUBECTL and EKSCTL on Jenkins_Server
1. AWSCLI: use to interact with AWS account.
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```
```
aws configure
```
@ To configure our EKS-Cluster-User that we created. This is the user that we want to connect from our vm to AWS account. ```aws configure``` will ask for the access_key_ID and secret_access_key: use the security-cred that we created in access_Key, which was downloaded in your local machine. Default region name: <give your region code name (ap-south-1)>, Default output format:<Just press enter>

2. KUBECTL: use to interact with K8's cluster.
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
3. EKSCTL: use to create EKS cluster.
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
#### Create EKS Cluster:
```
eksctl create cluster --name=<my-eks22> \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --version=1.30 \
                      --without-nodegroup

eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster <my-eks22> \
    --approve

eksctl create nodegroup --cluster=<my-eks22> \
                       --region=ap-south-1 \
                       --name=node2 \
                       --node-type=t3.medium \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=Key \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

### Step-4: Installing Jenkins on the Jenkins_Server instance:
1. Update server package
```
sudo apt update
```
2. Prerequisite: Java

Install OpenJDK 17 JRE Headless
```
sudo apt install openjdk-17-jre-headless -y
```
3. Install Jenkins
```
# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y
```      
4. Browse to ```http://<jenkins_server-ip_addr>:8080```

5. Installing docker on Jenkins_Server as well:
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the Docker packages
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
To give permission for use the docker: 2 options
```
sudo usermod -aG docker ubuntu
```
or
```
sudo chmod 666 /var/run/docker.sock
```
### Step-5: Installing SonarQube on SonarQube_Instance.
1. Update the server
```
sudo apt update
```
2. Install docker: we are going to use the docker to setup sonarqube 
```
use above script to install docker
```
3. Run SonarQube container
```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
4. Access SonarQube by opening a web browser and navigating to http://<SonarQube_instance_Ip>:9000.
### Step-6: Installing Nexus on Nexus_Instance.
