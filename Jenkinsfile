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
                sh 'npm install --no-optional'
                sh 'npm run build --skip-tests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "ratneshpuskar/docker-react:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
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
      targetPort: 80
  type: NodePort
  nodePort: 30007
"""
                    sh "echo '${deploymentYaml}' > deployment.yaml"
                    sh "echo '${serviceYaml}' > service.yaml"
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.58.215 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.201.58.215 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}