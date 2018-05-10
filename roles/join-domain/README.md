Join-Domain
=========

This role installs, configures the PowerBroker Identity Services Open Client and joins a domain. If you found
this role looking for a Linux domain joining solution and don't have prior experience with the PBIS client 
I highly recommend it. You can read more about it, and its enterprise offering at the BeyondTrust official 
website [here](https://www.beyondtrust.com/products/powerbroker-identity-services-open/). 

As for this role, I originally found it in its published state from Larry Smith Jr. From there I followed
several forks which have added some functionality and fixed some errors. I created this fork in an effort
to centralized all the changes made since its original publishing. I've compiled the various readme notes
into this single readme here. I plan to clean up and further detail fuctionality as I work with the role
over the coming weeks and months. 

I've done my best to ensure the original licensing was preserved as well as crediting any contributors I
was able to locate. If I'm missing anyone or anything, please let me know. 

Requirements
------------

##### Domain Information - Required
###### Domain Name - example.org
###### Domain Netbios Name - EXAMPLE
###### Domain OU - Ansible
###### Domain User - ansible (account w/privileges to join domain or delegation)
###### Domain Password - xxxxxxx

Role Variables
--------------

````
---
# defaults file for ansible-domain-join
ad_domain: '{{ pri_domain_name }}'  #define your active directory domain - FQDN - same as pri_domain_name?
ad_domain_netbios: EXAMPLE   # enter NETBIOS name...ex...example.org = EXAMPLE
ad_ou: Ansible  #define active directory OU to place computer accounts
ad_password: [] # enter your domain user account password to join AD --- Better to prompt for this in playbook
ad_user: [] # enter your domain user account to join AD --- Better to prompt for this in playbook
domain_sudo_groups:  #define AD Groups to grant sudo rights....domain admins is best at the very least...
  - Domain^Admins
pbis_debian_download: '{{ pbis_debian_file }}.sh'
#pbis_debian_file: pbis-open-8.2.2.2993.linux.x86_64.deb
pbis_debian_file: pbis-open-8.3.0.3287.linux.x86_64.deb
#pbis_debian_url: http://download.beyondtrust.com/PBISO/8.2.2/linux.deb.x64
pbis_debian_url: http://download.beyondtrust.com/PBISO/8.3
pbis_dl_dir: /opt  #defines where PBIS will be downloaded to and executed from
pri_domain_name: example.org  #defines primary domain name.


#### Download locations
# http://download.beyondtrust.com/PBISO/8.3/pbis-open-8.3.0.3287.linux.x86_64.deb.sh
# http://download.beyondtrust.com/PBISO/8.2.2/linux.deb.x64/pbis-open-8.2.2.2993.linux.x86_64.deb.sh
````

Dependencies
------------

None



Other Notes
----------------

Join Domain

Prerequisites:

To work with AWS VPC first edit the VPC DHCP option to resolve the domain name and the DNS address. Otherwise the /etc/resolv.conf will be overwritten.

Vars:

Make sure to insert to "domain_sudo_groups" the group name without spaces (e.g Domain Users = Domain^Users).


Example Playbook
----------------
#### Galaxy
-----------
    - hosts: servers
      roles:
         - { role: mrlesmithjr.domain-join }
#### GitHub
-----------
    - hosts: servers
      roles:
        - ansible-domain-join

License
-------

BSD

Author Information
------------------

Original Author:  

Larry Smith Jr.
- @mrlesmithjr
- http://everythingshouldbevirtual.com
- mrlesmithjr [at] gmail.com

Other Contributors:

Oleg Gohman  
- https://github.com/oleggohman/

Alexander Ray  
- https://github.com/alexray92/ansible-domain-join

Jon Anderson  
- https://github.com/janderslander/ 