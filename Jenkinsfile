pipeline {
    agent any

    environment {
        IMAGE_NAME = "kmaharwal1/cicd-e2e"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout App Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/KonpalM/cicd-end-to-end'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "Building Docker Image..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "Pushing Docker Image..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Checkout K8S Manifest Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/KonpalM/cicd-demo-manifests-repo.git'
            }
        }

        stage('Update K8S manifest & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'fbe1c471-7df9-4cc6-af09-ec0586e6dcd6',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )]) {
                    sh '''
                    echo "Updating deploy.yaml with new image tag..."
                    sed -i "s|image: .*$|image: '$IMAGE_NAME:$IMAGE_TAG'|g" deploy.yaml

                    echo "Committing changes..."
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins CI"
                    git add deploy.yaml
                    git commit -m "Update deploy.yaml to image tag $IMAGE_TAG"

                    echo "Pushing changes..."
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/KonpalM/cicd-demo-manifests-repo.git HEAD:main
                    '''
                }
            }
        }
    }
}
