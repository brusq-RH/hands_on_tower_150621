# Workshop Exercise - Inventories, credentials and ad hoc commands

**Read this in other languages**:
<br>![uk](../images/uk.png) [English](README.md),![france](../images/fr.png) [Française](README.fr.md)

## Table of Contents

* [Objective](#objective)
* [Guide](#guide)
* [Create an Inventory](#create-an-inventory)
* [Machine Credentials](#machine-credentials)
* [Configure Machine Credentials](#configure-machine-credentials)
* [Run Ad Hoc Commands](#run-ad-hoc-commands)
* [Challenge Lab: Ad Hoc Commands](#challenge-lab-ad-hoc-commands)

## Objective

Explore and understand the lab environment.  This exercise will cover

* Locating and understanding:

  * Ansible Tower [**Inventory**](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html)
  * Ansible Tower [**Credentials**](https://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html)

* Running ad hoc commands via the Ansible Tower web UI

## Guide

### Create an Inventory

The first thing we need is an inventory of your managed hosts. This is the equivalent of an inventory file in Ansible Engine. There is a lot more to it (like dynamic inventories) but let’s start with the basics.

You should already have the web UI open, if not: Point your browser to the URL you were given, similar to 

    https://student<N>.<LABID>.events.opentlc.com

replace `<N>` and `<LABID>` and log in as admin with the password given on the lab landing page.

Create the inventory:

* In the web UI menu on the left side, go to **RESOURCES → Inventories**, click the plus button on the right side and choose **Inventory**.
* NAME: Workshop Inventory
* ORGANIZATION: Default
* Click SAVE

Go back to the Inventories list. Open the **Workshop Inventory** and click the **HOSTS** button, the list will be empty since we have not added any hosts yet.

So let’s add some hosts. First we need to have the list of all managed hosts which are accessible to you within this lab. These can be found in an Ansible inventory file on the Tower node, it was created during deployment of the environment.

You should already have the code-server web UI and a terminal window open from the Accessing your Lab Environment section, if not refer back there to open it.

You can find the inventory information for your lab in the file ~/lab_inventory/hosts. In your code-server terminal, output them with cat, they should look like:

```bash
$ cat ~/lab_inventory/hosts
[all:vars]
ansible_user=student<X>
ansible_ssh_pass=PASSWORD
ansible_port=22

[web]
node1 ansible_host=22.33.44.55
node2 ansible_host=33.44.55.66
node3 ansible_host=44.55.66.77

[control]
ansible ansible_host=11.22.33.44
```

> **Warning**
>
> In your inventory the IP addresses will be different.

So back to Tower interface, Click on '+' button :
* NAME: node1
* In the variables, add **ansible_host: x.x.x.x** using the right IP found in your ~/lab_inventory/hosts file
* Click SAVE 
* Close the Host edition windows by clicking on the 'x' icon.

![Create host](images/create_host.png) 

Repeat the steps to add  node2 and node3 in the inventory.

At then end, clock on **RESOURCES → Inventories → Workshop Inventory** et **Hosts** , check the 3 nodes have well been added :

![hosts_added](images/3nodes_added.png) 




###  Machine Credentials

One of the great features of Ansible Tower is to make credentials usable to users without making them visible. To allow Tower to execute jobs on remote hosts, you must configure connection credentials.

> **Tip**
>
>This is one of the most important features of Tower: Credential Separation! Credentials are defined separately and not with the hosts or inventory settings.


As this is an important part of your Tower setup, why not make sure that connecting to the managed nodes from Tower is working?

To test access to the nodes via SSH do the following:

* In your browser bring up the terminal window in code-server (remember this runs on the Tower node)
* From here as user ec2-user SSH into node1 or one of the other nodes and execute sudo -i.

```bash
[student1@ansible ~]$ ssh ec2-user@node1
[ec2-user@node1 ~]$
sudo -i
[root@node1 ~]# exit
[ec2-user@node1 ~]$ exit
```

What does this mean?`
* Tower user `student<N>` can connect to the managed hosts with SSH key authentication as user ec2-user.
* User ec2-user can execute commands on the managed hosts as root with sudo.

###  Configure Machine Credentials
Now we will configure the credentials to access our managed hosts from Tower. In the **RESOURCES** menu choose **Credentials**. Now:

Click the '+' button to add new credentials

* NAME: Workshop Credentials
* ORGANIZATION: Click on the magnifying glass, pick Default and click SELECT
* CREDENTIAL TYPE: Click on the magnifying glass, pick Machine as type and click SELECT (you will have to use the search or cycle through the types to find it).
* USERNAME: ec2-user
* PRIVILEGE ESCALATION METHOD: sudo

> **Tip**
>
>Whenever you see a magnifiying glass icon next to an input field, clicking it will open a list to choose from.

As we are using SSH key authentication, you have to provide an SSH private key that can be used to access the hosts. You could also configure password authentication here.

Bring up your code-server terminal on Tower, and cat the SSH private key:

```bash
[student1@ansible ~]$ cat .ssh/aws-private.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2nnL3m5sKvoSy37OZ8DQCTjTIPVmCJt/M02KgDt53+baYAFu1TIkC3Yk+HK1
[...]
-----END RSA PRIVATE KEY-----`
```


* Copy the complete private key (including “BEGIN” and “END” lines) and paste it into the SSH PRIVATE KEY field in the web UI.
* Click SAVE

Go back to the **RESOURCES -> Credentials -> Workshop Credentials** and note that the SSH key is not visible.

You have now setup credentials to use later for your inventory hosts.


### Run Ad Hoc commands

It is possible to run run ad hoc commands from Ansible Tower as well.

* In the web UI go to **RESOURCES → Inventories → Workshop Inventory**

* Click the **HOSTS** button to change into the hosts view and select the three hosts by ticking the boxes to the left of the host entries.

* Click **RUN COMMANDS**. In the next screen you have to specify the ad hoc command:

  <table>
    <tr>
      <th>Parameter</th>
      <th>Value</th>
    </tr>
    <tr>
      <td>MODULE</td>
      <td>ping</td>
    </tr>
    <tr>
      <td>MACHINE CREDENTIAL</td>
      <td>Workshop Credentials</td>
    </tr>
  </table>

  * Click **LAUNCH**, and watch the output.

<hr>

The simple **ping** module doesn’t need options. For other modules you need to supply the command to run as an argument. Try the **command** module to find the userid of the executing user using an ad hoc command.

  <table>
    <tr>
      <th>Parameter</th>
      <th>Value</th>
    </tr>
    <tr>
      <td>MODULE</td>
      <td>command</td>
    </tr>
    <tr>
      <td>ARGUMENTS</td>
      <td>id</td>
    </tr>
  </table>

> **Tip**
>
> After choosing the module to run, Tower will provide a link to the docs page for the module when clicking the question mark next to "Arguments". This is handy, give it a try.

<hr>

How about trying to get some secret information from the system? Try to print out */etc/shadow*.

<table>
  <tr>
    <th>Parameter</th>
    <th>Value</th>
  </tr>
  <tr>
    <td>MODULE</td>
    <td>command</td>
  </tr>
  <tr>
    <td>ARGUMENTS</td>
    <td>cat /etc/shadow</td>
  </tr>
</table>

> **Warning**
>
> **Expect an error\!**

Oops, the last one didn’t went well, all red.

Re-run the last ad hoc command but this time tick the **ENABLE PRIVILEGE ESCALATION** box.

As you see, this time it worked. For tasks that have to run as root you need to escalate the privileges. This is the same as the **become: yes** used in your Ansible Playbooks.

### Challenge Lab: Ad Hoc Commands

Okay, a small challenge: Run an ad hoc to make sure the package "tmux" is installed on all hosts. If unsure, consult the documentation either via the web UI as shown above or by running `[ansible@tower ~]$ ansible-doc yum` on your Tower control host.

> **Warning**
>
> **Solution below\!**

<table>
  <tr>
    <th>Parameter</th>
    <th>Value</th>
  </tr>
  <tr>
    <td>yum</td>
    <td>command</td>
  </tr>
  <tr>
    <td>ARGUMENTS</td>
    <td>name=tmux</td>
  </tr>
  <tr>
    <td>ENABLE PRIVILEGE ESCALATION</td>
    <td>✓</td>
  </tr>
</table>

> **Tip**
>
> The yellow output of the command indicates Ansible has actually done something (here it needed to install the package). If you run the ad hoc command a second time, the output will be green and inform you that the package was already installed. So yellow in Ansible doesn’t mean "be careful"…​ ;-).

---
**Navigation**
<br>
[Previous Exercise](../1-intro) - [Next Exercise](../3-projects)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
