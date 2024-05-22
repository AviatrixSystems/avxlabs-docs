# Lab 10 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` works.

## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall functionality encompases several services, such as Distributed Firewalling, Threat Prevention, TLS Decryption, URL Filtering, Suricata IDS/IPS and Advanced NAT capabilities.

In this lab you will create additional logical containers, called `SmartGroups`, that group instances that present similarities inside a VPC/VNet/VCN, and then you will enforce rules among these SmartGroups (aka **_Distributed Cloud Firewalling Rules_**):

1) `intra-rule` = Rule applied within a Smart Group

2) `inter-rule` = Rule applied among SmartGroups

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the <span style='color:lightgreen'>Green</span> domain and the <span style='color:lightblue'>Blue</span> domain.
```

All the Test instances have been deployed with the typical <ins>CSP tags</ins>. 

```{important}
The **CSP tagging** is the recommended method for defining the SmartGroups.
```

In this lab you are asked to achieve the following requirements among the instances deployed across the three CSPs:

- Create a Smart Group with the name `"bu1"` leveraging the tag `"environment"`.
- Create a Smart Group with the name `"bu2"` leveraging the tag `"environment"`.
- Create an `intra-rule` that allows ICMP traffic within bu1.
- Create an `intra-rule` that allows ICMP traffic within bu2.
- Create an `intra-rule` that allows SSH traffic within bu1.
- Create an `inter-rule` that allows ICMP traffic only <ins>from</ins> bu2 <ins>to</ins> bu1.

```{figure} images/lab10-initial.png
---
height: 400px
align: center
---
Initial Topology Lab 10
```

## 3. Smart Groups Creation

Create two Smart Groups and classify each Smart Group, leveraging the CSP tag `"environment"`:

- Assign the name `"bu1"` to the Smart Group **#1**.
- Assign the name `"bu2"` to the Smart Group **#2**.

### 3.1. Smart Group “bu1”

Go to **CoPilot > SmartGroups** and click on `"+ SmartGroup"`.

```{figure} images/lab10-smart2.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#479608'>bu1</span>
- **CSP Tag Key**: <span style='color:#479608'>environment</span>
- **CSP Tag Value**: <span style='color:#479608'>bu1</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Resource Selection"`.

```{figure} images/lab10-smart3.png
---
align: center
---
Resource Selection
```

The CoPilot shows that there are two instances that perfectly match the condition:

- **aws-us-east2-spoke1-test1** in AWS
- **azure-us-west-spoke1-test1** in Azure

```{figure} images/lab10-smart4.png
---
align: center
---
Resources that match the condition
```

### 3.2. Smart Group “bu2”

Create another Smart Group clicking on the `"+ SmartGroup"` button.

```{figure} images/lab10-smart5.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#479608'>bu2</span>
- **CSP Tag Key**: <span style='color:#479608'>environment</span>
- **CSP Tag Value**: <span style='color:#479608'>bu2</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Resource Selection"`.

```{figure} images/lab10-smart6.png
---
align: center
---
Resource Selections
```

The CoPilot shows that there are three instances that match the condition:

- **aws-us-east2-spoke1-test2** in AWS
- **azure-us-west-spoke2-test1** in Azure
- **gcp-us-central1-spoke1-test1** in GCP

```{figure} images/lab10-smart7.png
---
align: center
---
Resources that match the condition
```

At this point, you have only created logical containers that do not affect the existing routing domain.

Let's verify that everything has been kept unchanged! Bear in mind that there is the `Greenfield-Rule` at the very top of your DCF rules list, whereby all kind of traffic will be permitted.

```{figure} images/lab10-newone2.png
---
align: center
---
Greenfield-Rule in action
```

### 3.3. Connectivity verification (ICMP)

Open a terminal window and SSH to the public IP of the instance **aws-us-east-2-spoke1-<span style='color:red'>test1</span>** (NOT test2), and from there ping the private IPs of each other instances to verify that the connectivity has not been modified.

```{note}
Refer to your POD for the private IPs.
```

```{figure} images/lab10-newone.png
---
align: center
---
Ping
```

```{figure} images/lab10-ping.png
---
align: center
---
Ping
```

```{figure} images/lab10-newone3.png
---
align: center
---
Ping
```

### 3.4.  Connectivity verification (SSH)

Verify also from the instance **aws-us-east-2-spoke1-test1** that you can SSH to the private instance in AWS, to the instance in GCP and likewise to the other two instances in Azure.

```{note}
Refer to your POD for the private IPs.
```

```{figure} images/lab10-sshtoaws.png
---
align: center
---
SSH to test2 in AWS US-East-2
```

```{figure} images/lab10-sshtogcp.png
---
align: center
---
SSH to GCP
```

```{figure} images/lab10-sshtoazure1.png
---
align: center
---
SSH to Azure Spoke1
```

```{figure} images/lab10-sshtoazure2.png
---
align: center
---
SSH to Azure Spoke2
```

```{figure} images/lab10-sshnew.png
---
align: center
---
SSH to AWS us-east-1-spoke1-test1
```

```{figure} images/lab10-sshnew2.png
---
align: center
---
SSH to AWS us-east-1-spoke1-test2
```

The previous outcomes confirm undoubtetly that the connectivity is working smoothly, despite the creation of those two new Smart Groups.

## 4. DCF Rules Creation
### 4.1. Build a Zero Trust  Network Architecture

First and foremost, let's move the `Explicit-Deny-Rule` at the very top of the list of your DCF rules.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default)**, click on the the `"two arrows"` icon on the righ-hand side of the `Explicit-Deny-Rule` and choose *`"Move Rule"`* at the very Top. 

Then click on **Save in Draft**.
```

```{figure} images/lab10-newedit.png
---
align: center
---
Move the rule
```

Then **commit** your change!

```{figure} images/lab10-commit.png
---
align: center
---
Commit
```

```{warning}
Zero Trust architecture is "Never trust, always verify", a critical component to enterprise cloud adoption success!
```

### 4.2. Create an intra-rule that allows ICMP inside bu1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newrule.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>bu1</span>
- **Destination Smartgroups**: <span style='color:#479608'>bu1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-rule1.png
---
align: center
---
Create Rule
```

At this point, there should be just one uncommitted rule at the very top, as depicted below.

```{figure} images/lab10-rule2.png
---
align: center
---
Current list of rules
```

### 4.2. Create an intra-rule that allows ICMP inside bu2

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-rule3.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>intra-icmp-bu2</span>
- **Source Smartgroups**: <span style='color:#479608'>bu2</span>
- **Destination Smartgroups**: <span style='color:#479608'>bu2</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>intra-icmp-bu1</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-intrabu2.png
---
align: center
---
intra-icmp-bu2
```

At this point, you will have two new rules marked as `New`, therefore you can proceed and click on the **Commit** button.

```{figure} images/lab10-rule5.png
---
align: center
---
Commit
```

## 5. Verification

Afte the creation of the previous Smart Groups and Rules, this is how the topology with the permitted protocols should look like:

```{figure} images/lab10-topology2.png
---
height: 400px
align: center
---
New Topology
```

### 5.1. Verify SSH traffic from your laptop to bu1

SSH to the Public IP of the instance **aws-us-east-2-spoke1-test1**.

```{figure} images/lab10-sshpod.png
---
align: center
---
SSH from your laptop
```

### 5.2. Verify ICMP within bu1 and from bu1 towards bu2

Ping the following instances from **aws-us-east-2-spoke1-test1**:

- **gcp-us-central1-spoke1-test1** in GCP
- **azure-west-us-spoke1-test1** in Azure
- **azure-west-us-spoke2-test1** in Azure

According to the rules created before, only the ping towards the **azure-us-west-spoke1-test1** will work, because this instance belongs to the same Smart Group bu1 as the instance from where you generated ICMP traffic.

```{figure} images/lab10-pingcheck.png
---
align: center
---
Ping
```

Let's investigate the logs:

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{tip}
Turn on the **Auto Refresh** knob. Moreover, refresh the web page to trigger the logs. You can also filter out based on the protocol **ICMP**.
```{figure} images/lab10-monitor.png
---
align: center
---
Auto Fefresh and Filter
```

```{figure} images/lab10-monitor99.png
---
align: center
---
Monitor
```

Now, let's try to ping the instance **aws-us-east-2-spoke1-test2** from **aws-us-east-2-spoke1-test1**. 

```{warning}
The instance **aws-us-east-2-spoke1-test1** is in the same VPC. Although these two instances have been deployed in two distinct and separate Smart Groups, the communication will occur until you don't enable the `"Security Group(SG) Orchestration"` (aka _intra-vpc separation_).
```

```{figure} images/lab10-pingtotest2.png
---
align: center
---
Ping
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Settings** and click on the `"Manage"` button, inside the `"Security Group (SG) Orchestration"` field.

```{figure} images/lab10-orchestration.png
---
align: center
---
SG Orchestration
```

Enable the **_SG orchestration_** feature on the **_aws-us-east-2-spoke1_** VPC, flag the checkbox  `"I understand the network impact of the changes"` and then click on **Save**.

```{figure} images/lab10-orchestration2.png
---
align: center
---
Manage SG Orchestration
```

Relaunch the ping from **aws-us-east-2-spoke1-<span style='color:#479608'>test1</span>** towards **aws-us-east-2-spoke1-<span style='color:red'>test2</span>**. 

```{figure} images/lab10-pingtotest2fail.png
---
align: center
---
Ping fails
```

```{important}
This time the ping fails. You have achieved a complete separation between Smart Groups deployed in the same VPC in AWS US-EAST-2, thanks to the Security Group Orchestration carried out by the **Aviatrix Controller**.
```

### 5.3. Verify SSH within bu1

SSH to the Private IP of the instance **_azure-west-us-spoke1-test1_** in Azure. Despite the fact that the instance is within the same Smart Group "bu1", the SSH will fail due to the absence of a rule that would permit SSH traffic within the Smart Group.

```{figure} images/lab10-sshfail.png
---
align: center
---
SSH fails
```

### 5.4. Add a rule that allows SSH in bu1

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newrule2.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>bu1</span>
- **Destination Smartgroups**: <span style='color:#479608'>bu1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>intra-icmp-bu2</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-sshbu1.png
---
align: center
---
Create rule
```

Click on `"Commit"` to enforce the new rule in the **Data Plane**.

```{figure} images/lab10-commitsshbu1.png
---
align: center
---
Commit
```

- Try once again to SSH to the Private IP of the instance **_azure-west-us-spoke1-<span style='color:red'>test1</span>_** in Azure in BU1.

This time the connection will be established, thanks to the new intra-rule.

```{figure} images/lab10-sshbu1ok.png
---
align: center
---
SSH ok
```

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{figure} images/lab10-logsshbu1.png
---
align: center
---
Logs 
```

From the log above is quite evident that the `"intra-ssh-bu1`" rule is permitting SSH traffic within the Smart Group bu1, successfully.

After the creation of the previous intra-rule, this is how the topology with the permitted protocols should look like:

```{figure} images/lab10-topologynew.png
---
height: 400px
align: center
---
New Topology
```

### 5.4. SSH to VM in bu2

SSH to the Public IP of the instance **_gcp-us-central1-spoke1-test1_**:

```{figure} images/lab10-sshtocentral.png
---
align: center
---
SSH to gcp-us-central1-spoke1-test1
```

### 5.5. Verify ICMP traffic within bu2

Ping the following instances:

- **aws-us-east-2-spoke1-test1** in AWS
- **aws-us-east-2-spoke1-test2** in AWS
- **azure-west-us-spoke1-test1** in Azure
- **azure-west-us-spoke2-test1** in Azure

According to the rules created before, only the ping towards the **azure-west-us-spoke2-test1** and **aws-us-east-2-spoke1-test2** will work, because these two instance belongs to the same Smart Group **bu2** as the instance from where you executed the ICMP traffic.

```{figure} images/lab10-pingtestgcp.png
---
align: center
---
Ping
```

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{figure} images/lab10-bu2monitor.png
---
align: center
---
Monitor
```

The logs above confirm that the ICMP protocol is permitted within the Smart Group bu2.
 
### 5.6. Inter-rule from bu2 to bu1

Create a new rule that allows ICMP FROM bu2 TO bu1.

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab10-newrule4.png
---
align: center
---
New Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>inter-icmp-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>bu2</span>
- **Destination Smartgroups**: <span style='color:#479608'>bu1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>intra-ssh-bu1</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-interssh.png
---
align: center
---
Create Rule
```

Enforce the this new rule in the data plane clicking on the `"Commit"` button.

```{figure} images/lab10-newcommit2.png
---
align: center
---
Commit
```

SSH to the Public IP of the instance **_azure-west-us-spoke<span style='color:#479608'>2</span>-<span style='color:#479608'>test1</span>_**.

Ping the following instances:
- **aws-us-east-2-spoke1-test1** in AWS
- **aws-us-east-2-spoke1-test2** in AWS
- **gcp-us-central1-spoke1-test1** in GCP
- **azure-west-us-spoke1-test1** in Azure

Thit time all pings will be successful, thanks to the inter-rule applied between bu2 and bu1.

```{figure} images/lab10-pingallok.png
---

align: center
---
Ping ok
```

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{figure} images/lab10-monitorfresh.png
---
align: center
---
Monitor
```

The logs clearly demonstrate that the inter-rule is successfully permitting ICMP traffic from bu2 to bu1.

After the creation of the previous inter-rule, this is how the topology with all the permitted protocols should look like.

```{figure} images/lab10-lastdrawing2.png
---
height: 400px
align: center
---
New Topology with the DCF rules
```

```{note}
The last inter-rule works smoothly only because the ICMP traffic is generated from the bu2, however, if you SSH to any instances in the Smart Group bu1, the ICMP traffic towards bu2 will fail due to the direction of the inter-rule that was created before: **FROM** bu2 **TO** bu1 (please note the direction of the arrow in the drawing).
```

```{figure} images/lab10-direction.png
---
align: center
---
From To
```

The inter-rule is Stateful in the sense that it will permit the echo-reply generated from the bu1 to reach the instance in bu2.
 
## 6. East-1 and the Multi-Tier Transit

### 6.1 Activation of the MTT

Let’s now also involve the AWS region **US-EAST-1**.

This time, you have to allow the ICMP traffic between the Smart Group **bu2** and the ec2 instance **_aws-us-east-1-spoke1-test2_**, solely.

```{figure} images/lab10-newtopology3.png
---
height: 400px
align: center
---
New Topology
```

SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_**.

Ping the following instance:

- **aws-us-east-1-spoke1-test2** in AWS

```{figure} images/lab10-pingfails10.png
---
align: center
---
Ping
```

The ping fails, therefore, let’s check the routing table of the Spoke Gateway **_azure-west-us-spoke2_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the gateway **_azure-west-us-spoke2_**

```{figure} images/lab10-spoke2azure.png
---
align: center
---
azure-west-us-spoke2
```

Then click on the `"Gateway Routes"` tab and check whether the destination route is present in the routing table or not.

```{figure} images/lab10-gatewayroutes.png
---
align: center
---
Gateway Routes
```

```{note}
The destination route is **not** inside the routing table, due to the fact that the Transit Gateway in AWS US-EAST-1 region has only <ins>one peering</ins> with the Transit Gateway in AWS US-EAST-2 region, therefore the Controller will install the routes that belong to US-EAST-1 only inside the routing tables of the Gateways in AWS US-EAST-2, excluding the rest of the Gateways of the MCNA. If you want to distribute the routes from AWS US-EAST-1 region in the whole MCNA, you have <ins>two possibilities</ins>:
```

- Enabling `"Full-Mesh"` on the Transit Gateways in **_aws-us-east1-transit_** VPC

    **OR**

- Enabling `"Multi-Tier Transit"`

Let’s enable the **MTT** feature, to see its beahvior in action!

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east-1-transit_**.

```{figure} images/lab10-mtt.png
---
align: center
---
aws-us-east-1-transit
```

Go to `"Settings"` tab and expand the `"“Border Gateway Protocol (BGP)”` section and insert the AS number **64512** on the empty field related to the `"“Local AS Number”`, then click on **Save**.

```{figure} images/lab10-mtt2.png
---
align: center
---
Settings
```

Repeat the previous action for the last Transit Gateway still without BGP ASN:

- **azure-west-us-transit**: <span style='color:#479608'>ASN **64515**</span>

```{figure} images/lab10-newlab.png
---
align: center
---
azure-west-us-transit
```

```{note}
Both the **aws-us-east-2-transit** and the the **gcp-us-central1-transit** got already configured with their ASNs during the Lab 8!
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east-2-transit_**.

```{figure} images/lab10-mtt3.png
---
align: center
---
aws-us-east-2-transit
```

Go to `"Settings"` tab and expand the `"General"` section and activate the `"Multi-Tier Transit"`, turning on the corresponding knob. 

Then click on **Save**.

```{figure} images/lab10-mtt4.png
---
align: center
---
Multi-Tier Transit
```

Let’s verify once again the routing table of the Spoke Gateway in **_azure-west-us-spoke2_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the relevant gateway **_azure-west-us-spoke2_**

```{figure} images/lab10-mtt5.png
---
align: center
---
azure-west-us-spoke2
```

This time if you click on the `"Gateway Routes"` tab, you will be able to see the destination route, **10.0.12.0/24**, in **aws-us-east1-spoke1** VPC.

```{figure} images/lab10-mtt6.png
---
align: center
---
10.0.12.0/24
```

- SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_**.

Ping the following instance:

- **aws-us-east-1-spoke1-test2** in AWS (refer to your personal POD portal for the private IP).

```{figure} images/lab10-mtt7.png
---
align: center
---
Ping
```

Although this time there is a valid route to the destination, thanks to the **MTT** feature, the pings still fails. 

```{warning}
The reason is that the ec2-instance  **aws-us-east-1-spoke1-test2** is not allocated to any Smart Groups yet!
```

### 6.2 Smart Group “east1”

Let’s create another Smart Group for the test instance **_aws-us-east-1-spoke1-test2_** in US-EAST-1 region in AWS.

Go to **Copilot > SmartGroups** and click on  `"+ SmartGroup"` button.

```{figure} images/lab10-mttnew.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#479608'>east1</span>
- **CSP Tag Key**: <span style='color:#479608'>Name</span>
- **CSP Tag Value**: <span style='color:#479608'>aws-us-east-1-spoke1-test2</span>

```{figure} images/lab10-mtt9.png
---
align: center
---
Resource Selection
```

The CoPilot shows that there is just one single instance that matches the condition:

- **aws-us-east-1-spoke1-test2** in AWS

Do not forget to click on **Save**.

### 6.3 Create an inter-rule that allows ICMP from bu2 towards east1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-mtt8.png
---
align: center
---
New Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#479608'>inter-icmp-bu2-east1</span>
- **Source Smartgroups**: <span style='color:#479608'>bu2</span>
- **Destination Smartgroups**: <span style='color:#479608'>east1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Then click on **Save In Drafts**.

```{caution}
Please note the direction of this new inter-rule: 

**FROM** bu2 **TO** east1
```

```{figure} images/lab10-lastrule.png
---
align: center
---
The Last Rule...
```

Now you can carry on with the last **commit**!

```{figure} images/lab10-lastcommit.png
---
align: center
---
Commit
```

### 6.4 Verify connectivity between bu2 and east1

- SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_** and ping the private IP of the ec2-instance **_aws-us-east-1-spoke1-test2_**

```{figure} images/lab10-lastping.png
---
align: center
---
Ping
```

This time the ping will be successful!

Check the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{figure} images/lab10-reallylast.png
---
align: center
---
inter-icmp-bu2-east1 Logs
```

## 7. Spoke to Spoke Attachment

Now that you have enabled the Distributed Cloud Firewall, the owner of the **_azure-west-us-spoke2-test1_** VM would like to communicate directly with the nearby **_azure-west-us-spoke1-test1_** VM, avoding that the traffic generated from the VNet is sent to the NGFW, first.

```{figure} images/lab10-spoke2spoke01.png
---
align: center
---
No More NGFW
```

### 7.1 Creating a Spoke to Spoke Attachment

Go to **Copilot > Cloud Fabric > Gateways > Spoke Gateways**, locate the **_azure-west-us-spoke2_** GW and click on the **`Manage Transit Gateway Attachment`** icon on the right side of its row.

```{figure} images/lab10-spoke2spoke02.png
---
align: center
---
Manage Transit Gateway Attachment
```

Select the **Spoke Gateway** tab and then choose the **azure-west-us-spoke1** GW from the drop-down window.

Do not forget to click on **Save**.

```{figure} images/lab10-spoke2spoke03.png
---
align: center
---
azure-west-us-spoke1
```

Now go to **CoPilot > Cloud Fabric > Topology** and check the new attachment between the two Spoke Gateways in Azure.

```{figure} images/lab10-spoke2spoke04.png
---
align: center
---
Spoke to Spoke Attachment
```

```{caution}
It will take approximately **2** minutes to reflect into the Topology.
```

Let's check the **Routing Table** of the **_Spoke2_** in Azure.

Go to **CoPilot > Cloud Fabric > Gateways**, select the **azure-west-us-spoke2**, then select the **Gateways Routes** tab and search for the subnet **`192.168.1.0`** on the right-hand side.

```{figure} images/lab10-spoke2spoke05.png
---
align: center
---
azure-west-us-spoke2
```

You will notice that the destination is now reachable with a lower metric (50)! 

The traffic generated from the **_azure-west-us-spoke2-test1_** VM will now prefer going through the Spoke-to-Spoke Attachment, for the communication with the Spoke1 VNet.

```{important}
The Aviatrix Cloud Fabric is very flexible and does not lock you in with solely a Hub and Spoke Topology!
```

```{figure} images/lab10-spoke2spoke06.png
---
align: center
---
Metric 50
```

```{figure} images/lab10-spoke2spoke07.png
---
align: center
---
Spoke to Spoke
```

`Congratulations, you have deployed the full-blown Aviatrix solution!`

```{figure} images/lab10-lastdrawing.png
---
height: 400px
align: center
---
Full-Blown Aviatrix Solution
```