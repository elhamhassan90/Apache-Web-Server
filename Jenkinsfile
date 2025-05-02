pipeline {
    agent any

    environment {
        VM3_IP = '192.168.142.129'
        REMOTE_USER = 'elham'
        REMOTE_PATH = '/tmp'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Bash Scripts to VM3') {
            steps {
                echo "Copying Bash Scripts to VM3 (${VM3_IP})..."
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no Bash-Scripts/users $REMOTE_USER@$VM3_IP:$REMOTE_PATH/
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no Bash-Scripts/groups-and-assign $REMOTE_USER@$VM3_IP:$REMOTE_PATH/
                    '''
                }
            }
        }

        stage('Execute Bash Scripts on VM3') {
            steps {
                echo "Executing Bash Scripts on VM3 (${VM3_IP})..."
                withCredentials([string(credentialsId: 'vm3-sudo-password', variable: 'SUDO_PASS'),
                                 sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no $REMOTE_USER@$VM3_IP "echo $SUDO_PASS | sudo -S bash /tmp/users"
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no $REMOTE_USER@$VM3_IP "echo $SUDO_PASS | sudo -S bash /tmp/groups-and-assign"
                    '''
                }
            }
        }
    }

    post {
        failure {
            mail to: 'elhamhassan252@gmail.com',
                 subject: "Pipeline Failed: ${env.JOB_NAME}",
                 body: "Something went wrong with job ${env.BUILD_URL}"
        }
    }
}

