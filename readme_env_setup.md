# CIDSE Ansible Setup Readme

This describes the setup process for creating the CIDSE Ansible server along with Jenkins and ARA for automation and analysis.


When finished, the overall workflow consists of


| Code & Commit                                               | Jenkins                                       | ARA Runtime Analysis                                                                    |
|-------------------------------------------------------------|-----------------------------------------------|-----------------------------------------------------------------------------------------|
| Make needed changes to Ansible code or update the inventory | Detects commits, run Ansible playbook site.yml | Review results of latest Jenkins build and investigate errors, or disconnected machines |


Starting from scratch the setup process is:

1. Clean install Ubuntu LTS, name the machine 'cidse-ansible'
2. Run updates & upgrades
3. Setup the Ansible repo
```
    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt-get install ansible
```
4. Install the following packages
    - sshpass, openjdk-8-jdk, ansible, git
5. Setup Jenkins
```
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    sudo apt-get install jenkins
```
6. Run apt update
7. Install the Jenkins package
8. Goto http://cidse-ansible:8080 to access Jenkins
9. Get the temporary password via a 'cat' of the file as shown in the webpage
10. Follow the prompts and make the following selections  
    - Add the following plugins:  
        - Dashboard
        - Throttle concurrent
        - Build pipeline
        - Parameterized trigger
        - Git parameter
        - GitHub
        - GitLab
    - Remove the following plugins:
        - Ant
        - Cradle
        - Subversion
        - SSH Slaves
11. Create the first user
    - Username and password are in the vault file within the repo
        - Use 'ansible vault view' to review credential information if needed
12. Add some additional needed plugins
    - Click on **Manage Jenkins**
    - Click on **Manage Plugins**
    - Click on the **Available** tab
    - Find and select **Ansible Plugin** and **Matrix Authorization Strategy**
    - Click **Download now and install after restart**
13. Next add the 'auto' user
    - Click on **Manage Jenkins**
    - Click on **Manage Users**
    - Click on **Create User**
        - Use credentials as stated in the main vault.yml
14. Set up permissions for 'auto' user
    - Click on **Manage Jenkins**
    - Click on **Configure Global Security**
    - Select **Matrix-based Security**
    - Select all permissions for your admin account
    - Select the following for the 'auto' account
        - Overall - Read
        - Job - Build
        - Job - Read
        - Job - Workspace
    - Click **Save**
15. Generate API key for 'auto' user
    - Log out and login in as the 'auto' user
    - Click on the username 'auto' in top right of window
    - Select **Configure**
    - Under the API Token section, click **Add New Token** and select **Generate**
    - Save this token in a safe place (eg. a Vault file, you will need it later)
16. Setup ARA for reviewing Ansible logs

In a terminal, run the following
```
cd
git clone <repo-url>
cd cidse-ansible
ansible-playbook ara.yml -Kk --ask-vault-pass
```
Running this playbook will complete the full setup for ARA Runtime Analysus.

You are now ready to build Jenkins jobs and review logs

|             |                            |
|-------------|----------------------------|
| Jenkins     | https://hostname-fqdn:8080 |
| Ansible ARA | http://hostname-fqdn       |