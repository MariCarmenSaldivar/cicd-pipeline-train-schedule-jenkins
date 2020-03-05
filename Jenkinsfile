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
                script {
                    app = docker.build("msaldivar/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:3000)'
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
                    docker.withRegistry('','DockerHub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeploytoProduction') {
            when{
                branch 'master'
            }
            steps{
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([sshUserPrivateKey(credentialsId: "stagingserver", keyFileVariable: 'keyfile')]) {
                    script{
                        sh "ssh -o StrictHostKeyChecking=no -i ${keyfile} ubuntu@ec2-3-84-5-6.compute-1.amazonaws.com"
                        sh "ssh -o StrictHostKeyChecking=no -i ${keyfile} ubuntu@ec2-3-84-5-6.compute-1.amazonaws.com \"docker pull msaldivar/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no -i ${keyfile} ubuntu@ec2-3-84-5-6.compute-1.amazonaws.com \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no -i ${keyfile} ubuntu@ec2-3-84-5-6.compute-1.amazonaws.com \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no -i ${keyfile} ubuntu@ec2-3-84-5-6.compute-1.amazonaws.com \"docker run --restart always --name train-schedule -p 8000:3000 -d msaldivar/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
