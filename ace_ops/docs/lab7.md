# Lab 7 - Aviatrix Cloud Firewall

In this lab you will activate the `Aviatrix Cloud Firewall`, a cloud-native security solution that provides distributed firewalling, traffic control, and network segmentation. Unlike centralized traditional firewalls, it simplifies security management across multi-cloud environments, automates policy enforcement, and can operate in either learning or enforcement modes, offering flexibility for securing both egress and east–west traffic.

## 1. SCENARIO

The BU2 database requires updates, but the VM resides in a private subnet. The BU2 DB owner has requested that only `apt-get` commands be permitted for egress traffic, while all other outbound traffic be blocked. All egress traffic must be monitored with **_logging_** enabled.

Actions:
- Enable the `Cloud Secure Egress` feature on the **_ace-azure-east-us-spoke2_** VNet.
- Create `Distributed Cloud Firewall` rules to enforce these requirements.

Furthermore, you are requested to create the **Distributed Cloud Firewall rules** that will enforce these requirements.

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Scenario Topology
```

## 2. CHANGE REQUEST

### 2.1 Secure Cloud Egress on Spoke2 VNet

- Enable the **Egress** on the VNet where **BU2 DB** resides.

```{tip}
Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and then click on the `"Enable Local Egress on VPC/VNets"` button.

Select the **_ace-azure-east-us-spoke2_** VPC.

Do not forget to click on **Add**.
```

```{figure} images/lab7-vpc.png
---
align: center
---
ace-azure-east-us-spoke2
```

```{important}
This action will install a `Default Route` in all the Private Routing Tables inside the Azure Spoke2 VNet. The Defautl Routes will point to the **_ace-azure-east-us-spoke2_** GW.

Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_ace-azure-east-us-spoke2_** Gateway, then click on the `VPC/VNet Route Tables` tab and select any **Private** Routing Tables from the `Route Table` field!

```{figure} images/lab7-defaultroute.png
---
align: center
---
Default Route injected by the AVX Controller
```

### 2.2 Secure Cloud Egress on VNet1 - TEST VNET

```{caution}
Replicate the steps applied to VNet2 by enabling the **Egress control** on the **_azure-east-us-spoke1_** VNet. The objective is to use the Spoke1 VNet <ins>as a test environment</ins> to identify the permitted domains.
```

- Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and then click on the `"Enable Local Egress on VPC/VNets"` button, then select the **_ace-azure-east-us-spoke1_** VPC.

Do not forget to click on **Add**.

```{figure} images/lab7-vpc45.png
---
align: center
---
ace-azure-east-us-spoke1
```

### 2.3 Distributed Cloud Firewall - Activation

- Enable the **Distributed Cloud Firewall** feature.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and then click on the `"Begin Using Distributed Cloud Firewall"` button, then to the subsequent `"Begin Using Distributed Cloud Firewall"` button, and last but not least to the `"Begin"` button
```

```{figure} images/lab7-enabledcf01.png
---
align: center
---
Begin Using Distributed Cloud Firewall
```

```{figure} images/lab7-enabledcf0222.png
---
align: center
---
Begin
```

The Aviatrix Controller has pushed a `Default- Action Rule` permitting all traffic; you should observe the impact right away.

```{figure} images/lab7-enabledcf0411.png
---
align: center
---
Default Action Rule
```

Now, click **Ruleset** and select the `V1 Policy List`.

```{figure} images/lab7-greenfield5666.png
---
align: center
---
V1 Policy List
```

- Create a new rule clicking on the **"+ Rule"** button.

```{figure} images/lab7-greenfield56661.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>Greenfield-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-greenfield566612.png
---
align: center
---
Greenfield-Rule
```

Click on **Commit**.

```{figure} images/lab7-greenfield5666123.png
---
align: center
---
Commit
```

- Use **_Spoke1 VNet_** as a test environment. SSH to the **BU1 DB**, then execute the _apt-get_ commands. This will help identify the domains that should be permitted.

```{figure} images/lab7-test.png
---
align: center
---
Spoke1 VNet1 as test VNet
```

You need to **SSH** into the **BU1 DB**. Since this VM does not have a public IP, you must first SSH into the **BU1 Frontend** VM, and then, from there, initiate an SSH connection to the private IP of the **BU1 DB**.

```{figure} images/lab7-test3.png
---
height: 400px
align: center
---
SSH to BU1 DB
```

- Apply the `All-Web` WebGroup to restrict HTTP/HTTPS traffic for capturing logs of accessed FQDNs.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and edit the _Greenfield-Rule_, clicking on the pencil icon on the right-hand side.

This time attach the `All-Web` WebGroup and then click on **Save In Drafts**.
```

```{figure} images/lab7-test4.png
---
height: 150px
align: center
---
Edit the Greenfield-Rule
```

```{figure} images/lab7-test5.png
---
align: center
---
All-Web
```

Do not forget to **Commit** your changes into the Data Path.

```{figure} images/lab7-test6.png
---
align: center
---
Commit
```

- Now execute the **_apt-get_** commands from the **BU1 DB** !

```bash
sudo apt-get update -y 
```
and
```bash
sudo apt-get upgrade -y
```

When you see these pop-up messages, just click on the **Enter** button on your keyboard!

```{figure} images/lab7-enter.png
---
align: center
---
Press Enter
```

```{figure} images/lab7-enter2.png
---
align: center
---
Press Enter again
```

- Check the domains hit by the *apt-get* commands

```{tip}
Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and select the *ace-azure-east-us-spoke**1*** on the `"VPC/VNets"` field.
```

```{figure} images/lab7-test7.png
---
align: center
---
Monitor
```

```{warning}
<ins>Identify all the necessary domains for successfully executing the apt-get commands by exclusively examining the entire list of logs. 

Any other domains should be disregarded.</ins>
```

The required domains are **five**!

<details>
  <summary>Click here for discovering the <span style='color:#33ECFF'>allowed domains</span></summary>
  
- motd.ubuntu.com
- azure.archive.ubuntu.com
- livepatch.canonical.com
- contracts.canonical.com
- esm.ubuntu.com
</details>

<details>
  <summary>Click here for discovering the <span style='color:#33ECFF'>prohibited domains</span></summary>
  
- storage.azure.net
- windows.net

```{figure} images/lab7-prohibited.png
---
height: 150px
align: center
---
Press Enter again
```
</details>

### 2.1 Detach the WebGroup

- Detach the **_All-Web_** Webgroup from the Greenfield-Rule.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and click on the pencil icon on the righ-hand side of the Greenfield-Rule entry.

Clear the `WebGroups` field and then click on **Save In Drafts**. 

Do not forget then to click on **Commit**.
```

```{figure} images/lab7-test8.png
---
align: center
---
Detaching the All-Web WebGroup
```

```{figure} images/lab7-lastone2.png
---
align: center
---
Commit your change
```

- Now that you have discovered the domains that are triggered when the **_apt-get_** commands are executed, you can proceed in disabling the `Egress` feature from the **Spoke1** VNet.

```{tip}
Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and click on the `Disable Local Egress` button 
beside the **_ace-azure-east-us-spoke1_** VNet entry.
```

```{figure} images/lab7-disableegress.png
---
height: 200px
align: center
---
Disable Egress on Spoke1 VNet
```

- SSH to the **BU2 DB** (this VM does not have a Public IP, whereby you need to SSH to BU1 Frontend first, and then from that VM, issue the SSH command towards the Private IP of BU2 DB).

```{figure} images/lab7-sshbu2db.png
---
height: 400px
align: center
---
BU2 DB ip address
```

Issue the following curl commands:

```bash
curl www.google.com
```
```bash
curl www.wikipedia.com
```
```bash
curl www.espn.com
```

All commands will be successful, however, this is not what the request asked you to configure...

- Enable the `ZTNA Model` (i.e.  **Zero Trust Network Architecture Model**), remove the **Greenfield-Rule**, and first insert an **Explicit-Deny** Rule at the bottom.

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and click on the `"+ Rule"` button.

```{figure} images/lab7-deleterulee23.png
---
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>Explicit-Deny-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

```{figure} images/lab7-deleterulee235.png
---
align: center
---
Explicit-Deny-Rule
```

Do not forget to click on **Save In Drafts**, and then **Commit** this new rule!

```{figure} images/lab7-deleterulee2356.png
---
height: 200px
align: center
---
Commit
```

### 2.2 ZTNA

Now you can proceed in deleting the Greenfield-Rule!

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules**, click on the `"three dots"` icon, on the right-hand side of the Greenfiled-Rule and then click on `"Delete Rule"`.
```

```{figure} images/lab7-deleterulee.png
---
height: 200px
align: center
---
Delete Rule
```

```{figure} images/lab7-deleterulee2.png
---
height: 200px
align: center
---
Commit
```

As soon as the **Explicit-Deny-Rule (editable)** is deployed on the top of the Rules list, the SSH session with the BU2 DB will be terminated. 

You will have to reestablish the SSH session with BU2 DB!

```{warning}
The East-West traffic is blocked now!
```

- Explore the **SmartGroups** section on the CoPilot

```{tip}
Go to **CoPilot > Groups**
```

You will notice two SmartGroups, called **BU1** and **BU2**, in addition to the pre-defined SmartGroups (i.e. *Anywhere* and *Public Internet*).

```{figure} images/lab7-bus.png
---
align: center
---
SmartGroups list
```

- Expand either SmartGroups to find out what VMs have been associated to each of these logical container.

```{figure} images/lab7-bus2.png
---
height: 200px
align: center
---
BU1's members
```

It is evident that BU1 Frontend turns out belonging to the BU1 SmartGroup, whereas the BU2 DB is a member of the BU2 SmartGroup.

```{important}
These two SmartGroups had been preprovisioned for you at the launch of the PODs.
```

- Create an **inter-rule** that allows SSH from BU1 to BU2.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

```{figure} images/lab7-deleterulee23567.png
---
align: center
---
inter-ssh-bu1-bu2
```

Do not forget to click on **Save In Drafts**, and then **Commit** this new rule!

```{figure} images/lab7-bus3new.png
---
height: 200px
align: center
---
Three Rules
```

- SSH to BU2 DB from the BU1 Frontend, then check the logs!

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**
```

```{figure} images/lab7-bus5.png
---
height: 150px
align: center
---
SSH logs
```

- Now create another SmartGroup that identifies the BU2 DB, solely.

```{tip}
Go to **CoPilot > Groups** and click on the `"+ SmartGroup"` button.
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU2-DB</span>
- **Matches all conditions (AND)**: <span style='color:#479608'>environment</span>
  --> **Cloud Tag Values**: <span style='color:#479608'>bu2</span>

Then click on `"+ Add condition"` button and select the following additonal parameters:

```{figure} images/lab7-bus50.png
---
align: center
---
+Add condition
```

- **Matches all conditions (AND)**: <span style='color:#479608'>Region</span>
  --> **Cloud Tag Values**: <span style='color:#479608'>eastus</span>


Do not forget to click on **Save**.

```{tip}
The SmartGroup can match **multiple tags** simultaneously, harnessing the boolean expressions!
```

```{figure} images/lab7-bu2db.png
---
align: center
---
BU2 DB SmartGroup
```

- Create a **WebGroup** that matches the domains and sub-domains that you found earlier.

```{tip}
Go to **CoPilot > Groups > WebGroups** and then click on `"+ WebGroup"`.
```{figure} images/lab7-webgroupp.png
---
align: center
---
BU2 DB SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create WebGroup"`:

- **Name**: <span style='color:#479608'>ubuntu-update</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'> *.ubuntu.com</span>
- **Domains/URLs**: <span style='color:#479608'> *.archive.ubuntu.com</span>
- **Domains/URLs**: <span style='color:#479608'> *.canonical.com</span>

then click on **Save**.

```{caution}
These are the domains identified before using the `Discovery Mode` (**= Greenfield-Rule + All-Web**).

- The FQDNs `*.ubuntu.com` and `*.canonical.com` are both using a first-level wilcard. These wildcards will be placeholders for *ntp.ubuntu.com*, *ftp.ubuntu.com*, *download.ubuntu.com*, *contracts.canonical.com*...
- The FQDN `*.archive.ubuntu.com` is using a second-level wilcard, that allows to create a subdivision within *archive.ubuntu.com*.
```

```{figure} images/lab7-webgroup.png
---
height: 400px
align: center
---
ubuntu-update
```

- Now create an **inter-rule** that allows BU2 DB to generate traffic towards Internet but only for reaching out the <ins>ubuntu servers</ins> !

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ubuntu-bu2db-internet</span>
- **Source Smartgroups**: <span style='color:#479608'>BU2-DB</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public internet</span>
- **WebGroups**: <span style='color:#479608'>ubuntu-update</span>
- **Protocol**: <span style='color:#479608'>Any</span>

- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule once again!

```{figure} images/lab7-lastrule.png
---
align: center
---
inter-ubuntu-bu2db-internet
```

```{figure} images/lab7-lastcommit.png
---
align: center
---
DCF rules list
```

- Now try issuing the following curl commands once again from the BU2 DB virtual machine:

```bash
curl www.google.com
```
```bash
curl www.wikipedia.com
```
```bash
curl www.espn.com
```

```{figure} images/lab7-explicitdeny101.png
---
height: 150px
align: center
---
Explicit Deny Rule matching
```

You will notice three **Drop** entries within the **Monitor** section that have matched the `Explicit-Deny-Rule`, successfully.

```{figure} images/lab7-explicitdeny.png
---
height: 150px
align: center
---
Explicit Deny Rule matching
```

- Now try to run the **apt-get** commands !

```bash
sudo apt-get update -y 
```
and
```bash
sudo apt-get upgrade -y
```

Once again, if you see these pop-up messages, just click on the **Enter** button on your keyboard!

```{figure} images/lab7-enter.png
---
align: center
---
Press Enter
```

```{figure} images/lab7-enter2.png
---
align: center
---
Press Enter again
```

- Now check the logs within the **Egress** section!

```{tip}
Go to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and select the **ace-azure-east-us-spoke2** VNet
```

```{figure} images/lab7-last.png
---
height: 250px
align: center
---
ace-azure-east-us-spoke2
```

```{figure} images/lab7-lastlog.png
---
height: 250px
align: center
---
Allowed domains!
```

You have successfully applied the Secure Egress Control, leveraging both the `Egress` feature and the `Distributed Cloud Firewall` rules!