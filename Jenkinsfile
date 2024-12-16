pipeline {
    agent any 
    
    environment {
        // Store GCP credentials and project ID as environment variables
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')
        GOOGLE_CLOUD_PROJECT = credentials('gcp-project-id')
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                // Checkout the main branch from the GitHub repository
                git branch: 'main', url: 'https://github.com/vijaygiduthuri/Netflix.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                // Build the Docker image from the Dockerfile in the repo
                sh "docker build -t netflix:latest ."
            }
        }
        // stage('Authenticate with Google Cloud') {
        //     steps {
        //         // Authenticate using the service account key
        //         sh """
        //         gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
        //         gcloud config set project ${GOOGLE_CLOUD_PROJECT}
        //         gcloud auth configure-docker us-central1-docker.pkg.dev
        //         """
        //     }
        // }
        stage('Tag Docker Image for Artifact Registry') {
            steps {
                // Tag the built Docker image for Artifact Registry
                sh "docker tag netflix:latest us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/docker-repo/netflix:latest"
            }
        }
        stage('Push Docker Image to Artifact Registry') {
            steps {
                // Push the tagged image to Artifact Registry
                sh "docker push us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/docker-repo/netflix:latest"
            }
        }
        stage('Cleanup Docker Image on VM') {
            steps {
                // Prune unused Docker images to free up space
                sh "docker image prune -f"
            }
        }
        stage('Cleanup Existing Container') {
            steps {
                // Stop and remove existing containers (if any)
                sh """
                if [ \$(docker ps -a -q -f name=netflix) ]; then
                    docker stop netflix || true
                    docker rm netflix || true
                fi
                """
            }
        }
        stage('Connect to GKE Cluster') {
            steps {
                // Authenticate and set up kubectl to interact with the GKE cluster
                sh """
                gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                gcloud config set project ${GOOGLE_CLOUD_PROJECT}
                gcloud container clusters get-credentials demo-gke-cluster --zone us-central1 --project ${GOOGLE_CLOUD_PROJECT}
                kubectl get nodes -o wide
                """
            }
        }
        stage('Deploy Application on GKE Using Helm') {
            steps {
                script {
                    // Navigate to the Helm chart directory and deploy or upgrade using Helm
                    dir('netflix-helm') {
                        // Use helm upgrade --install for idempotency (upgrade if exists, install if not)
                        sh "ls"
                        sh "ls .."
                        sh "pwd"
                        sh "helm upgrade --install netflix . --namespace default --debug"
                    }
                }
            }
        }
    }
}
