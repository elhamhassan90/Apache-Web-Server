# 🚀 Apache-Web-Server with Jenkins/Ansible


## 🌟 Project Description

This project aims to set up a Continuous Integration/Continuous Deployment (CI/CD) pipeline using Jenkins, Ansible, and GitLab. It involves provisioning virtual machines (VMs) with dedicated services, managing user access, integrating GitLab with Jenkins, and detecting code commits to autonomously execute an Ansible playbook. This playbook installs and configures Apache HTTP Server, and generates an email notification if the pipeline fails.


---

## 🗂️ Project Structure
```
Apache-Web-Server/
├── Jenkinsfile      
├── Bash-Scripts/            
│   ├── users
│   └── groups-and-assign
│
├── Ansible playbook          
│   ├── anible.cfg              
│   ├── inventory
│   ├── site.yml
│   └── roles/apache           
│       ├── tasks/
│ 	│ 	└── main.yml
│       ├── handlers/
│ 	│ 	└── main.yml
│       ├── vars/
│ 	│ 	└── main.yml
│       └── templates/
│		└── index.html.j2
├── images/              
│   ├── 1.png         
│   ├── 2.png
│   └── 3.png
└── README.md
```

---

---
## ☁️ Technologies Used

- **Ansible**
- **Jenkins** 
- **Gitlab** 
- **Docker**
- **3 CentOS VM** as development environment
-  **Gmail** for notification

---

---

## 📷 Included Screenshots
## 📷 Screenshots
### 🔹 Apache content as my cv as html web content
![Apache](./images/1.png)

---

### 🔹 Mail After success or failure of CICD build
![Notification](./images/3.png)

---



## Prerequisites

1. **VMware Workstation** installed on your machine (using CentOS 9 Stream image).
2. **VM1 - Jenkins Server**: A dedicated Jenkins server for orchestrating the CI/CD pipeline.
3. **VM2 - GitLab Instance**: Implementation of a private GitLab instance for hosting Git repositories.
4. **VM3 - Web Server**: Deployment of a web server with Apache HTTP Server service using Ansible.

## Setup

### Provisioning VMs

- **VM1: Jenkins Server**
  - IP: `192.168.142.128:8080`
  - Admin User: `elham`
  
- **VM2: GitLab Instance**
  - IP: `192.168.142.132`
  - Admin User: `elham`
  
- **VM3: Web Server with Apache HTTP Server**
  - IP: `192.168.142.129`
  - Admin User: `elham`

---
## Pipeline Workflow

1. **Git Commit**: A change in the GitLab repository triggers the Jenkins pipeline via webhook.
2. **Jenkins Pipeline**: Jenkins fetches the code and runs the **Ansible Playbook** to configure the Apache Web Server on VM3.
3. **Email Notification**: If the pipeline fails at any point, an email notification is sent to the relevant user(s).

---
---
4. **Creating a Private Git repository on Gitea**:
    - Create a Git repository called "Apache-Web-Server" in GitLab.
    - Push all files (Ansible playbook, roles, Jenkinsfile) to that repository. 

5. **GitLab Integration with Jenkins**:
    #### In GitLab Instance
    - Generate Access Token called "Jenkins_GitLab_API_Access" from the account settings. 
    #### In Jenkins web console
    - Install (GitLab, GitLab API, GitLab Authentication) Plugins.
    - Add a credential of kind "GitLab API token" by using "Jenkins_GitLab_API_Access" from GitLab.

6. **Detect a code commit from GitLab repo to trigger the Jenkins pipeline**:

    #### In Jenkins web console
    - Generate API Token called "GitLab_Webhook" from "admin > Configure". 
    #### In GitLab Instance
    - Add a Webhook in the repo from the repo settings using "GitLab_Webhook" from Jenkins.

7. **Jenkins Configuration**:
    #### To access the private GitLab repository
    - Add a credential of kind "Username with password" include "Jenkins_GitLab_API_Access".

    #### To Make the ansible playbook reach VM3
    - Add a credential of kind "SSH Username with private key" include a private key which its public key is on VM3.
    - Add a credential of kind "Secret text" include the VM3 apache user sudo password.

    #### To send the email notification successfully
    - Install (Email Extension, Email Extension Template) Plugins.
    - Add a credential of kind "Username with password" include the app password which generated in email app (Gmail).
   
## Ansible Playbook to deploy Apache server

 ### Configure the inventory file:
   ```ini
   [webservers]
   192.168.142.129
   ```
 ### Upade the ansible configuration file:
   ```ini
   [defaults]
   inventory = inventory
   remote_user = elham
   host_key_checking = no

  [privilege_escalation]
  become = yes
  become_user = root
  become_method = sudo
  become_ask_pass = no
  ```
 ### Install the role with ansible-galaxy command:
    ansible-galaxy init roles/webserver-role
 ### Ansible Playbook (WebServerSetup.yml)
    - name: Playbook to install and configure Apache HTTP Server on VM3
    hosts: webservers
    gather_facts: no
    roles:  
        - apache
 ### Ansible Role (apache)

   #### Tasks (roles/tasks/main.yml)
   ###### Install httpd server package 
    - name: install apache
    ansible.builtin.yum:
        name: httpd
        state: present
   ###### Start httpd service
    - name: start and enable httpd service
    ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
   ###### Allow HTTP traffic through firewall and notify the handler to restart firewall service  
    - name: allow HTTP traffic through firewall
    ansible.posix.firewalld:
        service: http
        permanent: yes
        state: enabled
    notify:
        - reload firewalld
   ###### Configure Apache home page and notify a handler to restart httpd service 
    - name: add custom root page
    ansible.builtin.template:
        src: index.html.j2
        dest: /var/www/html/index.html
        mode: 0644
    notify:
        - restart apache service
   #### Handlers (roles/handlers/main.yml)
   ###### Restart firewall service  
    - name: reload firewalld
    service:
        name: firewalld
        state: reloaded
   ###### Restart httpd service  
    - name: restart apache service
    ansible.builtin.service:
        name: httpd
        state: restarted
   #### Variables (roles/vars/main.yml)
   ###### Variable to be showed in the apache new home page
    username: "Eng. Elham Hasan Gouda Tammam Kedwany"
   #### Templates (roles/templates/index.html.j2)
   ###### Coding the new apache home page with html
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Elham Hasan - CV</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #e0e0e0; 
    }
    .container {
      width: 95%;
      max-width: 1200px; 
      margin: 40px auto;
      padding: 50px; 
      background-color: #fff; 
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
      min-height: 2200px; 
    }
    h1, h2, h3 {
      color: #333;
    }
    p, li {
      color: #444;
      line-height: 1.7;
      font-size: 16px;
    }
    ul {
      padding-left: 20px;
    }
    a {
      color: #007acc;
      text-decoration: none;
    }
    a:hover {
      text-decoration: underline;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Elham Hasan Gouda</h1>
    <h2>DevOps Engineer</h2>
    <p><strong>Mobile:</strong> (+20)1221607477</p>
    <p><strong>GitHub:</strong> <a href="https://github.com/ElhamHasan" target="_blank">github.com/ElhamHasan</a></p>
    <p><strong>Address:</strong> Naser City, Cairo, Egypt</p>
    <p><strong>LinkedIn:</strong> <a href="https://www.linkedin.com/elhamhasan" target="_blank">linkedin.com/elhamhasan</a></p>
    <p><strong>Email:</strong> elhamhassan252@gmail.com</p>
    <p><strong>Credly:</strong> <a href="https://www.credly.com/elham-hasan" target="_blank">credly.com/elham-hasan</a></p>

    <h3>Profile</h3>
    <p>Energetic, hardworking Junior DevOps Engineer, ITI graduate seeking to work for a reputable company where my educational background, skills and experience can be fully-utilized and enhanced.</p>

    <h3>Education</h3>
    <p><strong>Bachelor's degree, Engineering of communication and electronics</strong>, Tanta University | 2015 - 2020</p>
    <p>Grade: Good</p>
    <p>Graduation project: Computer Vision Based Mouse, Cursor control in Python (OpenCV) | Grade: Excellent</p>

    <h3>Internships</h3>
    <ul>
      <li>ITI (System Administration & DevOps Track) | Nov 2024 - Apr 2025</li>
      <li>ITI Online (Data Analyst Track “ITI Tech-Leaps”) | Apr 2022 - Aug 2022</li>
      <li>NTI (Big Data) | Jan 2021 - Feb 2021</li>
      <li>PPC (Summer Training) | Sep 2019</li>
    </ul>

    <h3>Experience</h3>
    <ul>
      <li>Freelancer (Linux Specialist) | Khamsat.com | Apr 2025 - Now</li>
      <li>ADSL Technical Support | Telecom Egypt (We) | Feb 2021 - Apr 2021</li>
    </ul>

    <h3>Certificates</h3>
    <ul>
      <li>Red Hat® Certified System Administrator (RHCSA®) – <a href="#">RedHat.com</a></li>
      <li>Huawei Cloud Certified Developer Associate (HCCDA) – <a href="#">HuaweiCloud.com</a></li>
      <li>Golden Badge of SQL and Python (HackerRank) – <a href="#">HackerRank.com</a></li>
      <li>AWS Certified SysOps Admin – Associate (SOA-C02) – <em>Ongoing</em> – <a href="#">AWS.com</a></li>
    </ul>

    <h3>Skills</h3>
    <ul>
      <li><strong>Networking:</strong> IT Essentials, CCNA, Database & OS Fundamentals</li>
      <li><strong>System Admin:</strong> Red Hat (Ansible), Windows Server (MCSA), Apache, Nginx</li>
      <li><strong>Cloud & Virtualization:</strong> VMware vSphere, Azure, Huawei Cloud</li>
      <li><strong>Automation & DevOps:</strong> Git, Terraform, Docker, Kubernetes</li>
      <li><strong>Development:</strong> Python, Bash Scripting</li>
      <li><strong>Soft Skills:</strong> Communication, Teamwork, Troubleshooting, Persistence</li>
    </ul>

    <h3>Projects</h3>
    <ul>
      <li>Multi-Tier AWS Infrastructure & Auto-Deployment – <a href="#">IAC (Terraform & AWS)</a></li>
      <li>User Group Manager GUI Menu – <a href="#">Bash/Linux Project</a></li>
      <li>Users & Groups Management System – <a href="#">Python/Linux</a></li>
      <li>Windows Server Admin Project – <a href="#">MCSA</a></li>
      <li>Interconnecting Cisco Devices – <a href="#">CCNA Project</a></li>
    </ul>

    <h3>Languages & Personal Info</h3>
    <ul>
      <li>Arabic (Native), English (Very Good)</li>
      <li>Driving License: Valid (Egypt)</li>
    </ul>
  </div>
</body>
</html>
```

## Jenkins File to deploy the Ansible Playbook

#### First stage: To run script to fetch the admin members when the failure of the pipeline 

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

#### Second stage: To run ansible playbook in the target machine (VM3) using "SSH_PRIVATE_KEY" & "SUDO_PASS" credentials

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


#### Send email notification in case of pipeline failure 

    post {
        success {
            emailext(
                to: "elhamhassan252@gmail.com",
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <html>
                        <body style="font-family: Arial, sans-serif;">
                            <h2 style="color: green;">✅ Pipeline Succeeded</h2>
                            <p><strong>Job Name:</strong> ${JOB_NAME}</p>
                            <p><strong>Build Number:</strong> ${BUILD_NUMBER}</p>
                            <p><strong>Status:</strong> SUCCESS</p>
                            <p>🔗 <a href="${BUILD_URL}">Click here to view full build details</a></p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html'
            )
        }

        failure {
            emailext(
                to: "elhamhassan252@gmail.com",
                subject: "❌ FAILURE: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <html>
                        <body style="font-family: Arial, sans-serif;">
                            <h2 style="color: red;">❌ Pipeline Failed</h2>
                            <p><strong>Job Name:</strong> ${JOB_NAME}</p>
                            <p><strong>Build Number:</strong> ${BUILD_NUMBER}</p>
                            <p><strong>Status:</strong> FAILURE</p>
                            <p>🔗 <a href="${BUILD_URL}">Click here to investigate the issue</a></p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html'
            )
        }
    }
}





## 👩‍💻 Author
**Elham**  
🔧 DevOps Enthusiast | System Admin | Automation Engineer | Web Designer  
🚀 Built locally on CentOS and Docker  
📬 GitHub: https://github.com/elhamhassan90  
🔗 LinkedIn: www.linkedin.com/in/elham-hasan-6b029433a  
---

⭐ *If you found this useful or inspiring, star the repo and con
