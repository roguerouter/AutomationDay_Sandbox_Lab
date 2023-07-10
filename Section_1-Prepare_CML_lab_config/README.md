# Prepare Cisco Modeling Labs (CML) and Ansible

README (skip if using your own CML deployment) - If you are using the Cisco Modeling Labs Sandbox

The first step to get the CML lab operational and start working with Ansible.  This directory generates the Ansible inventory files and Cisco Modeling Labs - Labfile.  Once your lab has started, from your WSL or Remote-SSH terminal in vscode run ssh developer@10.10.20.50 using the password documented in the CML Sandbox page.  Once connected, verify the default route of your sandbox using ip route.  You should see coming like this

```
(py3venv) [developer@devbox ~]$ ip route
default via 10.10.20.254 dev ens160 proto static metric 100 
10.10.20.0/24 dev ens160 proto kernel scope link src 10.10.20.50 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
```

The IP listed after "default via" is the address of your default route.  If this IP differs from the default configuration lab configuration of 10.10.20.254, change the cml_sandbox_gateway value in the lab_values.yaml file in the inventory directory.  If this is not different, move ahead and exit the ssh session on devbox.

README (skip if using CML Sandbox) - If you are using your own CML deployment

The 10.10.20.X IP addresses set in the lab_values.yaml file will need to be modified to support your specific environment.  These addresses need to be routable from your WSL or Remote-SSH host.  See "A note on altering inventory below".

## Setup Lab

To setup the lab in either the CML Sandbox or your own CML deployment, you need to run the generate_CML_lab.yaml playbook (see Running the Playbook below).  This will create a directory called "processed_files" which will contain: 

* inventory.yaml - Inventory file for all labs included in Section_3-Ansible_Labs.  Labs are soft linked to this file and will not work without the file existing here
* automation_day_template.yaml - CML lab configuration file.  If you opt not to work through the Section_2-Simple_API this file can be uploaded to CML to run the labs. 

## Folder Structure

Included in this directory are the following files and folders:

* ansible.cfg - Ansible configuration file containing default configuration for how we want Ansible to operate for the purposes of this lab.  For security purposes, in a production environment "host_key_checking" needs to be changed to True.
* inventory - This contains a lab_values.yaml file which will configure IP address for CML and the Ansible inventory.  More details below on configuration.
* processed_files - This directory is empty but will contain the processed templates that are created by the generate_CML_lab.yaml playbook.  The processed templates will result in an inventory.yaml file to allow Ansible to reach each host and the full lab configuration that can be imported to Cisco Modeling Labs
* templates - Jinja2 format templates with a few variables that are assigned from the lab_values.yaml file.
* generate_CML_lab.yaml - This is the Ansible playbook which will process the templates using the provided inventory and assign all variables to each file.  Output is saved to processed_files.
* check_cml.yaml - This Ansible playbook with check the hosts in CML to verify they are accessible from your local machine.  If you are going to run Section_2-Simple_API, follow the steps to deploy you

### A Note on altering inventory

The values listed below in lab_values.yaml can be altered for your environment.  The ubuntu_host_ens2 and ubuntu_host_ens2_gw variables in this lab point to an internet facing connection, and if changed need to be on a subnet that can reach the internet as well.  ENS2 is attached to the External Connector, operating in "bridge mode", that allows the ubuntu host to download necessary packages, clone this repository, and provide SSH access to run the playbooks. Your dns1 variable should use the DNS server IP of your local subnet, for home systems this is generally your router GW address or a public DNS like google on 8.8.8.8.  If ENS2 cannot reach the internet the cloud-config will fail to deploy and your lab will not operate.  The management and ubuntu_host_ens3 addresses can be changed as desired, so long as they are all in the same subnet.  You must assign the appropriate prefix mask for Ubuntu hosts that you change.

```
        cml_lab_title: "Practice Lab"
        cat8kv_r1_mgmt: 10.10.20.101
        cat8kv_r2_mgmt: 10.10.20.102
        cat9kv_sw1_mgmt: 10.10.20.103
        cat9kv_sw2_mgmt: 10.10.20.104
        n9kv_sw1_mgmt: 10.10.20.160
        cml_sandbox_gateway: "10.10.20.254"
        ubuntu_host_ens2: "10.10.20.108/24"
        domain_name: ansible.lab
        dns1: "8.8.8.8"
```

### Running the playbook

To execute the playbook run

```
ansible-playbook generate_CML_lab.yaml
```
## The instructions below are for users who do not want to run the Section_2-Simple_API lab.

If you intend to perform Section_2 labs and work with Postman, please skip the following steps as these will import the lab via the CML Web UI.  Section_2 labs import the lab and start it using the API calls available in CML.

### To upload lab to CML 

Once you run the generate_CML_lab.yaml playbook you'll see a "processed_files" folder get created in VScode.  Open this folder and right click the automation_day_template.yaml and select Download.  This will save the file to your local machine.  From the dashboard of the CML environment you are using:
* Select the IMPORT button
* Click the files to import bar
* Locate the downloaded file and open it
* Select import and allow a few seconds for processing
* Select go to lab

Once in the lab, located the simulate tab on the lower toolbar and then select start lab.  This will take approximately 3 - 5 minutes to complete.  Once complete there will be a green check next to the hosts.  If this is not showing the lab is not ready, please wait until all systems have a check before moving forward.

### Once the lab is uploaded and started

We need to "ping" all the hosts to make sure they are accessible from our Remote-SSH session.

**Step 1 - Verify hosts can be pinged from CML Devbox:**

```ansible-playbook devbox_ping_cml.yaml -i 10.10.20.50, -u developer -k```

The password will be the CML Devbox password listed in the Sandbox lab instrctions.  You should see the following output if all the hosts are running:

```        "stdout_lines": [
            "PING 10.10.20.101 (10.10.20.101) 56(84) bytes of data.",
            "64 bytes from 10.10.20.101: icmp_seq=1 ttl=255 time=0.837 ms",
            "",
            "--- 10.10.20.101 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 0.837/0.837/0.837/0.000 ms",
            "PING 10.10.20.102 (10.10.20.102) 56(84) bytes of data.",
            "64 bytes from 10.10.20.102: icmp_seq=1 ttl=255 time=0.645 ms",
            "",
            "--- 10.10.20.102 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 0.645/0.645/0.645/0.000 ms",
            "PING 10.10.20.103 (10.10.20.103) 56(84) bytes of data.",
            "64 bytes from 10.10.20.103: icmp_seq=1 ttl=255 time=1.43 ms",
            "",
            "--- 10.10.20.103 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 1.438/1.438/1.438/0.000 ms",
            "PING 10.10.20.104 (10.10.20.104) 56(84) bytes of data.",
            "64 bytes from 10.10.20.104: icmp_seq=1 ttl=255 time=1.47 ms",
            "",
            "--- 10.10.20.104 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 1.476/1.476/1.476/0.000 ms",
            "PING 10.10.20.105 (10.10.20.105) 56(84) bytes of data.",
            "64 bytes from 10.10.20.105: icmp_seq=1 ttl=255 time=2.05 ms",
            "",
            "--- 10.10.20.105 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 2.057/2.057/2.057/0.000 ms",
            "PING 10.10.20.108 (10.10.20.108) 56(84) bytes of data.",
            "64 bytes from 10.10.20.108: icmp_seq=1 ttl=64 time=2.27 ms",
            "",
            "--- 10.10.20.108 ping statistics ---",
            "1 packets transmitted, 1 received, 0% packet loss, time 0ms",
            "rtt min/avg/max/mdev = 2.273/2.273/2.273/0.000 ms"
            ]
```

If you do not see the output above, verify your hosts are all up and running in CML and try again.  On the CML sandbox it may take a couple attempts.  You may also want to ssh directly to developer@10.10.20.50 and ping the hosts directly from the devbox to get them to respond.  If this is successful move on to Step 2 below.

**Step 2 - Verify Hosts are reachable from Remote-SSH or WSL host**

Run the following command, when prompted, the password (sans quotes) is 'netops_admin' :

```ansible-playbook check_cml.yaml -i processed_files/inventory.yaml -u netops -k```

You should see the following output:

```
ok: [cat8kv_r1] => {
    "msg": "cat8kv_r1 is running"
}
ok: [cat8kv_r2] => {
    "msg": "cat8kv_r2 is running"
}
ok: [n9kv_sw1] => {
    "msg": "n9kv_sw1 is running"
}
ok: [n9kv_sw2] => {
    "msg": "n9kv_sw2 is running"
}
ok: [n9kv_sw3] => {
    "msg": "n9kv_sw3 is running"
}
```

If you do not see this output, please check Step 1 again and then try this Step afterwards.  If you do see the following output, you are now ready to start Section_3-Ansible_Labs!

To move forward:

* Expand the Section_3-Ansible_Labs folder
* Right click with your mouse on the AnsibleLab1-Show_Commands_and_Debug
* Select Open in Integrated Terminal
