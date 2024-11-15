pipeline {
    agent any
    tools { 
        nodejs 'NodeJS'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-react'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install --no-audit --no-fund'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    
                    sh 'docker build -t ' + imageName + ' .'
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                        
                        sh 'echo "${PASSWORD}" | docker login -u "${USERNAME}" --password-stdin'
                        sh 'docker push ' + imageName
                    }
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
                      name: react-app-deployment
                    spec:
                      replicas: 2
                      selector:
                        matchLabels:
                          app: react-app
                      template:
                        metadata:
                          labels:
                            app: react-app
                        spec:
                          containers:
                          - name: react-app
                            image: ratneshpuskar/docker-react:${env.BUILD_NUMBER}
                            ports:
                            - containerPort: 80
                    """
                    
                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: react-app-service
                    spec:
                      selector:
                        app: react-app
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 80
                          nodePort: 30007
                      type: NodePort
                    """
                    
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.232.118.58 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.232.118.58 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}