
pipeline {
    agent any

    environment {
        AWS_REGION     = 'us-east-1'
        AWS_ACCOUNT_ID = '405134482584'
        ECR_REPO_NAME  = 'jenkins1'
        EKS_CLUSTER    = 'my-eks-cluster'
        APP_NAME       = 'my-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ptejaswinj/Jenkinsandjava.git'
            }
        }

        stage('Check AWS Access') {
            steps {
                sh '''
                    aws --version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_URI = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${env.BUILD_NUMBER}"
                }

                sh '''
                    docker build -t "$IMAGE_URI" .
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                        --region "$AWS_REGION" |
                        docker login \
                        --username AWS \
                        --password-stdin \
                        "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

                    docker push "$IMAGE_URI"
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                        --region "$AWS_REGION" \
                        --name "$EKS_CLUSTER"

                    sed "s|IMAGE_URI|$IMAGE_URI|g" \
                        deploymentjava.yaml > deployment-rendered.yaml

                    kubectl apply -f deployment-rendered.yaml
                    kubectl apply -f servicelb.yaml

                    kubectl rollout status deployment/"$APP_NAME" \
                        --timeout=180s

                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully to EKS!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins Console Output.'
        }
    }
}
