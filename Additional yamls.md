
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


