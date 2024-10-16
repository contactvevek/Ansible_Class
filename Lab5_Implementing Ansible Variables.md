## Lab : Implementing Ansible Variables

In this lab, we will learn how to use variables in Ansible inside the playbook.

### Task 1: Configuring Packages in Ansible Using Variables

1. Create and change into the "file" directory inside the "labs" directory

    ```bash
    cd ~
    ```
    ```bash
    mkdir variables-lab && cd variables-lab
    ```
   
2. Create a new playbook for implementing variables

    ```bash
    vi implement-vars.yml
    ```

3. Add the following content to the `implement-vars.yml` file

    ```yaml
    ---
    - hosts: '{{ hostname }}'
      become: yes
      vars:
        hostname: all
        package1: apache2
        destination: /var/www/html/index.html
        source: index.html
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
          copy:
            src: '{{ source }}'
            dest: '{{ destination }}'
    ```
     **save the file using** `ESCAPE + :wq!`
   
5. Create a file called `index.html`**

    ```bash
    vi index.html
    ```
    ```html
    <html>
      <body style="background-color: white;">
        <h1 style="color: blue;">Welcome to CloudThat</h1>
        <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTpDuOE3FeeLNdUHy2ZXFIVapLOXZjRkn5UUA&s" 
             alt="Image">
      </body>
    </html>
    ```
     **save the file using** `ESCAPE + :wq!`
   
7. Run the playbook to see the implementation of variables**

    ```bash
    ansible-playbook implement-vars.yml
    ```

8. Use the public IP of any of the Ansible managed instances to view the webpage


### Task 2: Implementing Ansible Variables Using `extra-vars` Option

1. Create a file `index1.html` and copy the following contents into that file**

    ```bash
    vi index1.html
    ```

    ```html
    <html>
      <body style="background-color: white;">
        <h1 style="color: green;">This is the alternate Home Page</h1>
        <img src="https://content.cloudthat.com/consulting/wp-content/uploads/2023/01/03125800/Run-your-distributed-Applications-with-more-accuracy-at-a-boosted-speedd.png" 
         alt="CloudThat Image" style="width:500px; height:auto;">
      </body>
    </html>
    ```
     **save the file using** `ESCAPE + :wq!`
2. Implement the playbook using the following command

    ```bash
    ansible-playbook implement-vars.yml --extra-vars "source=/home/sirin_a/variables-lab/index1.html"
    ```
   - Ensure the absolute path of the new `index1.html` file is correct
     
3. Use the public IP of any of the Ansible managed instances to view the webpage


### Task 3: Configuring Variables as a Separate File and Implementing Ansible Playbook

1. Create a file `myvariables.yml` and add your variables

    ```bash
    vi myvariables.yml
    ```

    ```yaml
        ---
        hostname: all
        package1: apache2
        destination: /var/www/html/index.html
        source: /home/sirin_a/variables-lab/index2.html
    ```
     **save the file using** `ESCAPE + :wq!`
    
2. Create a file `index2.html` and copy the following contents into that file

    ```bash
    vi index2.html
    ```

    ```html
    <html>
      <body style="background-color: white;">
        <h1 style="color: blue;">Welcome to Ansible Training</h1>
        <img src="https://cdn-labgd.nitrocdn.com/DgsEbCQFApREClXUXMwcDAPWJfHtBIby/assets/images/optimized/rev-46e4eb1/content.cloudthat.com/consulting/wp-content/uploads/2024/03/13112158/Homepage-Banner-12-Anniversary.png" 
         alt="CloudThat 12th Anniversary" 
         style="width:500px; height:auto;">
      </body>
    </html>
    ```
     **save the file using** `ESCAPE + :wq!`
   
4. Modify the `implement-vars.yml` with the below-mentioned changes

    ```bash
    vi implement-vars.yml
    ```

    ```yaml
    ---
    - hosts: '{{ hostname }}'
      become: yes
      vars_files:
        - myvariables.yml
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
        - name: Copy required index.html to the document folder for httpd
          copy:
            src: '{{ source }}'
            dest: '{{ destination }}'
    ```
     **save the file using** `ESCAPE + :wq!`
   
6. Implement the Ansible playbook with the following command

    ```bash
    ansible-playbook implement-vars.yml
    ```

7. Use the public IP of any of the Ansible managed instances to view the webpage


