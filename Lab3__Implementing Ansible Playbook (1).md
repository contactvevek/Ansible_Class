## Lab : Implementing Ansible Playbook

In this lab, we will be creating our first playbook to install and configure the Apache web server and will execute the playbook.

### Task 1: Playbook to install apache web server

1. **Create a new directory `playbook-labs` in the `/home/sirin_a` directory**

    ```bash
    cd ~
    ```
    ```sh
     mkdir playbook-lab && cd playbook-lab
    ```

2. **Create a new file `install-apache-pb.yml` (the playbook)**

    ```sh
     vi install-apache-pb.yml
    ```

3. **Add the below code to the newly created playbook:**

    ```yaml
    ---
    - name: This play will install apache web servers on all the hosts
      hosts: all
      become: yes
      tasks:
        - name: Task1 will install apache2 using apt
          apt:
            name: apache2
            update_cache: yes
            state: latest
        - name: Task2 will upload custom index.html into all hosts
          copy:
            src: "index.html"
            dest: "/var/www/html/index.html"
        - name: Task3 will setup attributes for file
          file:
            path: /var/www/html/index.html
            owner: www-data
            group: www-data
            mode: 0644
        - name: Task4 will start apache2
          service:
            name: apache2
            state: started
    ```

    **save the file using** `ESCAPE + :wq!`

4. **Create a simple `index.html` file in the same `labs` directory**

    ```bash
     vi index.html
    ```

5. **Add the below code to the `index.html` file**

    ```html
        <html>
          <body>
            <h1>Welcome to CloudThat</h1>
            <img src="https://cdn-labgd.nitrocdn.com/DgsEbCQFApREClXUXMwcDAPWJfHtBIby/assets/images/optimized/rev-46e4eb1/content.cloudthat.com/consulting/wp-content/uploads/2024/03/13112158/Homepage-Banner-12-Anniversary.png" 
                 alt="CloudThat 12th Anniversary" 
                 style="width:500px; height:auto;">
          </body>
        </html>

    ```
    **save the file using** `ESCAPE + :wq!`
6. **Execute the playbook using the `ansible-playbook` command**

    ```bash
    ansible-playbook install-apache-pb.yml
    ```

7. **Verify the setup by doing a curl on both managed nodes to check the `index.html`**

    ```bash
    curl <private_ip_of_node1>
    curl <private_ip_of_node2>
    ```
---------------------------------------------------------------------------------------
### Task 2: Uninstall apache web server

1. With slight modification, we can change the playbook to uninstall apache (self exercise)
    ```bash
    cp install-apache-pb.yml uninstall-apache-pb.yml
    ```
2. Edit the file to uninstall package
    ```bash
    vi uninstall-apache-pb.yml
    ```
    Retain only first task. Replace `state: latest` with `state: absent` and
    **save the file using** `ESCAPE + :wq!`
3. Check if the playbook is ok
    ```bash
    ansible-playbook uninstall-apache-pb.yml --check
    ```
4. Now run the playbook and check in the browser if the web page exists.
    ```bash
    ansible-playbook uninstall-apache-pb.yml
    ```
    
5. Check the browser with the corresponding IPv4 DNS name and ensure webpage is removed.
