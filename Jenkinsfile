pipeline {
    agent any
    tools {
        maven 'maven-3.9.4'
    }
    stages {
        stage('Clone the repo from github') {
            steps {
                git branch: 'master', credentialsId: 'github-credentials', url: 'https://github.com/madinenitejaswini/webapp.git'
            }
        }
        stage('Build the code') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis using Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-9.9.1') {
                sh 'mvn sonar:sonar'
                }
            }            
        }
        stage('Build Docker Image and tag it to aws ecr') {
            steps {
                sh '''
                docker build . -t webapp:$BUILD_NUMBER
                docker tag webapp:$BUILD_NUMBER 361661913055.dkr.ecr.ap-south-1.amazonaws.com/webapp:$BUILD_NUMBER
                '''  
            }
        }
	stage('Push Docker Image to AWS ECR') {
            steps {
                 // configure AWS credentials
                withAWS(credentials: 'aws-credentials', region: 'ap-south-1') {
                    sh '''
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 361661913055.dkr.ecr.ap-south-1.amazonaws.com
                    docker push 361661913055.dkr.ecr.ap-south-1.amazonaws.com/webapp:$BUILD_NUMBER
                    '''
                }
            }
        }
        stage('Deployto AWS EKS') {
            steps {
               withAWS(credentials: 'aws-credentials', region: 'ap-south-1') {

                   // Connect to the EKS cluster
                   // Apply yaml file to eks cluster
                    sh '''
                     aws eks update-kubeconfig --name dev-cluster --region ap-south-1 
                     kubectl apply -f .
                     kubectl set image deployment/webapp webapp=361661913055.dkr.ecr.ap-south-1.amazonaws.com/webapp:$BUILD_NUMBER
                    '''
                }
            }
        }
    }
}
