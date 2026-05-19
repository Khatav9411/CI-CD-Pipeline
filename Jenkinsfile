pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kamleshcloud/simple-web-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-token',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                    sed -i "s|image: .*|image: $DOCKER_IMAGE:$IMAGE_TAG|g" k8s/deployment.yaml

                    git config user.name "jenkins"
                    git config user.email "jenkins@example.com"

                    git add k8s/deployment.yaml
                    git commit -m "Update image tag to $IMAGE_TAG" || echo "No changes to commit"

                    git push https://$GIT_USER:$GIT_TOKEN@github.com/Khatav9411/CI-CD-Pipeline.git HEAD:main
                    '''
                }
            }
        }
    }
}