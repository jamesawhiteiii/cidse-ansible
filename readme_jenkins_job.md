# Jenkins Job Setup with API Call

Use this as a guide to set up a Jenkins job to be triggered remotely via an API call (eg: a curl from a remote machine)

For further reference see [here](https://humanwhocodes.com/blog/2015/10/triggering-jenkins-builds-by-url/) for more detail.

## Steps

1. Ensure you have properly set up an 'auto' user and API key per the environment setup readme
2. Open Jenkins
3. Click on **New Item**
4. Name the new job
    - We use 'Provision_SA' for the Service Account job
        - *NOTE: Job names affect the generated URL for triggers*  
5. Click on **Freestyle Project**
6. Click **Ok**
7. Add a description
8. Check **GitHub Project** if using GitHub
9. Add the Repo URL to **Project URL**
10. Select **This Project Is Parameterized** if your API call will be providing some information to the Jenkins job build (eg: a hostname)
    - Add **cidse_newhost** as the name of the parameter
    - Add a description
    - Select **Trim the string**
11.  Check boxes for concurrency and build throttling as appropriate to your specific job
12. Select **Git** under **Source Control Management**
13. Enter repository URL and add credentials as needed
14. Specify the branch to build from (eg. master vs testing)
15. Check **Trigger builds remotely**
16. Generate or paste in the Job Auth Token
    - The Provision_SA token is stored in the main vault file
17. Under **Build Environment** select **Use secret texts or files**
18. Add **Username and password (separate)** for each of the following
    - Ansible SSH User/Pass (use these variable names)
        - jenk_oldtechs_user
        - jenk_oldtechs_pass
19. Add a build step of **Invoke Ansible Playbook**
    - Playbook: **./service-acct.yml**
    - File/Host List: **${cidse_newhost},**
    - Associate SSH credentials of preseed techs account
    - Associate vault password
    - Check **become**
    - Check **Disable SSH Host Key Checking**
    - Check **Unbuffered stdout**
    - Additional parameters: **-e "ansible_sudo_pass=${jenk_oldtechs_pass}"**
20. Click **Save**
21. Now test your new API trigger by issuing the following terminal command:
```
curl -u auto:117cc760a614eb5e961604c74b903a0e01 "http://cidse-ansible.cidse.dhcp.asu.edu:8080/job/Provision_SA/buildWithParameters?token=PEb9RAY2wjrtlrGrHckTEsf4ZxW4mXsx&cidse_newhost=test"
```