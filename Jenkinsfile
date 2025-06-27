pipeline {
    agent any

    environment {
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

        stage('Build & Push All Services') {
            steps {
                script {
                    def services = ['customer', 'gateway', 'products', 'proxy', 'shopping']

                    for (service in services) {
                        def imageName = "ashwanidevops321/${service}:latest"
                        dir(service) {
                            echo "🔧 Building image for ${service}"
                            def image = docker.build(imageName)
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                                echo "📤 Pushing image ${imageName} to DockerHub"
                                image.push()
                            }
                        }
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
                    string(credentialsId: 'deploy-path_1', variable: 'DEPLOY_PATH_1')
                ]) {
                    sshagent(credentials: [SSH_CREDENTIAL_TEST_ID]) {
                        sh """
                            echo "📁 Creating remote deployment directory..."
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} 'mkdir -p ${DEPLOY_PATH_1}'

                            echo "📤 Copying docker-compose.yml to remote server..."
                            scp ${SCRIPT} ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH_1}/docker-compose.yml

                            echo "🚀 Deploying containers on remote server..."
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                                set -e
                                cd ${DEPLOY_PATH_1} &&
                                docker-compose pull &&
                                docker-compose down &&
                                docker-compose up -d --remove-orphans
                            '
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
// This Jenkinsfile is designed to build and deploy a microservices application using Docker and Docker Compose.
// It includes stages for checking out the code, building and pushing Docker images for each service,
// and deploying the application to a remote server using SSH and Docker Compose.