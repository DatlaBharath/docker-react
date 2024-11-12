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
                sh 'npm install --no-optional'
                sh 'npm run build -- --skip-tests'
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker build -t ${imageName} .
                            docker push ${imageName}
                        '''
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
                        nodePort: 30007
                      selector:
                        app: docker-react
                    """
                    
                    writeFile(file: 'deployment.yaml', text: deploymentYaml)
                    writeFile(file: 'service.yaml', text: serviceYaml)
                    
                    sh '''
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.186.252 "kubectl apply -f -" < deployment.yaml
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.186.252 "kubectl apply -f -" < service.yaml
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}