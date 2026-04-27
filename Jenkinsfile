pipeline {
    agent any

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        IMAGE_NAME = "sumador"
        IMAGE_TAG = "${env.BUILD_NUMBER}"

        NEXUS_HOST = "nexus:8082"
        NEXUS_URL = "http://nexus:8082"
        NEXUS_REPO = "mydocker"
        CREDENTIALS_ID = "nexus-credentials"

        FULL_IMAGE_NAME = "${NEXUS_HOST}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Run tests') {
            steps {
                sh 'docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} npm test'
            }
        }

        stage('Dependency Scan - npm audit') {
            steps {
                
                sh 'docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} npm audit --audit-level=critical'

            }
        }

        stage('Image Scan - Trivy') {
            steps {
                sh '''
                docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy image \
                --severity CRITICAL \
                --exit-code 1 \
                ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}'
            }
        }

        stage('Push Image to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USERNAME',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    sh '''
                    echo "$NEXUS_PASSWORD" | docker login $NEXUS_HOST -u "$NEXUS_USERNAME" --password-stdin
                    docker push $FULL_IMAGE_NAME
                    '''
                }
            }
        }
    }

    post {
        always {
            sh '''
            docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
            docker rmi ${FULL_IMAGE_NAME} || true
            '''
        }
    }
}