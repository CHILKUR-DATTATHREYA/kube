pipeline {
    // Defines the execution environment. Using 'docker' is best practice for Node.js.
    agent {
        // Use a Docker image that already contains Node.js and basic build tools.
        docker {
            image 'node:16-alpine'
            // Ensure the Jenkins agent has Docker access if building outside the container.
            args '-u 0:0' // Optional: Run as root inside the container for permissions
        }
    }

    environment {
        // --- Deployment Variables ---
        DOCKER_REGISTRY = 'williamdockerhub' 
        DOCKER_IMAGE_NAME = 'my-kubetest'
        DOCKER_FULL_NAME = "${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        // IMPORTANT: Replace 'docker-hub-creds-id' with the actual ID of your Jenkins credential
        DOCKERHUB_CREDENTIALS = 'docker-hub-creds-id' 
    }

    stages {
        stage('Build and Test') {
            steps {
                echo 'Installing dependencies and running tests...'
                // Install and test steps run inside the 'node:16-alpine' container defined above
                sh 'npm install'
                // sh 'npm test' // Uncomment if you have a test script
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building final image: ${DOCKER_FULL_NAME}:${DOCKER_TAG}"
                // Use the Docker binary available on the host machine to build the image
                // The '.' refers to the current workspace where your Dockerfile resides.
                sh "docker build -t ${DOCKER_FULL_NAME}:${DOCKER_TAG} ."
            }
        }

        stage('Push Image to Registry') {
            steps {
                // Securely use credentials for Docker login
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CREDENTIALS, 
                    passwordVariable: 'DOCKER_PASSWORD', 
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker push ${DOCKER_FULL_NAME}:${DOCKER_TAG}"
                    
                    // Tag and push as 'latest' after a successful build
                    sh "docker tag ${DOCKER_FULL_NAME}:${DOCKER_TAG} ${DOCKER_FULL_NAME}:latest"
                    sh "docker push ${DOCKER_FULL_NAME}:latest"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Applying deployment and service to Kubernetes...'
                // These commands assume 'kubectl' is installed and configured on the Jenkins agent
                sh "kubectl apply -f my-kube1-deployment.yaml"
                sh "kubectl apply -f my-kube1-service.yaml"
            }
        }
    }
}
