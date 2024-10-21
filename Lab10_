## Lab : Implementing Ansible Roles
1. Navigate to your home directory
   
    ```bash
    cd ~/
    ```
2. Lets uninstall apche2. After that, we will use ansible role to install it.
    ```bash
    ansible-playbook ~/playbook-lab/uninstall-apache-pb.yml 
    ```
  - Ensure the correct path of the `yaml`
    
3. Install tree. A `tree` is a recursive directory listing program that produces a depth-indented listing of files. 
    ```bash
    sudo apt install tree -y
    ```
  
4. You can view your home directory structure in tree format with below command tree 
    ```bash
    tree ~/playbook-lab
    ```
5. Navigate back to your home directory
   ```bash
    cd ~
    ```
6. Create a roles directory and navigate into it
    ```bash
    mkdir roles && cd roles
    ```
7. Now inside the roles directory, create a different directory for roles, namely webrole and navigate it. 
  
    ```
    mkdir webrole && cd webrole/
    ```
8. Inside `webrole` create `tasks` and `file' for roles directory structure
    ```
    mkdir files tasks && cd files/
    ```
9. Create a simple `index.html` file in the `files` directory
    ```
    vi index.html
    ```
    Add the given content, by pressing `INSERT`
    ```
      <html>
        <body style="background-color: white;">
          <h1 style="color: blue;">We are performing the Roles Lab</h1>
          <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTpDuOE3FeeLNdUHy2ZXFIVapLOXZjRkn5UUA&s" 
               alt="Image">
        </body>
      </html>
    ```
    **save the file using** `ESCAPE + :wq!`
    
10. Then go to the `tasks` directory as below 
    ```
    cd .. && cd tasks/
    ```
11. create `main.yml` file to install web packages.
    ```
    vi main.yml
    ```
    Add the given content, by pressing `INSERT`
    ```
    ---
    - name: install apache2
      apt: 
        name: apache2 
        update_cache: yes 
        state: latest
    
    - name: uploading default index.html for host
      copy:
        src: /home/sirin_a/roles/webrole/files/index.html
        dest: /var/www/html
    
    - name: Setting up attributes for file
      file:
        path:  /var/www/html/index.html
        owner: sirin_a
        group: sirin_a
        mode:  0644
    
    - name: start apache2 service
      service:
        name: apache2 
        state: started
    ```
    **save the file using** `ESCAPE + :wq!`
    - Ensure 
    
13. After the creation of this file, we are done with the complete hierarchy of roles, so we will have a look at how it is exactly using tree command
    ```
    cd ../..
    ```
    ![image](https://github.com/user-attachments/assets/5d47a1da-0a16-4611-94f8-1eac467c9ad1)

14. Now create `dbrole` directory in the `roles` directory and navigate it
    ```
    mkdir dbrole && cd dbrole
    ```
15. Inside `dbrole` create `tasks` for roles directory structure
    ```
    mkdir tasks && cd tasks
    ```

16. Then create `main.yaml` file to install database packages.
    ```
    vi main.yaml
    ```
     Add the given content, by pressing `INSERT`
    ```
    ---
    - name: install mariadb-server
      apt:
        name: mariadb-server
        update_cache: yes
        state: latest
    - name: start mariadb
      service:
        name: mariadb
        state: started
    
    ```
     **save the file using** `ESCAPE + :wq!`

17. Navigate back to the parent directory
    ```
    cd ../..
    ```
18. View the directory structure using tree
    ```
    tree
    ```
    ![image](https://github.com/user-attachments/assets/758eb951-cf4a-4c94-8b00-eda4a8d2a846)

19. Now create the playbook as `implement-roles.yml`
    ```
    vi implement-roles.yml
    ```
     Add the given content, by pressing "INSERT".
    ```
    ---
     - hosts: all
       become: yes 
    
       roles:
         - webrole
         - dbrole
    ```   
     **save the file using** `ESCAPE + :wq!`
20. Run tree command in the parent directory `role` to view complete directory structure of it
     ```
    tree
    ```
     ![image](https://github.com/user-attachments/assets/fd67b410-73a5-41d6-b54d-2367d7a9eeac)

21. Execute the playbook
    ```
    ansible-playbook implement-roles.yml
    ```
22. Check the home page on browser. (Public DNS)
    It will show the webpage with msg "We are performing the Roles Lab"
