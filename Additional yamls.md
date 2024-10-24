
### Play level tagging
```yaml
---
- name: This is my First Play
  hosts: all
  become: yes
  connection: ssh
  gather_facts: no
  tags:
  - play1
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600  # Cache validity time in seconds
      tags:
        - apt-update

    - name: Install vsftpd
      apt:
        name: vsftpd
        state: latest
      tags:
        - packages
        - software
    - name: Start the vsftpd Service
      service:
        name: vsftpd
        state: started
      tags:
        - services
        - software
- name: This is my Second Play
  hosts: all
  become: yes
  tags:
  - play2
  tasks:
  - name: Installing list of packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
    - wget
    - tree
    - vim
    - git
```
```bash
ansible-playbook --tags "play2" tagslabs.yml
```
### Do Until
```yaml
---
- hosts: all
  tasks:
  - name: Check if the file is created
    stat:
      path: /tmp/somefile.txt
    register: file_status
    until: file_status.stat.exists
    retries: 5
    delay: 10
```

```bash
ansible-playbook file.yaml
```

![image](https://github.com/user-attachments/assets/e608b8ec-304f-4b99-b6d5-4ea4c01e78d1)

### any_errors_fatal 
If any task fails, the entire playbook execution will stop.
**Without any_errors_fatal**
By default, Ansible will continue running tasks on other hosts if one host encounters an error. For instance, if node-1 encounters an error during task execution, Ansible will still continue the playbook on node-2, node-3, etc.

![image](https://github.com/user-attachments/assets/1970dcfd-cedd-44b8-9e39-162cefd84a22)

```bash
vi any_errors_fatal.yaml
```

```yaml
---
  - hosts: '{{ hostname }}'
    become: yes
    any_errors_fatal: yes
    vars:
      hostname: all
      package1: apache2
    tasks:
      - name: Install defined package
        apt:
          name: '{{ package1 }}'
          update_cache: yes
          state: latest
      - name: Start desired service
        service:
          name: '{{ package1 }}'
          state: started
```
```bash
ansible-playbook any_errors_fatal.yaml
```
**With any_errors_fatal: yes**
When any_errors_fatal is set to yes, if one host (e.g., node-1) fails during task execution, all other hosts will immediately stop executing their tasks. The playbook will stop running across all hosts in the inventory.

![image](https://github.com/user-attachments/assets/a5be877b-b54a-410b-a8e4-ba908bf9edee)

