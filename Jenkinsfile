pipeline {
    agent {
        label 'executor-1'
    }
    tools {
        maven 'devmaven'
    }
    environment {
        ECR_REPO = '<ecr-repo-path>/service-1'
    }

    stages {
        stage ('cleanWS') {
            steps {
                cleanWs()
            }
        }
        stage ('code') {
            steps {
                git 'https://<source-code-path>'
            }
        }
        stage ('build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage ('CQA') {
            steps {
                withSonarQubeEnv ('sonar') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=devproject'
                }
            }
        }
        stage ('Qualitygates') {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
            }
        }
        stage ('Artifacts & Docker build') {
            steps {
                parallel {
                    stage ('Nexus Artifacts') {
                        steps {
                            nexusArtifactUploader artifacts: <nexus-grrovy-syntax>
                        }
                    }
                    stage ('Docker') {
                        steps {
                            sh 'docker build --tag $ECR_REPO:$BUILD_NUMBER .'
                        }
                    }
                }
            }
        }
        stage ('ImageScan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL --exit-code 1 -f table -o service-1.txt $ECR_REPO:$BUILD_NUMBER'
            }
            post {
                always {
                    archiveArtifacts artifacts: '*.txt', allowEmptyArchive: true
                }
            }
        }
        stage ('registry-push') {
            steps {
                sh '<jenkins-agent-IAM-evaluation>'
                sh 'docker push $ECR_REPO:$BUILD_NUMBER'
            }
        }
    }
}
