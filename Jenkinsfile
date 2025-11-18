pipeline {
    agent {
        label 'docker-agent-01' // deve essere un nodo con Docker nativo
    }

    environment {
        DOCKER_HUB_USER = 'admin'
        IMAGE_NAME = "flask-app-example"
        DOCKERHUB_CRED_ID = 'dockerhub-creds'
        KUBECONFIG_CRED_ID = 'k8s-kubeconfig' // Assicurati che l'ID esista in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Cloning repository on Docker agent..."
                checkout scm
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = "dev-${env.BUILD_NUMBER}"
                    env.FULL_IMAGE_NAME = "${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                    
                    echo "Docker image tag set to: ${env.IMAGE_TAG}"
                    echo "Full image name set to: ${env.FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building image ${env.FULL_IMAGE_NAME} with Docker..."
                sh "docker build -t ${env.FULL_IMAGE_NAME} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.DOCKERHUB_CRED_ID,
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        echo "Logging into Docker Hub..."
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"

                        echo "Pushing image..."
                        sh "docker push ${env.FULL_IMAGE_NAME}"

                        sh "docker logout || true"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    withKubeConfig([credentialsId: env.KUBECONFIG_CRED_ID]) {
                        echo "Deploying to Kubernetes with Helm..."
                        sh """
                            helm upgrade --install flask-app ./charts/flask-app-chart \
                                --namespace default \
                                --set image.repository=${env.DOCKER_HUB_USER}/${env.IMAGE_NAME} \
                                --set image.tag=${env.IMAGE_TAG} \
                                --wait \
                                --insecure-skip-tls-verify
                        """
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning up local image: ${env.FULL_IMAGE_NAME}"
                sh "docker rmi ${env.FULL_IMAGE_NAME} || true"
            }
        }
    }
}

