vm -----> Jenkins
vm -----> SonarQube
vm -----> Nexus
K8's -----> EKS

### Step-1: Create 3 vm's instances:
1. Login to your AWS Account
2. Launch vm with below configurations
  1. AMI: Ubuntu latest version
  2. Instance_Type: t2.large # 8 RAM and 2 CPU (for jenkis server only for other vm use t2.medium)
  3. NSG: Project-Demo-NSG-bishal
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
  4. Storage: 25 GiB.
 

### Step-2: Create a new user for creating EKS.
1. Goto IAM.
2. Select 'Users'
3. Create User
4. User name: eks-create-cluster-bishal
@ Note: Once user is created, assign the policy so that user can create EKS.

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

@ Note: Once all the policies are added and we need to add the inline policy also. For that:
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
6. 
8. 
### Step-

### Step-

### Step-

### Step-

### Step-
