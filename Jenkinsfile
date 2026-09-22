pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'prajwal11a/devops-maven-cicd:latest'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        SSH_CREDENTIALS = 'docker-k8s-ssh'
        K8S_HOST = '35.154.145.128'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(["${SSH_CREDENTIALS}"]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$K8S_HOST \
                        "kubectl apply -f deployment.yaml && \
                         kubectl rollout restart deployment/devops-maven-cicd && \
                         kubectl rollout status deployment/devops-maven-cicd"
                    '''
                }
            }
        }
    }
}
