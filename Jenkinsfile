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
        stage('Docker Build & Push') {
            steps {
                script {
                    // Builds the image using your repo name and build number tag
                    def app = docker.build("${IMAGE_REPO}:${IMAGE_TAG}")
                    
                    // Logs in to DockerHub using your credentials
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDS}") {
                        
                        // Pushes the image with the specific Build Number (e.g., parte15/zomato:42)
                        app.push()
                        
                        // Also pushes and overwrites the 'latest' tag (e.g., parte15/zomato:latest)
                        app.push('latest')
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
