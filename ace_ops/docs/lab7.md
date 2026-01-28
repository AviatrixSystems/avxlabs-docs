# Lab 7 - Aviatrix Cloud Firewall

In this lab you will activate the `Aviatrix Cloud Firewall`, a cloud-native security solution that provides distributed firewalling, traffic control, and network segmentation. Unlike centralized traditional firewalls, it simplifies security management across multi-cloud environments, automates policy enforcement, and can operate in either learning or enforcement modes, offering flexibility for securing both egress and east–west traffic.

## 1. SCENARIO

The database VNet owner requires that prohibited egress traffic be dropped. Please review egress traffic for **BU1-DB** and **BU2-DB** and ensure any restricted destinations are blocked.

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Scenario Topology
```

In your POD Portal, open the Gatus Dashboard for BU1-DB and BU2-DB and examine the External section to identify prohibited traffic.

```{figure} images/lab7-newgatus01.png
---
height: 400px
align: center
---
BU1-DB Gatus
```

```{figure} images/lab7-newgatus02.png
---
height: 400px
align: center
---
BU2-DB Gatus
```

```{important}
Even though the two virtual machines are deployed in private subnets, they can still access the public internet because the `Egress` feature was enabled when the POD was launched.

When `Local Egress` is enabled, **SNAT** is automatically turned on. As a result, all outbound traffic from the Spoke VPC/VNet is translated to use the gateway’s public IP address. In addition, the VPC/VNet’s default route (**0.0.0.0/0**) is updated to point to the Spoke gateway.
```

- Navigate to **CoPilot > Security > Egress > Egress VPC/VNets**. You’ll see that both **_ace-azure-east-us-spoke1_** and **_ace-azure-east-us-spoke2_** VNets are already configured with Local Egress enabled.

```{figure} images/lab7-newgatus03.png
---
height: 400px
align: center
---
Cloud Secure Egress
```

```{important}
This action installs a default route in every private route table within the Azure spoke VNets. These default routes point to the Aviatrix Spoke Gateway(s).

To verify, go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways**, select a spoke gateway (for example, ace-azure-east-us-spoke2), then open the `VPC/VNet Route Tables` tab. From the Route Table drop-down, select any private route table.
```{figure} images/lab7-defaultroute.png
---
align: center
---
Default Route injected by the AVX Controller
```

## 2. CHANGE REQUEST

### 2.1 Distributed Cloud Firewall

To control and enforce egress traffic, you must enable the `Distributed Cloud Firewall`.

#### 2.1.1 DCF - Activation

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

The Aviatrix Controller has pushed a `Default Action Rule` permitting all traffic; you should observe the impact right away.

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

#### 2.3.2 Ad-hoc Greenfield-Rule

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

### 2.4 Smart Groups

Create three Smart Groups to identify:
1) **BU1 Frontend**
2) **BU1 DB**
3) **BU2 DB**

- Navigate to **CoPilot > Groups > SmartGroups**, then click the `“+ SmartGroup”` button.

```{figure} images/lab7-SGnew00.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-aws-eu-west-1-spoke1-bu1-frontend</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-aws-eu-west-1-spoke1-bu1-frontend
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart31.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart311.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-aws-eu-west-1-spoke1-bu1-frontend_**

Create another Smart Group by clicking the `“+ SmartGroup”` button again.

```{figure} images/lab7-SGnew10.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-DB</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-azure-east-us-spoke1-bu1-db</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-azure-east-us-spoke1-bu1-db
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart41.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart411.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-azure-east-us-spoke1-bu1-db_**

Add another Smart Group by tapping the `“+ SmartGroup”` one more time.

```{figure} images/lab7-SGnew20.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU2-DB</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-azure-east-us-spoke2-bu2-db</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-azure-east-us-spoke2-bu2-db
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart51.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart511.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-azure-east-us-spoke2-bu2-db_**

#### 2.4.1 Updating the Greenfield Rule

Navigate to **CoPilot > Security > Distributed Cloud Firewall**, and click the `pencil icon` beside the **Greenfield Rule** to update its configuration.

```{figure} images/lab7-green00.png
---
height: 200px
align: center
---
Edit the Greenfiled-Rule
```

Select the `"All-Web"` WebGroup from the **_WebGroups_** field, then click **"Save In Drafts"**.

```{figure} images/lab7-green01.png
---
align: center
---
All-Web
```

```{important}
The All-Web WebGroup attached to the Greenfield-Rule will allow logging of the FQDNs being accessed.
```

Before clicking **Commit**, create another DCF Rule and then click `"+ Rule"`.

```{figure} images/lab7-green02.png
---
align: center
---
+Rule
```

### 2.5 DCF Rules

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu1frontend-bu1db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **PORT**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-interssh001.png
---
align: center
---
inter-ssh
```

Before clicking **Commit**, continue adding a new DCF Rule by clicking `"+ Rule"`.

```{figure} images/lab7-green03.png
---
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu1frontend-bu2db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-DB</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **PORT**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-interssh002.png
---
align: center
---
inter-ssh
```

You can now proceed and click **Commit** to enforce the policies in the data plane.

```{figure} images/lab7-interssh0023.png
---
align: center
---
Commit
```

```{caution}
The last two policies ensure you can securely SSH from the BU1 Frontend to the Azure database VMs in private subnets, which do not have public IP addresses.
```

### 2.6 Discovery Process

- Use **_Spoke1 VNet_** as a test environment. SSH to the **BU1 DB**, then execute the _apt-get_ commands. This will help identify the domains that should be permitted.

```{figure} images/lab7-test.png
---
align: center
---
Spoke1 VNet1 as test VNet
```

```{warning}
Both VNets are configured with private routing tables, each containing a default route that points to the Spoke gateway deployed in that VNet, enabled by the `Secure Cloud Egress` functionality.
```

You need to **SSH** into the **BU1 DB**. Since this VM does not have a public IP, you must first SSH into the **BU1 Frontend** VM, and then, from there, initiate an SSH connection to the private IP of the **BU1 DB**.

```{figure} images/lab7-test3.png
---
height: 400px
align: center
---
SSH to BU1 DB
```

You can explore the logs of your enforcement. Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and search for `"inter-ssh-bu1frontend-bu1db"`. You will see that the policy was triggered by the SSH action you performed from the BU1 Frontend to the BU1 DB.

```{figure} images/lab7-test-logs.png
---
height: 200px
align: center
---
Logs
```

- Now execute the **_apt-get_** commands from the **BU1 DB** !

```bash
sudo apt-get update -y 
```
and
```bash
sudo apt-get upgrade -y
```

If you see the pop-up message depicted below, please press the **Enter** key on your keyboard.

```{figure} images/lab7-enter.png
---
align: center
---
Press Enter
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
Identify all the domains required to successfully execute the apt-get commands by examining the entire logs. The domains hit should belong to _ubuntu/canonical_ domains, and you can derive subdomains using the wildcard *.
```

The required domains are **six**!

<details>
  <summary>Click here for discovering the <span style='color:#33ECFF'>allowed domains</span></summary>
  
- motd.ubuntu.com
- azure.archive.ubuntu.com
- changelogs.ubuntu.com
- livepatch.canonical.com
- contracts.canonical.com
- esm.ubuntu.com
</details>

### 2.7 ZTNA

Now that the discovery process is complete on the test VNet where the Azure Spoke1 Gateway is deployed, we can activate the ZTNA framework by inserting an explicit deny-all rule above the Greenfield-Rule.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**. 
```

- Clicking the `"+ Rule"` button.

```{figure} images/lab7-green53.png
---
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>ExplicitDenyAll</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>**Deny**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>inter-ssh-bu1frontend-bu1db</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-exdeall.png
---
align: center
---
ExplicitDenyAll
```

Click **Commit** to enforce the policies in the data plane.

```{figure} images/lab7-xdeall00.png
---
height: 300px
align: center
---
Commit
```

- SSH into the **BU2 DB** host (this VM has no public IP). Access must be via the **BU1 Frontend** first, then from that VM SSH to the private IP of BU2 DB.

```{figure} images/lab7-sshbu2db.png
---
height: 400px
align: center
---
BU2 DB ip address
```

Open the **Monitor** tab to access the logs, then search for `"inter-ssh-bu1frontend-bu2db"`. You’ll see that the policy was triggered by the SSH action you performed from the **BU1 Frontend**, this time aimed at the **BU2 DB**.

```{figure} images/lab7-test-logs02.png
---
height: 200px
align: center
---
Logs
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

```{figure} images/lab7-test-logs1234.png
---
height: 200px
align: center
---
Curl fails
```

The prior curls command will fail, as traffic is blocked by the **ExplicitDenyAll** rule.

### 2.8 WebGroup

- Create a **WebGroup** that matches the domains and subdomains identified previously.

```{tip}
Navigate to **CoPilot > Groups > WebGroups** and then click on `"+ WebGroup"`.
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

- The FQDNs `*.ubuntu.com` and `*.canonical.com` are both using a first-level wilcard. These wildcards will be placeholders for *ntp.ubuntu.com*, *ftp.ubuntu.com*, *download.ubuntu.com*, *contracts.canonical.com*, *changelogs.ubuntu.com*...
- The FQDN `*.archive.ubuntu.com` is using a second-level wilcard, that allows to create a subdivision within *archive.ubuntu.com*.
```

```{figure} images/lab7-webgroup.png
---
height: 400px
align: center
---
ubuntu-update
```

- Now create an **inter-rule** that permits **BU2 DB** to generate traffic to the **Internet**, but solely for reaching the _Ubuntu servers_.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and click on the `"+ Rule"` button.
```

```{figure} images/lab7-webgroup001.png
---
height: 400px
align: center
---
+Rule
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

- Now try to run the **apt-get** commands !

```bash
sudo apt-get update -y 
```
and
```bash
sudo apt-get upgrade -y
```

Once again, if you see any of the pop-up messages depicted below, please press the **Enter** key on your keyboard.

```{figure} images/lab7-enter.png
---
align: center
---
Press Enter
```

```{figure} images/lab7-enter909.png
---
align: center
---
Press Enter
```

- Now check the logs within the **Egress** section!

```{tip}
Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and select the **ace-azure-east-us-spoke2** VNet, then filter by `"Allowed"`.
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
height: 450px
align: center
---
Allowed domains!
```

```{caution}
In the logs, you’ll probably see an allowed entry for `"api.snapcraft.io"`. This domain was allowed <ins>before </ins>applying the ExplicitDenyAll rule and the specific inter-rule that uses the `ubuntu-update` WebGroup.
```

You have successfully applied Secure Egress Control, leveraging both the `Egress` feature and the `Distributed Cloud Firewall` policy.
