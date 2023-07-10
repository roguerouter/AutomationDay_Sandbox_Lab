# Ansible Lab 1 - Show Commands and Ansible Debug

Ansible offers us a means to touch multiple devices at once without using the CLI.  However, due to the way it operates, output is not always visible from the commands we went.  If we want to view the response from the host, particularly help with show commands, we need to store the response in a variable and output it in a secondary task.  This lab will experiment with sending show commands and viewing the output as well as the running config from our devices.

# General notes

In this Lab, and the following labs, there is an inventory directory and an ansible.cfg file.  The inventory directory contains a soft-link to the inventory file in Section_1-Prepare_CML_Lab_Config/processed_files/inventory.yaml.  If you following the lab, leave this file unchanged.  If you wish to add a different inventory to test these commands (using an alternate lab) you will need to add a new inventory file (name it something different) and edit ansible.cfg to change the default inventory.yaml file.  You'll also need to update the save_ios_config.yaml and save_nxos_config.yaml file to point at your desired hosts and not the lab hosts.

## Playbooks in this directory

* save_ios_config.yaml - Saves the running configuration of IOS and IOS-XE based platforms.  This will only run on lab hosts Cat8kv-R1, Cat8kv-R2, Cat9kv-sw1, and Cat9kv-sw2.
* save_nxos_config.yaml - Saves the running configuration of NX-OS based platforms.  This will only run on lab host n9kv-sw1.
* show_commands.yaml - This allows us to run show commmands on IOS, IOS-XE, and NX-OS based platforms, using a conditional to only run the command on the proper hosts.  The output from the host is returned on the terminal once the playbook completes.  Experiment with show commands and see how the data is returned.

## How to run the playbooks in this directory

By default, the inventory.yaml file has a variable set for *ansible_user*.   This user is netops and it is assigned throughout the lab.  If you are using a different user and it is not in the inventory as a var, run:
```
ansible-playbook *playbook* -u *username* -k
```
The -k option allows us to send the ssh password for the lab.  When prompted, use *netops_admin*

```
ansible-playbook show_commands.yaml -k
ansible-playbook save_ios_config.yaml -k
ansible-playbook save_nxos_config.yaml -k
```

Feel free to edit show_commands.yaml and change to IOS and NXOS show commands to experiment with how data gets returned from Ansible.  Once completed move on to the next Lab by right clicking AnsibleLab2-Basic_System_Configuration and selecting Open in Integrated Terminal