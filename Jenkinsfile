pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kamleshcloud/simple-web-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image'

                sh '''
                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }
    }
}