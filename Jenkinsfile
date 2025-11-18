pipeline {
    agent any 

    environment {
        // Use your specific Docker variables
        DOCKER_REGISTRY = 'williamdockerhub' 
        DOCKER_IMAGE_NAME = 'my-kubetest'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        // IMPORTANT: Replace 'docker-hub-creds' with the ID of your Jenkins credential
        DOCKERHUB_CREDENTIALS = 'docker-hub-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // 'checkout scm' is often implicit but good to keep.
            }
        }

        stage('Build Image') {
            steps {
                echo "Building image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG}"
                // The '.' indicates the Dockerfile is in the current workspace directory.
                sh "docker build -t ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                // Use stored credentials for secure login before pushing
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CREDENTIALS, 
                    passwordVariable: 'DOCKER_PASSWORD', 
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker push ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${env.DOCKER_TAG}"
                }
            }
        }
    }
}
