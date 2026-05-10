pipeline {
    agent any

environment {
    GIT_COMMIT_REV = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    SCANNER_HOME = tool 'sonarqube-scanner'
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
                sh 'cat fs-report.html'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Cartservice -Dsonar.projectKey=Cartservice '''
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
                    sh "docker build -t yukesh24/cartservice:${GIT_COMMIT_REV} ."
                   }    
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html yukesh24/cartservice:${GIT_COMMIT_REV}"
                sh 'cat image-report.html'
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    dir('src') {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                            sh "docker push yukesh24/cartservice:${GIT_COMMIT_REV}"
                         }
                      }
                   }
                }
            }
        }
    }
     post {
        always {
            cleanWs()
        }
        success {
            echo "Build ${BUILD_NUMBER} passed successfully!"
        }
        failure {
            echo "Build ${BUILD_NUMBER} failed. Checking logs..."
        }
    }
}
