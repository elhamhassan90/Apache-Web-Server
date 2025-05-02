pipeline {
    agent any

    environment {
        REMOTE_HOST = "192.168.142.129"
        REMOTE_USER = "elham"
        REMOTE_PATH = "/home/elham/Desktop"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Bash Scripts') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_PRIVATE_KEY'),
                    usernamePassword(credentialsId: 'sudo-password', passwordVariable: 'SUDO_PASS', usernameVariable: 'SUDO_USER')
                ]) {
                    sh '''
                        scp -i "$SSH_PRIVATE_KEY" -r ./Bash-Scripts/ ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}/
                        ssh -i "$SSH_PRIVATE_KEY" ${REMOTE_USER}@${REMOTE_HOST} "echo $SUDO_PASS | sudo -S chmod +x ${REMOTE_PATH}/Bash-Scripts/*"
                    '''
                }
            }
        }

        stage('Execute Bash Scripts') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_PRIVATE_KEY'),
                    usernamePassword(credentialsId: 'sudo-password', passwordVariable: 'SUDO_PASS', usernameVariable: 'SUDO_USER')
                ]) {
                    sh '''
                        ssh -i "$SSH_PRIVATE_KEY" ${REMOTE_USER}@${REMOTE_HOST} "echo $SUDO_PASS | sudo -S bash ${REMOTE_PATH}/Bash-Scripts/groups-and-assign"
                        ssh -i "$SSH_PRIVATE_KEY" ${REMOTE_USER}@${REMOTE_HOST} "echo $SUDO_PASS | sudo -S bash ${REMOTE_PATH}/Bash-Scripts/users"
                    '''
                }
            }
        }

        stage('Deploy Apache Web Server - Ansible playbook') {
            steps {
                echo 'Running Ansible playbook here...'
                // Place your Ansible playbook logic here
            }
        }
    }

    post {
        failure {
            emailext to: 'elhamhassan252@gmail.com',
                    subject: "Jenkins Build Failed: ${env.JOB_NAME}",
                    body: "The build has failed. Please check Jenkins console output for details."
        }
    }
}

