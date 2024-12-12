pipeline {
    agent any
    environment {
        DOCKERHUB_CREDS = credentials('docker')
    }
    stages {
        stage('Docker Image Build') {
            steps {
                echo 'Building Docker Image...'
                sh 'docker build --tag xtracoolbreeze/cw2-server:1.0 .'
                echo 'Docker Image Built Successfully!'
            }
        }

        stage('Test Docker Image') {
            steps {
                echo 'Testing Docker Image...'
                sh '''
                    docker stop test-container || true
                    docker rm test-container || true

                    docker image inspect xtracoolbreeze/cw2-server:1.0
                    docker run --name test-container -p 8081:8080 xtracoolbreeze/cw2-server:1.0
                    docker ps

                    docker stop test-container
                    docker rm test-container
                '''
            }
        }

        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('DockerHub Image Push') {
            steps {
                sh 'docker push xtracoolbreeze/cw2-server:1.0'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['jenkins-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@54.152.42.249 << EOF
                    kubectl apply -f deployment.yaml
                    kubectl rollout status deployment/cw2-server
                    EOF
                    '''
                }
            }
        }
    }
}
