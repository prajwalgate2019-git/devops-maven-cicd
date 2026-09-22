pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'prajwal11a/devops-maven-cicd:latest'
        K8S_HOST = '35.154.145.128'
        K8S_USER = 'ubuntu'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Project to Docker/K8s Server') {
            steps {
                sshagent(['docker-k8s-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} \
                        "rm -rf /home/ubuntu/devops-maven-cicd && mkdir -p /home/ubuntu/devops-maven-cicd"

                        scp -o StrictHostKeyChecking=no -r \
                        Dockerfile index.html deployment.yaml \
                        ${K8S_USER}@${K8S_HOST}:/home/ubuntu/devops-maven-cicd/
                    '''
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                sshagent(['docker-k8s-ssh']) {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_TOKEN'
                        )
                    ]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "
                                cd /home/ubuntu/devops-maven-cicd &&
                                docker build -t ${DOCKER_IMAGE} . &&
                                echo '${DOCKER_TOKEN}' | docker login -u '${DOCKER_USER}' --password-stdin &&
                                docker push ${DOCKER_IMAGE} &&
                                docker logout
                            "
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(['docker-k8s-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${K8S_USER}@${K8S_HOST} "
                            cd /home/ubuntu/devops-maven-cicd &&
                            kubectl apply -f deployment.yaml &&
                            kubectl rollout restart deployment/devops-maven-cicd &&
                            kubectl rollout status deployment/devops-maven-cicd
                        "
                    '''
                }
            }
        }
    }
}
