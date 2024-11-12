pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/docker-react', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install --only=dev'
                sh 'npm run build --if-present'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh """
                        docker build -t ${imageName} .
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo ${DOCKERHUB_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin'
                    sh 'docker push ratneshpuskar/docker-react:${env.BUILD_NUMBER}'
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
                          targetPort: 80
                          nodePort: 30008
                      selector:
                        app: docker-react
                    """
                    
                    sh """
                        echo "${deploymentYaml}" > deployment.yaml
                        echo "${serviceYaml}" > service.yaml 

                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 "kubectl apply -f -" < service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'The deployment was successful!'
        }
        failure {
            echo 'The deployment failed.'
        }
    }
}