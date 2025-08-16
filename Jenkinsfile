pipeline {
    agent any

    tools {
        maven 'maven-3'
    }

    environment {
        PROJECT_ID   = 'sharp-ring-407510'
        CLUSTER_NAME = 'gke-1'
        LOCATION     = 'asia-south1'
        CREDENTIALS_ID = 'kubernetes'
        DOCKERHUB_CREDS = 'docker'
    }

    stages {
        stage('Scm Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image"
                    dockerImage = docker.build("dockerhubdemos/devops:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker Image"
                    withCredentials([string(credentialsId: "${DOCKERHUB_CREDS}", variable: 'docker')]) {
                        sh "echo ${docker} | docker login -u dockerhubdemos --password-stdin"
                    }
                    dockerImage.push()
                    dockerImage.push("latest")
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withEnv(["PATH+TOOLS=${WORKSPACE}:/usr/local/bin"]) {
                    withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        script {
                            echo "Deployment started ..."

                            sh '''
                                set -e
                                
                                # Install kubectl if not present
                                if ! command -v kubectl >/dev/null 2>&1; then
                                    echo "Installing kubectl..."
                                    KUBECTL_VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
                                    curl -LO https://storage.googleapis.com/kubernetes-release/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl
                                    chmod +x kubectl
                                    mv kubectl ${WORKSPACE}/kubectl
                                else
                                    echo "kubectl already installed"
                                fi

                                # Install gke-gcloud-auth-plugin if not present
                                if ! command -v gke-gcloud-auth-plugin >/dev/null 2>&1; then
                                    echo "Installing gke-gcloud-auth-plugin..."
                                    sudo apt-get update -y
                                    sudo apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin
                                else
                                    echo "gke-gcloud-auth-plugin already installed"
                                fi

                                export PATH=${WORKSPACE}:$PATH

                                # Update image tag in manifests
                                sed -i "s/tagversion/${BUILD_ID}/g" deployment.yaml
                                sed -i "s/tagversion/${BUILD_ID}/g" serviceLB.yaml

                                echo "Activating service account..."
                                gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                                gcloud config set project ${PROJECT_ID}
                                gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}

                                echo "Applying Kubernetes manifests..."
                                kubectl apply -f serviceLB.yaml --validate=false
                                kubectl apply -f deployment.yaml --validate=false
                            '''
                        }
                    }
                }
            }
        }
    }
}
