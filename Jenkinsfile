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
                sh 'npm install --only=prod'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh """
                            echo "${DOCKER_HUB_PASSWORD}" | docker login -u "${DOCKER_HUB_USERNAME}" --password-stdin
                            docker build -t ${imageName} .
                            docker push ${imageName}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployment = '''
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
'''
                    def service = '''
apiVersion: v1
kind: Service
metadata:
  name: docker-react
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30007
  selector:
    app: docker-react
'''

                    sh """
                        echo '${deployment}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f -"
                        echo '${service}' | ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.233.111.101 "kubectl apply -f -"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'The build and deployment have been successful!'
        }
        failure {
            echo 'The build or deployment failed.'
        }
    }
}