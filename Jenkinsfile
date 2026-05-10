pipeline {
    agent any

    tools {
        nodejs 'nodejs23'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
        GIT_COMMIT_REV = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'LoadGeneratorService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
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
            stage('Sonarqube Analysis') {
                steps {
                    withSonarQubeEnv('sonarqube') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectName=LoadGeneratorService \
                       -Dsonar.projectKey=LoadGeneratorService '''
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
                                sh "docker build -t yukesh24/loadgeneratorservice:${GIT_COMMIT_REV} ."
                            }
                        }
                    }
                }
            stage('Trivy Image Scan') {
                steps {
                    sh "trivy image --format table -o loadgeneratorservice-image-report.html yukesh24/loadgeneratorservice:${GIT_COMMIT_REV}"
                }
            }
            stage('Docker Push') {
                steps {
                    script {
                            withDockerRegistry(credentialsId: 'docker-cred') {
                                sh "docker push yukesh24/loadgeneratorservice:${GIT_COMMIT_REV}"
                        }
                    }
                }
            }
        }
    }
