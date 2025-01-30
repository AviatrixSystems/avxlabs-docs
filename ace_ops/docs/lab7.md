# Lab 7 - Cloud Perimeter Security (Secure Cloud Egress)

The borderless nature of the cloud creates significant challenges in securing egress traffic, controlling network visibility, and managing spend. Provider-native solutions offer only basic security, high egress data costs, compliance challenges, and exposure to threats like data exfiltration. This leads to operational inefficiencies, increased security risks, and a slower time to market.

## 1. SCENARIO

BU2 DB needs to get updates, however the VM is inside a private subnet. The BU2 DB owner has raised a request that solely the *apt-get* commands should be allowed (<ins>other kind of egress traffic should be prohibited</ins>).

Furthermore, all egress traffic must be tracked, whereby **_Logging_** must be enabled.

You are requested to activate the `Egress` feature on the **_ace-azure-east-us-spoke2_** VNet, likewise on the **_ace-azure-east-us-spoke1_** VNet but just temporarily as a Test VNet.

In addition to this, you area also requested to create the DCF rules that would fulfil the aforementioned request.

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Scenario Topology
```

## 2. CHANGE REQUEST

- Enable the Egress on the VNet where the BU2 DB resides.

```{tip}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and then click on the `"Enable Local Egress on VPC/VNets"` button.

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

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_ace-azure-east-us-spoke2_** Gateway, then click on the `VPC/VNet Route Tables` tab and select any **Private** Routing Tables from the `Route Table` field!

```{figure} images/lab7-defaultroute.png
---
align: center
---
Default Route injected by the AVX Controller
```

- Enable the **Distributed Cloud Firewall** feature.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and then click on the `"Enable Distributed Cloud Firewall"` button, then to the subsequent `"Begin Using Distributed Cloud Firewall"` button, and last but not least to the `"Begin"` button
```

```{figure} images/lab7-enabledcf01.png
---
align: center
---
Enable DCF message
```

```{figure} images/lab7-enabledcf02.png
---
align: center
---
Begin using DCF message
```

```{figure} images/lab7-enabledcf03.png
---
align: center
---
Greenfield Rule message
```

You will immediately notice a pop-up message, informing you that the `Greenfield Rule` will be created for allowing all kind of traffic.

```{figure} images/lab7-enabledcf04.png
---
align: center
---
Default Rules
```

- Enable the `"Logging"` on the Greenfield-Rule.

```{tip}
Click on the pencil icon on the right-hand side of the entry **Greenfield-Rule**.
```

```{figure} images/lab7-editgreen.png
---
height: 150px
align: center
---
Edit the Greenfield-Rule
```

```{figure} images/lab7-enablelog.png
---
align: center
---
Enable Logging
```

Do not forget to click on **Save In Drafts** and then click on **Commit**, to push the change into the Data Plane.

```{figure} images/lab7-lastone.png
---
height: 150px
align: center
---
Commit
```

- Use the **_Spoke1 VNet_** as a Test VNet to find out the domains that should be allowed, in order to selectively execute the *apt-get* commands only towards the defined domains.

```{figure} images/lab7-test.png
---
align: center
---
Spoke1 VNet1 as test VNet
```

  - Enable the Egress on the **_Spoke1 VNet_** too.

```{tip}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and then click on the `"Enable Local Egress on VPC/VNets"` button.

Select the **_ace-azure-east-us-spoke1_** VPC.

Do not forget to click on **Add**.

```{figure} images/lab7-test20.png
---
height: 400px
align: center
---
Enable Local Egress 
```

```{figure} images/lab7-test2.png
---
height: 400px
align: center
---
ace-azure-east-us-spoke1
```

- SSH to the **BU1 DB** (this VM does not have a Public IP, whereby you need to SSH to BU1 Frontend first, and then from that VM, issue the SSH command towards the Private IP of BU1 DB).

```{figure} images/lab7-test3.png
---
height: 400px
align: center
---
SSH to BU1 DB
```

- Enable the `Discovery Mode` on the Greenfield-Rule

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and edit the Greenfield-Rule, clicking on the pencil icon on the right-hand side.

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
height: 150px
align: center
---
Commit
```

- Now execute the **_apt-get_** commands from the BU1 DB !

```bash
sudo apt-get update -y 
```
and
```bash
sudo apt-get upgrade -y
```

- Check the domains hit by the *apt-get* commands

```{tip}
Go to **CoPilot > Security > Egress > Monitor** and select the *ace-azure-east-us-spoke1* on the `"VPC/VNets"` field.
```

```{warning}
<ins>Identify all the required domains for launching successfully the *apt-get* commands, inspecting **solely** the whole list of logs.</ins>
```

The required domains are **five**!

<details>
  <summary>Click here for discovering the <span style='color:#33ECFF'>domains</span></summary>
  
- motd.ubuntu.com
- azure.archive.ubuntu.com
- livepatch.canonical.com
- contracts.canonical.com
- esm.ubuntu.com
</details>

```{figure} images/lab7-test7.png
---
height: 200px
align: center
---
Monitor
```

- Detach the **_All-Web_** Webgroup from the Greenfield-Rule.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the pencil icon on the righ-hand side.

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
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and click on the `remove` button 
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

- Enable the `ZTNA Model` (i.e.  **Zero Trust Network Architecture Model**), creating an **Explicit Deny Rule** first.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+Rule"` button.
```

Create an `Explicit Deny Rule`:

- **Name**: <span style='color:#479608'>Explicit-Deny-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

Do not forget once again to click on **Save In Drafts** and then click on **Commit**.

```{figure} images/lab6-egressrule.png
---
align: center
---
Explicit-Deny-Rule
```

As soon as the Explicit-Deny-Rule is deployed the SSH session with the BU2 DB will be terminated. 

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
- **CSP Tag Key**: <span style='color:#479608'>environment</span>
  --> **CSP Tag Value**: <span style='color:#479608'>bu2</span>
- **CSP Tag Key**: <span style='color:#479608'>Region</span>
  --> **CSP Tag Value**: <span style='color:#479608'>eastus</span>

```{figure} images/lab7-anothercondition.png
---
height: 300px
align: center
---
+Add condition
```

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
height: 150px
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

- Now check the logs within the **Egress** section!

```{tip}
Go to **CoPilot > Security > Egress > Monitor** and select the **ace-azure-east-us-spoke2** VNet
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