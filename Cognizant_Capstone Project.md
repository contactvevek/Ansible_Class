

#### Task 1: Launching the VM Instances in GCP

1. Go to the GCP Console and select Compute Engine.
2. launch 2 VM Instance.
6. SSH into one of the machine

#### Task 2: Execute the Following Commands for Installing the Ansible

* Check the IP is changed to “ansible-workstation” hostname and run the following command to update:
```bash
sudo apt update && sudo apt upgrade -y
```

* Install Python 3.9, python3-pip, and wget which are compatible with Ansible using the below command:
```bash
sudo apt install python3 python3-pip wget -y
```
#### Install via `apt` package manager
* Use the following command to add the Ansible PPA:
```bash
sudo apt-add-repository --yes --update ppa:ansible/ansible
```

* Now install Ansible by using the following command:
```bash
sudo apt install ansible -y
```

* Check the version of Ansible using the below command:
```bash
ansible --version
```

#### Task 3: Make Entry in the Host File
* Create `Ansible directory`  by running the following command:
```bash
sudo mkdir /etc/ansible
```
* To run all the commands on the localhost, make the following entry:

```bash
sudo vi /etc/ansible/hosts
```

* Add the `localhost` to the `ansible_hosts`
```
localhost ansible_connection=local
```
 save the file using `ESCAPE + :wq!`

* For enabling `PasswordLess authentication` follow the below steps:

* On the ansible-server
```bash
ssh-keygen -t rsa -b 2048
```
* Verify the  SSH key pair on your local machine:
```bash
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```
* Copy the public key to Managed Nodes:
```bash
cat ~/.ssh/id_rsa.pub
```
* Log into the remote server (managed Nodes) via another method (e.g., console access).
* Add the public key to `~/.ssh/authorized_keys` on the remote servers:
```bash
echo "your_public_key_content" >> ~/.ssh/authorized_keys
```
* Set the below permissions On the remote servers:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
* Enable Passwordless authentication
```
sudo visudo
```
* Paste the below, replace with your username and exit the vi editor
```
sirin_a ALL=(ALL) NOPASSWD: ALL
```
* exit from the managed node-1
```bash
exit
```
* Execute the below command on `ansible-server` to verify passwordless authentication has been enabled
```
ssh Your_User@PUBLIC_IP of node-1
```
```ansible
ansible all -m ping
```
#### Task 4: Install Jenkins on Local Server
* Create yaml file
```bash
vi jenkins.yaml
```
Add the below code in the file
```yaml
---
- name: Start installing Jenkins pre-requisites before installing Jenkins
  hosts: localhost
  become: yes
  gather_facts: no

  tasks:

    - name: Cleanup old Jenkins configurations and packages
      block:
        - name: Remove old Jenkins repository configuration
          file:
            path: /etc/apt/sources.list.d/jenkins.list
            state: absent

        - name: Remove any existing Jenkins installation
          apt:
            name: jenkins
            state: absent

        - name: Remove any leftover Jenkins data
          command: rm -rf /var/lib/jenkins
          ignore_errors: yes  # Ignore errors in case the directory does not exist

    - name: Update apt repository with latest packages
      apt:
        update_cache: yes
        upgrade: yes

    - name: Install JDK (openjdk-17-jdk) on Jenkins server
      apt:
        name: openjdk-17-jdk
        update_cache: yes

    - name: Install Jenkins apt repository key
      apt_key:
        url: https://pkg.jenkins.io/debian/jenkins.io-2023.key
        state: present

    - name: Configure the Jenkins apt repository
      apt_repository:
        repo: deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/
        filename: jenkins
        state: present

    - name: Update apt-get repository with "apt-get update"
      apt:
        update_cache: yes

    - name: Install Jenkins
      apt:
        name: jenkins
        update_cache: yes

    - name: Start Jenkins service
      service:
        name: jenkins
        state: started

    - name: Wait for Jenkins to create the initial admin password file
      wait_for:
        path: /var/lib/jenkins/secrets/initialAdminPassword

    - name: Display Jenkins admin password
      command: cat /var/lib/jenkins/secrets/initialAdminPassword
      register: out

    - debug:
        msg: >
          Jenkins is up and running!
          Access Jenkins at: http://<Public_IP>:8080 
          Initial Admin Password: {{ out.stdout }}
```
save the file using `ESCAPE + :wq!`
* Now execute the playbook
```bash
ansible-playbook jenkins.yaml
```
* Verify the Jenkins Landing Page: Open a web browser and navigate to your Jenkins landing page using your Jenkins server's public IP address. Replace `<Your_Jenkins_IP>` with the actual public IP.
 ```
 http://<Your_Jenkins_IP>:8080/
 ```
 ![image](https://github.com/user-attachments/assets/71bebf61-157e-4c99-81a6-54e28f11ba79)

* It requests the **InitialAdminPassword** during the setup.
  
* The **InitialAdminPassword** is displayed on the Ansible CLI when the playbook is executed
  
* Now, go back to Jenkin's landing page in the **Web Browser**:
  
  (Enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**)
.
* Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
  
* Click on **Install suggested Plugins** on the Customize Jenkins page.
  
* Once the plugins are installed, it gives you the page where you can create a New **Admin User**.
  
* Enter the **User Id** and **Password** followed by **Name** and **E-Mail ID** then click on **Save & Continue**.
  
* In the next step, on the Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

#### Task 5: Prepare the GitHub Repo

* Create a **GitHub Account** & **Public Repository** with name as **"capstone-project"**
  
* Now, Save your  `yaml` files, `Inventory` file, in the repository. 

**Note:** (To SignUp for GitHub Account, [Click Here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home))
![image](https://github.com/user-attachments/assets/c7f1e904-9f51-413f-a51c-e7f2ab62578c)
* Below files to your repository
* Copy the below content for your `Prometheous Server`
```yaml
---
- name: Start installing Jenkins pre-requisites before installing Jenkins
  hosts: localhost
  become: yes
  gather_facts: no

  tasks:

    - name: Cleanup old Jenkins configurations and packages
      block:
        - name: Remove old Jenkins repository configuration
          file:
            path: /etc/apt/sources.list.d/jenkins.list
            state: absent

        - name: Remove any existing Jenkins installation
          apt:
            name: jenkins
            state: absent

        - name: Remove any leftover Jenkins data
          command: rm -rf /var/lib/jenkins
          ignore_errors: yes  # Ignore errors in case the directory does not exist

    - name: Update apt repository with latest packages
      apt:
        update_cache: yes
        upgrade: yes

    - name: Install JDK (openjdk-17-jdk) on Jenkins server
      apt:
        name: openjdk-17-jdk
        update_cache: yes

    - name: Install Jenkins apt repository key
      apt_key:
        url: https://pkg.jenkins.io/debian/jenkins.io-2023.key
        state: present

    - name: Configure the Jenkins apt repository
      apt_repository:
        repo: deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/
        filename: jenkins
        state: present

    - name: Update apt-get repository with "apt-get update"
      apt:
        update_cache: yes

    - name: Install Jenkins
      apt:
        name: jenkins
        update_cache: yes

    - name: Start Jenkins service
      service:
        name: jenkins
        state: started

    - name: Wait for Jenkins to create the initial admin password file
      wait_for:
        path: /var/lib/jenkins/secrets/initialAdminPassword

    - name: Display Jenkins admin password
      command: cat /var/lib/jenkins/secrets/initialAdminPassword
      register: out

    - debug:
        msg: >
          Jenkins is up and running!
          Access Jenkins at: http://<Public_IP>:8080 
          Initial Admin Password: {{ out.stdout }}
```
* Copy the below content for your `inventory` file, modify the username and IP 
```yaml
node-1   ansible_ssh_host=10.128.0.54 ansible_ssh_user=sirin_a  
```
#### Task 6: Configure Ansible in Jenkins 
* Click on Manage Jenkins > Plugins > Available Plugins tab and search for **Ansible**, and **SSH Agent** and click Install.
* Once the installation is completed, click on **Go back to the top page**.
* On Home Page select Manage Jenkins > Tool.
* Inside Tool Configuration, look for **Ansible installations**, click Add Andible.
* Give the Name as **Ansible**, Slect "Install automatically", and Save the configuration.

  
#### Task 7: Set Up the SSH  Credentials in Jenkins
You need to give Jenkins access to your target nodes via SSH, so it can run the Ansible playbooks.

* Go to Manage Jenkins > Manage Credentials.
* Click on (global) under Domains.
* Add the Private Key 
* Click **Add Credentials**.    
* In the Kind dropdown, select **SSH Username with private key**.
* Enter the username that Ansible will use to SSH into your target nodes (i.e `sirin_a`).
* Give this credential an ID (e.g., `SSH-KEY`). 
* Now Select "Enter directly" under "Private Key". The private key can be copied form the Jenkins CLI using the below command
```
cat /home/sirin_a/.ssh/id_rsa
```
 (Copy the entire content of the Private Key, including the **First and Last line** till `5 hyphens` only.)  
* Once Copied, Paste it into the space provided for the **private key** then click on **Create**.
  
#### Task 8: Create a Jenkins Pipeline
A pipeline job will define the steps to pull the YAML files (Ansible playbooks) from GitHub and deploy them using Ansible.

* Go to **Jenkins Dashboard** and click **New Item**.
* Enter an item name (e.g., ansible-pipeline), select Pipeline, and click OK.
* Click on the Pipeline and add the below script
```
    pipeline {
    agent any

    environment {
        GITHUB_REPO_URL = 'https://github.com/sirinali07/cognizant-capstone-repo.git'
        GIT_BRANCH = 'master'
        INVENTORY_FILE = 'ansible_playbooks/inventory'
        PLAYBOOK_FILE = 'ansible_playbooks/prometheus.yaml'
        SSH_KEY_CREDENTIAL_ID = 'SSH-KEY'
        
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh '''
                        if [ -d "ansible_playbooks" ]; then
                            rm -rf ansible_playbooks
                        fi
                        git clone -b ${GIT_BRANCH} ${GITHUB_REPO_URL} ansible_playbooks
                    '''
                }
            }
        }

        stage('Set Up SSH Key') {
            steps {
                script {
                    sshagent([SSH_KEY_CREDENTIAL_ID]) {
                        echo 'SSH Key added for Ansible playbook execution.'
                    }
                }
            }
        }

        

        stage('Execute Ansible Playbook') {
            steps {
                script {
                    sshagent([SSH_KEY_CREDENTIAL_ID]) {
                        sh '''
                            ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i ${INVENTORY_FILE} ${PLAYBOOK_FILE} \
                            -e "target_user=sirin_a"
                        '''
                    }
                }
            }
        }
    }

 
}
```
* Click save and Build
* 
#### Task 8: Access Prometheus Web UI: 

* Open a browser and go to `http://<your-gcp-vm-ip>:9090`
- i.e http://35.232.129.112:9090/
- Ensure the whitelisting of the port 9090
![image](https://github.com/user-attachments/assets/da1bd9c3-c360-41b3-9043-65b977f63607)

