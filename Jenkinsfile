pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/docker-react']]
                ])
            }
        }
        stage('Build Project') {
            steps {
                sh 'npm install --no-optional'
                sh 'npm run build'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker build -t ${imageName} ."
                        sh "docker push ${imageName}"
                        sh 'docker logout'
                    }
                }
            }
        }
        stage('Create and Deploy YAMLs') {
            steps {
                script {
                    def deployment = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: docker-react
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: docker-react
                      template:
                        metadata:
                          labels:
                            app: docker-react
                        spec:
                          containers:
                          - name: docker-react
                            image: ratneshpuskar/docker-react:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 80
                    """
                    def service = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: docker-react
                    spec:
                      type: NodePort
                      selector:
                        app: docker-react
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 80
                          nodePort: 30007
                    """
                    writeFile file: 'deployment.yaml', text: deployment
                    writeFile file: 'service.yaml', text: service

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.109.185.68 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.109.185.68 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}