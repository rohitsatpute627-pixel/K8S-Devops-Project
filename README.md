K8S DevOps Project – AWS EKS CI/CD Pipeline
📌 Project Overview
This project demonstrates an end-to-end DevOps CI/CD pipeline for deploying a web application to Amazon EKS (Elastic Kubernetes Service).
The application source code is maintained in GitHub. A GitHub webhook triggers Jenkins, which:
Clones the latest source code.
Builds a Docker image.
Tags the image with the Jenkins build number and `latest`.
Pushes the image to Docker Hub.
Authenticates with AWS.
Configures `kubectl` for the EKS cluster.
Applies Kubernetes manifests.
Updates the Kubernetes Deployment with the newly built image.
Waits for the Kubernetes rolling update to complete.
The application is exposed externally through a Kubernetes `LoadBalancer` Service.
> **Source:** This README is based on the supplied project setup notes, including the Dev server, Jenkins Master, EKS cluster, Docker Hub, Kubernetes manifests, Jenkins pipeline, and CI/CD flow. 
---
🏗️ Project Architecture
```text
                         ┌──────────────────────┐
                         │      Developer       │
                         │   Changes Code       │
                         └──────────┬───────────┘
                                    │
                                    │ git push
                                    ▼
                         ┌──────────────────────┐
                         │       GitHub         │
                         │ K8S-Devops-Project   │
                         └──────────┬───────────┘
                                    │
                             Webhook Trigger
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │       Jenkins Master        │
                    │                              │
                    │  1. Clone Code              │
                    │  2. Build Docker Image       │
                    │  3. Tag Image               │
                    │  4. Push Image              │
                    │  5. Configure AWS/EKS       │
                    │  6. Deploy to Kubernetes    │
                    └──────────────┬───────────────┘
                                   │
                     ┌─────────────┴─────────────┐
                     │                           │
                     ▼                           ▼
          ┌───────────────────┐       ┌────────────────────┐
          │    Docker Hub     │       │     AWS EKS        │
          │                   │       │                    │
          │ docker-repo-5     │       │ cluster1           │
          │ image:1           │       │                    │
          │ image:latest      │       │  ┌──────────────┐  │
          └─────────┬─────────┘       │  │ Deployment   │  │
                    │                 │  │ replicas: 2  │  │
                    │ image pull      │  └──────┬───────┘  │
                    └────────────────►│         │          │
                                      │         ▼          │
                                      │  ┌──────────────┐  │
                                      │  │ Kubernetes   │  │
                                      │  │    Pods      │  │
                                      │  │  ┌────────┐  │  │
                                      │  │  │ Pod 1  │  │  │
                                      │  │  └────────┘  │  │
                                      │  │  ┌────────┐  │  │
                                      │  │  │ Pod 2  │  │  │
                                      │  │  └────────┘  │  │
                                      │  └──────┬───────┘  │
                                      │         │          │
                                      │         ▼          │
                                      │  ┌──────────────┐  │
                                      │  │ LoadBalancer │  │
                                      │  │    :80       │  │
                                      │  └──────┬───────┘  │
                                      └─────────┼───────────┘
                                                │
                                                ▼
                                      ┌──────────────────┐
                                      │      User        │
                                      │ Browser / Client │
                                      └──────────────────┘
```
---
🔄 CI/CD Flow
```text
Developer
   │
   │ Push code
   ▼
GitHub Repository
   │
   │ GitHub Webhook
   ▼
Jenkins Pipeline
   │
   ├── Clone Code
   │
   ├── Build Docker Image
   │
   ├── Tag Image
   │      ├── <BUILD_NUMBER>
   │      └── latest
   │
   ├── Push Image → Docker Hub
   │
   ├── Authenticate → AWS
   │
   ├── Configure kubectl → EKS
   │
   ├── kubectl apply
   │
   ├── kubectl set image
   │
   └── kubectl rollout status
             │
             ▼
        EKS Deployment
             │
             ▼
       Rolling Update
             │
             ▼
        New Pods Ready
             │
             ▼
      LoadBalancer :80
             │
             ▼
          Website
```
---
1. 🖥️ Development Server
The project uses an Ubuntu Server 24.04 LTS instance as the development server.
Server
```text
Name: dev server
Instance type: c7i-flex.large
AMI: Ubuntu Server 24.04 LTS (HVM), SSD Volume Type
```
Download application source
```bash
cd /root

wget https://www.tooplate.com/zip-templates/2130_waso_strategy.zip

apt-get update -y

apt install unzip -y

unzip 2130_waso_strategy.zip

cd /root/2130_waso_strategy
```
---
2. 🐳 Dockerfile
Create a `Dockerfile` inside the application directory:
```dockerfile
FROM ubuntu

RUN apt-get update -y
RUN apt-get install -y apache2

COPY . /var/www/html

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
Dockerfile explanation
Instruction	Purpose
`FROM ubuntu`	Uses Ubuntu as the base image
`RUN apt-get update -y`	Updates package information
`RUN apt-get install -y apache2`	Installs Apache web server
`COPY . /var/www/html`	Copies application files into Apache web root
`CMD [...]`	Starts Apache in the foreground
---
3. 📦 Push Application to GitHub
Install Git if required:
```bash
apt install git -y
```
Initialize the repository:
```bash
git init

git add *

git commit -m "Initial commit Waso Strategy Website"
```
Create the GitHub repository and connect it:
```bash
git remote add origin https://github.com/rohitsatpute627-pixel/K8S-Devops-Project.git

git push origin master
```
The repository name must match the repository configured in the Jenkins pipeline.
---
4. ⚙️ Jenkins Master
A separate Ubuntu server is used as the Jenkins Master.
```text
Name: jenkins-master
Instance type: c7i-flex.large
AMI: Ubuntu Server 24.04 LTS (HVM), SSD Volume Type
```
Install Java
```bash
apt update -y

apt install fontconfig openjdk-21-jre -y
```
Install Jenkins
```bash
wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | \
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

apt update -y

apt install jenkins -y

systemctl start jenkins
```
---
5. 🐳 Install Docker on Jenkins Master
```bash
apt install docker.io -y

systemctl start docker

systemctl enable docker

usermod -aG docker jenkins

systemctl restart jenkins

systemctl restart docker
```
Jenkins requires Docker access because the pipeline builds the application image on the Jenkins server.
---
6. 🔐 Jenkins Sudo Configuration
The supplied setup adds the Jenkins user to sudoers:
```text
jenkins ALL=(ALL) NOPASSWD: ALL
```
This provides Jenkins with unrestricted sudo access.
> **Security note:** For a production implementation, avoid unrestricted `NOPASSWD: ALL` where possible. Use the minimum permissions required by the pipeline.
---
7. 🌐 Jenkins Web Access and GitHub Webhook
Open TCP port `8080` in the Jenkins server security group.
Access Jenkins:
```text
http://<JENKINS_PUBLIC_IP>:8080
```
Configure the GitHub webhook:
```text
http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
```
When code is pushed to GitHub, the webhook triggers the Jenkins pipeline.
> The original setup notes contain a specific public IP for the webhook. This README intentionally uses `<JENKINS_PUBLIC_IP>` so the document does not expose an environment-specific address.
---
8. ☁️ Create Amazon EKS Cluster
Create an EKS cluster in:
```text
Region: ap-south-1
Cluster name: cluster1
Configuration: Custom configuration
EKS Auto Mode: Disabled
Kubernetes version: 1.36
Cluster IAM role: AmazonEKSClusterRole
```
The supplied setup notes indicate that cluster creation takes approximately 10 minutes.
---
9. 👷 Create EKS Worker Nodes
Go to:
```text
EKS
 → Clusters
 → cluster1
 → Compute
 → Node groups
 → Add
```
Create the worker node group.
The setup notes indicate that node group creation takes approximately 5 minutes.
---
10. 🔑 AWS IAM User
Create an IAM user:
```text
Username: user1
Access: Administrator access
```
Create an Access Key ID and Secret Access Key for the user.
> **Security recommendation:** Administrator access is used in the supplied lab setup, but production environments should use least-privilege IAM policies and preferably short-lived credentials/roles.
---
11. 🛠️ Install AWS CLI
The setup notes initially run:
```bash
aws configure
```
and note that AWS CLI may not be installed on the Ubuntu server.
Install it:
```bash
cd /root

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o awscliv2.zip

apt update -y

apt install -y unzip

unzip awscliv2.zip

/root/aws/install
```
Configure AWS:
```bash
aws configure
```
Enter:
```text
AWS Access Key ID
AWS Secret Access Key
Default region: ap-south-1
Output format: json
```
---
12. 🔐 EKS Security Group
The EKS API server uses port `443`.
Open TCP port `443` in the appropriate EKS cluster security group as required by the lab setup.
Navigate to:
```text
EKS
 → Clusters
 → cluster1
 → Networking
 → Cluster security group
```
---
13. 🔧 Configure kubeconfig
Connect to the Jenkins server and run:
```bash
aws eks --region ap-south-1 update-kubeconfig --name cluster1
```
Verify:
```bash
kubectl get nodes
```
---
14. 🧰 Install eksctl
```bash
cd /root

curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
| tar xz -C /tmp

mv /tmp/eksctl /usr/local/bin

eksctl version
```
---
15. ☸️ Install kubectl
```bash
cd /root

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x /root/kubectl

mv /root/kubectl /usr/local/bin/kubectl

kubectl version --client
```
---
16. 🔐 Give IAM User Access to EKS
Create an EKS IAM access entry for the IAM user.
Example:
```text
IAM principal ARN:
arn:aws:iam::<ACCOUNT_ID>:user/user1

Username:
abc

Access policy:
AmazonEKSClusterAdminPolicy
```
The supplied notes also use:
```bash
aws eks associate-access-policy \
  --cluster-name cluster1 \
  --principal-arn arn:aws:iam::<ACCOUNT_ID>:user/user1 \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster
```
Replace `<ACCOUNT_ID>` with the AWS account ID.
---
17. 👤 Give Jenkins Access to kubeconfig
Create the Jenkins Kubernetes configuration directory:
```bash
mkdir /var/lib/jenkins/.kube
```
Copy the kubeconfig:
```bash
cp /root/.kube/config /var/lib/jenkins/.kube
```
Change ownership:
```bash
chown -R jenkins:jenkins /var/lib/jenkins/.kube
```
Set secure permissions:
```bash
chmod 600 /var/lib/jenkins/.kube/config
```
Update kubeconfig:
```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name cluster1
```
Verify:
```bash
kubectl get nodes
```
---
18. ☸️ Kubernetes Deployment
Create the Kubernetes directory:
```bash
mkdir /k8s

cd /k8s
```
Create:
```text
deployment.yml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: waso-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: waso-app
  template:
    metadata:
      labels:
        app: waso-app
    spec:
      containers:
      - name: waso-container
        image: rohitsatpute45/docker-repo-5:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
```
Apply:
```bash
kubectl apply -f deployment.yml
```
Deployment explanation
```text
Deployment
   │
   ├── replicas: 2
   │
   ├── Pod 1
   │    └── waso-container
   │
   └── Pod 2
        └── waso-container
```
The deployment uses the label:
```text
app: waso-app
```
This label is used by the Service to identify the application Pods.
---
19. 🌐 Kubernetes Service
Create:
```text
service.yml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: waso-service
spec:
  type: LoadBalancer
  selector:
    app: waso-app
  ports:
    - port: 80
      targetPort: 80
```
Apply:
```bash
kubectl apply -f service.yml
```
Service traffic flow
```text
Internet
   │
   ▼
AWS Load Balancer
   │
   │ Port 80
   ▼
waso-service
   │
   ├──────────────┐
   ▼              ▼
Pod 1            Pod 2
Port 80          Port 80
```
The Service selects Pods using:
```yaml
selector:
  app: waso-app
```
---
20. 🐳 Docker Hub
Create the Docker Hub repository:
```text
docker-repo-5
```
The repository name must match the name used by the Jenkins pipeline:
```text
rohitsatpute45/docker-repo-5
```
---
21. 🔑 Jenkins Docker Hub Credentials
Go to:
```text
Jenkins
 → Manage Jenkins
 → Credentials
 → Add Credentials
```
Select:
```text
Username with Password
```
Use:
```text
ID: dockerhub
```
The pipeline uses this credential to authenticate with Docker Hub.
---
22. 🔐 Jenkins AWS Credentials
Install the AWS Credentials plugin:
```text
Jenkins
 → Manage Jenkins
 → Plugins
 → Available Plugins
 → AWS Credentials
```
Then create an AWS credential:
```text
Jenkins
 → Manage Jenkins
 → Credentials
 → Add Credentials
 → AWS Credentials
```
Use:
```text
ID: aws-cred
```
Add the Access Key ID and Secret Access Key associated with the IAM user used by this lab setup.
---
23. 🧩 Jenkins Pipeline
Create a Jenkins Pipeline job.
Configure:
```text
Pipeline
GitHub repository URL
GitHub hook trigger
Pipeline script
```
The supplied pipeline is:
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rohitsatpute45/docker-repo-5"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "cluster1"
        K8S_DIR = "/k8s"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/rohitsatpute627-pixel/K8S-Devops-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    docker push $DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Configure AWS & EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    sh """
                    export AWS_DEFAULT_REGION=$AWS_REGION

                    aws sts get-caller-identity

                    aws eks update-kubeconfig \
                        --region $AWS_REGION \
                        --name $CLUSTER_NAME
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    sh """
                    export AWS_DEFAULT_REGION=$AWS_REGION

                    kubectl apply -f /$K8S_DIR/ --validate=false

                    kubectl set image deployment/waso-deployment \
                        waso-container=$DOCKER_IMAGE:$IMAGE_TAG

                    kubectl rollout status deployment/waso-deployment
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }
    }
}
```
---
24. 🔍 Jenkins Pipeline Stages Explained
Stage 1 – Clone Code
```groovy
stage('Clone Code')
```
Jenkins clones:
```text
https://github.com/rohitsatpute627-pixel/K8S-Devops-Project.git
```
into the Jenkins workspace.
---
Stage 2 – Build Docker Image
```groovy
docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
```
If Jenkins build number is:
```text
1
```
the image becomes:
```text
rohitsatpute45/docker-repo-5:1
```
The pipeline also creates:
```text
rohitsatpute45/docker-repo-5:latest
```
---
Stage 3 – Push to Docker Hub
Jenkins logs into Docker Hub using:
```text
dockerhub
```
credential ID.
Then pushes:
```text
rohitsatpute45/docker-repo-5:<BUILD_NUMBER>
rohitsatpute45/docker-repo-5:latest
```
---
Stage 4 – Configure AWS and EKS
The pipeline uses:
```text
aws-cred
```
to authenticate to AWS.
It validates the identity:
```bash
aws sts get-caller-identity
```
Then configures the EKS context:
```bash
aws eks update-kubeconfig \
    --region ap-south-1 \
    --name cluster1
```
---
Stage 5 – Deploy to EKS
Kubernetes manifests are applied:
```bash
kubectl apply -f /k8s/ --validate=false
```
The Deployment image is updated:
```bash
kubectl set image deployment/waso-deployment \
    waso-container=rohitsatpute45/docker-repo-5:<BUILD_NUMBER>
```
Finally:
```bash
kubectl rollout status deployment/waso-deployment
```
waits for the rollout to complete.
---
25. 📊 Jenkins Environment Variables
Variable	Purpose	Example
`DOCKER_IMAGE`	Docker Hub repository	`rohitsatpute45/docker-repo-5`
`IMAGE_TAG`	Jenkins build-based version	`1`
`AWS_REGION`	AWS region	`ap-south-1`
`CLUSTER_NAME`	EKS cluster name	`cluster1`
`K8S_DIR`	Kubernetes manifest directory	`/k8s`
Example
For Jenkins build:
```text
BUILD_NUMBER = 1
```
the image becomes:
```text
rohitsatpute45/docker-repo-5:1
```
---
26. 🚀 Complete Deployment Flow
```text
                    ┌──────────────┐
                    │  Developer   │
                    └──────┬───────┘
                           │
                           │ git push
                           ▼
                    ┌──────────────┐
                    │    GitHub    │
                    └──────┬───────┘
                           │
                           │ webhook
                           ▼
                 ┌────────────────────┐
                 │ Jenkins Pipeline   │
                 └─────────┬──────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         Clone Code    Build Image   Tag Image
              │            │            │
              └────────────┼────────────┘
                           ▼
                    ┌──────────────┐
                    │  Docker Hub  │
                    └──────┬───────┘
                           │
                           │ image
                           ▼
                    ┌──────────────┐
                    │    AWS EKS   │
                    │   cluster1   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Deployment  │
                    │  replicas: 2 │
                    └──────┬───────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  Pod 1         Pod 2
                    │             │
                    └──────┬──────┘
                           ▼
                    ┌──────────────┐
                    │ LoadBalancer │
                    │    Port 80   │
                    └──────┬───────┘
                           │
                           ▼
                       Website
```
---
27. 🔄 Rolling Update
When a developer changes the application:
```text
index.html
```
and pushes the change to GitHub:
```bash
git add .
git commit -m "Update website"
git push origin master
```
the following process occurs:
```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
New Docker Image
   ↓
Docker Hub
   ↓
kubectl set image
   ↓
Kubernetes Deployment
   ↓
Rolling Update
   ↓
New Pods
   ↓
LoadBalancer
   ↓
Updated Website
```
The pipeline waits for rollout completion using:
```bash
kubectl rollout status deployment/waso-deployment
```
---
28. 🌐 Access the Application
After deployment, check the Service:
```bash
kubectl get svc
```
Example:
```text
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP
waso-service    LoadBalancer   ...            <LOAD_BALANCER_DNS>
```
Open the Load Balancer DNS in a browser:
```text
http://<LOAD_BALANCER_DNS>
```
---
29. 🧪 Verify Deployment
Check Nodes
```bash
kubectl get nodes
```
Check Pods
```bash
kubectl get pods
```
Check Deployment
```bash
kubectl get deployment
```
Check Service
```bash
kubectl get svc
```
Check Pod Details
```bash
kubectl describe pod <pod-name>
```
Check Application Logs
```bash
kubectl logs <pod-name>
```
Check Deployment Rollout
```bash
kubectl rollout status deployment/waso-deployment
```
---
30. 🐚 Enter a Kubernetes Pod
```bash
kubectl exec -it <pod-name> -- /bin/bash
```
Install Vim if required:
```bash
apt update -y

apt install -y vim
```
Edit the web page:
```bash
vim /var/www/html/index.html
```
The supplied project notes also describe repeating the process for the second Pod for testing purposes.
> For normal application delivery, changes should be made in source control and redeployed through CI/CD rather than manually editing running Pods.
---
31. 🧰 Useful Troubleshooting Commands
Jenkins
```bash
systemctl status jenkins
```
```bash
journalctl -u jenkins -f
```
Docker
```bash
systemctl status docker
```
```bash
docker ps
```
```bash
docker images
```
Kubernetes
```bash
kubectl get nodes
```
```bash
kubectl get pods -o wide
```
```bash
kubectl get deployment
```
```bash
kubectl get svc
```
```bash
kubectl describe pod <pod-name>
```
```bash
kubectl logs <pod-name>
```
EKS connectivity
```bash
aws sts get-caller-identity
```
```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name cluster1
```
```bash
kubectl get nodes
```
---
32. 🗂️ Suggested Repository Structure
```text
K8S-Devops-Project/
│
├── Dockerfile
├── deployment.yml
├── service.yml
├── Jenkinsfile
├── index.html
└── README.md
```
The exact repository contents can vary depending on how the application source is organized.
---
33. 🎯 Technologies Used
Technology	Role
AWS EC2	Dev and Jenkins servers
AWS EKS	Kubernetes cluster
Kubernetes	Container orchestration
Docker	Containerization
Docker Hub	Container image registry
Jenkins	CI/CD automation
Git	Source-code version control
GitHub	Source-code repository and webhook
AWS CLI	AWS management from Jenkins/server
kubectl	Kubernetes command-line management
eksctl	EKS cluster tooling
Apache	Web server inside the container
Ubuntu 24.04	Server/container operating system
---
34. 🔐 Security Considerations
The supplied lab setup contains several intentionally simple configurations that should be improved before production use:
Do not commit AWS Access Keys or Docker Hub passwords to Git.
Use Jenkins Credentials instead of hardcoding secrets.
Avoid `jenkins ALL=(ALL) NOPASSWD: ALL` in production.
Avoid giving an IAM user full Administrator access unless required.
Prefer IAM roles and short-lived credentials where possible.
Restrict security-group inbound rules to required source IPs.
Do not expose Jenkins port `8080` to the entire internet unnecessarily.
Use HTTPS for production endpoints.
Use a dedicated domain and TLS certificate for the application.
Consider image scanning and vulnerability management before deploying images.
Pin production image versions rather than relying only on `latest`.
Protect GitHub webhook and Jenkins endpoints.
---
35. 🏆 Project Outcome
This project demonstrates a complete CI/CD implementation:
```text
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Docker Hub
   ↓
AWS Authentication
   ↓
Amazon EKS
   ↓
Kubernetes Deployment
   ↓
Rolling Update
   ↓
LoadBalancer
   ↓
Web Application
```
The main DevOps concept demonstrated is automated application delivery from source-code commit to a running application on Kubernetes.
---
📚 Quick Interview Explanation
> "I built an end-to-end CI/CD pipeline using GitHub, Jenkins, Docker, Docker Hub, AWS EKS, and Kubernetes. The developer pushes code to GitHub, which triggers Jenkins through a webhook. Jenkins clones the code, builds a Docker image, tags it with the Jenkins build number, and pushes it to Docker Hub. Jenkins then authenticates with AWS, configures the EKS kubeconfig, applies the Kubernetes manifests, and updates the Deployment with the new Docker image. Kubernetes performs a rolling update across two replicas, and the application is exposed through a LoadBalancer Service."
---
📌 Project Flow in One Line
Developer → GitHub → Webhook → Jenkins → Docker Build → Docker Hub → AWS EKS → Kubernetes Deployment → Pods → LoadBalancer → Application
