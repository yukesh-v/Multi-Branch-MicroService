pipeline {
    agent any

environment {
    SCANNER_HOME = tool 'sonarqube-scanner'
} 
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'CheckoutService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }

        stage('Install & Tidy') {
            steps {
                sh 'go mod tidy'
                sh 'go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest'
            }
        }

        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html || true'
            }
        }

        stage('Lint & Format') {
            steps {
                sh 'golangci-lint run ./...'
            }
        }

        stage('Static Analysis') {
            steps {
                sh 'go vet ./...'
            }
        }

        stage('Security Scan (Govulncheck)') {
            steps {
                sh 'go install golang.org/x/vuln/cmd/govulncheck@latest'
                sh 'govulncheck ./...'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'go test -v ./... -cover'
            }
        }

        stage('Trivy fs Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh """${SCANNER_HOME}/bin/sonar-scanner \
                           -Dsonar.projectName=ProductCatalogService \
                           -Dsonar.projectKey=ProductCatalogService"""
                    }
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

        stage('Build Binary') {
            steps {
                sh 'CGO_ENABLED=0 GOOS=linux go build -o app .'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t yukesh24/checkoutservice:${BUILD_NUMBER} ."
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
