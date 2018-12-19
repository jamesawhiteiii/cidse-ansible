# CIDSE Ansible Readme

This file details this directory/repo as well as some basics on how to use Ansible in our environment. Should you
not find the answer you need in this readme you should consult the offical Ansible documentation. It is thorough and
includes excellent examples as well as discussion of best practices. In addition to understanding Ansible you should also
understand the basics of Git source control. To read more on setting up the CIDSE Ansible environment see the environment setup readme file.


## Basic Terminology

**Playbook** - Combination of inventory and roles (give a list of machines a role [aka purpose])  
**Role** - As shown in the directory structure below, a role represents settings, configuration files, packages etc for a specific purpose (eg: db or webserver)  
**Task** - These are individual script-like items in a role, for example you might install a package, verify a file exists, or edit a config file  
**Inventory** - An inventory file is a flat file that serves as a sort of database, it contains groups and hostnames. Groups can contain other groups or hosts

## Running a Playbook & Commands 

To run a playbook ensure your present working directory is the main 'ansible' folder. Perform a 'git-pull' to ensure you
have the latest version of all the Ansible files before running them against any hosts.    
The syntax for a playbook command is:
```
    ansible-playbook -i <inventoryfile> <host/group> <playbookname.yml> -Kk --ask-vault-pass
```

Useful options to be aware of:
```
| Option           | Use                                                                          |
|------------------|------------------------------------------------------------------------------|
| --ask-vault-pass | Prompts you for the vault password when running a command                    |
| -e               | Allows you to pass extra variables to Ansible for use in command or in tasks |
| -i               | Use this to specify an inventory file                                        |
| -k               | Prompts you for SSH password                                                 |
| -K               | Prompts you for sudo/become password                                         |
| -u               | Allows you to specify the SSH username                                       |
```

As a quick note: Ansible uses 'become' in place of the word 'sudo'. Direct sudo usage in Ansible is deprecated. You should always use 'become' and remember it works exactly as 'sudo' would.

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
ARA:
    Sets up ARA Ansible Runtime Analysis for the main Ansible machine
    This role is not part of the site.yml master playbook
    Used only for the setup of the cidse-ansible machine

Common:
   Disable Root Account
   Change Techs account password
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

Service-Acct:
    Sets up a local use account with admin permissions to only be used by Ansible
    The user and pass are vaulted in a group_vars file
    
Printers:
    Sets up all CIDSE Common printers, and enables cups. 

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

## Ansible Vault

Ansible Vault is a method for encrypting files that allows Ansible to parse and read them during execution. It also allows us to upload credentials as part of Git commits by encrypting the entire contents of files. Below is a refernce of commands used to handle 'vaulted' files within the Ansible repo here. The below section is copied from the Ansible documentation on using the Vault feature.

#### Creating Encrypted Files
To create a new encrypted data file, run the following command:

``` ansible-vault create foo.yml ```

First you will be prompted for a password. The password used with vault currently must be the same for all files you wish to use together at the same time. After providing a password, the tool will launch whatever editor you have defined with $EDITOR, and defaults to vi (before 2.1 the default was vim). Once you are done with the editor session, the file will be saved as encrypted data.

The default cipher is AES (which is shared-secret based).

#### Editing Encrypted Files
To edit an encrypted file in place, use the ansible-vault edit command. This command will decrypt the file to a temporary file and allow you to edit the file, saving it back when done and removing the temporary file:

``` ansible-vault edit foo.yml ```

#### Rekeying Encrypted Files
Should you wish to change your password on a vault-encrypted file or files, you can do so with the rekey command:

``` ansible-vault rekey foo.yml bar.yml baz.yml ```

This command can rekey multiple data files at once and will ask for the original password and also the new password.

#### Encrypting Unencrypted Files
If you have existing files that you wish to encrypt, use the ansible-vault encrypt command. This command can operate on multiple files at once:

``` ansible-vault encrypt foo.yml bar.yml baz.yml ```

#### Decrypting Encrypted Files
If you have existing files that you no longer want to keep encrypted, you can permanently decrypt them by running the ansible-vault decrypt command. This command will save them unencrypted to the disk, so be sure you do not want ansible-vault edit instead:

``` ansible-vault decrypt foo.yml bar.yml baz.yml ```

#### Viewing Encrypted Files

If you want to view the contents of an encrypted file without editing it, you can use the ansible-vault view command:

``` ansible-vault view foo.yml bar.yml baz.yml ```
