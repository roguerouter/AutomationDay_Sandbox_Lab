# Ansible Lab 5 - Routing Configuration

The final piece of the lab configuration.  After running the playbook in this lab you should be able to ping, and ssh to all the alpine hosts from your Ubuntu desktop.  Like interfaces, routing configuration is specific to a host and requires the use of host_vars.  We will be configuring OSPF process ID, area ID found within the process, network address, wildcard mask, and the area id for the network.  The routers use OSPF process 1 and area 0 for all networks.

## Folder and playbook details

* host_vars - Contains the networks, area, and process associated with the device being configured.  The ospf configuration is created in a list for ease of configuration.
* configure_routing.yaml - Ansible playbook, this will run a task that will loop through the lists in each host_var file and configure OSPF on the upstream device.  Additionally, three additional tasks will run to create a static route on cat9kv_sw1, cat9kv_sw2, and n9kv_sw1 pointing to their respective default gateway.

### How to run the playbooks

After running these playbooks, you will have complete IP reachability.  Allow time for the adjacenies to establish before trying to ping.

*Reminder: The -k option allows us to send the ssh password for the lab.  When prompted, use *netops_admin*

```
ansible-playbook configure_routing.yaml -k
```

### Tests

From your terminal, ssh to netops@10.10.20.108 (Ubuntu Host in CML) enter the netops_admin password and attempt to ping and ssh the internal CML hosts.

When this lab is complete, you should be able to perform the following:

- Verify all interfaces are up.  It may take some time for OSPF to come up after running the playbook.

  - ping 192.168.2.254
  - ping 192.168.2.253
  - ping 192.168.3.1
  - ping 192.168.3.2
  - ping 192.168.4.254
  - ping 192.168.4.253
  - ping 192.168.4.126
  - ping 192.168.4.125
  - ping 192.168.2.100
  - ping 192.168.4.64
  - ping 192.168.4.192

- ssh to alpine-0, alpine-1, alpine-2

  - ssh netops@192.168.2.100
  - ssh netops@192.168.4.64
  - ssh netops@192.168.4.192