pipeline {
    agent any
    options { timestamps() }
    environment {
        IMAGE = 'ghaith/monapp'  // Remplacez par votre nom Docker Hub
        TAG = "build-${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm  // Lit le repository Git
            }
        }
        
        stage('Docker Build') {
            steps {
                bat 'docker version'
                bat "docker build -t %IMAGE%:%TAG% ."
            }
        }
        
        stage('Smoke Test') {
            steps {
                bat """
                docker rm -f monapp_test 2>null || ver > null
                docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
                ping -n 3 127.0.0.1 > null
                curl -I http://localhost:8081 | find "200 OK"
                docker rm -f monapp_test
                """
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'USER', 
                    passwordVariable: 'PASS'
                )]) {
                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker tag %IMAGE%:%TAG% %IMAGE%:latest
                    docker push %IMAGE%:%TAG%
                    docker push %IMAGE%:latest
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Build+Test+Push OK'
        }
        failure {
            echo '❌ Build/Tests/Push KO'
        }
    }
}