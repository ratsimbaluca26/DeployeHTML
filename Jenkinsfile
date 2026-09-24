pipeline {
    agent any

    environment {
        // Remplacez par votre identifiant Docker Hub
        DOCKER_USER     = 'ratsimba4'
        IMAGE_NAME      = 'html-app'
        REGISTRY        = 'docker.io'
        // Remplacez par l'IP de votre VM Master Kubernetes
        K8S_API_SERVER  = 'https://192.168.56.10:6443'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('2. Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                    dockerLatest = docker.build("${DOCKER_USER}/${IMAGE_NAME}:latest")
                }
            }
        }

        stage('3. Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'docker-hub-credentials') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerLatest.push("latest")
                    }
                }
            }
        }

        stage('4. Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'k8s-token', variable: 'K8S_TOKEN')]) {
                    sh '''
                        # Injection des variables dans le manifeste Deployment
                        sed -i "s/__DOCKERHUB_USER__/${DOCKER_USER}/g" k8s/deployment.yaml
                        sed -i "s/__BUILD_NUMBER__/${BUILD_NUMBER}/g" k8s/deployment.yaml

                        # Application du Deployment et du Service sur le Master K8s
                        kubectl --server=${K8S_API_SERVER} \
                                --token=${K8S_TOKEN} \
                                --insecure-skip-tls-verify=true \
                                apply -f k8s/deployment.yaml

                        kubectl --server=${K8S_API_SERVER} \
                                --token=${K8S_TOKEN} \
                                --insecure-skip-tls-verify=true \
                                apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
    }
}