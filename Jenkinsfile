def AGENT_LABEL_MASTER = "Built-In-Node"
def AGENT_LABEL_SLAVE = "Jenkins-slave1"

pipeline {
    agent {
       label "${AGENT_LABEL_SLAVE}"
    }
    tools{
        jdk 'Java8'
    }
    
    environment {
        DOCKER_IMAGE_NAME = "dkalmode27/npmwebsite"
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
         
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        
        stage('Remove old canary deployment') {
            
            agent { 
               label "${AGENT_LABEL_MASTER}" 
            }
            
            when {
                branch 'master'
            }
                        
            steps {
                sh 'kubectl delete deploy train-schedule-deployment-canary'
            }
        }
        
        stage('Remove old canary service') {
            
            agent { 
               label "${AGENT_LABEL_MASTER}" 
            }
            
            when {
                branch 'master'
            }
                        
            steps {
                sh 'kubectl delete svc train-schedule-service-canary'
            }
        }
        
        stage('Create new canary deployment and service') {
            
            agent { 
               label "${AGENT_LABEL_MASTER}" 
            }
            
            when {
                branch 'master'
            }
                        
            steps {
                sh 'kubectl create -f train-schedule-kube-canary.yml'
            }
        }
        
        stage('DeployToProduction') {
            
            agent { 
               label "${AGENT_LABEL_MASTER}" 
            }
            
            when {
                branch 'master'
            }
                        
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh 'kubectl create -f train-schedule-kube.yml'
            }
        }
    }
}
