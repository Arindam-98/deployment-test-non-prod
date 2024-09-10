pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'arindam0998/my-django-app'
        DOCKER_HOST = "unix:///var/run/docker.sock"
        KUBECONFIG = "/var/jenkins_home/.kube/config" // Point to kubeconfig in Jenkins
    }

    stages {
        stage('Test Git Commands') {
            steps {
                sh 'git --version'
            }
        }

        stage('Checkout') {
            steps {
                // Checkout from GitHub
                git url: 'https://github.com/Arindam-98/deployment-test-non-prod.git', credentialsId: 'b1f704e2-6f8a-4167-85ec-0acbd9770d64'
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
                    // Apply Kubernetes manifests
                    sh '''
                    kubectl apply -f /var/jenkins_home/workspace/deployment.yaml --kubeconfig /var/jenkins_home/.kube/config
                    kubectl apply -f /var/jenkins_home/workspace/service.yaml --kubeconfig /var/jenkins_home/.kube/config
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