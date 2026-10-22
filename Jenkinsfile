pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials' // À adapter selon l'ID de vos identifiants sur Jenkins
        IMAGE_NAME = 'chali250/tictactoe-game'
        GIT_REPO_MANIFESTS = 'git@github.com:chali250/tictactoe-k8s-manifests.git' // Votre dépôt GitOps
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

        stage('Security Scan - Trivy') {
            steps {
                // Redirection du dossier temporaire et fix des flags Trivy
                sh '''
                    mkdir -p .trivy-cache/tmp
                    export TMPDIR=$(pwd)/.trivy-cache/tmp
                    export TRIVY_TMP_DIR=$(pwd)/.trivy-cache/tmp
                    trivy image --cache-dir .trivy-cache --severity HIGH,CRITICAL ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
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
                    // Configuration de git pour le push automatique
                    sh '''
                        git config --global user.email "jenkins@pipeline.com"
                        git config --global user.name "Jenkins CI"
                    '''
                    
                    // Cloner, mettre à jour le tag de l'image dans deployment.yaml et pousser vers le dépôt K8s
                    sh """
                        rm -rf tictactoe-k8s-manifests
                        git clone ${GIT_REPO_MANIFESTS}
                        cd tictactoe-k8s-manifests
                        
                        # Remplacement du tag dans le fichier de déploiement Kubernetes
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
        success {
            echo 'Pipeline exécuté avec succès ! ArgoCD va synchroniser les modifications.'
        }
        failure {
            echo 'Le pipeline a échoué. Vérifiez les logs de l étape en cours.'
        }
    }
}
