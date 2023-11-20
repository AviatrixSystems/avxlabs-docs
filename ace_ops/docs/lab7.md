# Lab 7 - Secure Egress

## 1. SCENARIO

BU2 DB needs to get updates, however the VM is inside a private subnet. The BU2 DB owner has raised a request that solely the *apt-get* commands should be allowed (<ins>other kind of egress traffic should be prohibited</ins>).

Furthermore, all egress traffic must be tracked, whereby **_Logging_** must be enabled.

You are requested to activate the `Egress` feature on the **_ace-azure-east-us-spoke2_** VPC solely, and create the rules that would fulfil the aforementioned request.

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Scenario Topology
```

## 2. CHANGE REQUEST

- Enable the Egress on the VPC where the BU2 DB resides.

```{tip}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and then click on the `"+ Local Egress on VPC/VNets"` button.

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
This action will install a `Default Route` within the Private Subnets, pointing to the **_ace-azure-east-us-spoke2_** GW.

```{figure} images/lab7-defaultroute.png
---
align: center
---
ace-azure-east-us-spoke2
```

- Enable the **Distributed Cloud Firewall** feature.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and then click on the `"Enable Distributed Cloud Firewall"` button, and then to the subsequent `"Begin Using Distributed Cloud Firewall"` button.
```

You will immediately notice a pop-up message, informing you that the `Greenfield Rule` will be created for allowing all kind of traffic in a typical **_Deny-List_** model.

Click on **Begin**.

```{figure} images/lab7-greenfield.png
---
align: center
---
Greenfield Rule
```

- Enable the `"Logging"` on the Greenfield-Rule.

```{tip}
Click on the pencil icon on the right-hand side of the entry **Greenfield-Rule**.
```

```{figure} images/lab7-editgreen.png
---
align: center
---
Edit the Greenfield-Ruyle
```

```{figure} images/lab7-enablelog.png
---
align: center
---
Enable Logging
```

Do not forget to click on **Save In Drafts** and then click on **Commit**, to push the change into the Data Plane.

- SSH to the BU2 DB (this VM does not have a Public IP, whereby you need to SSH to BU1 Frontend first, and then from that VM, issue the SSH command towards the Private IP of BU2 DB).

```{figure} images/lab7-sshbu2db.png
---
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

- Enable the `Allow-List Model` (i.e. ZTNA approach), creating an **Explicit Deny Rule** first.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+Rule"` button.
```

Create an `Explicit Deny Rule`:

- **Name**: <span style='color:#33ECFF'>Explicit-Deny-Rule</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Deny**</span>

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
Go to **CoPilot > SmartGroups**
```

You will notice two SmartGroups, called BU1 and BU2, in addition to the pre-defined SmartGroups (i.e. *Anywhere* and *Public Internet*).

```{figure} images/lab7-bus.png
---
align: center
---
SmartGroups list
```

- Expand either SmartGroups to find out what VMs have been associated to each of these logical container.

```{figure} images/lab7-bus2.png
---
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

- **Name**: <span style='color:#33ECFF'>inter-ssh-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>BU2</span>
- **Protocol**: <span style='color:#33ECFF'>TCP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** this new rule!

```{figure} images/lab7-bus3.png
---
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
align: center
---
SSH logs
```

- Now create another SmartGroup that identifies the BU2 DB, solely.

```{tip}
Go to **CoPilot > SmartGroups**
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>BU2-DB</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>name</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>ace-azure-east-us-spoke2-bu2-db</span>

Do not forget to click on **Save**.

```{figure} images/lab7-bu2db.png
---
align: center
---
BU2 DB SmartGroup
```

- Create a WebGroup that matches the *ubuntu* domain, its sub-domains and the *canonical* domain!

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > WebGroups** and then click on `"+ WebGroup"`.
```

Ensure these parameters are entered in the pop-up window `"Create WebGroup"`:

- **Name**: <span style='color:#33ECFF'>ubuntu-update</span>
- **Type**: <span style='color:#33ECFF'>Domains</span>
- **Domains/URLs**: <span style='color:#33ECFF'>*.ubuntu.com</span>
- **Domains/URLs**: <span style='color:#33ECFF'> *.archive.ubuntu.com</span>
- **Domains/URLs**: <span style='color:#33ECFF'> *.canonical.com</span>

then click on **Save**.

```{caution}
- The FQDNs `*.ubuntu.com` and `*.canonical.com` are both using a first-level wilcard. These wildcards will be placeholders for *ntp.ubuntu.com*, *ftp.ubuntu.com*, *download.ubuntu.com*, *contracts.canonical.com*...
- The FQDN `*.archive.ubuntu.com` is using a second-level wilcard, that allows to create a subdivision within *archive.ubuntu.com*.
```

```{figure} images/lab7-webgroup.png
---
align: center
---
ubuntu-update
```

- Now create an **inter-rule** that allows BU2 DB to generate traffic towards Internet but only for reaching out the <ins>ubuntu servers</ins> !

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#33ECFF'>inter-ubuntu-bu2db-internet</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU2-DB</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Public internet</span>
- **WebGroups**: <span style='color:#33ECFF'>ubuntu-update</span>
- **Protocol**: <span style='color:#33ECFF'>Any</span>

- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

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

You will notice three **Drop** entries within the Monitor section that have matched the `Explicit-Deny-Rule`, successfully.

```{figure} images/lab7-explicitdeny.png
---
align: center
---
inter-ubuntu-bu2db-internet
```

- Now try to run the apt-get commands !

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
align: center
---
ace-azure-east-us-spoke2
```

```{figure} images/lab7-lastlog.png
---
align: center
---
ace-azure-east-us-spoke2
```

You have successfully applied the Secure Egress Control, leveraging both the Egress feature and the DCF rules!