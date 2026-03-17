pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = "dockerhub-credentials"
        DOCKERHUB_USERNAME       = "wimetirix"
        IMAGE_NAME               = "wimetirix/wimetrix-app"
        IMAGE_TAG                = "latest"
        GITHUB_REPO              = "https://github.com/ak-idrees/wimetrix-app.git"
        BRANCH                   = "master"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Pulling code from GitHub..."
                git branch: "${env.BRANCH}",
                    url: "${env.GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh """
                    docker build \
                      -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} \
                      -t ${env.IMAGE_NAME}:${env.BUILD_NUMBER} \
                      .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo "🔐 Logging in to Docker Hub..."
                withCredentials([
                    usernamePassword(
                        credentialsId: "${env.DOCKERHUB_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo $DOCKER_PASS | docker login \
                          -u $DOCKER_USER \
                          --password-stdin
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "🚀 Pushing image to Docker Hub..."
                sh """
                    # Push with latest tag
                    docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}

                    # Push with build number tag (good for versioning)
                    docker push ${env.IMAGE_NAME}:${env.BUILD_NUMBER}
                """
            }
        }

        stage('Cleanup') {
            steps {
                echo "🧹 Cleaning up local images..."
                sh """
                    docker rmi ${env.IMAGE_NAME}:${env.IMAGE_TAG} || true
                    docker rmi ${env.IMAGE_NAME}:${env.BUILD_NUMBER} || true
                """
            }
        }

    }

    post {
        success {
            echo "✅ SUCCESS! Image pushed → ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
            echo "✅ Also pushed → ${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
        }
        failure {
            echo "❌ FAILED! Check logs above for errors"
        }
        always {
            echo "🏁 Pipeline finished — Build #${env.BUILD_NUMBER}"
        }
    }
}

