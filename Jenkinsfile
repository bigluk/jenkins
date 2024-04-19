pipeline{
    
    agent any
    
    tools{
        maven 'maven_3_9_6'
    }
    
    stages{
        
        stage("Build project"){
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/bigluk/jenkins']])
                sh 'mvn clean install'
            }
        }
        
        stage("Build Docker Image"){
            steps{
                sh 'docker build -f src/main/docker/Dockerfile.jvm -t bigluc/simple-jenkins-app .'
            }
        }
        
        stage("Push Image to Docker Hub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerHub-psw', variable: 'dockerHubPsw')]) {
                        sh 'docker login -u bigluc -p ${dockerHubPsw}'
                    }
                    sh 'docker push bigluc/simple-jenkins-app'
                }
            }
        }
        
    }
    
}