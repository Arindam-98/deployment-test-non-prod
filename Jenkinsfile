pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Clone the repository
                git 'https://github.com/your-repo/your-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    def app = docker.build("your-docker-image-name")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run tests using the Docker image
                    docker.image("your-docker-image-name").inside {
                        sh 'python manage.py test'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub and push the image
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials-id') {
                        def app = docker.build("your-docker-image-name")
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Deploy the app using kubectl
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        always {
            // Clean up workspace and Docker images after the pipeline
            cleanWs()
            script {
                docker.image("your-docker-image-name").inside {
                    sh 'docker system prune -f'
                }
            }
        }
    }
}