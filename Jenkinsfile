pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'arindam0998/my-django-app'
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }

    stages {
        stage('Test Git Commands') {
            steps {
                sh 'git --version'
            }
        }

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Arindam-98/deployment-test-non-prod.git'
            }
        }

        stage('Debug Docker CLI') {
            steps {
                script {
                    sh '''
                    docker info
                    ls -l /workspace/certs
                    cat /workspace/certs/ca.pem
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        def app = docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                       kubectl apply -f /var/jenkins_home/workspace/deployment.yaml
                       kubectl apply -f /var/jenkins_home/workspace/service.yaml
                       '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            script {
                docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").inside {
                    sh 'docker system prune -f'
                }
            }
        }
    }
}