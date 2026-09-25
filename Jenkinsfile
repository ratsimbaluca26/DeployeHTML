pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'ratsimba14'
        KUBECONFIG_SERVER = 'https://192.168.56.10:6443'
    }

    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Construction de l'image avec le numéro de build Jenkins ($BUILD_NUMBER)
                    sh "docker build -t ${DOCKERHUB_USER}/html-app:${BUILD_NUMBER} ."
                    
                    // Authentification et push sur Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo \$PASS | docker login -u \$USER --password-stdin"
                        sh "docker push ${DOCKERHUB_USER}/html-app:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Remplacement dynamique des placeholders dans le fichier YAML
                    sh """
                        sed -i 's/__DOCKERHUB_USER__/${DOCKERHUB_USER}/g' k8s/deployment.yaml
                        sed -i 's/__BUILD_NUMBER__/${BUILD_NUMBER}/g' k8s/deployment.yaml
                    """

                    // Application du fichier manifest mis à jour
                    withCredentials([string(credentialsId: 'k8s-token', variable: 'K8S_TOKEN')]) {
                        sh """
                            docker run --rm -i \
                              -v \$(pwd)/k8s:/k8s \
                              bitnami/kubectl:latest \
                              --server=${KUBECONFIG_SERVER} \
                              --token=\${K8S_TOKEN} \
                              --insecure-skip-tls-verify=true \
                              apply -f /k8s/deployment.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Nettoyage des images locales générées
            sh "docker rmi ${DOCKERHUB_USER}/html-app:${BUILD_NUMBER} || true"
        }
    }
}