pipeline {
    agent any

    environment {
        DOCKER_IMAGE_ID = 'microservice-image'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        SSH_CREDENTIAL_TEST_ID = 'ssh-key-id-in-jenkins'
        SCRIPT = 'docker-compose.yml'

    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE_ID}")
                }
            }
        }

         stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_IMAGE_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Remote Server') {
            when {
                expression { fileExists(SCRIPT) }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'deploy-user', variable: 'DEPLOY_USER'),
                    string(credentialsId: 'deploy-host', variable: 'DEPLOY_HOST'),
                    string(credentialsId: 'deploy-path_1', variable: 'DEPLOY_PATH_1'),
                ]) {
                    sshagent(credentials: [SSH_CREDENTIAL_TEST_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} 'mkdir -p ${DEPLOY_PATH_1}'
                            scp ${SCRIPT} ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH_1}/docker-compose.yml

                            ssh ${DEPLOY_USER}@${DEPLOY_HOST} << EOF
                                cd ${DEPLOY_PATH_1}
                                docker-compose pull
                                docker-compose down
                                docker-compose up -d --remove-orphans
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}