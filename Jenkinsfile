pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    tools {
        maven 'Maven-3.9.12'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Create ZIP') {
            steps {
                sh 'zip -j devops-project.zip index.html'
            }
        }

        stage('Deploy App 1') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'app1-ubuntu-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        unzip -p devops-project.zip index.html > webserver1.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver1.html \
                            "$SSH_USER@10.0.11.201:/tmp/webserver1.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@10.0.11.201" \
                            'sudo mv /tmp/webserver1.html /var/www/html/webserver1.html && sudo chmod 644 /var/www/html/webserver1.html'

                        rm -f webserver1.html
                    '''
                }
            }
        }

        stage('Deploy App 2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'app2-amazonlinux-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        unzip -p devops-project.zip index.html > webserver2.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver2.html \
                            "$SSH_USER@10.0.12.58:/tmp/webserver2.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@10.0.12.58" \
                            'sudo mv /tmp/webserver2.html /usr/share/nginx/html/webserver2.html && sudo chmod 644 /usr/share/nginx/html/webserver2.html'

                        rm -f webserver2.html
                    '''
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'devops-project.zip',
                             fingerprint: true
        }
    }
}
