# Basic of working with APIs

*Note, you will need to download and install Postman from "https://www.postman.com/downloads/".  You also need to sign up for an account to import the Environment and CML collections.  If you do not wish to do this, return to Section_1 and complete the instructions 'for users who do not want to run the Section_2-Simple_API lab.'*

An Application Programming Interface (API) provides an interface by which we can directly interact with the software or network devices.  Cisco Modeling Labs contains an API that allows you to interact and control functionality within the lab environment.  The API can be accessed from within the CML tool, under Tools -> API Documentation (see image below) .  The API contains an extensive list of functions that allow you to create, delete, update, and stop/start the labs, nodes, links, and other functions of the CML server.  The goal of this lab is to give you hands on experience with simple API calls.

![alt text](../images/CML_API_Docs.png "CML API Documentation")

## Folder Structure

* Postman_Import - Contains the environment and collections for import to Postman for running this lab.  Instructions to import are below.
* yaml2json.yaml - This is an Ansible playbook to conver the automation_day_lab.yaml file to a JSON file which we will use to import into CML.

## Importing the Postman files

1. Launch postman and sign in
2. When the workspace loads, you should see My Workspace and Collections/Environment/History to the left.  If this is not expanded click collections.  This will expose an import button.
3. Click Import
4. From the import window, choose "folders"
5. Navigate to the Postman_Import folder in this repository, and select open.
6. You will be prompted to import an Environment and Collection, select Import.
7. The Collection and Environment will populate to your workspace.

**Demo of Actions**

![alt text](../images/import_postman.gif "Importing Postman Environment and Collections")

### How to use these files

The Postman collection is intended to give you a sample of what an API can do for you.  To better understand the results from the API you'll need to browse the CML API Documentation portal on your CML server.

**Authentication**

1. In postman, on the left hand of the window select environments and the CML environment.  In current value column, fill in your username, password, and the URL of your CML server (for sandbox this is https://10.10.20.161), but leave token and LAB_ID blank.  The URL should be https://*dns or ip of cml server* do not add a forward slash at the end of the address (EX. https://1.1.1.1 or https://mycmlserver.domain.local).  Select Save in the environment window.
2. Once complete, select collections
   1. Choose POST CML - Authentication
      1. In the upper right hand corner, if you see a dropdown that says "No Environment" click it and select CML.  This will activate your environment for the lab.
   2. Notice, the URL has a variable for {{BASE_URL}}.  If you highlight this value you'll see the value you configured in the environment.
   3. Click the *Body* tab.  You'll see JSON data that is being sent to the server.  The body uses the variables you entered into the environment for your username and password.
   4. Click the *Tests* tab.  This is a small piece of code that parses the response from the server and stores your authentication token into your environment.  Every time this script runs it will update the value in your environment
3. Click Send, in the lower portion of Postman, you'll see a text string that contains your Bearer Token for the API.  This token will now appear in your environment.

**View Labs**

1. Now that we have our login token, click the *View Labs* task.
2. Select the *Authorization* tab.  You should see a selection of Bearer Token, with a variable of token in the Token field.  If you highlight over this variable you will see that it has obtained your token from the environment thanks to our previous task.
3. This is a GET request, there is no body data that needs to be sent to the server, we just want to hit *send* and see the labs (if any) running on the server.
4. If you have a lab running on this system, you should get a response, if all you get back is [] then don't worry this is because nothing is on the server.  We will add one shortly.  Users on CML Sandbox will see the default lab that is configured on the box.

**Import Lab**

1. Before we run this task we need to convert our Lab file into a JSON structure.  We can do this using the yaml2json.yaml Ansible playbook.
   1. If you are not already in the Section_2-Simple_API folder, right click it on the explorer pane and select Open in Integrated Terminal
   2. Run the below command
      ```
      ansible-playbook yaml2json.yaml
      ```
   3. You should have a file called automation_day_import.json.  Select the file in VScode to open it up, as we need to copy the contents to Postman.
   4. You'll want to do a "Select All" and "Copy".  Generally Control-A/Command-A and Control-C/Command-C will perform this, otherwise you can do it from application tool bar for your editor.  Using Selection -> Select All, followed up Edit -> Copy 
2. Return to Postman, and select *Import Lab*
3. Select the *Body* tab
4. Choose the *raw* button, and change the dropdown type from Text to JSON (if not already done)
5. Paste the contents of the JSON file we created into the body.  Scroll to the top of the body section, lets change the title.
6. Click Send, if everything went ok, we'll see a Status of 200 OK in the lower body section, and JSON output containing ID and a string, and warning (which should be blank)
7. Copy the alpha-numeric string in the id field (Ex. c60a1665-6e73-4c18-a677-f9f3ef14af49), we'll need this for the next task.

**Lab Operating State**

1. Skip *Start Lab* and select *Lab Operating State*
2. Notice, our URL now has a variable for *LAB_ID*.  We need to add the value we copied from the previous task to the *LAB_ID* variable in our environment.  Select Environment -> CML 
3. In the LAB_ID, under current value, paste your alpha-numeric string into the current value field, hit Save in the upper right hand corner of the window.
4. Return to Collections and Select *Lab Operating State*
5. This is another GET request, when we hit send we should get some JSON data indicating that our Lab exists, when it was created, the title, and ownership as a few examples.  Go ahead and hit send.
6. Your should now see the output in the lower half of the window.  The state should show "DEFINED_ON_CORE" which means it exists, but we haven't started it.

**Start Lab**

1. Select the *Start Lab* task
2. This is a PUT request, we use PUT because we want to modify the current operation of the lab, which in this case is stopped
3. We do not need to change anything, click Send, you should not get any content in the lower portion of the screen, however, you should receive a status of *204 No Content* indicating the task was successful.
4. Return to *Lab Operating State* and click Send
5. We now see that the lab is started, this does not mean however it is fully operational.  To see that, add "check_if_converged" to the end of the URI and click send
6. While the system is pending full start up, the result of checked_if_converged will show false.  You can browse to the lab in your CML environment to see when all the nodes show green checkboxes.  Once complete you should see the API return true as a result.

## CML Sandbox Users

Once you get a true response from postman.  Return to the Section_1-Prepare_CML_lab_config by right clicking the folder, and selecting Open in Integrated Terminal.  You need to perform the following steps.

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