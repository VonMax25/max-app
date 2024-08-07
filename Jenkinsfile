pipeline {
    agent { label 'agent9' }  // Replace 'wsl-agent' with the label of your agent

    environment {
        GITHUB_REPO_URL = 'https://github.com/VonMax25/max-app.git'
        BRANCH_NAME = 'main'  // Replace with your branch name if it's not 'main'
        GITHUB_CREDENTIALS_ID = 'drama-github-token'  // Replace with your Jenkins GitHub credentials ID
        DOCKERHUB_CREDENTIALS_ID = 'drama-dockerhub-cred'  // Replace with your Jenkins Docker Hub credentials ID
        DOCKERHUB_REPO = 'vonmax25/max-app1'  // Replace with your Docker Hub repository
    }

    stages {
        stage('Agent Details') {
            steps {
                echo "Running on agent: ${env.NODE_NAME}"
                sh 'uname -a'  // Print system information
                sh 'whoami'    // Print the current user 
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: "${env.GITHUB_REPO_URL}", credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.9.6/bin/mvn clean package'  // Simple Maven build
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker --version'  // Verify Docker installation
                    sh "docker build -t ${env.DOCKERHUB_REPO}:latest ."  // Build Docker image
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh "docker push ${env.DOCKERHUB_REPO}:latest"
                        sh 'docker logout'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh "docker run --name max-app-con --rm -d -p 8000:8080 ${env.DOCKERHUB_REPO}:latest"  // Run Docker container in detached mode
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker containers and images...'
            sh 'docker rm $(docker ps -a -q) || true'
            sh 'docker rmi $(docker images -q) || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
