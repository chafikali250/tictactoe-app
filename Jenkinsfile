pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = 'chali250/tictactoe-game'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    appImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        appImage.push("${BUILD_NUMBER}")
                        appImage.push("latest")
                    }
                }
            }
        }

        stage('Update GitOps Manifests') {
            steps {
                script {
                    sh '''
                        git config --global user.email "jenkins@pipeline.com"
                        git config --global user.name "Jenkins CI"
                    '''
                    
                    sh """
                        rm -rf tictactoe-k8s-manifests
                        git clone ${GIT_REPO_MANIFESTS}
                        cd tictactoe-k8s-manifests
                        
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml
                        
                        git add deployment.yaml
                        git commit -m "CI: Update Tic-Tac-Toe image tag to ${BUILD_NUMBER}"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            // Supprime l'image Docker locale et nettoie le système pour libérer le disque
            sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
            sh "docker system prune -f"
        }
        success {
            echo 'Pipeline exécuté avec succès ! ArgoCD va synchroniser la nouvelle version.'
        }
        failure {
            echo 'Le pipeline a échoué. Vérifiez les logs de l étape en cours.'
        }
    }
}
