pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'arindam0998/my-django-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Arindam-98/deployment-test-non-prod.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build and tag the Docker image with prefix
                    def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run tests using the Docker image
                    docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").inside {
                        sh 'python manage.py test'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials-id') {
                        def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
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