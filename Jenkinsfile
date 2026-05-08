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
                git branch: 'RecommendationService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install --no-cache-dir -r requirements.txt'
                sh 'pip install pytest ruff mypy bandit'
            }
        }
        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html'
            }
        }

        stage('Lint & Format Check') {
            steps {
                sh 'ruff check .'
            }
        }

        stage('Static Type Check') {
            steps {
                sh 'mypy .'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'bandit -r .'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest --junitxml=results.xml'
            }

            stage('Trivy fs Scan') {
                steps {
                    sh 'trivy fs --format table -o fs-report.html .'
                }
            }
            stage('Sonarqube Analysis') {
                steps {
                    withSonarQubeEnv('sonarqube') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectName=RecommendationService \
                       -Dsonar.projectKey=RecommendationService '''
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
                        dir('src') {
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker build -t yukesh24/recommendationservice:${WORKSPACE} ."
                            }
                        }
                    }
                }
            }
            stage('Trivy Image Scan') {
                steps {
                    sh "trivy image --format table -o recommendationservice-image-report.html yukesh24/recommendationservice:${WORKSPACE}"
                }
            }
            stage('Docker Push') {
                steps {
                    script {
                        dir('src') {
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker push yukesh24/recommendationservice:${WORKSPACE}"
                            }
                        }
                    }
                }
            }
        }
    }
}
