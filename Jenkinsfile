pipeline {
    agent any

    environment {
        DOCKER_REPO   = "ganesamoorthir"
        IMAGE_NAME    = "capstone-website"
        IMAGE_TAG     = "${BUILD_NUMBER}"
        K8S_NAMESPACE = "default"
        RELEASE_DAY   = "24"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Validate Release Date') {
            steps {
                script {
                    def day = sh(script: "date +%d", returnStdout: true).trim()
                    if (day != env.RELEASE_DAY) {
                        error "❌ Release allowed only on ${RELEASE_DAY}th of the month"
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/GanesamoorthiR/capstone-devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh """
                docker push ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -n ${K8S_NAMESPACE} -f k8s/
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful on ${RELEASE_DAY}th"
        }
        failure {
            echo "❌ Deployment blocked or failed"
        }
        always {
            sh "docker system prune -f || true"
        }
    }
}
