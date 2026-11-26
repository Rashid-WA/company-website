pipeline {

    agent {
        label 'build'
    }

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKERHUB_USERNAME = 'dockerxhib'
        DOCKERHUB_REPOSITORY = 'dockerxhib/company-website'
        DOCKERHUB_CREDENTIAL_ID = 'cred'

        DEPLOYMENT_SERVER_IP = '172.31.4.166'
        DEPLOYMENT_SSH_CREDENTIAL_ID = 'ssh-key-cred'

        GITHUB_REPOSITORY_URL = 'https://github.com'

        DEPLOY_USER = 'ubuntu'
        CONTAINER_NAME = 'company-website'

        IMAGE_NAME = 'dockerxhib/company-website'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPOSITORY_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t "$IMAGE_NAME:$IMAGE_TAG" .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKERHUB_CREDENTIAL_ID,
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        export DOCKER_CONFIG=$(mktemp -d)
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        CLEAN_USER=$(printf '%s' "$DH_USER" | tr -d '[:space:]')
                        CLEAN_TOKEN=$(printf '%s' "$DH_TOKEN" | tr -d '[:space:]')

                        printf '%s' "$CLEAN_TOKEN" | docker login -u "$CLEAN_USER" --password-stdin
                        docker push "$IMAGE_NAME:$IMAGE_TAG"
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sshagent(
                    credentials: [env.DEPLOYMENT_SSH_CREDENTIAL_ID]
                ) {
                    sh '''
                        mkdir -p ~/.ssh
                        ssh-keyscan -H "$DEPLOYMENT_SERVER_IP" >> ~/.ssh/known_hosts 2>/dev/null

                        ssh -o StrictHostKeyChecking=yes \
                            "$DEPLOY_USER@$DEPLOYMENT_SERVER_IP" \
                            bash -s -- \
                            "$IMAGE_NAME:$IMAGE_TAG" \
                            "$CONTAINER_NAME" <<'REMOTE'

                            set -e

                            IMAGE="$1"
                            CONTAINER="$2"

                            echo "Pulling Docker image..."
                            docker pull "$IMAGE"

                            echo "Removing previous container..."
                            if docker container inspect "$CONTAINER" >/dev/null 2>&1; then
                                docker stop "$CONTAINER"
                                docker rm "$CONTAINER"
                            fi

                            echo "Starting new container..."
                            docker run -d \
                                --name "$CONTAINER" \
                                --restart unless-stopped \
                                -p 9090:80 \
                                "$IMAGE"

                            docker ps

REMOTE
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(
                    credentials: [env.DEPLOYMENT_SSH_CREDENTIAL_ID]
                ) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes \
                            "$DEPLOY_USER@$DEPLOYMENT_SERVER_IP" \
                            'curl --fail --silent --show-error \
                            --retry 5 --retry-connrefused \
                            --retry-delay 2 \
                            http://localhost:9090/ > /dev/null'

                        echo "Application deployed successfully!"
                        echo "Website: http://$DEPLOYMENT_SERVER_IP:9090"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}
