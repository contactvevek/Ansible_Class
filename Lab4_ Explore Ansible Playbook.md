## Lab : Exploring more on Ansible Playbooks
-----------------------------------------------------------------------------------
### Task 1: Creating a playbook and adding content to a file
1. Navigate to the directory
  ```bash
  cd /home/sirin_a/playbook-labs
  ```
2. Create a YAML playbook file called `putfile.yml`
    ```bash
    vi putfile.yml
    ```
    
    Add the given content, by pressing "INSERT" 
    
    ```yaml
    ---
    - hosts: all
      become: yes
      tasks:
        - name: Creating a new user cloudthat
          user:
            name: cloudthat
        - name: Creating a directory for the new user
          file:
            path: /home/cloudthat
            state: directory
    ```
  
  
    **save the file using** `ESCAPE + :wq!`
  
3. Now run the playbook as follow
    ```bash
    ansible-playbook putfile.yml
    ```
4. Now modify the playbook to add another child directory and a file inside cloudthat's home directory
    ```bash
    vi putfile.yml
    ```
    Add the given content, by pressing "INSERT" 
    ```yaml
    ---
    - hosts: all
      become: yes
      tasks:
        -   name: Creating a new user cloudthat
            user:
              name: cloudthat
        -   name: Creating a directory for the new user
            file:
              path: /home/cloudthat
              state: directory
        -   name: creating a folder named ansible
            file:
              path: /home/cloudthat/ansible
              state: directory
        -   name: creating a file within the folder ansible
            file:
              path: /home/cloudthat/ansible/hello.txt
              state: touch
        -   name: Changing owner and group with permission for the file within the folder named ansible
            file:
              path: /home/cloudthat/ansible/hello.txt
              owner: root
              group: cloudthat
              mode: 0665
        -   name: adding content to the file created named hello.txt
            lineinfile:
              path: /home/cloudthat/ansible/hello.txt
              line: "This is line 1"
    
    ```
    **save the file using** `ESCAPE + :wq!`
  
  
5. We can now run the playbook step-by-step to observe what happens at each stage:
    ```bash
    ansible-playbook putfile.yml --step
    ```
6. Verify the Content of the File are added in `hello.txt`
    ```bash
    ansible all -a "sudo cat /home/cloudthat/ansible/hello.txt"
    ```
    
7. Verify if the permissions / ownership is as per the playbook
    ```bash
    ansible all -a "sudo ls -l /home/cloudthat/ansible/"
    ```
