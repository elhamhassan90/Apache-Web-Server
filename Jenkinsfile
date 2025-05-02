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
                withCredentials([
                    string(credentialsId: 'vm3-sudo-password', variable: 'SUDO_PASS'),
                    sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_KEY')
                ]) {
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

    // Send email notification in case of pipeline failure 
    post {
        failure {
            script {
                env.DATE = new Date().format('yyyy-MM-dd')
                emailext(
                    subject: "Pipeline Failed: ${JOB_NAME}",
                    to: "elhamhassan252@gmail.com",
                    from: "webserver@jenkins.com",
                    replyTo: "webserver@jenkins.com",
                    body: """<html>
                                <body> 
                                    <h2>${JOB_NAME} — Build ${BUILD_NUMBER}</h2>
                                    <div style="background-color: white; padding: 5px;"> 
                                        <h3 style="color: black;">Pipeline Status: FAILURE</h3> 
                                    </div> 
                                    <p>Check Pipeline Failed Reason <a href="${BUILD_URL}">console output</a>.</p>
                                    <p>Web Admins: ${MEMS}.</p>
                                    <p>Pipeline Execution Date: ${DATE}.</p>
                                </body> 
                            </html>""",
                    mimeType: 'text/html'
                )
            }
        }
    }
}

