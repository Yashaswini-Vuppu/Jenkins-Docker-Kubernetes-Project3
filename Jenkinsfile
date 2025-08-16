pipeline {
    agent any

    tools {
        maven 'maven-3'
    }

    environment {
        PROJECT_ID   = 'sharp-ring-407510'
        CLUSTER_NAME = 'gke-1'
        LOCATION     = 'asia-south1'
        CREDENTIALS_ID = 'kubernetes'  // GCP Service Account Key
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    echo "Building with Maven..."
                    sh "mvn clean install -DskipTests"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image"
                    def dockerImage = docker.build("dockerhubdemos/devops:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker Image"
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
            environment {
                PATH = "$WORKSPACE:$PATH"
            }
            steps {
                withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo "Deployment started ..."

                        // Install kubectl if not present
                        sh '''
                            set -e
                            if [ ! -f "$WORKSPACE/kubectl" ]; then
                              echo "Installing kubectl..."
                              curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                              chmod +x kubectl
                              mv kubectl $WORKSPACE/
                            else
                              echo "kubectl already installed in workspace"
                            fi
                        '''

                        // Install gke-gcloud-auth-plugin if not present
                        sh '''
                            if ! command -v gke-gcloud-auth-plugin >/dev/null 2>&1; then
                              echo "Installing gke-gcloud-auth-plugin..."
                              sudo apt-get update -y
                              sudo apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin
                            fi
                        '''

                        // Auth and configure cluster
                        sh '''
                            echo "Activating service account..."
                            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                            gcloud config set project ${PROJECT_ID}
                            gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}

                            echo "Updating manifests..."
                            sed -i "s/tagversion/${BUILD_ID}/g" serviceLB.yaml
                            sed -i "s/tagversion/${BUILD_ID}/g" deployment.yaml

                            echo "Applying Kubernetes manifests..."
                            $WORKSPACE/kubectl apply -f serviceLB.yaml
                            $WORKSPACE/kubectl apply -f deployment.yaml
                        '''
                    }
                }
            }
        }
    }
}
