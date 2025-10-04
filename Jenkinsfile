@Library('testsharedlibrary') _

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
                    dockerBuild("$IMAGE_NAME", "$IMAGE_TAG")
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    testImage("$CONTAINER_NAME", "$IMAGE_NAME", "$IMAGE_TAG")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    pushImage("$IMAGE_NAME", "$IMAGE_TAG", "$DOCKER_HUB_USER")
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
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa 192.168.99.11 >> ~/.ssh/known_hosts
                    
                    ssh vagrant@192.168.99.11 \
                        -o SendEnv=DOCKER_HUB_USER \
                        -o SendEnv=IMAGE_NAME \
                        -o SendEnv=IMAGE_TAG \
                        -o SendEnv=CONTAINER_NAME \
                        -C "$command1 && $command2 && $command3"
                '''
                }
            }
        }
    }
}