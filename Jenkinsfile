pipeline {
    agent any

    environment {
        DOCKER_HUB_USER      = 'chali250'
        IMAGE_NAME           = 'tictactoe-game'
        BUILD_TAG            = "${BUILD_NUMBER}"
        GITHUB_MANIFEST_REPO = 'github.com/chafikali250/tictactoe-k8s-manifests.git'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                echo '📥 Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('2. Build & Test') {
            steps {
                echo '🧪 Exécution des tests...'
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('3. Build & Push Docker Image') {
            steps {
                echo '🐳 Construction et publication de l\'image Docker...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_TAG} ."
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ."
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('4. Security Scan with Trivy') {
            steps {
                echo '🛡️ Scan de sécurité avec Trivy...'
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_TAG}"
            }
        }

        stage('5. Update Manifest Repo') {
            steps {
                echo '📝 Mise à jour des manifestes GitOps...'
                withCredentials([gitUsernamePassword(credentialsId: 'github-credentials', gitToolName: 'git-tool')]) {
                    sh """
                        git clone https://${GITHUB_MANIFEST_REPO} manifest-repo
                        cd manifest-repo
                        sed -i 's|image: .*|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_TAG}|g' deployment.yaml
                        git config user.email "jenkins@pipeline.com"
                        git config user.name "Jenkins CI"
                        git commit -am "Update image tag to build ${BUILD_TAG}"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage du workspace...'
            deleteDir()
        }
    }
}
