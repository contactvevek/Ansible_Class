## Troubleshooting and Debugging in Ansible

### Task 1 : Blocks with Ansible Playbook

In Ansible, the block and rescue keywords are used to handle tasks more gracefully, allowing you to define how certain task failures are managed. 
Structure of block and rescue
- block - Contains tasks that will be executed.
- rescue - Contains tasks that will be executed if any task in the block fails.
- always (optional) - Contains tasks that will always be executed, whether the block succeeded or failed.

Create and edit `blklab.yml` in the `error-labs` directory. Notice that the `web_package` variable is an invalid package. 
Due to the invalid package in a block, tasks under rescue will run

```bash
cd ~ && mkdir error-labs && cd error-labs
```
Create a playbook file named `blklab.yml`

```bash
vi blklab.yml
```
Add the given content, by pressing `INSERT`

```yaml
---
  - hosts: all
    become: yes
    user: sirin_a
    connection: ssh
    gather_facts: no
    tasks:
    - block:
      - name: Install {{ web_package }} package
        apt:
          name: "{{ web_package }}"
          state: latest
      rescue:
      - name: Install {{ db_package }} package
        apt:
          name: "{{ db_package }}"
          state: latest
      always:
      - name: Start {{ db_service }} service
        service:
          name: "{{ db_service }}"
          state: started
    vars:
      web_package: apche2
      db_package: mariadb-server
      db_service: mariadb
 ```
save the file using `ESCAPE + :wq!`

Execute the playbook, Block tasks fail and that Rescue tasks is running due to the failure of block tasks. The Always tasks run independently

```bash
ansible-playbook blklab.yml
```

![image](https://github.com/user-attachments/assets/007f3737-40f2-4db9-ad25-71d697ca92ce)


Now fix the package name in the Playbook (web_package: apache2) and run the Playbook again

```bash
ansible-playbook blklab.yml
```

Notice that the tasks under rescue are not running as block tasks ran successfully.
![image](https://github.com/user-attachments/assets/80dfff79-fd7f-4636-b97a-ed190599ca28)


### Task 2 : Ignoring Failed Commands

In Ansible, by default, when a task fails, the execution of the playbook is halted for that host. 

However, you can override this behavior using the `ignore_errors: yes` directive to allow the playbook to continue executing subsequent tasks, even if a failure occurs in one of the tasks.

Create a playbook file named `ignore_errors.yml`
```bash
vi ignore_errors.yml
```

Add the given content, by pressing `INSERT`

```yaml
---
- hosts: all
  become: true

  tasks:
  - name: Install security updates
    apt:
     name: "{{ item }}"
     state: latest
    loop:
    - openssl
    - openssh
    ignore_errors: yes 

  - name: Check if docker is installed
    command: docker --version
    register: output
    ignore_errors: yes    

  - name: Debug output of the docker version check
    debug:
     var: output

  - name: Install docker
    apt:
     name: docker.io
     state: present
    when: output.failed  # Correcting syntax to check if the command failed

```
save the file using `ESCAPE + :wq!`

Execute the playbook using the ansible-playbook command
```bash
ansible-playbook ignore_errors.yml
```
Monitor the output of the playbook execution. You should see the following:
- The playbook will attempt to install security updates. If any updates fail, it will continue due to the ignore_errors: yes directive.
- It will check if Docker is installed, and if the command fails, it will log the failure without stopping the execution.
- The debug task will show the output of the Docker version check, including any error messages.
- Finally, if Docker was not installed, it will proceed to install Docker.

### Task 3 : Ansible Notify and Handlers 

In Ansible, The run_once directive ensures that a task is executed only once, regardless of how many hosts are in the play.

Create a playbook file named `notify-handlers.yaml`
```bash
vi notify-handlers.yaml
```
Add the given content, by pressing `INSERT`

```yaml
---
- name: Install and configure Apache Web Server
  hosts: all
  become: yes
  tasks:
  - name: Install Apache
    apt:
     name: apache2
     state: present
    notify: Restart Apache

  - name: Deploy index.html
    copy:
     content: "<html><h1>Welcome to node {{ ansible_hostname }}</h1></html>"
     dest: var/www/html/index.html
    notify: Restart Apache

  handlers:
  - name: Restart Apache
    service:
     name: apache2
     state: restarted
```
save the file using `ESCAPE + :wq!`
Execute the playbook using the ansible-playbook command
```bash
ansible-playbook notify-handlers.yaml
```
### Task 4 : run_once


Create a playbook file named `run_once.yaml`
```bash
vi run_once.yaml
```
Add the given content, by pressing `INSERT`

```yaml
---
- hosts: all
  become: yes
  vars:
    file_url: "https://download.docker.com/mac/static/stable/x86_64/docker-27.3.1.tgz" # URL for the file to download
    temp_path: "/tmp/docker.tgz"  # Path for the downloaded file
    dest_path: "/etc/docker/docker.tgz"  # Destination path on all servers
    download_host: "node-1"  # Specify the host for the run_once task

  tasks:
    - name: Ensure destination directory exists on all servers
      file:
        path: "/etc/docker"
        state: directory
        mode: '0755'

    - name: Download Docker binary on a specific server
      get_url:
        url: "{{ file_url }}"
        dest: "{{ temp_path }}"
      run_once: yes
      delegate_to: "{{ download_host }}"  # Run only on the defined host

    - name: Distribute Docker binary to all servers
      copy:
        src: "{{ temp_path }}"
        dest: "{{ dest_path }}"
        mode: '0644'
```
save the file using `ESCAPE + :wq!`

Execute the playbook using the ansible-playbook command
```bash
ansible-playbook run_once.yaml
```

### Task 5 : Ansible Playbook Execution and Validation

**Running the Playbook in Dry-Run Mode**

   ```bash
   ansible-playbook install-apache-pb.yml --check
   ```
   - This command runs the playbook in check mode (also called dry-run mode). 
   It simulates what would happen if the playbook were run, but makes no actual changes to the target systems.

**Checking the Syntax of the Playbook**

```bash
ansible-playbook install-apache-pb.yml --syntax-check
```
- This command performs a syntax check to ensure that the YAML structure and Ansible syntax are valid before executing the playbook.

**Running the Playbook with Different Verbosity Levels**

```bash
ansible-playbook install-apache-pb.yml -v
```

```bash
ansible-playbook install-apache-pb.yml -vv
```

```bash
ansible-playbook install-apache-pb.yml -vvv
```
```bash
ansible-playbook install-apache-pb.yml -vvvv
``` 
- -v: Shows which tasks are running and their results (changed, ok, failed).
- -vv: Shows additional details about the task input/output.
- -vvv: Includes even more granular detail about what Ansible is doing.
- -vvvv: Displays full debug information, useful for troubleshooting SSH connections or other deep issues.

**Limiting the Playbook to Specific Hosts**

```bash
ansible-playbook install-apache-pb.yml --limit node1
```
- Runs the playbook on a specific host or group of hosts (node1 in this case) as defined in the inventory.


**Showing Task Differences**

```bash
ansible-playbook install-apache-pb.yml --diff
```
- Shows the differences between the current state and the desired state for tasks that support it (e.g., file, template tasks).

**Listing All Tasks Without Execution**

```bash
ansible-playbook install-apache-pb.yml --list-tasks
```
- This command lists all tasks in the playbook without executing them.

**Running Playbook in Interactive Step-by-Step Mode**

```bash
ansible-playbook install-apache-pb.yml --step
```
- This command prompts for user confirmation before running each task in the playbook.




