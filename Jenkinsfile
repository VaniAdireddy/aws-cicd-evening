pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TRIVY_CACHE_DIR = '/var/tmp/trivy'
        }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'prod' , url: 'https://github.com/VaniAdireddy/aws-cicd-evening.git'
            }
        }
        stage('Versioning') {
            steps {
                script {
                sh 'mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}'
                }
            }
        }
        stage('Maven Compile') {
            steps {
               echo 'Maven Compile Started'
               sh 'mvn compile'
            }
        } 
         stage('Maven Test') {
            steps {
               echo 'Maven Test Started'
               sh 'mvn test'
            }
        } 
        stage('File System Scan by Trivy') {
            steps {
               echo 'Trivy Scan Started'
               sh 'trivy fs --format table --output trivy-fs-output.txt .'
            }
        } 
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=SpringBootApp -Dsonar.projectKey=SpringBootApp \
                                                       -Dsonar.java.binaries=. -Dsonar.exclusions=**/trivy-fs-output.txt '''
               }
            }
        } 
        stage('Quality Gate') {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true, credentialsId: 'sonar'  
              }
            } 
        }
        stage('Maven Package') {
            steps {
               echo 'Maven package Started'
               sh 'mvn package'
          }
        } 
    }
}