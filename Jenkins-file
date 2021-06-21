pipeline {
    agent any

    stages {
        stage('git pull & compile') {
            steps {
                git "https://github.com/Debasish960/my-project-1.git"
                sh "/opt/maven/apache-maven-3.8.1/bin/mvn clean compile"
                sh "/opt/maven/apache-maven-3.8.1/bin/mvn sonar:sonar -Dsonar.projectKey=my-project-1 -Dsonar.host.url=http://65.2.116.192:9000 -Dsonar.login=baaed209d3a9a2828f7748988cca14d0dc55ceac"
            }
        }
        stage('package $ deploy artifact to NEXUS') {
            steps {
                sh "/opt/maven/apache-maven-3.8.1/bin/mvn package"
                sh "/opt/maven/apache-maven-3.8.1/bin/mvn deploy"
            }
        }
        stage('build docker image') {
            steps {
                sh "docker build -t debasish7/my-project-img ."
            }
        }
        stage('docker login & push to DockerHub') {
            steps {
                withCredentials([string(credentialsId: 'DockerHub', variable: 'Docker')]) {
                sh "docker login -u debasish7 -p ${Docker}"
                }
                sh "docker push debasish7/my-project-img"
            }
        }
        stage('run a container over ssh') {
            steps {
                sshagent(['Docker_Server_SSH']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.17.123 docker rm -f project-container || true'
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.17.123 docker run -it --name project-container -p 8080:8080 -d debasish7/my-project-img /bin/bash'
                }
            }
        }
    }
}
