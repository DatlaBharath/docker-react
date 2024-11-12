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
                    sh 'npm run build --skip-tests'
                }
            }
            stage('Docker Build and Push') {
                steps {
                    script {
                        def repoName = 'docker-react'.toLowerCase()
                        def imageName = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                        withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                            sh 'echo ${DOCKERHUB_PASSWORD}'
                            sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                            sh "docker build -t ${imageName} ."
                            sh "docker push ${imageName}"
                        }
                    }
                }
            }
            stage('Deploy to Kubernetes') {
                steps {
                    script {
                        def repoName = 'docker-react'.toLowerCase()
                        def deploymentYaml = """
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: ${repoName}
                        spec:
                          replicas: 1
                          selector:
                            matchLabels:
                              app: ${repoName}
                          template:
                            metadata:
                              labels:
                                app: ${repoName}
                            spec:
                              containers:
                              - name: ${repoName}
                                image: ratneshpuskar/${repoName}:${env.BUILD_NUMBER}
                                ports:
                                - containerPort: 80
                        """
                        def serviceYaml = """
                        apiVersion: v1
                        kind: Service
                        metadata:
                          name: ${repoName}
                        spec:
                          type: NodePort
                          selector:
                            app: ${repoName}
                          ports:
                          - port: 80
                            targetPort: 80
                            nodePort: 30007
                        """
                        sh "ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 'kubectl apply -f -' <<< \""\"${deploymentYaml}\"\""
                        sh "ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@13.234.110.244 'kubectl apply -f -' <<< \""\"${serviceYaml}\"\""
                    }
                }
            }
        }
        post {
            success {
                echo 'Deployment was successful!'
            }
            failure {
                echo 'Deployment failed!'
            }
        }
    }