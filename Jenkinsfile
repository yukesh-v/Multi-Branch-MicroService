pipeline {
    agent any

    tools {
        nodejs 'nodejs23'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'CurrencyService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }
        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html'
            }
        }
        stage('Compilation') {
            steps {
                sh 'find . -name "*.js" -exec node --check {} +'
            }
        }
        stage('Trivy fs Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=CurrencyService -Dsonar.projectKey=CurrencyService"""
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker build -t yukesh24/currencyservice:${BUILD_NUMBER} ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o currencyservice-image-report.html yukesh24/currencyservice:${BUILD_NUMBER}"
            }
        }
        stage('Docker Push') {
            steps {
                script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker push yukesh24/currencyservice:${BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
