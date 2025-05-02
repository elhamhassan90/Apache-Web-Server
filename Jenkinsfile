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

        stage('Run Ansible Playbook') {
            steps {
                echo "Running Ansible Playbook to configure Apache Web Server on ${VM3_IP}..."
                withCredentials([string(credentialsId: 'vm3-sudo-password', variable: 'SUDO_PASS')]) {
                    sh '''
                        cd "Ansible playbook"
                        ansible-playbook site.yml \
                            --extra-vars "ansible_sudo_pass=$SUDO_PASS"
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: "elhamhassan252@gmail.com",
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "The pipeline ran successfully. Check it here: ${BUILD_URL}"
            )
        }

        failure {
            emailext(
                to: "elhamhassan252@gmail.com",
                subject: "❌ FAILURE: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "The pipeline failed. Check the error here: ${BUILD_URL}"
            )
        }
    }
}

