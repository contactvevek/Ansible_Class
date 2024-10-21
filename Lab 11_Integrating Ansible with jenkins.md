### Task 1: Install Jenkins Server
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
      ```
      save the file using `ESCAPE + :wq!`
   * Now execute the `jenkins.sh` script file
      ```
      . ./jenkins.sh
      ```
   * One it's done move to Task 2.
### Task 2: Login to the Jenkins Server
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
     
### Task 3: Be ready with GitHub Repo
  * Create a **GitHub Account** & **Public Repository** with name as **"Yaml-Repo"**
  * Now, Save your  `yaml` files, `Inventory` file, and `Jenkinsfile` in the repository. Alternatively, you can refer to any of your existing project GitHub repositories that contain Ansible YAML files.

   **Note:** (To SignUp for GitHub Account, [Click Here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home))
   
  * You need to generate a **Personal Access Token** (PAT) in GitHub.
   
  * Click Here To Generate the Token:</summary>
     
     * First, Go to your `GitHub homepage,` Click on the top right `profile Icon` and then `settings`
     * Click on `Developer settings` (At the bottom on the left side menu). Click on `Personal Access Token` and then Click on `Generate New Token.`
     * Under '**Select Scopes**' select all items. Click on '**generate token**'.
     * Once generated, **Copy and save the token in a safe location as it is not visible again in GitHub.**
     
        **Example:** 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
   
   

### Task 4: Configure Ansible in Jenkins 
   * Ensure you have the following Jenkins plugins installed:
   
   * `Git Plugin:` Allows Jenkins to pull code from GitHub.
   * `GitHub Integration Plugin:` Enables webhooks from GitHub.
   * `Ansible Plugin:` Enables integration with Ansible to run playbooks.

##### Do the below in Jenkin's Dashboard:

   * Click on Manage Jenkins > Plugins > Available Plugins tab and search for **Ansible** and click Install.
   * Once the installation is completed, click on **Go back to the top page**.
   * On Home Page select Manage Jenkins > Tool.
   * Inside Tool Configuration, look for **Ansible installations**, click Add Andible.
   * Give the Name as **Ansible**, Slect "Install automatically", and Save the configuration.

##### Set Up SSH Credentials in Jenkins
  You need to give Jenkins access to your target nodes via SSH, so it can run the Ansible playbooks.

  * Go to Manage Jenkins > Manage Credentials.
  * Click on (global) under Domains.
  * Click **Add Credentials**.
    ![image](https://github.com/user-attachments/assets/08eb2602-0ea5-478a-9dc9-273071031f55)

  * In the Kind dropdown, select **SSH Username with private key**.
  * Enter the username that Ansible will use to SSH into your target nodes (i.e `sirin_a`).
  * Give this credential an ID (e.g., ansible-ssh-key). This will be referenced in the Jenkinsfile.
  * Now Select "Enter directly" under "Private Key" 
  * Upload the private key that corresponds to the SSH key used by Ansible to access the target nodes.
  * **Note:** To get the `Private Key` go to `Jenkins Server` and Execute the below command:
      ```
      cat /home/sirin_a/.ssh/id_rsa
      ```
     (Copy the entire content of the Private Key, including the **First and Last line** till `5 hyphens` only.)
     
   * Once Copied, Paste it into the space provided for the **private key** then click on **Create**..

      
    * `Note` : Ensure the target nodes have Ansible set up for passwordless SSH access. Ansible should be able to connect to these nodes using the SSH credentials configured in Jenkins.
    
##### Create a Jenkins Pipeline
  A pipeline job will define the steps to pull the YAML files (Ansible playbooks) from GitHub and deploy them using Ansible.
  
  * Go to **Jenkins Dashboard** and click **New Item**.
  * Enter Enter an item name (e.g., ansible-pipeline), select Pipeline, and click OK.
  * Scroll down to Pipeline.
  * Under **Pipeline** Definition, select Pipeline script from SCM.
  * Choose **Git** as the SCM.
  * In Repository URL, enter your github repo (i.e https://github.com/sirinali07/Yaml-Repo.git).
  * Under Credentisls select Add **Jenkins** 
     ![image](https://github.com/user-attachments/assets/21e771c0-d911-4f8d-aae5-24a8baa54776)
  * Then you will be prompted to the Jenkins Credentials Provider page. Under Add Credentials, you can add your GitHub Username (i.e sirin_a), Password (Github Token), and Description (i.e Git-Credentials). Then click on Add.
    ![image](https://github.com/user-attachments/assets/67ddf5e7-ce48-418e-8d7e-e358d17deef7)

  * Set the branch (e.g., main or master).
  * Set Repository browser to **Auto**.
  * In the `Script Path`  add the Jenkins script file name (i.e **Jenkinsfile) and Save the configuration.










