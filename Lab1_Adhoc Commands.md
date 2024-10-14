
## Lab 2: Exploring Adhoc Commands

In this lab, you will explore some common ad-hoc commands starting from memory details to copying files from one host to another.

Launch 2 more Ubuntu VM `managed-node1` and `managed-node2` and add ssh key to it.

### Step 1: Make Entry in the Host File
Create `Ansible directory`  by running the following command:
```bash
sudo mkdir /etc/ansible
```
To run all the commands on the localhost as well, make the following entry:

```bash
sudo vi /etc/ansible/hosts
```

Add the `private ip` of the managed nodes to the `ansible_hosts`
```ini
node1   ansible_host=<node1-private-ip>  
node2   ansible_host=<node2-private-ip> 
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


### Step 2: Get Memory Details
Run the following command to get the memory details:
```
ansible all -m command -a "free -h"
```

### Step 3: Create a New User on All Hosts
Use the below command to create a new user on all the hosts:
```
ansible all -m user -a "name=ansible-new home=/home/ansible-new" --become
```

### Step 4: verify the created users
Run the following command to verify:
```
ansible all -m command -a "tail /etc/passwd"
```

### Step 5: Create a File Inside the Directory
Use the following command to create a file inside the home directory of `ansible-new` user in one of the node:
```
ansible node2 -m file -a "dest=/home/ansible-new/demo.txt mode=600 state=touch" --become
```

### Step 6: Add Content to the File
Run the following command to add content to the file:
```
ansible nodde2 -b -m lineinfile -a 'dest=/home/ansible-new/demo.txt line="This server is managed by Ansible"'
```

### Step 7: SSH to the Host to Check the File
Use the below command to SSH to the host where the file was created and check if the file is created or not. Enter exit to exit from the managed node:
```
ssh your-username@Private_IP_node2
```
* Replace "your-username" with your actual username.
```
sudo cat /home/ansible-new/demo.txt
```

### Step 8: Exit Back to the Control Node and Create a File
Exit back to the control node and create a file using touch:
```
exit
```
```
touch test.txt
```

### Step 9: Copy the Test File to the Directory
Use the following command to copy the test file to the directory that we created:
```
ansible node2 -m copy -a "src=test.txt dest=/home/ansible-new/test" --become
```


