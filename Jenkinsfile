
pipeline {
    agent any

    stages {
        stage('Build') {
            steps { // <--- REQUIRED: steps block opens here
                script {
                    def imageTag = "my-k8s-app:${env.BUILD_NUMBER}"
                    dir('kube') {
                        // 1. Build
                       bat "docker build -t ${imageTag} -f Dockerfile ."
                        
                        // 2. Tag and Push (Ensure 'your-registry-username' is correct)
                        bat "docker tag ${imageTag} abhizoe/my-k8s-app:${env.BUILD_NUMBER}"
                        bat "docker push abhizoe/my-k8s-app:${env.BUILD_NUMBER}"
                        
                        env.IMAGE_TAG = "abhizoe/my-k8s-app:${env.BUILD_NUMBER}"
                    }
                }
            } // <--- REQUIRED: steps block closes here
        }

        stage('Test') {
            steps {
                echo "Running tests..." 
                // Add actual test commands here if needed
            }
        }

        stage('Deploy') {
            steps { // <--- REQUIRED: steps block opens here
                script {
                    dir('kube') {
                        echo "Deploying application with image tag: ${env.IMAGE_TAG}"
                        bat "sed -i 's|node-app-image-placeholder|${env.IMAGE_TAG}|g' node-app-deployment.yaml"
                        
                        // 2. Apply the manifest files to the Kubernetes cluster
                        bat 'kubectl apply -f node-app-deployment.yaml'
                        bat 'kubectl apply -f node-app-service.yaml'
                        
                        // 3. Wait for the deployment to complete
                        bat 'kubectl rollout status deployment/node-app-deployment'
                        
                        echo "Deployment to Kubernetes complete."
                    }
                }
            } // <--- REQUIRED: steps block closes here
        }
    }
}
