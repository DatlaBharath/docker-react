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
                sh 'npm install --skip-tests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imgName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imgName} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                        sh "docker push ${imgName}"
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
                      name: docker-react
                      labels:
                        app: docker-react
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
                      name: docker-react
                    spec:
                      type: NodePort
                      ports:
                        - port: 80
                          nodePort: 30007
                          protocol: TCP
                      selector:
                        app: docker-react
                    """
                    sh "echo \"${deploymentYaml}\" | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.206.66.91 'kubectl apply -f -'"
                    sh "echo \"${serviceYaml}\" | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@15.206.66.91 'kubectl apply -f -'"
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}