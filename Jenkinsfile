pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
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
    }
}