pipeline {
    environment {
        IMAGE_NAME = 'website'
        IMAGE_TAG = 'v1'
        DOCKER_PASSWORD = credentials('dockerhub-password')
        DOCKER_USERNAME = 'ulrichsteve'
        HOST_PORT = 8080
        CONTAINER_PORT = 80
        IP_DOCKER = '172.17.0.1'
    }
    agent any
    stages {
        stage ('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }
        stage ('Test') {
            steps {
                script {
                    sh '''
                        docker run --rm -dp ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                        curl -I http://${IP_DOCKER}
                        docker stop ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage ('Publish') {
            steps {
                script {
                    sh '''
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                        docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage ('Deploy Prod') {
            environment {
                SERVER_IP = '44.201.122.166'
                SERVER_USER = 'ubuntu'
            }
            when {
                branch 'main'
            }
            steps {
                script {
                    sshagent(['key']) {
                        sh '''
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker rm -f ${IMAGE_NAME} || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker pull ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} || echo 'Image Download success'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker run -dp ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                            sleep 15
                            curl -I ${SERVER_IP}
                        '''
                    }
                }
            }
        }
        stage ('Deploy Staging') {
            environment {
                SERVER_IP = '3.82.216.17'
                SERVER_USER = 'ubuntu'
            }
            when {
                branch 'staging'
            }
            steps {
                script {
                    sshagent(['key']) {
                        sh '''
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker rm -f ${IMAGE_NAME} || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker pull ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} || echo 'Image Download success'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker run -dp ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                            sleep 15
                            curl -I ${SERVER_IP}
                        '''
                    }
                }
            }
        }
        stage ('Deploy Review') {
            environment {
                SERVER_IP = '35.175.138.157'
                SERVER_USER = 'ubuntu'
            }
            when {
                branch 'review'
            }
            steps {
                script {
                    sshagent(['key']) {
                        sh '''
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker rm -f ${IMAGE_NAME} || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker pull ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} || echo 'Image Download success'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l ${SERVER_USER} ${SERVER_IP} "docker run -dp ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                            sleep 15
                            curl -I ${SERVER_IP}
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}
