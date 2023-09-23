# RHCE Notes and Exam Practice
This is mostly a combination of notes from both Sander van Vugt's book and video courses, plus some of my own experience peppered in. Please note that the following is mostly based around RHEL 8, but should be generally applicable to most Ansible setups.
## Lesson 1: Installing Ansible
- Ansible is a server/client architecture, but it does not have any agents on the managed hosts
- The server is generally called the "Ansible Control Node"
- SSH is used to connect to the managed computers
- You do need python, preferably python3, installed on all managed and control nodes.
- Ansible needs a dedicated user account, preferably non-root wherein each node needs sudo privileges
- The Ansible package is in EPEL, but most people generally install it via pip
- To install via EPEL:
    - `subscription-manager repos -enable=ansible-2-for-rhel-8-x86_64-rpms`
    - `dnf install ansible`
- To install via pip:
    - `dnf install python3`
    - `dnf install python3-pip`
    - `su -ansible; pip3 install ansible -user`
- May be necessary to run on all nodes:
    - `alternatives -set python /usr/bin/python3`
    - Make sure that all of the nodes have an ansible user
    - `echo "ansible ALL(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible`
    - Generate and copy around the ssh key for the ansible user
## Lesson 2: Setting up a managed environment
- An inventory list can be either dynamic or static
- You can set variables in your inventory list, but it is kinda bad practice to do so
- A host can be a member of multiple groups
- Nested groups exist
- Ranges can be used
    - For example, `server[1:20]` or `192.168.100.[100:250]`
- By default, inventory files are located in `/etc/ansible/hosts`
- You can define an alternative location in `ansible.cfg`
- Ansible has an implicit `localhost` defined, so there's no need to define it
- Become settings are not used on localhost, but it runs as the user that issued the Ansible command
- The `ansible.cfg` file is considered in order of precedence
    - `/etc/ansible/ansible.cfg` is the default
    - `~/.ansible.cfg` will override the default if it exists
    - `/.ansible.cfg` is in the current project directory and will override all of the above if it exists
    - If a variable called `ANSIBLE_CONFIG` exists, it will take priority
- `ansible-navigator` or `ansible -version` can help you figure out what config is currently being used
- The baseline ansible.cfg values to change are
    - `inventory`
    - `ask_sudo_pass`
    - `host_key_checking`
    - `deprecation_warnings`
    - `become`
    - `become_method`
    - `become_user`
    - `become_ask_pass`
## Lesson 3: Using Ad Hoc commands
- Some essential modules
    - `ping` will verify if ansible has the ability to login and that python is installed
    - `service` checks if a service is running
    - `command` will run any command but not through a shell
    - `shell` will run any command through a shell
    - `raw` runs a command on the remote host without a need for python
    - `copy` copies files to a managed host
    - `package` helps update packages in all types of package managers
    - `dnf` uses dnf to manage packages
- The following command is an example of using the raw command to help set up a host to be managed before it even has ansible installed
    - `ansible -u root -i inventory ansible3 --ask-pass -m raw -a "yum install python3"`
## Lesson 4: Getting started with playbooks
- If you want to undo the changes made by a playbook, you need to write a dedicated undo playbook
- in `~/.vimrc`, include the following:
    - `autocmd FileType yaml setlocal ai ts=2 sw=2 et`
- You can add `-v` up to four times to add verbosity
    - `-v` will show task results
    - `-vv` will show task results and task config
    - `-vvv` will show info about connections to managed hosts
    - `-vvvv` will show info about plugins, users to run scripts, and the names of the scripts executed
- `-c` preforms a dry run\
## Lesson 5: Working with variables and facts
- Ideally, playbooks should be void of variables
- You can keep variables in several different places
    - Playbook
    - Inventory file
    - Inclusion files (preferred)
- Some requirements:
    - Must start with a letter
    - Can only contain letters, numbers, and underscores
    - In conditional statements, no curly braces are needed to refer to a variable's values
    - If a variable is the first element, quotes are necessary
    - The most specific variable will get precedence
        - This order is:
            - cmd line > host > playbook > global
    - For host vars:
        - The name of the vars directory must be `host_vars`
        - The file must match the inventory name of the host
    - You can also do `group_vars` to define variables for a specific group
    - You can not `loop` or `with_items` on a dictionary
    - `.facts` files on managed hosts are custom fact files
        - These files are stored in `/etc/ansible/facts.d`
        - You can access these programmatically with `ansible_facts.ansible_local`
    - The results of a task's commands can be stored as a varible using the `register` parameter
    - This command will let you see custom facts on each managed node
        - `ansible all -m setup -a "filter=ansible_local"`
## Lesson 6: Task control
- Handlers will only run if a task has changed something
## Lesson 7: Deploying files
- Common file modules include:
    - `lineinfile` ensures that a line is in a file
    - `blockinfile` does the same but with multiple lines
    - `fetch` retrieves a file from a remote machine
    - `file` sets attributes to files, create, removes files, and manages symlinks
    - `sefcontext` sets SELinux file context in the policy
        - You need to ensure that `policycoreutils-python-utils` is installed
## Lesson 8 and 9: Roles and collections
- To install rhel system roles, run the following
    - `sudo dnf install rhel-system-roles`
- Each role you make should have its own repo for easy maintenance
- Roles will execute before anything else in a playbook, unless it's specified as a `pre_task`
- You can override what file paths are used for collections by setting `collections_path` in `ansible.cfg` or setting the `ANSIBLE_COLLECTIONS_PATH` environment variable
## Lesson 10: Large environments
- Set `forks=n` to configure the max number of simultaneous hosts
## Lesson 11: Troubleshooting Ansible
- Set `log_path` in `ansible.cfg` to write logs to a specific destination
## Lesson 12: Managing software
- By default, package facts are not gathered when ansible gathers facts. That is why `package_facts` exists
    - This module can be used to check if a package is installed
