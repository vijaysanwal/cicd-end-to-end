pipeline {

    agent any

    environment {
        AWS_REGION    = 'ap-south-1'
        ECR_REGISTRY  = '892387177992.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = 'test'
        IMAGE_TAG     = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Application') {
            steps {
                git(
                    url: 'https://github.com/vijaysanwal/cicd-end-to-end.git',
                    branch: 'vijay'
                )
            }
        }

        stage('Build Docker') {
            steps {
                sh '''
                    echo "Build Docker Image"

                    docker build \
                      -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        echo "Login to AWS ECR"

                        aws ecr get-login-password \
                          --region ${AWS_REGION} | \
                        docker login \
                          --username AWS \
                          --password-stdin ${ECR_REGISTRY}

                        echo "Push image to ECR"

                        docker push \
                          ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Checkout K8S Manifest') {
            steps {
                git(
                    url: 'https://github.com/vijaysanwal/cicd-demo-manifests-repo.git',
                    branch: 'vijay'
                )
            }
        }

        stage('Update K8S Manifest') {
            steps {
                sh '''
                    echo "Before update:"
                    cat deploy.yaml

                    echo "Updating image tag to ${IMAGE_TAG}"

                    sed -i -E "s|(892387177992\\.dkr\\.ecr\\.ap-south-1\\.amazonaws\\.com/test:)[0-9]+|\\1${IMAGE_TAG}|g" deploy.yaml

                    echo "After update:"
                    cat deploy.yaml

                    git config user.name "Jenkins"
                    git config user.email "jenkins@localhost"

                    git add deploy.yaml

                    git commit \
                      -m "Updated deploy yaml | Jenkins Pipeline" || true

                    echo "Manifest updated successfully"
                '''
            }
        }
    }
}
