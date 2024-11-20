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
        stage ("Scan GitHub Repository Using Trivy") {
            steps {
                sh "trivy fs ."
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t netflix:latest ."
            }
        }
        stage ("Scan Docker Image Using Trivy") {
            steps {
                sh "trivy image netflix:latest"
            }
        }
        stage ("Deploy to Docker Conatiner") {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 netflix:latest"
            }
        }
    }
}    
