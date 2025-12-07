pipeline {

    agent { label 'slave-1' }   // run on the docker-enabled worker node

    environment {
        DOCKERHUB_USER = 'satyajitpawar'
        IMAGE_REPO     = 'docker_jenkins'
        IMAGE_TAG      = 'latest'
        CONTAINER_NAME = 'myhttpd'
        HOST_PORT      = '8080'
        CONTAINER_PORT = '80'
    }

    triggers {
        pollSCM('* * * * *')   // check repo every 1 minute
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Satyajit-Pawar/docker_jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub2_cred',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                    echo "Pushing image to DockerHub..."
                    docker push ${DOCKERHUB_USER}/${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    set -e
                    # Remove existing container if exists
                    if docker ps -aq -f name=${CONTAINER_NAME} | grep -q . ; then
                        echo "Stopping & removing old container..."
                        docker rm -f ${CONTAINER_NAME}
                    fi

                    echo "Starting new container..."
                    docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${DOCKERHUB_USER}/${IMAGE_REPO}:${IMAGE_TAG}

                    echo "Application running at: http://<server-ip>:${HOST_PORT}"
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
