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
        stage('Scm Checkout') {
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
                echo "Testing..."
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
        
        stage("Push Docker Image") {
            steps {
                script {
                    echo "Push Docker Image"
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker push dockerhubdemos/devops:${env.BUILD_ID}
                        """
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubernetes', variable: 'GCP_KEY')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}
                        kubectl apply -f deployment.yaml
                    '''
                }
            }
        }

    }
}
        
