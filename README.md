# ansible-war-deploy

## A playbook to deploy a WAR on Tomcat7.

This project will have the Ansible playbook I use to deploy a war to Tomcat7.

The playbook will have some specific configuration that I used to use.

I'm sharing this playbook for many reasons:

* As this is my 1st production oriented playbook I thought that my commits history will maybe help someone else understand the basics of playbooks in an other way.
* Was looking for a playbook that do what this playbook did, but failed to find that, so maybe my playbook will help someone else creating his own playbook to deploy a war.
* I recommand that you check the commits changes to find why I use that task or so.. As I'm still leearning Ansible, I'll try to keep the commits changes minor & comments clear.

# Key Concepts
  - **Playbooks:** YAML files where Ansible code is written.
  - **Modules:** Predefined units of work that Ansible executes.
  - **Inventories:** Lists of nodes or hosts to be managed.
  - **Roles:** A way to group tasks, variables, files, templates, and modules in a structured manner.
  - **Tasks:** Actions executed by Ansible on the managed nodes.
  - **Handlers:** Special tasks that are run only when notified.
## Basic Components
  - Inventory
    - The inventory file lists the hosts and groups of hosts that Ansible will manage. It can be a simple text file or a more complex dynamic inventory script.

Example of a simple inventory file (hosts):
```
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com
```
  - Playbooks
    - Playbooks define the tasks to be executed on the managed hosts.

Example of a simple playbook (site.yml):
```
- hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true

- hosts: dbservers
  become: yes
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present

    - name: Start MySQL
      service:
        name: mysql
        state: started
        enabled: true
```
  - Roles
    - Roles allow you to organize playbooks into reusable components.

Directory structure for roles:
```
site.yml
roles/
  common/
    tasks/
    handlers/
    templates/
    files/
    vars/
    defaults/
    meta/
```
Example of a role (roles/common/tasks/main.yml):
```
- name: Ensure NTP is installed
  apt:
    name: ntp
    state: present

- name: Ensure NTP is running
  service:
    name: ntp
    state: started
    enabled: true
```
Running Ansible
To execute a playbook, use the ansible-playbook command:
```
ansible-playbook -i hosts site.yml
```
  - Advanced Features
    - Variables
      - Variables can be defined in playbooks, roles, or inventory files and used to customize behavior.

Example of using variables:
```
- hosts: webservers
  become: yes
  vars:
    nginx_port: 8080
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Configure Nginx to listen on custom port
      lineinfile:
        path: /etc/nginx/sites-available/default
        regexp: 'listen 80 default_server;'
        line: 'listen {{ nginx_port }} default_server;'

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true
```
  - Handlers
    - Handlers are tasks that run only when notified. They are typically used for restarting services after configuration changes.

Example of using handlers:
```
- hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Configure Nginx
      lineinfile:
        path: /etc/nginx/sites-available/default
        regexp: 'listen 80 default_server;'
        line: 'listen 8080 default_server;'
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```
  - Templates
    - Ansible uses Jinja2 templating to enable dynamic file creation.

Example of using templates:

Template file (nginx.conf.j2):

```
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Playbook using the template:
```
- hosts: webservers
  become: yes
  vars:
    nginx_port: 8080
    server_name: "example.com"
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Deploy Nginx configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

# How would you handle a situation where a task needs to be executed conditionally based on the outcome of a previous task?

Answer: Use the when clause in Ansible to conditionally execute tasks based on the outcome of previous tasks or the value of variables. You can also use register to capture the result of a task and use it in subsequent conditions.
Example:
```
yaml
Copy code
- name: Check if a file exists
  stat:
    path: /path/to/file
  register: file_stat

- name: Perform action if file exists
  command: /path/to/command
  when: file_stat.stat.exists
```
# Structure of an Ansible Role
Ansible roles have a standardized directory structure:
```
roles/
  myrole/
    tasks/
      main.yml
    handlers/
      main.yml
    templates/
    files/
    vars/
      main.yml
    defaults/
      main.yml
    meta/
      main.yml
```
**tasks/:** Contains the main list of tasks to be executed by the role.
**handlers/:** Contains handlers, which are tasks that are triggered by other tasks.
**templates/:** Contains Jinja2 templates that can be dynamically populated with variables.
**files/:** Contains files that can be copied to remote hosts.
**vars/:** Contains variables specific to the role.
**defaults/:** Contains default variables for the role, which can be overridden by other variables.
**meta/:** Defines metadata about the role, such as dependencies on other roles.
Creating an Ansible Role
Step 1: Create the Role Directory Structure
You can manually create the directory structure or use the ansible-galaxy command to generate it:
```
ansible-galaxy init myrole
```
Step 2: Define Tasks
Edit the tasks/main.yml file to define the tasks for the role:
```
# roles/myrole/tasks/main.yml
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Ensure Nginx is started
  service:
    name: nginx
    state: started
    enabled: true
```
Step 3: Define Handlers
If you need handlers, define them in the handlers/main.yml file:
```
# roles/myrole/handlers/main.yml
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```
Step 4: Define Variables
Define role-specific variables in the vars/main.yml file:

```
# roles/myrole/vars/main.yml
nginx_port: 80
```
Define default variables in the defaults/main.yml file:
```
# roles/myrole/defaults/main.yml
nginx_user: www-data
```
Step 5: Define Templates
If you need to use templates, place them in the templates/ directory and reference them in your tasks:
```
Template file (nginx.conf.j2):

server {
    listen {{ nginx_port }};
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Task using the template:

```
# roles/myrole/tasks/main.yml
- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: Restart Nginx
```
Step 6: Define Metadata
Define metadata about the role, such as dependencies, in the meta/main.yml file:
```
# roles/myrole/meta/main.yml
dependencies:
  - { role: another_role, some_variable: some_value }
Using an Ansible Role
To use the role in a playbook, reference it in the roles section:
```
```
# site.yml
- hosts: webservers
  become: yes
  roles:
    - role: myrole
      nginx_port: 8080
```
Example Role
Let's create a role called webserver that installs and configures Nginx.

Generate the role structure:
```
ansible-galaxy init webserver
Define tasks in tasks/main.yml:

# roles/webserver/tasks/main.yml
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: Restart Nginx

- name: Ensure Nginx is started
  service:
    name: nginx
    state: started
    enabled: true
Define handlers in handlers/main.yml:

# roles/webserver/handlers/main.yml
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
Define default variables in defaults/main.yml:


# roles/webserver/defaults/main.yml
nginx_port: 80
Create a template file templates/nginx.conf.j2:
server {
    listen {{ nginx_port }};
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Use the role in a playbook site.yml:

 site.yml
```
- hosts: webservers
  become: yes
  roles:
    - role: webserver
      nginx_port: 8080
```


# Error Handling
- Error handling in Ansible is crucial for creating robust automation scripts that can gracefully handle and recover from failures. Ansible provides several mechanisms for error handling, including the use of ignore_errors, failed_when, block, rescue, and always.

## Basic Error Handling
- **ignore_errors**
    - The ignore_errors directive allows a task to continue even if it fails. This can be useful when a failure is non-critical or expected.
```
- name: Install a non-critical package
  apt:
    name: non-critical-package
    state: present
  ignore_errors: yes
failed_when
```
The failed_when directive allows you to specify conditions under which a task is considered failed. This is useful when the default success/failure conditions are not sufficient.

```
- name: Check if a service is running
  shell: systemctl status myservice
  register: service_status
  failed_when: "'inactive' in service_status.stdout"
```
- **Advanced Error Handling**

  - **block, rescue, and always**
    - Ansible’s block, rescue, and always keywords provide structured error handling similar to try/catch/finally in traditional programming languages.

**block:** Contains the tasks to be executed.
**rescue:** Contains the tasks to be executed if any task in the block fails.
**always:** Contains the tasks to be executed regardless of success or failure.
```
- name: Structured error handling example
  hosts: all
  tasks:
    - block:
        - name: This is a task that might fail
          command: /bin/false
      rescue:
        - name: This is a rescue task
          debug:
            msg: "Task failed, but we handled it"
      always:
        - name: This task runs always
          debug:
            msg: "This runs regardless of success or failure"
```
Example Playbook with Error Handling
Here’s a more comprehensive example that combines several error-handling techniques:

```
- name: Comprehensive error handling playbook
  hosts: all
  tasks:
    - block:
        - name: Attempt to create a directory
          file:
            path: /tmp/mydir
            state: directory

        - name: Attempt to create a file in the directory
          file:
            path: /tmp/mydir/myfile
            state: touch

        - name: Simulate a command failure
          command: /bin/false

      rescue:
        - name: Handle the failure by removing the directory
          file:
            path: /tmp/mydir
            state: absent

        - name: Notify about the failure
          debug:
            msg: "An error occurred, but it has been handled"

      always:
        - name: Ensure cleanup always runs
          file:
            path: /tmp/mydir
            state: absent

        - name: Final notification
          debug:
            msg: "Cleanup completed"
```
**Custom Error Messages**
Using msg in the debug module within a rescue block can help provide custom error messages:
```
- name: Custom error message example
  hosts: all
  tasks:
    - block:
        - name: Simulate a task that fails
          command: /bin/false
      rescue:
        - name: Custom error message
          debug:
            msg: "The previous task failed, but it has been handled"
```
Summary
Effective error handling in Ansible involves understanding and using the various mechanisms available for managing and recovering from task failures. By using directives like ignore_errors, failed_when, and structured error handling with block, rescue, and always, you can create more resilient and maintainable automation scripts.
