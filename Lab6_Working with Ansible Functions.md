## Lab : Working with Ansible Functions

### Task 1: Loops with Ansible Playbook

1. Navigate home directory and create new directory there

    ```bash
    cd ~ && mkdir functions && cd functions
    ```

2. Create and edit `looplab.yml` to define the loop to add a list of users

    ```bash
    vi looplab.yml
    ```

3. Enter the contents of the file as shown below:

    ```yaml
    ---
    - hosts: all
      become: yes
      connection: ssh
      tasks:
        - name: Adding a number of users
          user:
            name: "{{ item }}"
            state: present
          with_items:
            - userX
            - userY
            - userZ
    ```
     **save the file using** `ESCAPE + :wq!`
   
5. Run the Ansible playbook

    ```bash
    ansible-playbook looplab.yml
    ```

6. Verify if the users mentioned in the list were added by using an Ansible ad-hoc command

    ```bash
    ansible all -a "tail -n 3 /etc/passwd"
    ```


### Task 2: Tags with Ansible Playbooks

1. Create and edit `tagslabs.yml` in the same directory

    ```bash
    vi tagslabs.yml
    ```

2. Enter the contents of the file as shown below

    ```yaml
    ---
    - hosts: all
      become: yes
      connection: ssh
      gather_facts: no
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
        - name: Start the vsftpd Service
          service:
            name: vsftpd
            state: started
          tags:
            - services

    ```
     **save the file using** `ESCAPE + :wq!`
3. Execute the playbook using tags. Notice that  the tasks associated with the mentioned tags is skipping and remaining task is running
    ```
    ansible-playbook --tags "packages" tagslabs.yml
    ```
4. Execute the playbook using tags. Notice that  the tasks associated with the mentioned tags is skipping and remaining tasks are running
    ```
    ansible-playbook --skip-tags "packages" tagslabs.yml
    ```

### Task 3: Prompts with Ansible Playbooks

1. Create and edit `promptlab.yml` in the same directory

    ```bash
    vi promptlab.yml
    ```

2. Enter the contents of the file as shown below:

    ```yaml
    ---
    - hosts: all
      become: yes
      connection: ssh
      vars_prompt:
        - name: pkginstall
          prompt: Which package do you want to install?
          default: telnet
          private: no
      tasks:
        - name: Install the package specified
          apt:
            pkg: "{{ pkginstall }}"
            state: latest
    ```
     **save the file using** `ESCAPE + :wq!`
   
4. When the playbook is run, the user will be presented with a prompt. The package entered will be installed

    ```bash
    ansible-playbook promptlab.yml
    ```

5. Verify if the specified package apache is installed. SSH into one of the machines and verify using the command shown below:

    ```bash
    ssh sirin_a@<managed_node_private_ip>
    ```
    - Replace the UserName(sirin_a) with yours.
    ```bash
    dpkg -l | grep apache
    ```

    Similarly, installation on the other client machine can be verified as well.


