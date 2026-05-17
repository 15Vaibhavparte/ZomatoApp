pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        // Define Docker credentials and image details here for easy updates
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
                    // This securely grabs your 'docker' credential from Jenkins
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        
                        // Log in to DockerHub using the standard CLI
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        
                        // Tag and Push
                        sh "docker tag zomato ${IMAGE_REPO}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_REPO}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script {
                   withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                       
                       // Docker requires you to be logged in to pull vulnerability databases
                       sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                       
                       sh "docker-scout quickview ${IMAGE_REPO}:${IMAGE_TAG}"
                       sh "docker-scout cves ${IMAGE_REPO}:${IMAGE_TAG}"
                       sh "docker-scout recommendations ${IMAGE_REPO}:${IMAGE_TAG}"
                   }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                // Added a cleanup step so consecutive builds don't fail due to the 'zomato' container name already being in use
                sh "docker rm -f zomato || true"
                sh "docker run -d --name zomato -p 3000:3000 ${IMAGE_REPO}:${IMAGE_TAG}"
            }
        }
    }
}
