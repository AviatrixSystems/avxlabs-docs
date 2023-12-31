# Lab 7 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` works.

## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall functionality encompases several services, such as Distributed Firewalling, Threat Prevention, TLS Decryption, URL Filtering, Suricata IDS/IPS and Advanced NAT capabilities.

In this lab you will create additional logical containers, called `SmartGroups`, that group instances that present similarities inside a VPC/VNet/VCN, and then you will enforce rules among these SmartGroups (aka **_Distributed Cloud Firewalling Rules_**):

1) `intra-rule` = Rule applied <ins>within</ins> a Smart Group

2) `inter-rule` = Rule applied <ins>among</ins> SmartGroups

## 3. Azure Spoke2

Up until now, in Azure you have only worked on **_azure-west-us-spoke1_**. 

This lab will introduce finally **_azure-west-us-spoke2_** GW, that is a gateway in the top-right corner of this topology that is already deployed (hence it is red), <ins>although its attachment has not been deployed yet</ins>!

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Initial Topology
```

## 3.1 Azure Spoke2 Attachment

First, you will need to configure the grey Aviatrix Spoke-Transit connection in the topology between **_azure-west-us-spoke2_** and **_azure-west-us-transit_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **_azure-west-us-spoke2_**, clicking on the pencil icon:

```{figure} images/lab7-spoke.png
---
align: center
---
Edit Spoke GW
```

Attach **_azure-west-us-spoke2_** (pre-configured VNet) to **_azure-west-us-transit_** as shown below.
Do not forget to click on **Save**.

```{figure} images/lab7-attachment.png
---
align: center
---
Attachment
```

This is how the topology would look like, after the completion of the previous action.

```{figure} images/lab7-newversiontopo.png
---
align: center
---
Topology
```

## 3.2. Network Domain Association

You need also to associate **_azure-west-us-spoke2_** to the <span style='color:lightgreen'>Green</span> Network Domain.

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Click the *pencil icon* to edit the **Green** network domain.

```{figure} images/lab7-editnd.png
---
align: center
---
Edit Green
```

Select the gateway **_azure-west-us-spoke2_** from the drop-down window, selecting the `"Associations"` field.

Then click **Save**:

```{figure} images/lab7-nd2.png
---
align: center
---
Association
```

After this step, this is how the topology should look like.

```{figure} images/lab7-finaltopology.png
---
height: 400px
align: center
---
Lab 7 Topology
```

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the <span style='color:lightgreen'>Green</span> domain and the <span style='color:lightblue'>Blue</span> domain.
```

## 3.3. Existing DCF rules modification

```{warning}
On Lab 6 (Egress), the DCF functionality was enabled. The current permitted rules are the `"Inspect-DNS"`, that is only allowing traffic towards UDP port 53, and the `"allow-domains"` , that is only allowing http/https traffic towards two domains. Any other kind of traffic is denied because of the presence of the `"Explicit-Deny-Rule"`. 

Let's move the **_Greenfield-Rule_**  on the top of the list of the DCF rules!
```

```{figure} images/lab7-dcfrule.png
---
align: center
---
DCF rules
```

- Modify the Greenfield-Rule Priority

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default)**, click on the the `"two arrows"` icon on the righ-hand side of the **Greenfield-Rule** and choose *`"Move Rule"`* at the very **Top**.
Then click on **Save in Draft**.
```

```{figure} images/lab7-top.png
---
align: center
---
Move at the Top
```

Do not forget to click on **Commit**.

```{figure} images/lab7-edit.png
---
align: center
---
Commit your changes
```

- Apply the "Logging" option to the Greenfield-Rule

```{tip}
Click on the pencil icon beside the Greenfield-Rule row, then turn on the toggle for Logging and click on **Save in Drafts**.

Once again, click on **Commit**.
```

```{figure} images/lab7-newone2.png
---
align: center
---
Edit the Greenfield-Rule
```

```{figure} images/lab7-newone3.png
---
align: center
---
Editing the Greenfield-Rule
```

```{figure} images/lab7-newone4.png
---
align: center
---
Commit
```

## 4. New DCF request

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
Initial Topology Lab 7
```

## 4.1 Smart Group Creation

Create two Smart Groups and classify each Smart Group, leveraging the CSP tag `"environment"`:

- Assign the name `"bu1"` to the Smart Group **#1**.
- Assign the name `"bu2"` to the Smart Group **#2**.

### 4.1.1. Smart Group “bu1”

Go to **CoPilot > SmartGroups** and click on `"+ SmartGroup"`.

```{figure} images/lab10-smart2.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>bu1</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>environment</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>bu1</span>

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

### 4.1.2. Smart Group “bu2”

Create another Smart Group clicking on the `"+ SmartGroup"` button.

```{figure} images/lab10-smart5.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>bu2</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>environment</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>bu2</span>

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

## 4.2. Connectivity verification (ICMP)

Open a terminal window and SSH to the public IP of the instance **aws-us-east-2-spoke1-<span style='color:red'>test1</span>** (NOT test2), and from there ping the private IPs of each other instances to verify that the connectivity has not been modified.

```{note}
Refer always to your personal POD for the private IPs.
```

```{figure} images/lab10-newone.png
---
align: center
---
SSH to test1 in us-east-2 VPC
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

## 4.3.  Connectivity verification (SSH)

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

## 5. Rules Creation
### 5.1. Build a Zero Trust  Network Architecture

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

### 5.2. Create an intra-rule that allows ICMP inside bu1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newrule.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#33ECFF'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-rule1.png
---
align: center
---
Create Rule
```

At this point, there would be one uncommitted rule at the very top, as depicted below.

```{figure} images/lab10-rule2.png
---
align: center
---
Current list of rules
```

### 5.3. Create an intra-rule that allows ICMP inside bu2

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-rule3.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>intra-icmp-bu2</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - **Existing Rule**: <span style='color:#33ECFF'>intra-icmp-bu1</span>
  
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

## 6. Verification

Afte the creation of the previous Smart Groups and Rules, this is how the topology with the permitted protocols should look like:

```{figure} images/lab10-topology2.png
---
height: 400px
align: center
---
New Topology
```

### 6.1. Verify SSH traffic from your laptop to bu1

SSH to the Public IP of the instance **aws-us-east-2-spoke1-test1**.

```{figure} images/lab10-sshpod.png
---
align: center
---
SSH from your laptop
```

### 6.2. Verify ICMP within bu1 and from bu1 towards bu2

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
Turn on the **Auto Refresh** knob. Moreover, refresh the web page to trigger the logs.
```

```{figure} images/lab10-monitor.png
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

Relaunch the ping from **aws-us-east-2-spoke1-<span style='color:#33ECFF'>test1</span>** towards **aws-us-east-2-spoke1-<span style='color:red'>test2</span>**. 

```{figure} images/lab10-pingtotest2fail.png
---
align: center
---
Ping fails
```

```{important}
This time the ping fails. You have achieved a complete separation between Smart Groups deployed in the same VPC in AWS US-EAST-2, thanks to the **Security Group Orchestration** carried out automatically by the **Aviatrix Controller**.
```

### 6.3. Verify SSH within bu1

SSH to the Private IP of the instance **_azure-west-us-spoke1-test1_** in Azure. Despite the fact that the instance is within the same Smart Group "bu1", the SSH will fail due to the absence of a rule that would permit SSH traffic within the Smart Group.

```{figure} images/lab10-sshfail.png
---
align: center
---
SSH fails
```

### 6.4. Add a rule that allows SSH in bu1

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newrule2.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Protocol**: <span style='color:#33ECFF'>TCP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - **Existing Rule**: <span style='color:#33ECFF'>intra-icmp-bu2</span>

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

This time the SSH session will be established, thanks to the new intra-rule.

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

### 6.4. SSH to VM in bu2

SSH to the Public IP of the instance **_gcp-us-central1-spoke1-test1_**:

```{figure} images/lab10-sshtocentral.png
---
align: center
---
SSH to gcp-us-central1-spoke1-test1
```

### 6.5. Verify ICMP traffic within bu2

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
 
### 6.6. Inter-rule from bu2 to bu1

Create a new rule that allows ICMP FROM bu2 TO bu1.

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab10-newrule4.png
---
align: center
---
New Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>inter-icmp-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - **Existing Rule**: <span style='color:#33ECFF'>intra-ssh-bu1</span>
  
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

SSH to the Public IP of the instance **_azure-west-us-spoke<span style='color:#33ECFF'>2</span>-<span style='color:#33ECFF'>test1</span>_**.

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
 
## 7. East-1 and the Multi-Tier Transit

### 7.1 Activation of the MTT

Let’s now also involve the AWS region **US-EAST-1**.

This time, you have to allow the ICMP traffic between the Smart Group **bu2** and the ec2 instance **_aws-us-east1-spoke1-test2_**, solely.

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

Let’s enable this time the MTT feature!

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

Repeat the previous action for the remaning Transit Gateways:

- **aws-us-east-2-transit**: <span style='color:#33ECFF'>ASN **64513**</span>
- **gcp-us-central1-transit**: <span style='color:#33ECFF'>ASN **64514**</span>
- **azure-west-us-transit**: <span style='color:#33ECFF'>ASN **64515**</span>

Go to C**oPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east-2-transit_**.

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

This time if you click on the `"Gateway Routes"` tab, you will be able to see the destination route in **aws-us-east1-spoke1** VPC.

```{figure} images/lab10-mtt6.png
---
align: center
---
10.0.12.0/23
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

Although this time there is a valid route to the destination, thanks to the **MTT** feature, the pings fails. 

```{warning}
The reason is that the ec2-instance  **aws-us-east-1-spoke1-test2** is not allocated to any Smart Groups yet!
```

### 7.2 Smart Group “east1”

Let’s create another Smart Group for the test instance **_aws-us-east-1-spoke1-test2_** in US-EAST-1 region in AWS.

Go to **Copilot > SmartGroups** and click on  `"+ SmartGroup"` button.

```{figure} images/lab10-mttnew.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>east1</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>Name</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>aws-us-east1-spoke1-test2</span>

```{figure} images/lab10-mtt9.png
---
align: center
---
Resource Selection
```

The CoPilot shows that there is just one single instance that matches the condition:

- **aws-us-east-1-spoke1-test2** in AWS

Do not forget to click on **Save**.

### 7.3 Create an inter-rule that allows ICMP from bu2 towards east1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-mtt8.png
---
align: center
---
New Rule
```

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>inter-icmp-bu2-east1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>east1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Then click on **Save In Drafts**.

```{caution}
Please note the direction of this new inter-rule: **FROM** bu2 **TO** east1
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

### 7.4 Verify connectivity between bu2 and east1

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

`Congratulations, you have deployed the full-blown Aviatrix solution!`

```{figure} images/lab10-lastdrawing.png
---
height: 400px
align: center
---
Full-Blown Aviatrix Solution
```