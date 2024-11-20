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
                git branch: 'main', url: 'https://github.com/vijaygiduthuri/Netflix-Clone.git'
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t netflix ."
            }
        }
        stage ("Deploy to Docker Conatiner") {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 netflix:latest"
            }
        }
    }
}    
