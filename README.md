# CI-CD-Pipeline-for-Flask-Application-on-AWS-EKS

# Project Summary

This project demonstrates a complete CI/CD pipeline for deploying a Flask application to AWS EKS (Elastic Kubernetes Service) using Jenkins, Docker, AWS ECR, and Kubernetes. It covers the entire DevOps lifecycle—from code integration, security scanning, containerization, and pushing images to ECR, to deploying the application on Kubernetes in a highly scalable cloud environment.


#Step 1: Install Required Tools
Since you don’t have eksctl or kubectl installed, let's start by installing all the necessary tools.

1.1 Install AWS CLI
The AWS CLI is needed to interact with AWS services.

📌 Step 1: Update System Packages

sudo apt update && sudo apt upgrade -y


📌 Step 2: Download AWS CLI Installer

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"


📌 Step 3: Install Unzip (If Not Installed)

sudo apt install unzip -y


📌 Step 4: Extract the AWS CLI Package

unzip awscliv2.zip


📌 Step 5: Run the AWS CLI Installation

sudo ./aws/install


📌 Step 6: Verify Installation

aws --version

📌 Step 7: Configure AWS CLI

aws configure

You'll be asked to enter:

AWS Access Key ID

AWS Secret Access Key

Default region (e.g., us-east-1)

Output format (json or yaml)



1.2 Install eksctl

eksctl is a CLI tool for creating and managing EKS clusters.


curl -LO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"

tar -xzf eksctl_Linux_amd64.tar.gz

sudo mv eksctl /usr/local/bin/

eksctl version



1.3 Install kubectl

kubectl is used to interact with Kubernetes.


curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client

1.4 Install Docker

sudo apt install -y docker.io

sudo systemctl start docker

sudo systemctl enable docker

docker --version


1.5 Install Helm (Kubernetes Package Manager)

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm version



#Step 2: Create an AWS EKS Cluster

Now, let's create an EKS cluster with 1 control plane (master node) and 2 worker nodes.

eksctl create cluster --name flask-cluster --region us-east-1 --nodegroup-name flask-workers --node-type t3.medium --nodes 2

This process may take 10-15 minutes.


Verify the cluster is created:

kubectl get nodes


#Step 3: Set Up Jenkins

We’ll install Jenkins inside an EC2 instance.

3.1 Install Java

sudo apt install -y openjdk-11-jdk

java -version

3.2 Install Jenkins


wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins


3.3 Access Jenkins

Find your instance's public IP, then open Jenkins in your browser:

http://<EC2_PUBLIC_IP>:8080


To get the admin password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Follow the Jenkins setup wizard and install the suggested plugins.

3.4 Install Plugins

Go to Manage Jenkins → Manage Plugins → Install:

Docker Pipeline
Kubernetes CLI
AWS CLI
Pipeline


#Step 4: Set Up Docker and AWS ECR

Now, we need to set up Docker and Amazon Elastic Container Registry (ECR) for storing our Flask app images.

4.1 Create an ECR Repository

aws ecr create-repository --repository-name flask-repo

Retrieve your ECR URL:

aws ecr describe-repositories

4.2 Authenticate Docker with ECR

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ECR_URL>


#Step 5: Create a Flask App

#5.1 Flask App Structure

flask-app/
│── app.py
│── requirements.txt
│── Dockerfile
│── deployment.yaml
│── service.yaml
│── jenkinsfile

#5.2 app.py

from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask on AWS EKS!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


#5.3 requirements.txt

Flask==2.0.1

#5.4 Dockerfile

FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]


#5.5 Kubernetes Deployment (deployment.yaml)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - name: flask-app
        image: <ECR_URL>/flask-app:latest
        ports:
        - containerPort: 5000


#5.6 Kubernetes Service (service.yaml)

apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: LoadBalancer
  selector:
    app: flask
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000


#Step 6: Jenkins Pipeline (Jenkinsfile)

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "<ECR_URL>/flask-app"
        DOCKER_TAG = "latest"
    }

    stages {
        stage("Clone Repository") {
            steps {
                git branch: "main", url: "https://github.com/your-repo/flask-app.git"
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
                }
            }
        }

        stage("Push Image to ECR") {
            steps {
                script {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $DOCKER_IMAGE"
                    sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
                }
            }
        }

        stage("Deploy to EKS") {
            steps {
                script {
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                }
            }
        }
    }
}



Apply Your Kubernetes YAML Files

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl get pods




# Authenticate with ECR
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/b0m7c0k4

# Build and tag the image
docker build -t flask-app .
docker tag flask-app:latest public.ecr.aws/b0m7c0k4/jenkins-flask-repository:latest

# Push the image to ECR
docker push public.ecr.aws/b0m7c0k4/jenkins-flask-repository:latest


kubectl rollout restart deployment flask-app





#Step 7: Deploy and Test

Once the Jenkins pipeline runs successfully, check the deployment:

kubectl get pods
kubectl get services

Find the EXTERNAL-IP of the LoadBalancer service and visit it in your browser.







Project Name: CI/CD Pipeline for Flask Application on AWS EKS
Project Summary
This project demonstrates a complete CI/CD pipeline for deploying a Flask application to AWS EKS (Elastic Kubernetes Service) using Jenkins, Docker, AWS ECR, and Kubernetes. It covers the entire DevOps lifecycle—from code integration, security scanning, containerization, and pushing images to ECR, to deploying the application on Kubernetes in a highly scalable cloud environment.

Key Components & Workflow
Version Control

The source code is stored in GitHub and automatically pulled by Jenkins.
Continuous Integration (CI)

Jenkins Pipeline automates the process of pulling the latest code, scanning for security vulnerabilities, and building a Docker image.
Containerization

The Flask application is packaged into a Docker container using a Dockerfile and stored in AWS Elastic Container Registry (ECR).
Security & Quality Checks

SonarQube is used for code quality analysis.
Trivy scans the Docker image for vulnerabilities.
Continuous Deployment (CD) to AWS EKS

The containerized application is deployed to an AWS EKS Cluster using Kubernetes Deployment and Service Manifests.
A LoadBalancer Service exposes the application to the internet.
Automation & Scalability

Jenkins automates deployments upon every code push, ensuring seamless and zero-downtime releases.
Kubernetes enables auto-scaling to handle traffic surges dynamically.

🚀 Interview Answers for Your DevOps Project (AWS EKS + Jenkins CI/CD + Flask App)
Here’s a structured way to explain your project during an interview and impress the interviewer with clear, practical answers!


1️⃣ CI/CD Pipeline Implementation (Jenkins)
❓ Question: How does Jenkins automate the build, test, and deployment process?
💡 Answer:
Jenkins is used as the CI/CD automation tool in this project. Here’s how the pipeline works:

Code Integration – The pipeline starts by pulling the latest code from GitHub using the git plugin in Jenkins.
Static Code Analysis – Before building, the pipeline runs SonarQube to check for security vulnerabilities and code quality issues.
Docker Image Build – Once the code passes scanning, Jenkins uses Docker to build a containerized image of the Flask application.
Security Scanning – The built Docker image is scanned using Trivy to check for vulnerabilities.
Push to AWS ECR – After a successful build, the image is pushed to AWS Elastic Container Registry (ECR).
Deploy to AWS EKS – Finally, Jenkins applies Kubernetes manifests (Deployment, Service, Ingress) to deploy the app on Amazon EKS.
🔥 Key Benefits:
✅ Eliminates manual deployments
✅ Ensures quality and security before deployment
✅ Enables quick rollback in case of failure


2️⃣ Containerization & Orchestration (Docker & Kubernetes)
❓ Question: How do Docker and Kubernetes work together for scalable deployments?
💡 Answer:

Docker is used to package the Flask application into a container, making it portable and consistent across different environments.
Kubernetes (EKS) is used to manage these Docker containers at scale. It automates deployment, scaling, and networking.
The containerized Flask app runs inside Kubernetes Pods, and Kubernetes ensures high availability by balancing traffic using a Service.
🔥 Real-World Use Case:
If traffic increases, Kubernetes can automatically scale the number of replicas to handle the load.


3️⃣ AWS EKS Setup & Cluster Management
❓ Question: What are the steps to provision an EKS cluster and deploy applications?
💡 Answer:
To set up an EKS cluster, follow these steps:

Create IAM Roles – Grant necessary permissions for eksctl and worker nodes.
Install Tools – Install kubectl, aws-cli, and eksctl on your EC2 instance.
Create the EKS Cluster – Use eksctl to provision the cluster:

eksctl create cluster --name flask-cluster --region us-east-1 --nodegroup-name flask-nodes --node-type t3.medium --nodes 2

Deploy Flask App to EKS – Apply Kubernetes manifests:
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

Expose the App using an Ingress Controller – This allows external access.🔥 Key Benefits:
✅ Fully managed Kubernetes service
✅ Auto-scaling and self-healing pods
✅ Secure and integrated with AWS services

4️⃣ Security & Quality Assurance (SonarQube & Trivy)
❓ Question: Why are SonarQube and Trivy used in production-grade applications?
💡 Answer:

SonarQube scans the Flask application’s source code for bugs, vulnerabilities, and code smells.
Trivy scans the Docker image for known security vulnerabilities before deployment.
🔥 Example:
If SonarQube detects a SQL injection vulnerability, the pipeline fails early, preventing insecure code from reaching production.

5️⃣ Challenges & Solutions
❓ Question: What challenges did you face, and how did you resolve them?

Challenge	Solution
Kubernetes configuration issues	- Used kubectl describe pod <pod_name> and kubectl logs <pod_name> to debug misconfigurations.
Managing AWS authentication for Jenkins	- Created an IAM role for Jenkins and attached it using an EC2 instance profile.
Debugging failed pipeline stages	- Used Jenkins logs (/var/log/jenkins/jenkins.log) to identify errors and fix them.
Disk space issues on Jenkins EC2 instance	- Increased storage, cleared unused Docker images (docker system prune).

🚀 Final Summary (For Interview Conclusion)
💡 Project Name: Flask Microservice Deployment on AWS EKS using Jenkins CI/CD

✔ This project automates the CI/CD pipeline using Jenkins to build, test, scan, and deploy a Flask app.
✔ It leverages Docker for containerization and AWS EKS for Kubernetes orchestration.
✔ SonarQube & Trivy ensure code quality and security before deployment.
✔ The deployment is scalable, secure, and integrated with AWS cloud-native services.







📌 Step 1: Development & Source Control
Developer writes code (Flask application)
Code is pushed to GitHub
⬇️

📌 Step 2: Jenkins CI/CD Pipeline
1️⃣ Jenkins triggers build on new GitHub push
2️⃣ SonarQube scan for code quality
3️⃣ Docker image build (docker build)
4️⃣ Trivy scan for vulnerabilities
5️⃣ Push Docker image to AWS ECR
6️⃣ Apply Kubernetes manifests (kubectl apply)

⬇️

📌 Step 3: AWS EKS Deployment
1️⃣ Set up AWS EKS Cluster (eksctl create cluster)
2️⃣ Configure Kubernetes CLI (kubectl)
3️⃣ Deploy Flask app to EKS
4️⃣ Service exposure via LoadBalancer

⬇️

📌 Step 4: Monitoring & Debugging
Check pod & service status (kubectl get pods/services)
Handle errors (kubectl describe pod)
CI/CD monitoring in Jenkins
    
