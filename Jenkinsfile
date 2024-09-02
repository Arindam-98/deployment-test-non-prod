pipeline {
    agent any

    environment {
    DOCKER_IMAGE = 'arindam0998/my-django-app'
    DOCKER_TLS_VERIFY = "1"
    DOCKER_HOST = "tcp://127.0.0.1:64410"
    DOCKER_CERT_PATH = "/home/armma/.minikube/certs"
    MINIKUBE_ACTIVE_DOCKERD = "minikube"
    }

    stages {

        stage('Test Git Commands') {
            steps {
                sh '''
                git --version
                '''
            }
        }
        
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Arindam-98/deployment-test-non-prod.git' , credentialsId: 'b1f704e2-6f8a-4167-85ec-0acbd9770d64'
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
                script {
                    sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
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