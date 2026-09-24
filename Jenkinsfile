pipeline {
    agent any

    environment {
        DOCKER_USER     = 'ratsimba14'
        IMAGE_NAME      = 'html-app'
        CREDENTIALS_ID  = '714e3aa4-04ce-4f28-bbd8-f0b955e811f7'
        K8S_API_SERVER  = 'https://192.168.56.10:6443'
        // Injection sécurisée du token Kubernetes dans l'environnement
        K8S_TOKEN       = credentials('k8s-token')
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('2. Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} -t ${DOCKER_USER}/${IMAGE_NAME}:latest .'
            }
        }

        stage('3. Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER_ID', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER_ID} --password-stdin'
                    sh 'docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${DOCKER_USER}/${IMAGE_NAME}:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
    steps {
        sh '''
            # 1. Remplacement des variables
            sed -i "s/__DOCKERHUB_USER__/ratsimba14/g" k8s/deployment.yaml
            sed -i "s/__BUILD_NUMBER__/${BUILD_NUMBER}/g" k8s/deployment.yaml

            # 2. Application sur K8s avec le volume monté (-v $PWD:/workspace -w /workspace)
            docker run --rm -v $PWD:/workspace -w /workspace bitnami/kubectl:latest \
              --server=https://192.168.56.10:6443 \
              --token=${K8S_TOKEN} \
              --insecure-skip-tls-verify=true \
              apply -f k8s/deployment.yaml

            docker run --rm -v $PWD:/workspace -w /workspace bitnami/kubectl:latest \
              --server=https://192.168.56.10:6443 \
              --token=${K8S_TOKEN} \
              --insecure-skip-tls-verify=true \
              apply -f k8s/service.yaml
        '''
    }
}
    }

    post {
        always {
            sh 'docker image prune -f'
        }
    }
}