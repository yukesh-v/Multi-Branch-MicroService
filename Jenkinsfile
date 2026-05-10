pipeline {
    agent any
 
environment {
    GIT_COMMIT_REV = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
} 
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'CartService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
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
                    dir('src') {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker build -t yukesh24/cartservice:${GIT_COMMIT_REV} ."
                     }
                   }    
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html yukesh24/cartservice:${GIT_COMMIT_REV}"
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    dir('src') {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push yukesh24/cartservice:${GIT_COMMIT_REV}"
                      }
                   }
                }
            }
        }
    }
}
