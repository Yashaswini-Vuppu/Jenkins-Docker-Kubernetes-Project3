pipeline {
    agent any
    tools {
        maven 'maven-3'
    }

    environment {
        PROJECT_ID     = 'sharp-ring-407510'
        CLUSTER_NAME   = 'gke-1'
        LOCATION       = 'asia-south1'
        CREDENTIALS_ID = 'kubernetes'
        WORKSPACE_BIN  = "$WORKSPACE/bin"
    }

    stages {
        stage('Scm Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    myimage = docker.build("dockerhubdemos/devops:${env.BUILD_ID}")
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push dockerhubdemos/devops:${env.BUILD_ID}
                    """
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'kubernetes', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh """
                            set -e
                            mkdir -p $WORKSPACE_BIN

                            echo "Installing kubectl..."
                            curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                            chmod +x kubectl
                            mv kubectl $WORKSPACE_BIN/kubectl

                            echo "Installing Google Cloud SDK..."
                            rm -rf $WORKSPACE/google-cloud-sdk
                            curl -sSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-417.0.0-linux-x86_64.tar.gz | tar -xz -C $WORKSPACE
                            
                            GCP_BIN="$WORKSPACE/google-cloud-sdk/bin"
                            export PATH=$WORKSPACE_BIN:$GCP_BIN:\$PATH

                            echo "Activating service account..."
                            bash -c "gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS"
                            bash -c "gcloud config set project $PROJECT_ID"
                            bash -c "gcloud container clusters get-credentials $CLUSTER_NAME --zone $LOCATION --project $PROJECT_ID"

                            echo "Applying Kubernetes manifests..."
                            sed -i 's/tagversion/${BUILD_ID}/g' serviceLB.yaml
                            sed -i 's/tagversion/${BUILD_ID}/g' deployment.yaml
                            bash -c "kubectl apply -f serviceLB.yaml"
                            bash -c "kubectl apply -f deployment.yaml"
                        """
                    }
                }
            }
        }
    }
}
