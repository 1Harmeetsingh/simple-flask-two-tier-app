pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                git url: "https://github.com/1Harmeetsingh/simple-flask-two-tier-app.git", branch: "main"
            }
        }
        stage("Build & Test"){
            steps{
                sh "sudo -S chown $USER /var/run/docker.sock"
                sh "docker build . -t flaskapp"
            }
        }
        stage("Push to DockerHub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag flaskapp ${env.dockerHubUser}/simple-flask-two-tier-app:latest"
                    sh "docker push ${env.dockerHubUser}/simple-flask-two-tier-app:latest" 
                }
            }
        }
        stage("Deploy"){
            steps{
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
