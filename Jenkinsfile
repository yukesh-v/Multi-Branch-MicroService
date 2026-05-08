pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
        // Using a variable for the image name to keep things clean
        IMAGE_NAME = "yukesh24/recommendationservice:${BUILD_NUMBER}"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'RecommendationService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
            ./venv/bin/pip install --upgrade pip
            
            # CRITICAL FIX: Install these first to support building grpcio wheel
            ./venv/bin/pip install "setuptools<82.0.0" wheel
            
            ./venv/bin/pip install --no-cache-dir --no-build-isolation -r requirements.txt
            ./venv/bin/pip install pytest ruff mypy bandit
                '''
            }
        }

        stage('Gitleaks Scan') {
            steps {
                // Ensure gitleaks is installed on your Jenkins agent
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html || true'
            }
        }

        stage('Lint & Security') {
            steps {
                sh '''
                    ./venv/bin/ruff check .
                    ./venv/bin/mypy .
                    ./venv/bin/bandit -r .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh './venv/bin/pytest --junitxml=results.xml'
            }
            post {
                always {
                    junit 'results.xml'
                }
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
                    sh """${SCANNER_HOME}/bin/sonar-scanner \
                       -Dsonar.projectName=RecommendationService \
                       -Dsonar.projectKey=RecommendationService"""
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
                    // Use the environment variable we defined at the top
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html ${IMAGE_NAME}"
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }
    }
}
