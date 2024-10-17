
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
