pipeline {
    agent any
    parameters {
        string(name: 'VERSION', description: 'Enter the APP VERSION')
    }
    environment {
        AWS_ACCOUNT_ID = "730335278282"
        REGION = "ap-south-1"
        REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/catalogue"
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REGISTRY_CREDENTIALS = 'docker-creds'
    }
    stages {
        stage('Clone') {
            steps {
                script {
                    echo "Clone started"
                    gitInfo = checkout scm
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    sh "docker build -t catalogue:${VERSION} ."
                }
            }
        }
        stage('Image push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-auth', region: "${REGION}") {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com
                            docker tag catalogue:${VERSION}  ${REPO_URI}:${VERSION}
                            docker push ${REPO_URI}:${VERSION}
                        """
                    }
                }
            }
        }
        stage('Image push to Docker Hub') {
            steps {
                script {
                    // Using withCredentials to securely access Docker Hub credentials
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // Logging into Docker Hub and pushing the image
                        sh """
                            echo \${DOCKER_PASSWORD} | docker login -u \${DOCKER_USERNAME} --password-stdin
                            docker tag catalogue:${VERSION} macharavikishore/catalogue:${VERSION}
                            docker push macharavikishore/catalogue:${VERSION}
                        """
                    }
                }
            }
        }
        stage('Deployment') {
            steps {
                script {
                    withAWS(credentials: 'aws-auth', region: "${REGION}") {
                        sh """
                            aws eks update-kubeconfig --region ${REGION} --name spot-cluster
                            cd helm
                            helm install catalogue . --set deployment.imageVersion=${VERSION}
                        """
                    }
                }
            }
        }
    }
}
