

## Task 1 : Filter Plugin:
* Filter plugins in Ansible allow you to manipulate and transform data directly within playbooks. 
* Filters can be applied to variables, facts, or other data structures, offering a flexible way to process and format data on the fly. 
* Filters are invoked using the | (pipe) symbol, which is followed by the filter name and any optional arguments.
* Filters are used in various cases, including but not limited to string manipulation, data formatting, sorting, and filtering items in lists or dictionaries.

1. Create a playbook `filter.yaml` 
   ```bash
   vi filter.yaml
   ```
   Add the below code in it
   ```yaml
   ---
   - hosts: all
     become: yes
     tasks:
     - name: Convert message to uppercase
       debug:
          msg: "{{ 'hello world' | upper }}"
   ```
   save the file using `ESCAPE + :wq!`
2. Run the Playbook
   ```bash
   ansible-playbook filter.yaml
   ```
3. After running the playbook, you should see the string "hello world" is successfully converted to "HELLO WORLD" and displayed using the debug module.

   
## Task 2 : Lookup Plugin:

* Lookup plugins are used to fetch data from external sources such as files, environment variables, or cloud services, and they are commonly used to dynamically populate values in playbooks.
  
1. Create the file `/tmp/test.txt`
   ```bash
   echo "This is a test file for Ansible lookup plugin." > /tmp/test.txt
   ```
2. Create a playbook `lookup.yml` to reads the contents of the file `/tmp/example.txt` using the `lookup plugin`.
   ```bash
   vi lookup.yml
   ```
   Add the below code in it
   ```yaml
   ---
   - name: Read content from a file using lookup plugin
     hosts: localhost
     tasks:
       - name: Fetch content from file
         debug:
           msg: "{{ lookup('file', '/tmp/example.txt') }}"

   ```
   save the file using `ESCAPE + :wq!`
3. Run the Playbook
   ```bash
   ansible-playbook lookup.yml
   ```
4. After running the playbook, you should see the content of the `/tmp/test.txt` file
   
## Task 3 : Working With Ansible Callback Plugins:
**Callback Plugins**

* Callback plugins in Ansible allow you to customize the output of playbook runs and add notifications based on playbook events. 
* They can be used to enhance the user experience by providing real-time feedback and can also facilitate integration with external systems for logging, monitoring, or alerting.

1. Create a Working Directory
   Start by setting up a directory for your project. This will contain your playbook, callback plugin, and files.
    ```bash
    mkdir plugins && cd plugins
    ```
2. Create a new file `plugin.yml` (the playbook)
    ```bash
    vi plugin.yml
    ```
    Add the below code to the playbook
   
    ```yaml
    - name: Installing Packages on Managed Node
      become: yes
      hosts: all
      tasks:
      - apt:
          name: httpd
          state: present
    ```
    save the file using `ESCAPE + :wq!`
    - Note that `httpd` will not work on your `Ubuntu managed node`. We are intentionally using it for testing purposes to see if the `callback plugin` correctly captures the task failure.
4. Create a Directory for Callback Plugins
    ```bash
    mkdir callback_plugins && cd callback_plugins
    ```
5. Now, create your custom callback plugin script: 
    ```bash
    vi callback_plugin.py
    ```
    Add the below code in it
    ```py
    from ansible.plugins.callback import CallbackBase
   
   class CallbackModule(CallbackBase):
       CALLBACK_VERSION = 2.0
       CALLBACK_TYPE = 'notification'
       CALLBACK_NAME = 'custom_callback'
   
       def v2_playbook_on_start(self, playbook):
           self._display.display("My Playbook is starting...")
   
       def v2_playbook_on_stats(self, stats):
           self._display.display("My Playbook is finished...")
   
       # This method is triggered when a task fails
       def v2_runner_on_failed(self, result, ignore_errors=False):
           host = result._host  # Get the name of the host where the task failed
           task_name = result.task_name  # Get the name of the failed task
           error_message = result._result.get('msg', 'No error message')  # Capture the error message
           
           # Display the failure message
           self._display.display(f"Task '{task_name}' failed on host '{host.name}' with error: {error_message}")
   
           # You could also send this message to external services like Slack, a logging system, etc.
    ```
    save the file using `ESCAPE + :wq!`

6. Update Ansible Configuration `(ansible.cfg)`   
    ```bash
    vi ~/.ansible.cfg
    ```
    Add the following lines to the configuration file to specify the callback plugin path
    ```bash
    [defaults]
    callback_plugins = ~/plugins
    ```
    - This tells Ansible to look in the ~/plugins/callback_plugins directory for custom callback plugins.
    - If the `[defaults]` section already exists, simply add `callback_plugins = ~/plugins/callback_plugins` below it.
7. Now, return to `plugins` directory
    ```bash
    cd ..
    ```
8. Once done with all the above steps, you run tree command to get below the directory structure.

     ![image](https://github.com/user-attachments/assets/87af4eb2-f8c5-4cc4-b3a5-da626511f73b)

   
9. Run the Playbook
    ```bash
    ansible-playbook plugin.yml
    ```
10. Verify the Callback Output
   When you run the playbook, you should see the custom messages from your callback plugin and
   By using the `v2_runner_on_failed` method in callback plugins, you can easily catch task failures during playbook execution.
   This allows you to provide detailed error messages or trigger alerts when something goes wrong, improving the overall reliability and transparency of your Ansible automation tasks.:

      ![image](https://github.com/user-attachments/assets/5256e288-db84-48b8-ab6d-cb53dc68f436)



*Reference Links:*

https://docs.ansible.com/ansible/latest/plugins/callback.html

https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins/callback
