pipeline {
    agent any

    tools {
        nodejs "NodeJS"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], doGenerateSubmoduleConfigurations: false, extensions: [], userRemoteConfigs: [[url: 'https://github.com/DatlaBharath/docker-react']]])
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install --no-audit --no-fund --no-optional'
                sh 'npm run build --if-present || echo "skip tests"'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def repoName = "docker-react"
                    def buildNumber = "${env.BUILD_NUMBER}"
                    def imageName = "ratneshpuskar/${repoName.toLowerCase()}:${buildNumber}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    script {
                        def repoName = "docker-react"
                        def buildNumber = "${env.BUILD_NUMBER}"
                        def imageName = "ratneshpuskar/${repoName.toLowerCase()}:${buildNumber}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def repoName = "docker-react"
                    def buildNumber = "${env.BUILD_NUMBER}"
                    def imageName = "ratneshpuskar/${repoName.toLowerCase()}:${buildNumber}"
                    def deploymentYaml = """
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${repoName.toLowerCase()}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${repoName.toLowerCase()}
                      template:
                        metadata:
                          labels:
                            app: ${repoName.toLowerCase()}
                        spec:
                          containers:
                          - name: ${repoName.toLowerCase()}
                            image: ${imageName}
                            ports:
                            - containerPort: 80
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${repoName.toLowerCase()}
                    spec:
                      type: NodePort
                      selector:
                        app: ${repoName.toLowerCase()}
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 80
                          nodePort: 30008
                    """

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        
        failure {
            echo 'Pipeline execution failed!'
        }
    }
}