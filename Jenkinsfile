pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-react'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def nodeHome = tool 'NodeJS'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def customImage = docker.build("ratneshpuskar/react-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ratneshpuskar/react-app:${env.BUILD_ID}
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/react-app-deployment react-app=ratneshpuskar/react-app:${env.BUILD_ID} --record
                        kubectl rollout status deployment/react-app-deployment
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
    }
}