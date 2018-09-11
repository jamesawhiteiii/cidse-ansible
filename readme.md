# CIDSE Ansible Readme

This file details this directory/repo as well as some basics on how to use Ansible in our environment. Should you
not find the answer you need in this readme you should consult the offical Ansible documentation. It is thorough and
includes excellent examples as well as discussion of best practices. In addition to understanding Ansible you should also
understand the basics of Git source control.


## Basic Terminology

**Playbook** - Combination of inventory and roles (combine a list of machines and give them a purpose)  
**Role** - As shown in the directory structure below, a role is settings, configuration files, packages etc for a specific purpose  
**Task** - These are individual script-like items in a role, for example you might install a package or verify a file exists

## Running a Playbook & Commands 

To run a playbook ensure your present working directory is the main 'ansible' folder. Perform a 'git-pull' to ensure you
have the latest version of all the Ansible files before running them against any hosts.    
The syntax for a playbook command is:
```
    ansible-playbook -i <inventoryfile> <host/group> <playbookname.yml> -K
```

-i -- specifies which inventory file to lookup hosts / host groups in  
-K -- asks for the sudo password before starting the playbook  
-k -- asks for the SSH password before starting (used to connect with user account/password and for debugging issues with SSH connectivity)  
-u -- allows you to pass a user to SSH as [Example: *ansible-playbook ... -u techs*]

#### Ad-hoc Commands

Sometimes its useful to run an Ansible command directly from the CLI, the syntax for that is:
```
    ansible -i <inventory> <host/group> -m <module> [args] -u <user> -k
```
The preceeding example shows running a command from a module against an inventory, using a module, connecting as a certain
user and prompting for both the SSH pass and sudo password. Below is an example of an ad-hoc ping command. Keep in mind
the Ansible ping is not your standard ping. It performs a full SSH connection to the remote machine. The ping module can
be useful in troubleshooting connectivtiy issues with machines.
```
    ansible -i <inventory> <host/group> -m ping -u someserviceaccount -Kk
```

## Current Roles Overview

```
Common:
   Enables serial over USB for all users
   Adds repos
        Chrome
        Sublime Text
        PBIS-Open
   Runs an apt-get update & upgrade
   Installs common packages
        byobu      (Used for kernel cleanup cron job)
        cifs-utils
        clamav-daemon
        clamtk
        git
        gksu
        google-chrome-stable
        grub2-splashimages
        openssh-server
        open-vm-tools
        pbis-open
        sublime-text
    Copies End-User Wallpaper
    Sets up cron job for kernel cleanup once a week
	
Landscape:
    Checks current status
    Joins Landscape if not already
	
Join-Domain:
    Checks current status
    Binds to AD if not already bound
```
	

## Directory Structure     

To get more background on the purpose of files and folders within the directory structure [click here](https://leucos.github.io/ansible-files-layout)

```
production                # inventory file for production machines
staging                   # inventory file for staging or testing

ansible.cfg               # specifies ansible configuration (takes precendence over the file in /etc/ansible)

group_vars/
   group1                 # here we assign variables to particular groups of machines as defined in inventory
   group2                 # ""
host_vars/
   hostname1              # if systems need specific variables, put them here (not used very often)
   hostname2              # ""

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role", not all the below directories will be used
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted (closest thing to a script)
        handlers/         #
            main.yml      #  <-- handlers file (think of these as triggered tasks)
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```