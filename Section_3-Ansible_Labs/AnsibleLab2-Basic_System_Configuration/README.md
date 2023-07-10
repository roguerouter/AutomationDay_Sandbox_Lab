# Ansible Lab 2 - Basic System Configuration

Ansible allows us to create playbooks that run tasks in an organization fashion.  In this lab we will configure basic system configuration items.  These include:

* Device Banner
* NTP server
* Hostname (using the inventory hostname of the device) and Domain Name

This example uses a simple playbook named *basic_system_config.yaml* to call files in the tasks directory named *banner.yaml*, *ntp.yaml*, and *hostname.yml*.  These files contain the actual modules Ansible will use to configure our devices.  Using a tasks directory can create a more organized structure for managing multiple unique configurations vs. having them in a single or multiple playbooks.

This example also contains a group_vars folder.  Ansible provides multiple methods for configuring variables that across groups of devices.  In our lab, we want to define the domain name and ntp server to use across the lab.  If we wanted group specific variables, we could create a file named routers.yaml (for our router group in inventory.yaml) and switches.yaml (for the switches group in our inventory.yaml)

Our NTP server is located on our Ubuntu host, running on at *172.16.1.100*.  Copy this IP address into the group_vars/all.yaml file and replace *<UBUNTU_HOST>*

## Folder and playbook details

* group_vars - contains an all.yaml file that will apply configuration to all hosts in this lab
* tasks - Contains three tasks for configuring ntp, hostname and domain, and banner.  This is a simple tasks example.  More advanced tasks and folder structure can be created to run against specific hosts or groups of host.
* templates - Contains banner.cfg which will configure the banner on the host.  Feel free to change the contents and save.
* basic_system_config.yaml - The playbook, calls each task in the tasks directory and runs them against our lab devices.  Tasks have a tag field assigned to them to allow us to specify a task to run versus running the whole playbook.

### How to run the playbook

Below are a couple commands to test Ansible operation.  The first command runs the playbook on all hosts.  For the second, update the banner file, save the contents and run the playbook with --tag banner.  Only the banner task will run and it will update all the host.  Finally the last command allows us to limit the banner job to only our Nexus host.  Update banner again and run the job.  Look at the host in SSH to the management address or look at CML and see what the results look like on the CLI.

*Reminder: The -k option allows us to send the ssh password for the lab.  When prompted, use *netops_admin*

```
ansible-playbook basic_system_config.yaml -k
ansible-playbook basic_system_config.yaml -k --tag banner
ansible-playbook basic_system_config.yaml -k --tag banner --limit "n9kv-sw1"
```

### Next Lab

Once completed, move on to the next Lab by right clicking AnsibleLab3-VLAN_Configuration and selecting Open in Integrated Terminal