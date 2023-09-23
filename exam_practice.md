# RHCE Exam Practice
This practice exam is based on a version of the exam centered around RHEL 8, but should be useful regardless. Answers to each task are intentionally left out of the repo for obvious reasons.
## Lab requirements
- To work through this practice exam, you will need a total of 5 servers
    - One server needs to be configured as a control host
    - The other 4 servers are configured as managed servers, using the names ansible1 through ansible4. These servers must meet the following requirements
        - At least 1GB of RAM
        - At least 20GB disk space on the primary disk, /dev/sda
        - At least 5GB disk space on a secondary disk, /dev/sdb, which only exist on ansible1 and ansible3
        - The root user account has been configured with a matching password on all servers
        - The control server has a user account "ansible". SSH public and private keys have been generated, but no further configuration has been done yet.
## Tasks
- Configure the control host with a static inventory, as well as the ansible.cfg file. In the static inventory, configure the following host groups:
    - Group `test` with `ansible1` as a member
    - Group `dev` with `ansible2` as a member
    - Group `prod` with `ansible3` and `ansible4` as a members
    - Group `servers` with groups `dev` and `prod` as members
    - Ensure that hosts can be reached through their FQDN as well as their short name
- Create an ad-hoc command that subscribes all hosts to a Red Hat subscription and run it in a shell script
- In that same shell script, include the following:
    - Install python
    - Create a user with the name `ansible`
    - Create a sudo config that allows the `ansible` user to run task with root privileges
    - Use an ad-hoc command to test connectivity to all remote hosts
- Write a playbook that installs software packages:
    - `Nmap` and `Wireshark` in the groups `dev`, `test`, and `prod`.
    - All packages from the package group `Virtualization host` on the group `prod`
    - Servers in the group `prod` must be fully updated
- Write a playbook that sets up storage and meets the following requirements:
    - If a second disk is found, a volume group with the name `vgfiles` should be created. This volume group may use all available disk space
    - If no second disk is found, print the line `no second disk` and fail further task execution on this host
    - An LVM logical volume with the name `lvfiles` and a size of 1GB should be created
    - The LVM should be formatted with XFS
    - The LVM should be persistently mounted on the directory `/lvfiles`
- Use the RHEL system role that manages time in a playbook with the name `time.yaml`
    - Ensure that `control.example.com` is used as the time server
    - Ensure that the appropriate parameter allows changing time even if a large difference exists between time on the host and server exists.
- Create a playbook that installs software on the managed host according to the following requirements:
    - Custom facts with the names `package1` and `package2` are available on `ansible2` and not on other hosts
    - The value of `package1` is set to `smb`; the value of `package2` is set to `mariadb-server`
    - Write a playbook that pushes these custom facts to `ansible2`
    - Write a script with the name `mypackages.sh`, which uses package fact gathering to show the properties of only the packages that were just installed. Write the output of this script file to `packages-out.txt` to the current directory on `control`
- Write a playbook that changes SELinux to permissive on `ansible3`.
    - If the mode is changed successfully, the host should reboot after waiting for 30 seconds. 
    - If no change is made, the playbook should print `no change was made`
- Write an ansible playbook that meets the following requirements:
    - Host group variables are used to set the variable `username=birch` and `groupname=profs` on hosts in the group `dev`. Likewise, `username=dawn` and `groupname=students` should be set on the group `prod`
    - The passwords for the users should be obtained from a vault-encrypted file.
        - The vault file is protected with the password `vaultpw`
        - The user passwords are set to `userpw`
- Write a playbook that generates a file with the name `/tmp/hosts` based on discovered inventory information. This file must have the same format as `/etc/hosts`
- Write a requirements file that installs the `nginx` role and the `docker` roll created by `geerlingguy` 
    - Run these roles against `ansible1`
- Write a playbook that installs, starts, and enables the `httpd` service
    - Ensure that the service is listening on port `88` and is accessible through the firewall.
    - Write a play that tests access to the service and prints an error message if the service could not be accessed
- Create a custom role called motd that meets the following requirements
    - Uses a jinja2 template to create a `motd` file stating when the file was created, the hostname of the machine, and the name of the system admin
    - Create a playbook called `run_motd.yaml` that runs this role against Ansible2
- Install and use the `raveshaww.updates` collection from Ansible Galaxy