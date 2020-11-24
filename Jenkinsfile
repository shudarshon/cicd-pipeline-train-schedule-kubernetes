pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "shudarshon/train-schedule"
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
                branch 'masterx'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dhub-uname-pwd') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                  kubeconfigId: 'kube-master-secret',
                  configs: 'train-schedule-kube.yml',
                  enableConfigSubstitution: true
               )
            }
        }
    }
}
