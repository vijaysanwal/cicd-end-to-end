pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '892387177992.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = 'test'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
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

        stage('Push the artifacts') {
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

        stage('Checkout K8S manifest SCM') {
            steps {
                git(
                    url: 'https://github.com/vijaysanwal/cicd-end-to-end.git',
                    branch: 'vijay'
                )
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                            cat deploy.yaml

                            sed -i "s/32/${BUILD_NUMBER}/g" deploy.yaml

                            cat deploy.yaml

                            git add deploy.yaml

                            git commit -m "Updated deploy yaml | Jenkins Pipeline" || true

                            git push \
                              https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/vijaysanwal/cicd-demo-manifests-repo.git \
                              HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
