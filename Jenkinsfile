pipeline {
    agent any
    
    tools {
        NodeJS 'nodejs'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-react'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install --omit=dev'
            }
        }
        
        stage('Build Docker Image') {
            environment {
                DOCKER_IMAGE = "ratneshpuskar/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}"
            }
            steps {
                script {
                    def dockerImage = "${DOCKER_IMAGE}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh '''
                            echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin
                            docker push ${DOCKER_HUB_USERNAME}/${JOB_NAME.toLowerCase()}:${BUILD_NUMBER}
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
  name: ${JOB_NAME.toLowerCase()}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${JOB_NAME.toLowerCase()}
  template:
    metadata:
      labels:
        app: ${JOB_NAME.toLowerCase()}
    spec:
      containers:
      - name: ${JOB_NAME.toLowerCase()}
        image: ratneshpuskar/${JOB_NAME.toLowerCase()}:${BUILD_NUMBER}
        ports:
        - containerPort: 80
"""
                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: ${JOB_NAME.toLowerCase()}
spec:
  type: NodePort
  selector:
    app: ${JOB_NAME.toLowerCase()}
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
"""
                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml
                    sh 'scp -i /var/test.pem deployment.yaml service.yaml ubuntu@13.233.111.101:/tmp'
                    sh '''
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f /tmp/deployment.yaml"
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f /tmp/service.yaml"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment succeeded!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}