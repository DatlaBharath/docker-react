pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-react'
            }
        }

        stage('Build Project') {
            steps {
                sh 'npm install --no-optional'
                sh 'npm run build --skip-tests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo ${DOCKERHUB_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin'
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker push ${imageName}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: docker-react-deployment
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

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: docker-react-service
                    spec:
                      type: NodePort
                      selector:
                        app: docker-react
                      ports:
                      - protocol: TCP
                        port: 80
                        nodePort: 30007
                    """

                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.232.118.58 'kubectl apply -f -' <<< "${deploymentYaml}" """
                    sh """ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.232.118.58 'kubectl apply -f -' <<< "${serviceYaml}" """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}