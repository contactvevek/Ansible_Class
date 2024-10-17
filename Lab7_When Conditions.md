### Task 1 : Using `When` Condition to install packages on `Ubuntu`

This task installs a list of packages on all managed nodes running Ubuntu

Create the playbook  `when_condition1.yml`
```
vi when_condition1.yml
```
Add the given content, by pressing "INSERT"
```yaml
- name: Installing list of packages
  hosts: all
  become: yes
  tasks:
  - name: install packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - tree
      - curl
      - wget
      - telnet
      - vsftpd
    when: ansible_facts['distribution'] == 'Ubuntu'

```
save the file using `ESCAPE + :wq!`

Now execute your Ansible playbook 
```bash
ansible-playbook when_condition1.yml
```
After running the playbook, check if the packages are installed on the managed nodes:
```bash
ansible all -b -m shell -a "dpkg -l | grep -E 'tree|curl|wget|telnet|vsftpd'"
```

### Task 2 : Using `When` Condition to install `apache2` 

This task will tests whether Apache is installed on the managed nodes and installs it if it is not.

Create the playbook  `when_condition2.yml`
```
vi when_condition2.yml
```
Add the given content, by pressing `INSERT`

```yaml
---
- name: Installing  Apache on the Ubuntu
  hosts: all
  become: yes
  tasks:
  - name: Gather package facts
    package_facts:
  - name: Install Apache if not already installed
    apt:
      name: apache2
      state: present
    when: "'apache2' not in ansible_facts.packages"
```
save the file using `ESCAPE + :wq!`

Now execute your Ansible playbook:

```bash
ansible-playbook when_condition2.yml
```
After running the playbook, check if the packages are installed on the managed nodes:
```bash
ansible all -b -m shell -a  "dpkg -l | grep apache2"
```

### Task 3 : Install packages using multiple `When` Conditions 

This task installs development tools based on the operating system type and checks if the system has more than 4GB of RAM.

Create the playbook  `when_condition3.yml`
```
vi when_condition3.yml
```
Add the given content, by pressing `INSERT`
```yaml
---
- name: Install development tools based on OS type and memory
  hosts: all
  become: yes
  tasks:

    - name: Gather memory facts
      setup:
        gather_subset:
          - hardware  # Ensure that memory facts are gathered

    - name: Install development tools on Ubuntu if RAM > 4GB
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - build-essential
        - git
        - vim
      when: ansible_facts['os_family'] == "Debian" and ansible_facts['memtotal_mb'] > 4096

    - name: Install development tools on Red Hat if RAM > 4GB
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - httpd
      when: ansible_facts['os_family'] == "RedHat" and ansible_facts['memtotal_mb'] > 4096
```
save the file using `ESCAPE + :wq!`

Now execute your Ansible playbook:

```bash
ansible-playbook when_condition3.yml
```
After running the playbook, check if the packages are installed on the managed nodes:
```bash
ansible all -b -m shell -a "dpkg -l | grep -E 'build-essential|git|vim'"
```
