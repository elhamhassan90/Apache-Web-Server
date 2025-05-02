pipeline {
    agent any

    environment {
        REMOTE_USER = 'elham'
        REMOTE_HOST = '192.168.142.129' // VM3 - Web Server
        VM1_HOST = '192.168.142.128' // VM1 - Ansible installed on host
        ANSIBLE_DIR = '/home/elham/ansible'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Bash Scripts to VM3') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh '''
                        echo "Copying bash scripts to VM3 ($REMOTE_HOST)..."
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no bash-scripts/*.sh ${REMOTE_USER}@${REMOTE_HOST}:/tmp/
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "chmod +x /tmp/*.sh"
                    '''
                }
            }
        }

        stage('Execute Bash Scripts on VM3') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY'),
                    usernamePassword(credentialsId: 'vm3-sudo-password', usernameVariable: 'SUDO_USER', passwordVariable: 'SUDO_PASS')
                ]) {
                    sh '''
                        echo "Running install_apache.sh on VM3..."
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "echo $SUDO_PASS | sudo -S /tmp/install_apache.sh"

                        echo "Running configure_vhosts.sh on VM3..."
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "echo $SUDO_PASS | sudo -S /tmp/configure_vhosts.sh"
                    '''
                }
            }
        }

        stage('Trigger Ansible from VM1 Host') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh '''
                        echo "Running Ansible playbook from VM1 host ($VM1_HOST)..."
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${VM1_HOST} "cd ${ANSIBLE_DIR} && ansible-playbook -i inventory apache-deploy.yml"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            emailext(
                subject: "Jenkins Job Failed: ${env.JOB_NAME}",
                body: "Build #${env.BUILD_NUMBER} failed. Please check Jenkins logs.",
                to: "elhamhassan252@gmail.com"
            )
        }
    }
}

