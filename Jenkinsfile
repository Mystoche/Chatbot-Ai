pipeline {
    // Default agent: any (allows builds on any available Jenkins node)

    // Trigger pipeline execution on every push to the master branch (can be customized)
    triggers {
        githubPush events: 'push', branches: 'master' // Can be adjusted for other branches
    }

    stages {
        // Checkout the code from GitHub
        stage('Checkout') {
            steps {
                script {
                    git branch: 'master', credentialsId: 'github-credentials', url: '<your-git-repository-url>'
                }
            }
        }

        // Build Docker image for Rasa
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t rasa-image .' // Assuming a Dockerfile exists at the root
            }
        }

        // Train and Test Rasa model inside the container
        stage('Train and Test Rasa Model') {
            steps {
                sh 'docker run rasa-image rasa train'
                sh 'docker run rasa-image rasa test'
            }
        }

        // Deploy the Rasa image to Minikube (assuming Minikube is already set up)
        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'minikube status || minikube start' // Start Minikube if not running

                    // Dynamic kubectl command based on Minikube version
                    sh 'minikube version | grep 1.24 && kubectl="minikube kubectl" || kubectl="kubectl"'

                    // Load Minikube Docker environment
                    sh 'eval $(minikube docker-env)'

                    // Deploy Rasa image as a deployment
                    sh "$kubectl create deployment rasa-deployment --image=rasa-image"

                    // Expose Rasa service with a NodePort
                    sh "$kubectl create service type=NodePort port=5005 rasa-deployment"
                }
            }
        }

        
    }
}
