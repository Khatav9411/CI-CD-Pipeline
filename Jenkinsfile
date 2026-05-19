pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kamleshcloud/simple-web-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_NAMESPACE = "simple-web-app"
        DEPLOYMENT_NAME = "simple-web-app-deployment"
        CONTAINER_NAME = "simple-web-app"
    }

    stages {
        stage('SonarQube Scan') {
            steps {
                echo 'Running SonarQube Code Analysis'
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

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

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying image to Kubernetes'
                sh '''
                kubectl set image deployment/$DEPLOYMENT_NAME \
                $CONTAINER_NAME=$DOCKER_IMAGE:$IMAGE_TAG \
                -n $KUBE_NAMESPACE

                kubectl rollout status deployment/$DEPLOYMENT_NAME -n $KUBE_NAMESPACE
                '''
            }
        }
    }
}