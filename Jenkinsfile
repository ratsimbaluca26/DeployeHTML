pipeline {
    agent any

    environment {
        DOCKER_USER     = 'ratsimba14'
        IMAGE_NAME      = 'html-app'
        // Votre ID d'identifiants exact dans Jenkins
        CREDENTIALS_ID  = 'd945e418-1ac8-49dc-a539-27174d966816'
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
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER_ID} --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
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