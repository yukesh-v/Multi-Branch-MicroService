pipeline {
    agent any

 tools {
        
        gradle 'gradle'
        jdk 'jdk17'
    }
    
environment{
    SCANNER_HOME = tool 'sonarqube-scanner'
}

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'Adservice', url: 'https://github.com/yukesh-v/Multi-Branch-MicroService.git'
            }
        }
         stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --report-format table --report-path gitleaks-report.html'
            }
        }
         stage('Grant Persmissions') {
            steps {
                sh 'chmod +x ./gradlew'
            }
        }
        stage('Gradle Compile') {
            steps {
                sh './gradlew compileJava assemble -x verifyGoogleJavaFormat'
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
                 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Adservice -Dsonar.projectKey=Adservice -Dsonar.java.binaries=.'''
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
        stage('Gradle Build') {
            steps {
                sh './gradlew clean build --info'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                  withDockerRegistry(credentialsId: 'docker-cred') {
                     sh 'docker build -t yukesh24/adservice:${WORKSPACE} .'   
                }
              }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o adservice-image-report.html yukesh24/adservice:${WORKSPACE} '
            }
        }
        stage('Docker Push') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker-cred') {
                     sh 'docker push yukesh24/adservice:${WORKSPACE} '
                 }
               }
            }
        }
    }
}
