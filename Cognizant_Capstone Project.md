
## Lab 1: Installation and Configuration of Ansible

### Task 1: Launching an Instance in GCP

1. Go to the GCP Console and select Compute Engine.
2. Select VM Instances and click on CREATE INSTANCE.
3. Enter the name of the instance `ansible-server` and select the Zone in which the instance is going to launch.
4. Select machine type. In the boot disk section, click on Change and select Ubuntu 24.04.
5. Click on create and launch the instance.
6. SSH into the machine

### Task 2: Execute the Following Commands for Installing the Ansible Server

1. Check the IP is changed to “ansible-workstation” hostname and run the following command to update:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. Install Python 3.9, python3-pip, and wget which are compatible with Ansible using the below command:
    ```bash
    sudo apt install python3 python3-pip wget -y
    ```
#### Install via `apt` package manager
1. Use the following command to add the Ansible PPA:
   ```bash
   sudo apt-add-repository --yes --update ppa:ansible/ansible
   ```

2. Now install Ansible by using the following command:
   ```bash
   sudo apt install ansible -y
   ```

3. Check the version of Ansible using the below command:
   ```bash
   ansible --version
   ```

### Task 3: Make Entry in the Host File
Create `Ansible directory`  by running the following command:
```bash
sudo mkdir /etc/ansible
```
To run all the commands on the localhost as well, make the following entry:

```bash
sudo vi /etc/ansible/hosts
```

Add the `private ip` of the managed nodes to the `ansible_hosts`
```
localhost ansible_connection=local
```
**save the file using** `ESCAPE + :wq!`

In real life situations, one of the managed node may be used as the ansible control node.
In such cases, we can make it a managed node, by adding localhost in hosts inventory file.

![image](https://github.com/user-attachments/assets/5dee6a8c-f016-40c2-a8a2-75101c36ff70)


For enabling `PasswordLess authentication` follow the below steps:

On the ansible-server
```bash
ssh-keygen -t rsa -b 2048
```
Verify the  SSH key pair on your local machine:
```bash
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```
Copy the public key to Managed Nodes:
```bash
cat ~/.ssh/id_rsa.pub
```
Log into the each remote server (managed Nodes) via another method (e.g., console access).
Add the public key to `~/.ssh/authorized_keys` on the remote servers:
```bash
echo "your_public_key_content" >> ~/.ssh/authorized_keys
```

Set the below permissions On the remote servers:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
```bash
exit
```
Execute the below command on `ansible-server` to verify passwordless authentication has been enabled
```
ssh Your_User@PUBLIC_IP of managed-node1
```
```
ssh Your_User@PUBLIC_IP of managed-node2
```
```ansible
ansible all -m ping
```


.
