pipeline {
    agent any
    tools {
        maven 'mymaven'
    }
    
    stages {
        stage('CleanWs') {
            steps {
                cleanWs()
            }
        }
        stage('Code') {
            steps {
                git 'https://github.com/CharanPolamarasetti/Jenkins-Docker.git'
            }
        }
        stage('Build') {
            steps{
                sh 'mvn clean package'
                sh 'cp -r target Docker-app'
            }
        }
        stage('CQA') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devopsproject"
                }
            }
        }
        stage('QualityGates'){
            steps{
                waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube'
            }
        }
        stage('Artifacts') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexus-token', groupId: 'com.visualpathit', nexusUrl: '3.148.232.154:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'artifactrepo', version: 'v2'
            }
        }
        stage('Build Images') {
            steps {
                sh 'docker build --tag dbimage:1.0 Docker-db'
                sh 'docker build --tag appimage:1.0 Docker-app'
            }
        }
        stage('ImageScan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL -f table -o dbimage-scan.txt dbimage:1.0'
                sh 'trivy image --severity HIGH,CRITICAL -f table -o appimage-scan.txt appimage:1.0'
                archiveArtifacts artifacts: '*.txt'
            }
        }
        stage('Registry-ECR') {
            steps {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 285065163560.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker tag dbimage:1.0 285065163560.dkr.ecr.us-east-2.amazonaws.com/dbrepo:1.0'
                sh 'docker push 285065163560.dkr.ecr.us-east-2.amazonaws.com/dbrepo:1.0'
                sh 'docker tag appimage:1.0 285065163560.dkr.ecr.us-east-2.amazonaws.com/apprepo:1.0'
                sh 'docker push 285065163560.dkr.ecr.us-east-2.amazonaws.com/apprepo:1.0'
            }
        }
    }
}