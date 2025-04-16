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
