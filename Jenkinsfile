pipeline {
    agent any
    
    tools {
        nodejs "NodeJS"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-react'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install --silent'
                sh 'npm run build --silent --skip-tests'
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUsername')]) {
                    script {
                        sh "echo $dockerHubPassword | docker login -u $dockerHubUsername --password-stdin"
                        def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                        sh "docker tag ${imageName} ratneshpuskar/docker-react:latest"
                        sh "docker push ${imageName}"
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
  selector:
    app: docker-react
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30007
  type: NodePort
"""

                    sh """
echo "${deploymentYaml}" > deployment.yaml
echo "${serviceYaml}" > service.yaml
ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.58.215 "kubectl apply -f -" < deployment.yaml
ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.58.215 "kubectl apply -f -" < service.yaml 
"""
                }
            }
        }
    }

    post {
        success {
            echo 'Job succeeded!'
        }
        failure {
            echo 'Job failed!'
        }
    }
}