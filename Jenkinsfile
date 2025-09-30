pipeline {
    agent none

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
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
    }
}