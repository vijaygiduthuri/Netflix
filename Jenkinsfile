pipeline {
    agent any 
    
    environment {
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
                git branch: 'main', url: 'https://github.com/vijaygiduthuri/Netflix.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t netflix:latest ."
            }
        }
        stage('Authenticate with Google Cloud') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                    sh "gcloud config set project ${GOOGLE_CLOUD_PROJECT}"
                    sh "gcloud auth configure-docker us-central1-docker.pkg.dev"
                }
            }
        }
        stage('Tag Docker Image for Artifact Registry') {
            steps {
                sh "docker tag netflix:latest us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/docker-repo/netflix:latest"
            }
        }
        stage('Push Docker Image to Artifact Registry') {
            steps {
                sh "docker push us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/docker-repo/netflix:latest"
            }
        }
        stage ('Cleanup Docker Image on VM'){
            steps {
                // Remove all unused Docker images
                sh "docker image prune -f"
            }
        }
        stage('Cleanup Existing Container') {
            steps {
                // Stop and remove the existing container if it exists
                sh """
                if [ \$(docker ps -a -q -f name=netflix) ]; then
                    docker stop netflix || true
                    docker rm netflix || true
                fi
                """
            }
        }
        stage('Deploy Docker Image from Artifact Registry') {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/docker-repo/netflix:latest"
            }
        }
    }
}
