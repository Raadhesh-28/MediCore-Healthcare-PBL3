
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'raadhesh/healthcare-app'
        K8S_DEPLOYMENT = 'healthcare-deployment'
        K8S_CONTAINER = 'healthcare-container'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building MediCore Docker image'
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ./app
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                echo 'Testing the Docker image'
                sh '''
                    set -e
                    docker run -d --name healthcare-ci-test -p 18082:80 ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    cleanup() {
                        docker rm -f healthcare-ci-test >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:18082/ > /dev/null; then
                            echo "Healthcare application test passed"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Healthcare application test failed"
                    exit 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/${K8S_DEPLOYMENT} \
                        ${K8S_CONTAINER}=${DOCKER_IMAGE}:${BUILD_NUMBER}

                    kubectl rollout status deployment/${K8S_DEPLOYMENT} \
                        --timeout=180s

                    kubectl get pods
                '''
            }
        }
    }

    post {
        success {
            echo 'MediCore CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}

