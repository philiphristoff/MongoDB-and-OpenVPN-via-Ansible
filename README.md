# MongoDB-and-OpenVPN-via-Ansible
This repo will contain my solutions to a series of practical tasks, using the power of Ansible. The tasks are as follows:
- Set up an Ubuntu VM (e.g. with Vagrant or in the Cloud) with a running Mongo db using an Ansible playbook.
- Set up a second Ubuntu VM which should regularly store backups of the Mongo db in the first VM using an Ansible playbook.
- Set up a VPN server (with Ansible) in one VM and connect to it from the other.

# Solution and Workflow:
## 1. Configuring the two needed VMs in Microsoft Azure:

   The first needed step was configuring the two Ubuntu VMs in Azure. The VMs were both setup up with headless Ubuntu 20.04s, 1GB of RAM, and dynamic hard drives.
   An important point in the configuration to pay attention to is putting both VMs in the same subnet to avoid connectivity issues. The VMs were named Ubuntu1 and Ubuntu2, the first one functioning as
   an Ansible control node, MongoDB and OpenVPN server, while the second one completed regular backups of the MongoDB on the first host and took the role of an OpenVPN client.

   ![image](https://github.com/philiphristoff/MongoDB-and-OpenVPN-via-Ansible/assets/110902715/a9025d4e-3aec-4e01-9314-f2cd2bd1f9d2)

   ![image](https://github.com/philiphristoff/MongoDB-and-OpenVPN-via-Ansible/assets/110902715/d38e7a5b-befb-4805-85fb-91569b5e3792)



## 2. Setting up MongoDB on Ubuntu1:
 
   This step is where the first Ansible playbook comes into play, it can be found here: MongoDB_Playbooks/mongodb_setup.yml
   The playbook serves to install MongoDB on the localhost via these steps:
   - Adding the GPG key.
   - Adding the MongoDB repository to the system.
   - Updating the APT package cache.
   - Installing MongoDB using APT.
   - Allowing remote connections by editing the mongod.conf file. (This is important for the next part of the task)
   - Creating a remote user that will later be used to execute the backups. (This is important for the next part of the task)

## 3. Configuring MongoDB Backups on Ubuntu2:
 
   Two playbooks were used for this step, MongoDB_Playbooks/mongodb_backup.yml and mongodb_backup_scheduler.yml. They could have been fit into one single playbook but this logical separation allows
   for our playbooks to be more flexible and have a higher factor of reusability.
   The scheduler playbook is simply used to launch the main backup playbook every 12 hours with the help of cron jobs. Moving on to the main backup playbook, the steps it performs are as follows:
   - Installing MongoDB tools.
   - Creating backup directory on Ubuntu2 where the backups are to be temporarily stored.
   - Retrieving the backup from Ubuntu1 using the mongodump command. (This is where the remoteUser that was created earlier comes into play)
   - Archiving the backup.
   - Cleaning up the temporary backup directory.

## 4. Setting up OpenVPN server side on Ubuntu1:

   Moving on to the third task from the list, setting up an OpenVPN server-client connection between Ubuntu1 and Ubuntu2. Two playbooks are used for these purposes OpenVPN_Playbooks/openvpn_client.yml and openvpn_server.yml.
   For the server side of the setup, the openvpn_server.yml playbook is used. The steps the playbook goes through are as follows:
   - Updating the APT cache.
   - Installing OpenVPN
   - Copying the server.conf file to the required location (/etc/openvpn)
   - Restarting OpenVPN

## 5. Setting up OpenVPN client side on Ubuntu2:

   This step uses the playbook located at OpenVPN_Playbooks/openvpn_client.yml. The steps the playbook goes through are as follows:
   - Updating the APT cache.
   - Installing OpenVPN
   - Copying the client.conf file to the required location (/etc/openvpn)
   - Restarting OpenVPN

It's important to mention that both step 4. and step 5. use predefined server and client configuration files. A good idea for a farther advancement of this project would be to automate the creation of these files as well.
