
### Task 1: Installing Packages Using Ansible Loops
Create a new playbook file called `install_packages.yml`

```yaml
---
- name: This is Loop Lab
  become: yes
  hosts: all
  tasks:
  - name: Installing all defined packages
    apt: 
      name: "{{ item }}"
      state: present
    loop:
    - wget
    - curl
    - tree
    - vsftpd 
```
Copy and paste the content in your playbook:

Run the playbook using the following command:
```bash
ansible-playbook install_packages.yml
```
#### Task 2: Creating Users with Specific UIDs Using Ansible
Create a playbook file called `create_users.yml`
```yaml
---
- hosts: all
  become: yes
  connection: ssh
  tasks:
    - name: Adding a number of users
      user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        state: present
      with_items:
        - { name: "userX", uid: 2000}
        - { name: "userY", uid: 3000}
        - { name: "userZ", uid: 4000}
```
Copy and paste the content in your playbook:

Run the playbook using the following command:
```bash
ansible-playbook create_users.yml
```
Verify the created users 
```bash
ansible all -m command -a "tail /etc/passwd" -b
```
