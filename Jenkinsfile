pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'arindam0998/my-django-app'
        DOCKER_HOST = "unix:///var/run/docker.sock"
        KUBECONFIG = "/var/jenkins_home/.kube/config" // Point to kubeconfig in Jenkins
        KUBE_NAMESPACE = 'default' // Set your desired namespace
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
                    // Build Docker image using Dockerfile from repo
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Push image to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        def app = docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        app.push("${env.BUILD_NUMBER}") // Push with build number
                        app.push("latest") // Tag latest version
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes manifests
                    sh '''
                    kubectl apply -f ${WORKSPACE}/deployment.yaml --kubeconfig ${KUBECONFIG} -n ${KUBE_NAMESPACE}
                    kubectl apply -f ${WORKSPACE}/service.yaml --kubeconfig ${KUBECONFIG} -n ${KUBE_NAMESPACE}
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify pods and services
                    sh '''
                    kubectl get pods -n ${KUBE_NAMESPACE} --kubeconfig ${KUBECONFIG}
                    kubectl get services -n ${KUBE_NAMESPACE} --kubeconfig ${KUBECONFIG}
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up workspace
            script {
                docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").inside {
                    // Clean up Docker resources
                    sh 'docker system prune -f'
                }
            }
        }
    }
}