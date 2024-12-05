pipeline {
    agent any 
    
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/vijaygiduthuri/Netflix.git'
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t netflix:latest ."
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
        stage ("Deploy to Docker Conatiner") {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 netflix:latest"
            }
        }
    }
}    
