pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = 'chali250/tictactoe-game'
        GIT_REPO_MANIFESTS = 'git@github.com:chali250/tictactoe-k8s-manifests.git'
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
                sh '''
                    GLOBAL_CACHE="/var/lib/jenkins/.cache/trivy"
                    mkdir -p ${GLOBAL_CACHE}/tmp
                    export TMPDIR=${GLOBAL_CACHE}/tmp
                    export TRIVY_TMP_DIR=${GLOBAL_CACHE}/tmp
                    trivy image --cache-dir ${GLOBAL_CACHE} --severity HIGH,CRITICAL ${IMAGE_NAME}:${BUILD_NUMBER}
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
        success {
            echo 'Pipeline exécuté avec succès ! ArgoCD va synchroniser la nouvelle version.'
        }
        failure {
            echo 'Le pipeline a échoué. Vérifiez les logs de l étape en cours.'
        }
    }
}
