pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16' // Using node16 to match your Docker container environment
    }
    environment {  
        // Define Docker and Image details here
        DOCKER_CREDS = 'docker'
        IMAGE_REPO = 'parte15/zomato'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/15Vaibhavparte/ZomatoApp.git'
            }
        }
        
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    // Securely inject Docker credentials using native Jenkins commands
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        
                        // Tag and push using the dynamic build number
                        sh "docker tag zomato ${IMAGE_REPO}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_REPO}:${IMAGE_TAG}"
                        
                        // Push 'latest' as a backup
                        sh "docker tag zomato ${IMAGE_REPO}:latest"
                        sh "docker push ${IMAGE_REPO}:latest"
                    }
                }
            }
        }
        
        
        stage ("Deploy to Kubernetes Cluster") {
            steps {
                // 1. Apply the baseline configurations from your GitHub repo
                sh "kubectl apply -f Kubernetes/deployment.yaml"
                sh "kubectl apply -f Kubernetes/service.yaml"
                sh "kubectl apply -f Kubernetes/node-service.yaml"
                
                // 2. Force the deployment to use the exact image we just built & pushed
                // Syntax: kubectl set image deployment/<Deployment-Name> <Container-Name>=<Image>
                sh "kubectl set image deployment/zomato zomato=${IMAGE_REPO}:${IMAGE_TAG}"
                
                // 3. Monitor the rollout to ensure pods boot up successfully
                sh "kubectl rollout status deployment/zomato"
            }
        }
    }
}