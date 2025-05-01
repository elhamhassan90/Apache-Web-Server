pipeline {
    agent any
    stages {
        stage('Bash Script Execution') {
            steps {
                script {
                    // Initialize the variable to avoid the error of null if the script didn't execute successfully
                    env.MEMS = "Undefined - Destination unreachable"
                    
                    // Use the withCredentials block to temporarily add the SSH private key to the environment
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_PRIVATE_KEY'),
                                     string(credentialsId: 'vm3-sudo-password', variable: 'SUDO_PASS')]) {
                        // Run bash-scripts
                        sh """
                            scp -i ${SSH_PRIVATE_KEY} -r ./Bash\ Scripts/ elham@192.168.142.129:/home/elham/Desktop/
                            ssh -i ${SSH_PRIVATE_KEY} elham@192.168.142.129 "echo ${SUDO_PASS} | sudo -S chmod a+x /home/elham/Desktop/Bash\ Scripts/*"
                        """
                        // Get the list of group members
                        env.MEMS = sh(script: "ssh -i ${SSH_PRIVATE_KEY} elham@192.168.142.129 'echo ${SUDO_PASS} | sudo -S /home/elham/Desktop/Bash\ Scripts/groups-and-assign.sh'", returnStdout: true).trim()
                        echo "Members: ${MEMS}"
                    }
                }
            }
        }

        stage('Deploy Apache Web Server - Ansible playbook') {
            steps {
                // Use the withCredentials block to temporarily add the SSH private key to the environment
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins_vm3_ssh_key', keyFileVariable: 'SSH_PRIVATE_KEY'),
                                 string(credentialsId: 'vm3-sudo-password', variable: 'SUDO_PASS')]) {
                    // Run the ansible-playbook command
                    sh "ansible-playbook site.yml --private-key=${SSH_PRIVATE_KEY} --extra-vars 'ansible_sudo_pass=${SUDO_PASS}' -i 192.168.142.129,"
                }
            }
        }
    }
    
    post {
        failure {
            script {
                // Send an email if the pipeline fails
                env.DATE = new Date().format('yyyy-MM-dd')
                emailext (
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
                                    <p> Check Pipeline Failed Reason <a href="${BUILD_URL}">console output</a>.</p>
                                    <p> Web Admins: ${MEMS}.</p>
                                    <p> Pipeline Execution Date: ${DATE}.</p>
                                </body> 
                            </html>""",
                    mimeType: 'text/html'
                )
            }
        }
    }
}

