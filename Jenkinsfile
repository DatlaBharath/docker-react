pipeline {
    agent any
    tools {
        maven "Maven"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DatlaBharath/docker-react', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    script {
                        def dockerImage = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                        sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin"
                        sh "docker push ${dockerImage}"
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
                          name: docker-react-deployment
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
                          name: docker-react-service
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

                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)
                    
                    sh """ ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f -" < deployment.yaml """
                    sh """ ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f -" < service.yaml """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}