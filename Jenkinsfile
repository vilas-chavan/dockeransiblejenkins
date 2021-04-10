pipeline{
    agent any
    
    environment{
        PATH = "/opt/maven3/bin:$PATH"
        DOCKER_TAG = gerVersion()
    }
    
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', url: 'https://github.com/vilas-chavan/dockeransiblejenkins.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh 'mvn clean package'
            }
        }
        
        stage('Docker Build'){
            steps{  
                    
                    sh "docker build . -t vilaschavan80/vkapp:${DOCKER_TAG}"
                    sh "docker build . -t vilaschavan80/vkapp:latest"
            }
        }
        
         stage('DockerHub Push'){
            steps{
                    withCredentials([string(credentialsId: 'docker-hub', variable: 'DockerHubPwd')]) {
                    sh "docker login -u vilaschavan80 -p  ${DockerHubPwd}"
                 }
                
                sh "docker push vilaschavan80/vkapp:${DOCKER_TAG}"
                sh "docker push vilaschavan80/vkapp:latest"
            }
        }
        
        stage('Docker remove images'){
            steps{  
                    
                    sh "docker rmi vilaschavan80/vkapp:${DOCKER_TAG}"
                    sh "docker rmi vilaschavan80/vkapp:latest"
            }
        }
        
         stage('Docker Deploy'){
            steps{
                   ansiblePlaybook become: true, credentialsId: 'docker-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'Ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def gerVersion(){
   def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
   return commitHash
}