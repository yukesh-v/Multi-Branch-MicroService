pipeline {
    agent any

 tools {
        
        gradle 'gradle'
        jdk 'jdk17'
    }
    
environment{
    SCANNER_HOME = tool 'sonarqube-scanner'
    BUILD_CONFIG = 'Release'
}

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'CartService', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }
         stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html'
            }
        }
         stage('Dotnet Restore') {
            steps {
                sh 'dotnet restore'
            }
        }
        stage('Dotnet Test') {
            steps {
                sh 'dotnet test --configuration ${env.BUILD_CONFIG} --no-build --logger:trx --results-directory ./test-results'
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
                 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=CartService -Dsonar.projectKey=CartService'''
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
        stage('Dotnet Build') {
            steps {
                sh 'dotnet build --configuration ${env.BUILD_CONFIG} --no-restore'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dir('src') {
                  withDockerRegistry(credentialsId: 'docker-cred') {
                     sh 'docker build -t yukesh24/cartservice:${WORKSPACE} .' 
                  }    
                }
              }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o cartservice-image-report.html yukesh24/cartservice:${WORKSPACE} '
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    dir('src') {
                   withDockerRegistry(credentialsId: 'docker-cred') {
                     sh 'docker push yukesh24/cartservice:${WORKSPACE} '
                   }
                 }
               }
            }
        }
    }
}
