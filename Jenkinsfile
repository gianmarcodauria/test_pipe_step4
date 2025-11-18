pipeline {
    agent { label 'docker-agent-01' }

    environment {
        DOCKER_HUB_USER = 'admin'
        IMAGE_NAME = "flask-app-example"
        DOCKERHUB_CRED_ID = 'dockerhub-creds'
        KUBECONFIG_CRED_ID = 'k8s-kubeconfig'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = "dev-${env.BUILD_NUMBER}"
                    env.FULL_IMAGE_NAME = "${env.DOCKER_HUB_USER}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                    echo "Full image: ${env.FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.FULL_IMAGE_NAME} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CRED_ID,
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    sh "docker push ${env.FULL_IMAGE_NAME}"
                    sh "docker logout"
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withKubeConfig([credentialsId: env.KUBECONFIG_CRED_ID]) {
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

        stage('Cleanup') {
            steps {
                sh "do

