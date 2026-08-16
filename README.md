**K8S DevOps Project – AWS EKS CI/CD Pipeline
📌 Project Overview**
**This project demonstrates an end-to-end DevOps CI/CD pipeline for deploying a web application to Amazon EKS (Elastic Kubernetes Service).
The workflow is:**
```text
Developer
   ↓
GitHub
   ↓
GitHub Webhook
   ↓
Jenkins
   ↓
Docker Build
   ↓
Docker Hub
   ↓
AWS EKS
   ↓
Kubernetes Deployment
   ↓
Pods
   ↓
LoadBalancer
   ↓
Web Application
```
The application is a web-based Waso Strategy website running on Apache inside a Docker container.
---
**🏗️ Architecture**
```text
                         Developer
                            |
                         git push
                            |
                            v
                       GitHub Repo
                            |
                     GitHub Webhook
                            |
                            v
                     Jenkins Pipeline
                       /          \
                      /            \
                     v              v
                Docker Hub       AWS EKS
                     |              |
                     |          Deployment
                     |              |
                     |           Pods x2
                     |              |
                     +----------> LoadBalancer
                                    |
                                    v
                              Web Application
```
Main Components
Component	Purpose
AWS EC2	Dev server and Jenkins server
GitHub	Source code repository
Jenkins	CI/CD automation
Docker	Application containerization
Docker Hub	Docker image registry
Amazon EKS	Kubernetes cluster
Kubernetes	Container orchestration
kubectl	Kubernetes CLI
eksctl	EKS management tool
Apache	Web server inside container
---
**1. Prerequisites**
Before starting, make sure you have:
AWS account
GitHub account
Docker Hub account
Basic Linux knowledge
Basic Docker knowledge
Basic Kubernetes knowledge
AWS IAM permissions
SSH access to EC2 instances
AWS Resources
This project uses:
```text
AWS Region: ap-south-1
EKS Cluster: cluster1
EC2 Instance Type: c7i-flex.large
OS: Ubuntu Server 24.04 LTS
```
---
**2. Project Repository Structure**
Recommended GitHub repository structure:
```text
K8S-Devops-Project/
│
├── Dockerfile
├── Jenkinsfile
├── deployment.yml
├── service.yml
├── index.html
└── README.md
```
---**
**3. Step 1 – Create Development Server**
**Create an EC2 instance for application development.
Recommended configuration
```text
Instance type: c7i-flex.large
AMI: Ubuntu Server 24.04 LTS
```
Connect to the server using SSH.
---
**4. Step 2 – Download Application Run:**
```bash
cd /root

wget https://www.tooplate.com/zip-templates/2130_waso_strategy.zip

apt-get update -y

apt install unzip -y

unzip 2130_waso_strategy.zip

cd /root/2130_waso_strategy
```
---**
**5. Step 3 – Create Dockerfile**
Create a file named:
```text
Dockerfile
```
Add:
```dockerfile
FROM ubuntu

RUN apt-get update -y
RUN apt-get install -y apache2

COPY . /var/www/html

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
Dockerfile Explanation
Instruction	Purpose
`FROM ubuntu`	Uses Ubuntu as the base image
`RUN apt-get update -y`	Updates package information
`RUN apt-get install -y apache2`	Installs Apache
`COPY . /var/www/html`	Copies website files to Apache web root
`CMD`	Starts Apache in foreground mode
---
**6. Step 4 – Push Project to GitHub**
Install Git if required:
```bash
apt install git -y
```
Initialize Git:
```bash
git init
```
Add files:
```bash
git add .
```
Commit:
```bash
git commit -m "Initial commit Waso Strategy Website"
```
Create a GitHub repository.
Example:
```text
K8S-Devops-Project
```
Add the remote repository:
```bash
git remote add origin https://github.com/<YOUR_USERNAME>/K8S-Devops-Project.git
```
Push the code:
```bash
git push -u origin master
```
> If your GitHub default branch is `main`, use `main` instead of `master`.
---
7. Step 5 – Create Jenkins Server
Create another EC2 instance.
```text
Name: jenkins-master
Instance type: c7i-flex.large
AMI: Ubuntu Server 24.04 LTS
```
Connect to the Jenkins server.
---
**8. Step 6 – Install Java**
```bash
apt update -y

apt install fontconfig openjdk-21-jre -y
```
Verify:
```bash
java -version
```
---
**9. Step 7 – Install Jenkins**
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
Check Jenkins:
```bash
systemctl status jenkins
```
---
**10. Step 8 – Install Docker on Jenkins**
```bash
apt install docker.io -y

systemctl start docker

systemctl enable docker
```
Add Jenkins to Docker group:
```bash
usermod -aG docker jenkins
```
Restart services:
```bash
systemctl restart jenkins
systemctl restart docker
```
Verify:
```bash
docker --version
```
---**
**11. Step 9 – Jenkins Security Group****
Allow Jenkins web access.
Open:
```text
TCP 8080
```
in the Jenkins EC2 security group.
Access Jenkins:
```text
http://<JENKINS_PUBLIC_IP>:8080
```
> For production, restrict port 8080 to trusted IP addresses instead of exposing it to the entire internet.
---
**12. Step 10 – Configure GitHub Webhook**
In GitHub:
```text
Repository
→ Settings
→ Webhooks
→ Add webhook
```
Payload URL:
```text
http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
```
Content type:
```text
application/json
```
Enable:
```text
Just the push event
```
This webhook triggers Jenkins whenever code is pushed to GitHub.
---
**13. Step 11 – Create EKS Cluster**
Create an Amazon EKS cluster.
Configuration:
```text
Region: ap-south-1
Cluster name: cluster1
Configuration: Custom
EKS Auto Mode: Disabled
Kubernetes version: 1.36
Cluster IAM role: AmazonEKSClusterRole
```
Wait for the cluster to become active.
---
14. Step 12 – Create EKS Worker Nodes
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
After creation, verify that the nodes are available.
---
**15. Step 13 – Create IAM User**
For the lab setup, create:
```text
Username: user1
Access: Administrator access
```
Create:
```text
Access Key ID
Secret Access Key
```
> For production, use least-privilege IAM policies and IAM roles/short-lived credentials instead of permanent administrator credentials.
---
**16. Step 14 – Install AWS CLI****
On Jenkins server:
```bash
cd /root

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o awscliv2.zip

apt update -y

apt install -y unzip

unzip awscliv2.zip

/root/aws/install
```
Verify:
```bash
aws --version
```
Configure:
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
**17. Step 15 – Configure EKS Access**
Install and configure the EKS tools.
Update kubeconfig:
```bash
aws eks --region ap-south-1 update-kubeconfig --name cluster1
```
Verify:
```bash
kubectl get nodes
```
---
**18. Step 16 – Install eksctl**
```bash
cd /root

curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
| tar xz -C /tmp

mv /tmp/eksctl /usr/local/bin

eksctl version
```
---
**19. Step 17 – Install kubectl**
```bash
cd /root

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x /root/kubectl

mv /root/kubectl /usr/local/bin/kubectl

kubectl version --client
```
---
**20. Step 18 – Give IAM User Access to EKS**
In AWS:
```text
EKS
→ Clusters
→ cluster1
→ Access
→ IAM access entries
→ Create
```
Use the IAM principal ARN of your `user1`.
Access policy:
```text
AmazonEKSClusterAdminPolicy
```
You can also associate the policy using AWS CLI:
```bash
aws eks associate-access-policy \
  --cluster-name cluster1 \
  --principal-arn arn:aws:iam::<ACCOUNT_ID>:user/user1 \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster
```
Replace:
```text
<ACCOUNT_ID>
```
with your AWS account ID.
---
**21. Step 19 – Give Jenkins Access to Kubernetes**
Create Jenkins kubeconfig directory:
```bash
mkdir /var/lib/jenkins/.kube
```
Copy kubeconfig:
```bash
cp /root/.kube/config /var/lib/jenkins/.kube/
```
Change ownership:
```bash
chown -R jenkins:jenkins /var/lib/jenkins/.kube
```
Set permissions:
```bash
chmod 600 /var/lib/jenkins/.kube/config
```
Update kubeconfig:
```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name cluster1
```
Test:
```bash
kubectl get nodes
```
---
22. Step 20 – Create Kubernetes Deployment
Create:
```text
deployment.yml
```
Add:
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
          image: <DOCKERHUB_USERNAME>/docker-repo-5:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```
Apply:
```bash
kubectl apply -f deployment.yml
```
Check:
```bash
kubectl get deployment
kubectl get pods
```
---
**23. Step 21 – Create Kubernetes Service**
Create:
```text
service.yml
```
Add:
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
Check:
```bash
kubectl get svc
```
Wait until the `EXTERNAL-IP` or load balancer hostname is available.
---
**24. Step 22 – Create Docker Hub Repository**
Create a Docker Hub repository:
```text
docker-repo-5
```
Your final image will look like:
```text
<DOCKERHUB_USERNAME>/docker-repo-5:<BUILD_NUMBER>
```
and:
```text
<DOCKERHUB_USERNAME>/docker-repo-5:latest
```
---
**25. Step 23 – Add Docker Hub Credentials to Jenkins**
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
Username: <DOCKERHUB_USERNAME>
Password: <DOCKERHUB_TOKEN_OR_PASSWORD>
```
For better security, use a Docker Hub access token instead of your account password.
---
**26. Step 24 – Add AWS Credentials to Jenkins**
Install:
```text
AWS Credentials
```
Plugin:
```text
Jenkins
→ Manage Jenkins
→ Plugins
→ Available Plugins
→ AWS Credentials
```
Create credentials:
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
Access Key ID: <AWS_ACCESS_KEY>
Secret Access Key: <AWS_SECRET_KEY>
```
---
**27. Step 25 – Create Jenkins Pipeline**
Create:
```text
Jenkins
→ New Item
→ Pipeline
```
Configure:
```text
GitHub repository URL
GitHub hook trigger
Pipeline script
```
Recommended approach: store the pipeline in the repository as:
```text
Jenkinsfile
```

**28. Step 26 – Jenkinsfile**
Create a file named:
```text
Jenkinsfile
```
Use:
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "<DOCKERHUB_USERNAME>/docker-repo-5"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "cluster1"
        K8S_DIR = "/k8s"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/<GITHUB_USERNAME>/K8S-Devops-Project.git'
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                    echo $DOCKER_PASS | docker login \
                        -u $DOCKER_USER \
                        --password-stdin

                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    docker push $DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Configure AWS & EKS') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]
                ]) {
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
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]
                ]) {
                    sh """
                    export AWS_DEFAULT_REGION=$AWS_REGION

                    kubectl apply -f /k8s/ --validate=false

                    kubectl set image deployment/waso-deployment \
                        waso-container=$DOCKER_IMAGE:$IMAGE_TAG

                    kubectl rollout status \
                        deployment/waso-deployment
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
Replace these values
```text
<DOCKERHUB_USERNAME>
<GITHUB_USERNAME>
```
with your actual usernames.
---
**29. Step 27 – Jenkins Pipeline Stages**
Stage 1 – Clone Code
Jenkins downloads the latest source code from GitHub.
```text
GitHub → Jenkins Workspace
```
Stage 2 – Build Docker Image
Jenkins builds the Docker image:
```bash
docker build -t <DOCKERHUB_USERNAME>/docker-repo-5:$BUILD_NUMBER .
```
Stage 3 – Push Image
The image is pushed to Docker Hub:
```text
docker-repo-5:<BUILD_NUMBER>
docker-repo-5:latest
```
Stage 4 – Configure AWS
Jenkins authenticates with AWS and configures EKS:
```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region ap-south-1 --name cluster1
```
Stage 5 – Deploy
Kubernetes manifests are applied:
```bash
kubectl apply -f /k8s/
```
Then the Deployment image is updated:
```bash
kubectl set image deployment/waso-deployment \
waso-container=<DOCKERHUB_USERNAME>/docker-repo-5:$BUILD_NUMBER
```
Finally Jenkins waits for the rollout:
```bash
kubectl rollout status deployment/waso-deployment
```
---
**30. Complete CI/CD Flow**
```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins
    |
    +--> Clone Code
    |
    +--> Build Docker Image
    |
    +--> Tag Image
    |
    +--> Push Image to Docker Hub
    |
    +--> Authenticate with AWS
    |
    +--> Configure EKS
    |
    +--> Apply Kubernetes YAML
    |
    +--> Update Deployment Image
    |
    v
Amazon EKS
    |
    v
Kubernetes Deployment
    |
    +--> Pod 1
    |
    +--> Pod 2
    |
    v
LoadBalancer
    |
    v
Web Application
```
---
**31. Test the Deployment**
Check nodes:
```bash
kubectl get nodes
```
Check Pods:
```bash
kubectl get pods
```
Check Deployment:
```bash
kubectl get deployment
```
Check Service:
```bash
kubectl get svc
```
Check logs:
```bash
kubectl logs <pod-name>
```
Check Pod details:
```bash
kubectl describe pod <pod-name>
```
Check rollout:
```bash
kubectl rollout status deployment/waso-deployment
```
---
**32. Access the Application**
Get the LoadBalancer:
```bash
kubectl get svc
```
Example:
```text
NAME           TYPE           EXTERNAL-IP
waso-service   LoadBalancer   <LOAD_BALANCER_DNS>
```
Open:
```text
http://<LOAD_BALANCER_DNS>
```
The Waso Strategy website should be displayed.
---
**33. Test CI/CD Automation**
Change the application:
```text
index.html
```
Then:
```bash
git add .
git commit -m "Update website"
git push
```
The GitHub webhook triggers Jenkins.
Jenkins will:
```text
1. Pull latest code
2. Build a new Docker image
3. Tag it using BUILD_NUMBER
4. Push image to Docker Hub
5. Connect to EKS
6. Update Kubernetes Deployment
7. Perform rolling update
8. Wait for rollout completion
```
Verify:
```bash
kubectl rollout status deployment/waso-deployment
```
Then refresh the LoadBalancer URL.

**34. Useful Troubleshooting Commands**
Jenkins
```bash
systemctl status jenkins
journalctl -u jenkins -f
```
Docker
```bash
systemctl status docker
docker ps
docker images
```
Kubernetes
```bash
kubectl get nodes
kubectl get pods -o wide
kubectl get deployment
kubectl get svc
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```
AWS / EKS
```bash
aws sts get-caller-identity

aws eks update-kubeconfig \
--region ap-south-1 \
--name cluster1

kubectl get nodes
```
---
35. Optional – Enter a Kubernetes Pod
To enter a Pod:
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
> For a real CI/CD workflow, do not manually modify running Pods. Make changes in GitHub and let Jenkins redeploy the application.
---
36. Security Best Practices
This project is designed as a learning/lab project. Before using it in production:
Do not commit AWS credentials to GitHub.
Do not commit Docker Hub passwords.
Use Jenkins Credentials for secrets.
Prefer IAM roles and short-lived credentials.
Use least-privilege IAM policies.
Avoid unrestricted `NOPASSWD: ALL`.
Restrict Jenkins port `8080`.
Restrict Kubernetes API access.
Use HTTPS/TLS for production.
Use image vulnerability scanning.
Prefer immutable image tags instead of only `latest`.
Protect GitHub webhooks.
Do not expose sensitive configuration files.

**37. Important GitHub Security Note**
Never put credentials directly inside:
```text
Jenkinsfile
deployment.yml
README.md
```
For example, do not upload:
```text
AWS Access Key
AWS Secret Key
Docker Hub Password
SSH Private Key
```
If credentials are accidentally pushed to GitHub, revoke/rotate them immediately.
---
**38. Interview Explanation**
You can explain this project in an interview like this:
> I built an end-to-end CI/CD pipeline using GitHub, Jenkins, Docker, Docker Hub, AWS EKS, and Kubernetes. When a developer pushes code to GitHub, a webhook triggers Jenkins. Jenkins clones the code, builds a Docker image, tags it using the Jenkins build number, and pushes it to Docker Hub. Jenkins then authenticates with AWS, configures kubectl for the EKS cluster, applies the Kubernetes manifests, and updates the Deployment with the new image. Kubernetes performs a rolling update across two replicas, and the application is exposed through a LoadBalancer Service.
---
**39. Project Outcome**
The project demonstrates:
```text
Source Code Management
        ↓
GitHub
        ↓
CI/CD Automation
        ↓
Jenkins
        ↓
Containerization
        ↓
Docker
        ↓
Image Registry
        ↓
Docker Hub
        ↓
Cloud Kubernetes
        ↓
Amazon EKS
        ↓
Deployment
        ↓
Rolling Update
        ↓
LoadBalancer
        ↓
Web Application
```
---
⭐ Skills Demonstrated
```text
AWS
EC2
EKS
IAM
Docker
Docker Hub
Jenkins
Git
GitHub
Kubernetes
kubectl
eksctl
Linux
Apache
CI/CD
Rolling Deployment
AWS CLI
```
---
👨‍💻 Author
Rohit Satpute
AWS | Cloud | DevOps | Kubernetes
