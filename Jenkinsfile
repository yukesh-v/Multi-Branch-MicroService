pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'EmailService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m pip install -r requirements.txt'
                sh 'python3 -m pip install pytest ruff mypy bandit'
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
                       -Dsonar.projectName=Emailservice \
                       -Dsonar.projectKey=Emailservice '''
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
                                sh "docker build -t yukesh24/emailservice:${BUILD_NUMBER} ."
                            }
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
                        dir('src') {
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker push yukesh24/emailservice:${BUILD_NUMBER}"
                            }
                        }
                    }
                }
            }
        }
    }

