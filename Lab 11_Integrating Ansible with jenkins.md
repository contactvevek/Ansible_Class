#### Task 1: Install Jenkins Server
   * Create a file named `jenkin.sh`
      ```
       vi jenkins.sh
      ```
      Add the below code in the script file
      ```
       sudo apt update -y
       sudo apt upgrade -y
       sudo apt install -y openjdk-17-jdk
       java --version
       curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
       echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
       sudo apt update -y
       sudo apt install -y jenkins
       sudo apt install sshpass
       sudo systemctl start jenkins
       sudo systemctl enable jenkins
       sudo systemctl status jenkins
       sudo apt update && sudo apt install ansible -y #for Ansible Installation
      ```
      save the file using `ESCAPE + :wq!`
   * Now execute the `jenkins.sh` script file
      ```
      . ./jenkins.sh
      ```
   * One it's done move to Task 2.
     
#### Task 2: Login to the Jenkins Server
   * Verify the Jenkins Landing Page: Open a web browser and navigate to your Jenkins landing page using your Jenkins server's public IP address. Replace `<Your_Jenkins_IP>` with the actual public IP.
     ```
     http://<Your_Jenkins_IP>:8080/
     ```
     ![image](https://github.com/user-attachments/assets/71bebf61-157e-4c99-81a6-54e28f11ba79)
   
   * It requests the **InitialAdminPassword** during the setup.)
      
   * To obtain the **InitialAdminPassword**, access the Jenkins Server by SSHing it.
   
   * From Jenkins Server execute the below command and copy the password.
     ```
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
      ![image](https://github.com/user-attachments/assets/ed3b8543-feb8-4b8e-b753-73c6f6b909b5)
   
   * Now, go back to Jenkin's landing page in the **Web Browser**:
      
      (Enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**)
   
   * Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
   * Click on **Install suggested Plugins** on the Customize Jenkins page.
   * Once the plugins are installed, it gives you the page where you can create a New **Admin User**.
   * Enter the **User Id** and **Password** followed by **Name** and **E-Mail ID** then click on **Save & Continue**.
   * In the next step, on the Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

     
#### Task 3: Be ready with GitHub Repo
  * Create a **GitHub Account** & **Public Repository** with name as **"ansible_files"**
  * Now, Save your  `yaml` files, `Inventory` file, in the repository. 

   **Note:** (To SignUp for GitHub Account, [Click Here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home))
   
  * You need to generate a **Personal Access Token** (PAT) in GitHub (Optional, when required)
   
  * Click Here To Generate the Token:</summary>
     
     * First, Go to your `GitHub homepage,` Click on the top right `profile Icon` and then `settings`
     * Click on `Developer settings` (At the bottom on the left side menu). Click on `Personal Access Token` and then Click on `Generate New Token.`
     * Under '**Select Scopes**' select all items. Click on '**generate token**'.
     * Once generated, **Copy and save the token in a safe location as it is not visible again in GitHub.**
     
        **Example:** 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
   
   

#### Task 4: Configure Ansible in Jenkins 
   * Ensure you have the following Jenkins plugins installed:
   
       * `Git Plugin:` Allows Jenkins to pull code from GitHub.
       * `GitHub Integration Plugin:` Enables webhooks from GitHub.
       * `Ansible Plugin:` Enables integration with Ansible to run playbooks.
       * `SSH Agent Plugin`: To manage and provide SSH credentials 

   * Click on Manage Jenkins > Plugins > Available Plugins tab and search for **Ansible** and click Install.
   * Once the installation is completed, click on **Go back to the top page**.
   * On Home Page select Manage Jenkins > Tool.
   * Inside Tool Configuration, look for **Ansible installations**, click Add Andible.
   * Give the Name as **Ansible**, Slect "Install automatically", and Save the configuration.

#### Task 5: Enable password-less authentication on the worker node
   * On the `Jenkins CLI` execute the below commands
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
   * Log into the each remote server (managed Nodes) via another method (e.g., console access).
  
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
   * Paste the below and exit the vi editor
     ```
     sirin_a ALL=(ALL) NOPASSWD: ALL
     ```
     ```bash
     exit
     ```
   * Execute the below command on `Jenkin-Server` to verify passwordless authentication has been enabled
     ```bash
     ssh Your_User@PUBLIC_IP of managed-node1
     ```
     ```bash
     ssh Your_User@PUBLIC_IP of managed-node2
     ```
    
### Task 6: Set Up SSH and the GitHub Credentials in Jenkins
  You need to give Jenkins access to your target nodes via SSH, so it can run the Ansible playbooks.

  * Go to Manage Jenkins > Manage Credentials.
  * Click on (global) under Domains.
  * Add the Private Key 
      * Click **Add Credentials**.    
      * In the Kind dropdown, select **SSH Username with private key**.
      * Enter the username that Ansible will use to SSH into your target nodes (i.e `sirin_a`).
      * Give this credential an ID (e.g., SSH-KEY). 
      * Now Select "Enter directly" under "Private Key". The private key can be copied form the Jenkins CLI using the below command
        ```
        cat /home/sirin_a/.ssh/id_rsa
        ```
         (Copy the entire content of the Private Key, including the **First and Last line** till `5 hyphens` only.)  
       * Once Copied, Paste it into the space provided for the **private key** then click on **Create**.

#### Task 7: Create a Jenkins Pipeline
  A pipeline job will define the steps to pull the YAML files (Ansible playbooks) from GitHub and deploy them using Ansible.
  
  * Go to **Jenkins Dashboard** and click **New Item**.
  * Enter an item name (e.g., ansible-pipeline), select Pipeline, and click OK.
  * Click on the Pipeline and add the below script
```
    pipeline {
    agent any

    environment {
        GITHUB_REPO_URL = 'https://github.com/sirinali07/Yaml-Repo.git'
        GIT_BRANCH = 'master'
        INVENTORY_FILE = 'ansible_playbooks/inventory'
        PLAYBOOK_FILE = 'ansible_playbooks/install-apache2.yaml'
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
    
 








