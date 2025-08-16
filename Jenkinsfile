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
        
        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'kubernetes', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo "Deployment started ..."

                        // ✅ Install kubectl + gke plugin if not present
                        sh '''
                            echo "Installing kubectl and gke-gcloud-auth-plugin..."
                            sudo apt-get update -y
                            sudo apt-get install -y kubectl google-cloud-sdk-gke-gcloud-auth-plugin
                        '''

                        // Replace tagversion with BUILD_ID
                        sh "sed -i 's/tagversion/${env.BUILD_ID}/g' serviceLB.yaml"
                        sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"

                        // Authenticate with GCP + deploy
                        sh """
                            echo "Activating service account..."
                            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                            gcloud config set project ${env.PROJECT_ID}
                            gcloud container clusters get-credentials ${env.CLUSTER_NAME} --zone ${env.LOCATION} --project ${env.PROJECT_ID}

                            echo "Applying Kubernetes manifests..."
                            kubectl apply -f serviceLB.yaml
                            kubectl apply -f deployment.yaml
                        """ 

                        echo "Deployment Finished ..."
                    }
                }
            }
        }
    }
}
