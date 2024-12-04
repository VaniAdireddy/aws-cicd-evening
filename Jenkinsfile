def registry = 'https://trialeabhm5.jfrog.io/'
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
        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogaccess"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/springbootApp.jar",
                                  "target": "jfrog-jenkins-mvn-libs-release",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        }
        stage('Build Docker Image and TAG') {
            steps {
                script {
                    // Build the Docker image using the renamed JAR file
                    script {
                            sh 'docker build -t springboot:latest .'
                   }
                }   
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table --scanners vuln -o trivy-image-report.html springboot:latest'
            }
        }
        stage('Archive Report') {
            steps {
                // Archive the Trivy report for later reference
                archiveArtifacts artifacts: 'trivy-image-report.html', fingerprint: true
            }
        }
        stage('Push Docker Image To Docker Hub') {
            steps {
                script {
                    // Build the Docker image using the renamed JAR file
                    script {
                            sh 'docker image tag springboot:latest vasanthaadireddy/springboot:latest'
                            sh 'docker push vasanthaadireddy/springboot:latest'
                   }
                }   
            }
        }
        stage('Push Docker Image To AWS ECR') {
            steps {
                
                    // Build the Docker image using the renamed JAR file
                    script {
                            sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 841162703137.dkr.ecr.us-east-1.amazonaws.com'
                            sh 'docker tag springboot:latest 841162703137.dkr.ecr.us-east-1.amazonaws.com/springboot:latest'
                            sh 'docker push 841162703137.dkr.ecr.us-east-1.amazonaws.com/springboot:latest'
                   }  
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'eks2', contextName: '', credentialsId: 'k8-cred', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://9E499A1CFC8D7B65DEF7D0D86F3B966B.gr7.us-east-1.eks.amazonaws.com') {
                sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    }
}