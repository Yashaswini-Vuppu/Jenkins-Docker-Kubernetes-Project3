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
    }
    
    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean install'
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'mvn test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'whoami'
                script {
                    myimage = docker.build("dockerhubdemos/devops:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker Image..."
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker push dockerhubdemos/devops:${env.BUILD_ID}
                        """
                    }
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                echo "Starting deployment to Kubernetes..."
                withCredentials([file(credentialsId: env.CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        # Authenticate with GCP
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud container clusters get-credentials $CLUSTER_NAME --zone $LOCATION --project $PROJECT_ID

                        # Update image tags in YAML files
                        sed -i "s/tagversion/${BUILD_ID}/g" serviceLB.yaml
                        sed -i "s/tagversion/${BUILD_ID}/g" deployment.yaml

                        # Apply manifests
                        kubectl apply -f serviceLB.yaml
                        kubectl apply -f deployment.yaml
                    '''
                }
                echo "Deployment completed."
            }
        }
    }
}
