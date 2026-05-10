pipeline {
    agent any

    environment {
        
       GIT_COMMIT_REV = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
       SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'RecommendationService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }

        stage('Gitleaks Scan') {
            steps {
                // Ensure gitleaks is installed on your Jenkins agent
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html || true'
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
                 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Recommendationservice -Dsonar.projectKey=Recommendationservice '''
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
                    sh "docker build -t yukesh24/recommendationservice:${GIT_COMMIT_REV} ."
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html yukesh24/recommendationservice:${GIT_COMMIT_REV}"
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push yukesh24/recommendationservice:${GIT_COMMIT_REV}"
                    }
                }
            }
        }
    }
}
