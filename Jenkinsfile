pipeline {
    agent any
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'EmailService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }
        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html'
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
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker build -t yukesh24/emailservice:${BUILD_NUMBER} ."
                         }
                     }
                }
            }
            stage('Trivy Image Scan') {
                steps {
                    sh "trivy image --format table -o emailservice-image-report.html yukesh24/emailservice:${BUILD_NUMBER}"
                }
            }
            stage('Docker Push') {
                steps {
                    script {
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker push yukesh24/emailservice:${BUILD_NUMBER}"
                        }
                    }
                }
            }
        }
    }

