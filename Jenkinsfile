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
    }
}