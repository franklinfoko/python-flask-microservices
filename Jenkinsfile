pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'frontend', description: 'This is the name of my docker image for frontend application')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'This is the tag of my docker image for frontend application')
        string(name: 'GITHUB_USER', defaultValue: 'franklinfoko', description: 'The username of github')
        string(name: 'CONTAINER_NAME', defaultValue: 'frontend-container', description: 'This is the name of the container')
        string(name: 'DOCKER_HUB_USER', defaultValue: 'franklinfoko', description: 'The user of the docker hub')
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} frontend/'
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    sh '''
                        echo "Launch test container"
                        docker run -d -p 5000:5000 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                        echo "Test container"
                        curl -I http://$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME}):5000
                        echo "Delete test container..."
                        docker rm -f ${CONTAINER_NAME}
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials', 
                        passwordVariable: 'DOCKERHUB_PASSWORD', 
                        usernameVariable: 'DOCKERHUB_USER'
                    )]) {
                        sh '''
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                            docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASSWORD
                            docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            environment {
                DOCKER_HUB_USER = "franklinfoko"
                IMAGE_NAME = "frontend"
                IMAGE_TAG = "latest"
                CONTAINER_NAME = "frontend-container"
            }
            steps {
                sshagent(credentials: ['ssh-cred']) {
                sh '''
                    command1="docker pull $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    command2="docker rm -f $CONTAINER_NAME || echo 'app not found'"
                    command3="docker run -d -p 5000:5000 --name $CONTAINER_NAME $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                    [ -d /home/vagrant/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    
                    ssh vagrant@192.168.99.11 \
                        -o SendEnv=DOCKER_HUB_USER \
                        -o SendEnv=IMAGE_NAME \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_NAME \
                        -C "$command1 && $command2 && command3"
                '''
                }
            }
        }
    }
}