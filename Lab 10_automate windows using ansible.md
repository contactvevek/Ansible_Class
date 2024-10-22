
### Task 1: Launching a Windows VM in GCP

1. Go to the GCP Console and select Compute Engine.
2. Select `VM Instances` and click on `CREATE INSTANCE`.
3. Enter the name of the instance `windows-server` and select the Zone in which the instance is going to launch.
4. Select machine type (i.e e2-standard-4). In the boot disk section, click on Change and select `Windows Server 2022 Datacenter`.
5. Click on create and launch the instance.
6. Whitelist port 5985(winrm-http) and 5986(winrm-https) in the firewall.
7. Reset Windows Password and Download `RDP` file to connect with "Windows-Server"

### Task 2: Windows Server Configuration

  * Ensure that your Windows server is reachable and has PowerShell enabled.
  * Set up Windows Remote Management (WinRM) for Ansible to connect and manage it.
    
#### Enable and Configure WinRM on the Windows Server

1. Open PowerShell as Administrator

2. Get the current WinRM configuration
 ```powershell
 winrm get winrm/config
 ```
3. Set the WinRM service to start automatically and start the service
  ```powershell
 Set-Service WinRM -StartupType Automatic
 Start-Service WinRM
 ```

4. Enable basic authentication  
  ```powershell
  winrm set winrm/config/service/auth '@{Basic="true"}'
 ```
5. Allow unencrypted traffic  
  ```powershell
 winrm set winrm/config/service '@{AllowUnencrypted="true"}'
 ```
6. Configure trusted hosts (for testing, allows connections from all hosts)
  ```powershell
 winrm set winrm/config/client '@{TrustedHosts="*"}'
 ```
7. Create a self-signed certificate for HTTPS (replace with your server's IP address)
  ```powershell
 $cert = New-SelfSignedCertificate -DnsName "35.188.200.132" -CertStoreLocation Cert:\LocalMachine\My
 ```
8. Bind the certificate to WinRM
  ```powershell
  $thumbprint = $cert.Thumbprint
  ```
  ```powershell
  winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=""35.188.200.132""; CertificateThumbprint=""$thumbprint""}"
  ```
 *Note:* If WinRM listener with the same address and transport (HTTPS) is already configured on the system, you need to delete the existing listener before creating a new one with the desired configuration
 
 ![image](https://github.com/user-attachments/assets/a0cbf878-5c9f-47b2-af91-4df0956d82f6)
 
 * Run the following command to view all current listeners:
 ```powershell
 winrm enumerate winrm/config/listener
 ```
 * Delete the Existing HTTPS Listener
 Identify the HTTPS listener from the output (where Transport=HTTPS) and then delete it using the following command:
 ```powershell
 winrm delete winrm/config/Listener?Address=*+Transport=HTTPS
 ```
 * creating a new one with the desired configuration
 ```powershell
 winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=""35.188.200.132""; CertificateThumbprint=""$thumbprint""}"
 ```
9. Enable the firewall rule for WinRM
  ```powershell
 netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
 ```
10. Verify WinRM configuration
  ```powershell
  winrm get winrm/config
  ```


### Task 3: Set Up Ansible Control Node
 Install necessary dependencies on the Ansible control node:

 ```bash
 pip3 install requests pywinrm --break-system-packages
 ```
 Verify the winrm installation
  ```bash
 python3 -c "import winrm"
 ```
 Configure Ansible Inventory File
 
 Edit your Ansible inventory file to include your Windows server details:

 ```yaml
  [windows]
  winserver ansible_host=35.188.200.132 ansible_user=sirin_a ansible_password=":PCj%u~E&72PW,P" ansible_port=5986 ansible_connection=winrm ansible_winrm_server_cert_validation=ignore ansible_winrm_transport=basic
```
**Test Ansible Connection to the Windows Server**
 ```bash
 ansible windows -m win_ping
```
***Let's Automate Windows Using Ansible***

### Task 4: Install PuTTY on Windows Server
 Navigate home directory and create new directory there
 ```bash
 cd ~ && mkdir win-yaml && cd win-yaml
 ```
 Create `win-putty.yml` to install PuTTy on windows server
 ```bash
 vi win-putty.yml
 ```
 Add the given content, by pressing "INSERT"
 ```yaml
 ---
 - name: Install PuTTY on Windows server
   hosts: windows
   tasks:
     - name: Download and install PuTTY from URL
       win_package:
         path: "https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.81-installer.msi"
         product_id: "{D6CB24DD-E28C-497D-9837-0475D0E78E33}"
         arguments: "/quiet"
         state: present
 ```
 save the file using `ESCAPE + :wq!`
 
 Run the Ansible playbook
 ```bash
 ansible-playbook win-putty.yml
 ```
### Task 5: Install 7-Zip Using Chocolatey
 Create the playbook to install the 7zip package on your Windows server using Chocolatey:
 ```
 vi  7zip.yml
 ```
 Add the given content, by pressing "INSERT"
 ```yaml
 ---
 - name: Install a package using Chocolatey on Windows server
   hosts: windows
   tasks:
   - name: Install 7zip using Chocolatey
     win_chocolatey:
       name: 7zip
       state: present
 ```
 save the file using `ESCAPE + :wq!`
 
 Now execute your Ansible playbook 
 
 ```bash
 ansible-playbook 7zip.yml
 ```
### Task 6: Install IIS (Web Server) on Windows
 Create the playbook  `install-iis.yml`
 ```
 vi install-iis.yml
 ```
 Add the given content, by pressing `INSERT`
 ```yaml
 ---
 - name: Manage Windows Hosts
   hosts: windows
   tasks:
     - name: Install IIS (Web-Server)
       win_feature:
         name: Web-Server
         state: present
 
     - name: Ensure IIS service is running
       win_service:
         name: W3SVC
         start_mode: auto
         state: started
 
     - name: Verify IIS installation by fetching default page
       win_uri:
         url: http://localhost
         method: GET
         return_content: yes
       register: result
 
     - name: Display the result of the default page request
       debug:
         var: result.content
 ```
 
  save the file using `ESCAPE + :wq!`
  
  Now execute your Ansible playbook 
  ```bash
  ansible-playbook install-iis.yml
  ```
### Task 7: Run Custom Windows Command 
 Execute custom Windows commands using the `win_command` module:
 Create the playbook
 ```bash
 vi win-command.yml
 ```
  Add the given content, by pressing `INSERT`
 ```yaml
 ---
 - name: test cmd from win_command module
   hosts: windows
   tasks:
     - name: run netstat and return Ethernet stats
       win_command: netstat -e
       register: netstat
     - debug: var=netstat
 ```
  save the file using `ESCAPE + :wq!`
   
 Now execute your Ansible playbook 
 
 ```bash
 ansible-playbook win-command.yml
 ```
