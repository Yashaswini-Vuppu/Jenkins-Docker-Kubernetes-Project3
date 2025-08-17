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
                    // Ensure this image name matches what's in your deployment.yaml
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
            steps{
                echo "Deployment started ..."
                sh 'ls -ltr'
                sh 'pwd'
                // Apply BUILD_ID to the deployment.yaml, as service.yaml doesn't need it
                sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
                
                echo "Start deployment of service.yaml"
                // Deploy the Service first. No verification needed for a Service.
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'service.yaml', credentialsId: env.CREDENTIALS_ID])
                
                echo "Start deployment of deployment.yaml"
                // Deploy the Deployment second, and verify its readiness.
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
                echo "Deployment Finished ..."
            }
        }
    }
}
