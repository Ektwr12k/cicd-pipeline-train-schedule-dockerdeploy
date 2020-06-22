pipeline {
    agent any
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
                echo 'Starting build docker image'
                script {
                    application = docker.build("ektwr12k/train-schedule") 
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Pushing docker image'
                script {
                    docker.withRegistry('https://hub.docker.com/r/ektwr12k/linuxacademy', 'DockerHub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production Server') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsID: 'production_server', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$PRODUCTION_SERVER \"docker pull ektwr12k/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$PRODUCTION_SERVER \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$PRODUCTION_SERVER \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$PRODUCTION_SERVER  \"docker run --restart always --name train-schedule -p 8080:8080 -d ektwr12k/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
