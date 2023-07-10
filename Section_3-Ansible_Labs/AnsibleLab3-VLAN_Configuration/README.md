# Ansible Lab 3 - VLAN Configuration

Lab 3 focuses on configuring VLANs on the two Catalyst 9000v switches and the Nexus 9000v switch.  This lab adds a new folder called host_vars.  Like the group_vars folder in lab2 which could provide group wide configuration, this folder contains configuration that is unique to each host in the lab.  In this lab we have a unique VLAN to set on each switch that will later be associated to the Alpine linux host and the router uplink port for the subnet the device is connected.  This VLAN is assigned in the host_vars/*host* file located in the lab directory.  The group_vars file follows and allows us to configure a system wide VLAN that will be used to place unused ports, to take them out of the default VLAN in the next lab.

## Folder and playbook details

The VLANs in the group and host vars directories are stored in a list.  This allows us to loop through the variables in the list and run a single task multiple times.  If we wanted to do this in the playbook, we would need 3 tasks, one for each switch, with all the VLANs we want to have configured on the switch.  

* group_vars - contains a configuration for VLAN 999 to assigned to the switches group.  Notice, unlike lab 2 where we did all.yaml, we are creating a group specific yaml file with our desired config variables.  There is a routers.yaml file that contains a VLAN.  However the playbook ONLY runs on switches so this will be ignored.
* host_vars - contains host specific vlan configuration.  If we wanted to add additional unique VLANs to a host, this is where we would assign those values.
* assign_vlans.yaml - Ansible playbook that loops through the lists in host_vars and group_vars to assign VLAN configurations as specified.  This will only create the VLAN, the next lab assigned the switchport to a VLAN.

Last note, the groups_vars vlans will not apply, this is intentional.  Since we have a key of vlans under host_vars and a key of vlans under group_vars, the key value of vlan in the host_vars takes precendence due to how Ansible operates.

## What if we didn't use a list to loop

The current playbooks have two tasks, one for deploying IOS vlans with a config *name* and *vlan_id*, at the end we see a loop that runs through the *vlans* list configured for each host in host_vars.  If we didn't use a list, we would need to create three tasks and specify that we only want to run them when our host equals the desired host.  The below is a completely valid configuration but it doesn't scale well for more complex, larger host environments.

```
  - name: (IOS) Configure VLAN interfaces - cat9k-sw1
    cisco.ios.ios_vlans:
      config:
      - name: Ansible_Vlan_5
        vlan_id: 5
      - name: Shutdown_VLAN_999
        vlan_id: 999
    when: {{inventory_hostname}} == "cat9kv-sw1"

  - name: (IOS) Configure VLAN interfaces - cat9k-sw2
    cisco.ios.ios_vlans:
      config:
      - name: Ansible_Vlan_20
        vlan_id: 5
      - name: Shutdown_VLAN_999
        vlan_id: 999
    when: {{inventory_hostname}} == "cat9kv-sw2"

  - name: (NX-OS) Configure VLAN interfaces - n9kv-sw1
    cisco.nxos.nxos_vlans:
      config:
      - name: Ansible_Vlan_30
        vlan_id: 30
      - name: Shutdown_VLAN_999
        vlan_id: 999
    when: {{inventory_hostname}} == "n9kv-sw1"
```
### How to run the playbook

After running the playbook below, ssh to each switch using the netops user and verify the vlan has been create.  Alternatively, return to lab 1 and update the "show_commands.yaml" file to have perform a *show vlan* on the switches.  If you choose to do this, you should change the playbook to only run against hosts: switches

*Reminder: The -k option allows us to send the ssh password for the lab.  When prompted, use *netops_admin*

```
ansible-playbook assign_vlans.yaml -k
```

We've only added one VLAN to each network device.  If you want, experiment with lists and add another vlan to each host.  You simple need a new line with the following:
```
Ex.:
- name: Ansible_vlan_10
  id: 10
```
```
Result:
./host_vars/n9kv_sw1.yaml
---
vlans:
- name: Ansible_vlan_30
  id: 30
- name: Ansible_vlan_10
  id: 10
```

### Next Lab

Once completed, move on to the next Lab by right clicking AnsibleLab4-Interface_Configuration and selecting Open in Integrated Terminal