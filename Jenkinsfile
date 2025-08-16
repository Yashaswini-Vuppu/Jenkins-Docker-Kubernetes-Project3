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
        WORKSPACE_BIN  = "${env.WORKSPACE}/bin"
        KUBECONFIG     = "${env.WORKSPACE}/kubeconfig"
    }

    stages {
        stage('SCM Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps { sh 'mvn test' }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    myimage = docker.build("dockerhubdemos/devops:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push dockerhubdemos/devops:${env.BUILD_ID}
                    """
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'kubernetes', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh '''
                            set -e
                            mkdir -p "$WORKSPACE/bin"

                            echo "Installing kubectl..."
                            bash -c "curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                            chmod +x kubectl
                            mv kubectl "$WORKSPACE/bin/kubectl"

                            echo "Installing Google Cloud SDK..."
                            rm -rf "$WORKSPACE/google-cloud-sdk"
                            curl -sSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-417.0.0-linux-x86_64.tar.gz | tar -xz -C "$WORKSPACE"
                            export PATH="$WORKSPACE/bin:$WORKSPACE/google-cloud-sdk/bin:$PATH"

                            echo "Activating service account..."
                            bash -c "gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS"
                            bash -c "gcloud config set project $PROJECT_ID"

                            echo "Generating kubeconfig..."
                            bash -c "gcloud container clusters get-credentials $CLUSTER_NAME --zone $LOCATION --project $PROJECT_ID --internal-ip=false --kubeconfig=$KUBECONFIG"

                            echo "Updating Kubernetes manifests..."
                            sed -i 's/tagversion/${BUILD_ID}/g' serviceLB.yaml
                            sed -i 's/tagversion/${BUILD_ID}/g' deployment.yaml

                            echo "Deploying to Kubernetes..."
                            bash -c "$WORKSPACE/bin/kubectl --kubeconfig=$KUBECONFIG apply -f serviceLB.yaml"
                            bash -c "$WORKSPACE/bin/kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml"
                        '''
                    }
                }
            }
        }
    }
}
