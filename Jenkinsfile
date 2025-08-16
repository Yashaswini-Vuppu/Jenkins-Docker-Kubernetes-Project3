pipeline {
    agent any

    tools {
        maven 'maven-3'
    }

    environment {
        PROJECT_ID    = 'sharp-ring-407510'
        CLUSTER_NAME  = 'gke-1'
        LOCATION      = 'asia-south1'
        CREDENTIALS_ID = 'kubernetes'
        DOCKERHUB_USER = 'dockerhubdemos'
        IMAGE_NAME    = 'devops'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "Building Docker image: ${imageTag}"
                    sh "docker build -t ${imageTag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    withCredentials([string(credentialsId: 'docker', variable: 'dockerPassword')]) {
                        sh """
                            echo "${dockerPassword}" | docker login -u ${DOCKERHUB_USER} --password-stdin
                            docker push ${imageTag}
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG
                        kubectl apply -f deployment.yaml
                    """
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up Docker images/containers to free space"
            sh '''
                docker system prune -af || true
                docker volume prune -f || true
            '''
        }
        success {
            echo "✅ Build, Push and Deploy completed successfully!"
        }
        failure {
            echo "❌ Build failed. Please check logs."
        }
    }
}
