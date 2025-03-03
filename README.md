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


$5.5 Kubernetes Deployment (deployment.yaml)



