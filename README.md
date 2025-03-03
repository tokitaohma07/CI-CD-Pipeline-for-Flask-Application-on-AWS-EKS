# CI-CD-Pipeline-for-Flask-Application-on-AWS-EKS

# Project Summary

This project demonstrates a complete CI/CD pipeline for deploying a Flask application to AWS EKS (Elastic Kubernetes Service) using Jenkins, Docker, AWS ECR, and Kubernetes. It covers the entire DevOps lifecycle—from code integration, security scanning, containerization, and pushing images to ECR, to deploying the application on Kubernetes in a highly scalable cloud environment.


Step 1: Install Required Tools
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


