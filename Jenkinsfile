pipeline {
    agent any
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'CheckoutService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }
         stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --exit-code 1'
            }
        }
        stage('Trivy fs Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh "DOCKER_BUILDKIT=1 docker build -t yukesh24/checkoutservice:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html yukesh24/checkoutservice:${BUILD_NUMBER}"
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push yukesh24/checkoutservice:${BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
