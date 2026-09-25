pipeline {
    agent any

    environment {
        DOCKER_USER     = 'ratsimba14'
        IMAGE_NAME      = 'html-app'
        CREDENTIALS_ID  = '714e3aa4-04ce-4f28-bbd8-f0b955e811f7'
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
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} -t ${DOCKER_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('3. Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER_ID', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER_ID --password-stdin'
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('4. Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'k8s-token', variable: 'K8S_TOKEN')]) {
                    sh """
                        # Combinaison propre du deployment et du service séparés par '---'
                        {
                          sed "s/__DOCKERHUB_USER__/${DOCKER_USER}/g; s/__BUILD_NUMBER__/${BUILD_NUMBER}/g" k8s/deployment.yaml
                          printf "\n---\n"
                          cat k8s/service.yaml
                        } | docker run --rm -i bitnami/kubectl:latest \
                          --server=${K8S_API_SERVER} \
                          --token=\$K8S_TOKEN \
                          --insecure-skip-tls-verify=true \
                          --request-timeout=120s \
                          apply -f -
                    """
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