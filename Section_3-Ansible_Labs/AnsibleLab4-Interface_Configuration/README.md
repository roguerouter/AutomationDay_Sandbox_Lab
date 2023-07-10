# Ansible Lab 4 - Interface Configuration

Interface configuration is a highly host specific configuration.  Generally, no two interfaces are alike, and different platforms have different naming schemes for the interface assignment.  This lab does not have a group_vars folder.  Instead each host can be in the host_vars directory with the host specific configurations.  For the routers we need to configure the IP addressing to be used on the ports between the Ubuntu host, between cat8kv-rtr1 and cat8kv-rtr2, cat8kv-rtr1 and cat9kv-sw1, and finally cat8kv-rtr2 and switches cat9kv-sw2 and n9kv-sw1.  For the switches, we need to perform multiple tasks on each host.  First we need to configure the Alpine Linux desktops and their router uplink, next we need to configure the SVI interface that assigns the subnet IP to reach the upstream router. Finally, the switches will shutdown all their unused interfaces and place them in VLAN 999.

## Folder and playbook details

* host_vars - Contains the host specific interfaces, IP addressing, and for switches the vlan configuration.
* tasks - contains the three tasks to configure switchport interface and vlan, SVIs, and shutdown unused switchports.
* configure_router_interfaces.yaml - Configures the router interfaces
* configure_switch_interfaces.yaml - Runs each task in the tasks directory to configure the switches

### How to run the playbooks

After running these playbooks, you will have IP connectivity in the lab.  You can ssh to your Ubuntu host (default IP 10.10.20.108) with netops/netops_admin  However, since we haven't created the routing configuration yet you can't ping all the devices.  Once you run the router playbook, you should be able to ping 192.168.1.2 which is the router interface connecting to the Ubuntu host.  Routing will be configured in the next lab.

*Reminder: The -k option allows us to send the ssh password for the lab.  When prompted, use *netops_admin*

```
ansible-playbook configure_router_interfaces.yaml -k
ansible-playbook configure_switch_interfaces.yaml -k
```

### Next Lab

Once completed, move on to the next Lab by right clicking AnsibleLab5-Routing_Configuration and selecting Open in Integrated Terminal