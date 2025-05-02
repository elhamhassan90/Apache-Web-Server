pipeline {
    agent any

    environment {
        SSH_KEY = credentials('elham-ssh-key')
        SUDO_PASS = credentials('elham-sudo-password')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Bash Scripts to VM3') {
            steps {
                echo 'Copying Bash Scripts to VM3 (192.168.142.129)...'
                withCredentials([sshUserPrivateKey(credentialsId: 'elham-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no Bash-Scripts/users elham@192.168.142.129:/tmp/
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no Bash-Scripts/groups-and-assign elham@192.168.142.129:/tmp/
                    '''
                }
            }
        }

        stage('Execute Bash Scripts on VM3') {
            steps {
                echo 'Executing Bash Scripts on VM3 (192.168.142.129)...'
                withCredentials([
                    string(credentialsId: 'elham-sudo-password', variable: 'SUDO_PASS'),
                    sshUserPrivateKey(credentialsId: 'elham-ssh-key', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no elham@192.168.142.129 "echo $SUDO_PASS | sudo -S bash /tmp/users"
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no elham@192.168.142.129 "echo $SUDO_PASS | sudo -S bash /tmp/groups-and-assign"
                    '''
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo 'Running Ansible Playbook to configure Apache Web Server on 192.168.142.129...'
                withCredentials([string(credentialsId: 'elham-sudo-password', variable: 'SUDO_PASS')]) {
                    sh '''
                        cd "Ansible playbook"
                        ansible-playbook site.yml --extra-vars ansible_sudo_pass=$SUDO_PASS
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                emailext (
                    to: 'elhamhassan252@gmail.com',
                    subject: "Build result: ${currentBuild.currentResult}",
                    body: """
                        Jenkins Job: ${env.JOB_NAME}
                        Build Number: ${env.BUILD_NUMBER}
                        Result: ${currentBuild.currentResult}
                        Check console output at: ${env.BUILD_URL}
                    """
                )
            }
        }
    }
}

