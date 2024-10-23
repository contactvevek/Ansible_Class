## Lab : Implementing Jinja2 Templates

#### Task 1 : Jinja2 Template for System Information

1. **Navigate Home Directory and create a directory for  `Jinja2 templates`**
    ```bash
    cd ~ && mkdir templates && cd templates
    ```
2. **In that directory create a template file, let’s say `new-motd.j2`**
    ```bash
    vi new-motd.j2
    ```
3.  **Add the below code to the jinja2 template**
    ```
    System Information:
    Operating System: {{ ansible_os_family }}
    Architecture: {{ ansible_architecture }}
    Distribution Version: {{ ansible_distribution }}
    ```
      **save the file using** `ESCAPE + :wq!`
      - This template will dynamically display system information such as OS, Architecture, and Distribution version.

4. **Now create the playbook that will use the Jinja2 template to render the `/etc/motd` file on the managed nodes.**
    ```
    vi implement-jinja2.yml
    ```
      **Add the below code to the newly created playbook:**
    ```yaml
    ---
    - name: playbook
      hosts: all
      gather_facts: true
      become: yes
      tasks:
        - name: Render template
          template:
            src: new-motd.j2
            dest: /etc/motd
            owner: sirin_a
            group: sirin_a
            mode: 0644
    ```
      **save the file using** `ESCAPE + :wq!`
5. **Now execute the playbook**
      ```bash
      ansible-playbook implement-jinja2.yml  
      ```
   
6. **Now we will check what our template is exactly doing, where it is putting the values.**
7. **For that, do the following:**
      ```bash 
      ansible all -m shell -a "cat /etc/motd"
      ```
     ***Alternatively,* you can SSH into a managed node and check "welcome Message" which trigered from "/etc/motd"**
      ```bash
      ssh sirin_a@<Managed-Node-IP>
      ```
       - Replace "your-username (sirin_a)" with your actual username.

      ```
      exit
      ```
#### Task 2 : Jinja2 Template for Dynamic HTML Web Page
Using a Jinja2 template to dynamically render the server's IP address on the web page
  
1. **Check inventory file for the managed nodes IP's**
      ```bash
      cat /etc/ansible/hosts
      ```
2. **Create a HTML file called `index.html.j2`**
      ```bash
      vi index.html.j2
      ```
      Add the given content, by pressing `INSERT`
      ```html
        <html>
          <body style="background-color: white;">
            <h1 style="color: blue;">Welcome to CloudThat</h1>
            <h3>My IPv4 Address is {{ ansible_host }}</h3>
          </body>
        </html>
      ```
      save the file using `ESCAPE + :wq!`
      * This will display the server’s IP address dynamically on the webpage.
     
3. **Create a new Ansible playbook to install Apache and deploy the HTML page:**
      ```bash
      vi Install-apache.yaml
      ```
      Add the given content, by pressing `INSERT`
      ```yaml
        ---
        - hosts: '{{ hostname }}'
          become: yes
          vars:
            hostname: all
            package1: apache2
            destination: /var/www/html/index.html
            source: index.html.j2
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
            - name: Copy required index.html to the document folder for apache2
              template:
                src: '{{ source }}'
                dest: '{{ destination }}'
      ```
      save the file using `ESCAPE + :wq!`
5. **Run the Ansible Playbook**
      ```bash
      ansible-playbook Install-apache.yaml
      ```
6. Copy paste the public IP  of the Ansible managed instances in the browser to view the webpage
      * http://"Managed-Node-1-Public-IP"
      * http://"Managed-Node-1-Public-IP" 
