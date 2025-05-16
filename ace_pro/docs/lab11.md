# Lab 11 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` works.

## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall is a powerful feature of the `Aviatrix Cloud Firewall`, designed to enhance security across your cloud infrastructure. It offers a comprehensive suite of capabilities, including Distributed Firewalling, Threat Prevention, TLS Decryption, URL Filtering, Suricata Intrusion Detection and Prevention Systems (IDS/IPS), Advanced Network Address Translation (NAT) capabilities, and Micro-Segmentation.

In this lab, you will have the opportunity to create additional logical containers known as `Smart Groups`. These Smart Groups are designed to categorize instances within a VPC/VNet/VCN that share similar characteristics. Once you have organized your instances into these groups, you will then enforce security policies among them through **Distributed Cloud Firewalling Rules**. This approach not only streamlines management but also enhances security by allowing for targeted rule enforcement based on the specific behaviors and attributes of the grouped instances.

1) `intra-rule` = Rule applied within a Smart Group

2) `inter-rule` = Rule applied among Smart Groups

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the <span style='color:lightgreen'>Green</span> domain and the <span style='color:lightblue'>Blue</span> domain.
```

All test instances have been deployed with the standard cloud service provider **(CSP) tags**.

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
Initial Topology Lab 11
```

## 3. Smart Groups Creation

Create two Smart Groups and classify each one using the CSP tag `"environment"`: 

- Assign the name `"bu1"` to the Smart Group **#1**.
- Assign the name `"bu2"` to the Smart Group **#2**.

### 3.1 Smart Group “bu1”

Go to **CoPilot > Groups > SmartGroups** and click on `"+ SmartGroup"`.

```{figure} images/lab10-smart2.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>bu1</span>
- **Matches all conditions (AND)/environment**: <span style='color:#479608'>bu1</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Preview"`.

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

### 3.2 Smart Group “bu2”

Create another Smart Group clicking on the `"+ SmartGroup"` button.

```{figure} images/lab10-smart5.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>bu2</span>
- **Matches all conditions (AND)/environment**: <span style='color:#479608'>bu2</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Preview"`.

```{figure} images/lab10-smart6.png
---
align: center
---
Resource Selection
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

At this stage, you have created logical containers that do not impact the existing routing domain, thanks to the `Connetion Policy` applied in **Lab 3**. Now, it’s time to thoroughly define Distributed Cloud Firewall (DCF) rules to govern **East-West** traffic.

Below is the current list of your DCF Rules within the **Distributed Cloud Firewall** section of your CoPilot:

```{figure} images/lab10-newone2.png
---
height: 300px
align: center
---
Complete DCF Rules List
```

### 3.3 Connectivity Verification (ICMP) Using Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select **_aws-us-east-2-spoke1-test1_**.

Insert the credentials available on your POD Portal and then click on **"Sign in"**.

```{figure} images/lab10-gatus00.png
---
height: 400px
align: center
---
Gatus
```

```{figure} images/lab10-gatus01.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test1
```

The **East-West** traffic is disrupted. The instance **_aws-us-east-2-spoke1-test1_** can only reach **test2** due to intra-VPC traffic, which bypasses the Aviatrix Spoke Gateway (i.e. the **Enforcement Security Point**).

### 3.4 Connectivity Verification (ICMP) Using SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Open a terminal window and SSH into the public IP of the instance **aws-us-east-2-spoke1-<span style='color:red'>test1</span>** (_NOT test2_). From there, ping the private IPs of each other instances to verify that the connectivity is indeed **broken**!

```{note}
Refer to your POD for the private IPs.
```

```{figure} images/lab10-newone.png
---
height: 400px
align: center
---
SSH
```

```{figure} images/lab10-newone3.png
---
align: center
---
Ping
```

```{figure} images/lab10-newone43.png
---
align: center
---
Ping
```

```{note}
From the outcomes above, you can see that only the **aws-us-east-2-spoke1-<span style='color:red'>test2</span>** instance is pingable. This is because the ICMP traffic from aws-us-east-2-spoke1-test1 to aws-us-east-2-spoke1-test2 is NOT passing through the Spoke Gateway.

The ICMP traffic is utilizing the standard behavior of intra-VPC communication.
```

### 3.5 Connectivity Verification (SSH) Using Gatus App

From the Gatus App open on **_aws-us-east-2-spoke1-test1_**, verify the SSH section.

```{figure} images/lab10-gatus02.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test1
```

The SSH test is only successful against **_aws-us-east-2-spoke1-test2_** due to intra-VPC traffic.

### 3.6  Connectivity Verification (SSH) Using SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Verify also from the instance **aws-us-east-2-spoke1-test1** that you **_can't_** SSH to any other instances, except to the **aws-us-east-2-spoke1-test2**, due to the fact that the SSH connection in this case, is established once again within the VPC, <ins>bypassing the Spoke Gateway (i.e. the DCF Enforcement Point)</ins>!

```{note}
Refer to your POD for the private IPs.
```

```{figure} images/lab10-sshtoaws.png
---
align: center
---
SSH to test2 in AWS US-East-2--> OK
```

```{figure} images/lab10-sshnew2.png
---
align: center
---
SSH fails towards the other instances
```

The previous results clearly confirm that connectivity is disrupted, allowing only `intra-vpc traffic`.

## 4. DCF Rules Creation
### 4.1. Create an intra-rule that allows ICMP inside bu1

```{warning}
Zero Trust architecture is "Never trust, always verify", a critical component to enterprise cloud adoption success!
```

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

Click on **Commit**.

```{figure} images/lab10-rulecommitted00.png
---
align: center
---
Rule committed
```

### 4.2. Create an intra-rule that allows ICMP inside bu2

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-rule3.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

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

Now proceed and click on the **Commit** button.

```{figure} images/lab10-rulecommitted01.png
---
align: center
---
Rule committed
```

## 5. Verification

Following the creation of the previous Smart Groups and Rules, the topology with the permitted protocols will appear as follows:

```{figure} images/lab10-topology2.png
---
height: 400px
align: center
---
New Topology
```

### 5.1 Verify ICMP within bu1 and from bu1 towards bu2 Using the Gatus App

Open the Gatus App on **_aws-us-east-2-spoke1-test1_** and verify the ICMP Traffic.

```{figure} images/lab10-gatus80.png
---
align: center
---
Gatus
```

**_aws-us-east-2-spoke1-test1_** and **_azure-west-us-spoke1-test1_** can ping each other due to the intra-rule applied to the SmartGroup "bu1".

However, **_aws-us-east-2-spoke1-test1_** is also capable of pinging **_aws-us-east-2-spoke1-test<span style='color:red'>2</span></summary>_**, although the latter belongs to "bu2". Once again, this behaviour stems from the fact that the two instances reside in the same VPC.

### 5.2 Verify SSH traffic from your laptop to bu1 Using SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

SSH to the Public IP of the instance **aws-us-east-2-spoke1-test1**.

```{figure} images/lab10-sshpod.png
---
align: center
---
SSH from your laptop
```

```{caution}
The SSH session from your laptop to the **_aws-us-east-2-spoke1-test1_** instance is not affected by any DCF rules, because the connection is established directly through the **AWS IGW**.
```

```{figure} images/lab10-laptop.png
---
align: center
---
SSH from your laptop - logical diagram
```

### 5.3 Verify ICMP within bu1 and from bu1 towards bu2 Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

Now, let's try to ping the instance **aws-us-east-2-spoke1-test2** from **aws-us-east-2-spoke1-test1**. 

```{figure} images/lab10-pingtotest2.png
---
align: center
---
Ping
```

**aws-us-east-2-spoke1-test1** can successfully ping **aws-us-east-2-spoke1-test<span style='color:red'>2</span></summary>** because the communication occurs within the VPC, bypassing the Spoke Gateway.

### 5.4 SG Orchestration

Let's investigate the logs:

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and then click on the filter icon and filter out based on the rule `"intra-icmp-bu1"`.

```{tip}
Filter out based on the protocol **ICMP**.
```{figure} images/lab10-monitor100.png
---
align: center
---
Filter
```

```{figure} images/lab10-monitor.png
---
align: center
---
Outcomes
```

Based on the outcome above, you will not see 10.0.1.10 (aws-us-east-2-spoke1-test2) because it belongs to bu2, which does not match the intra-icmp-bu1 rule. 

However, **aws-us-east-2-spoke1-test1** can ping **aws-us-east-2-spoke1-test2** due to intra-VPC communication, without needing to match any of the deployed DCF rules.

```{figure} images/lab10-sgorch01.png
---
align: center
---
tes1 and test2 in AWS US-EAST-2
```

```{figure} images/lab10-sgorch02.png
---
align: center
---
tes1 and test2 in AWS US-EAST-2
```

```{warning}
The instances **aws-us-east-2-spoke1-test1** and **aws-us-east-2-spoke1-test2** are in the same VPC. Although these two instances have been deployed in <ins>two distinct and separate Smart Groups</ins>, the communication will occur until you don't enable the `"Security Group(SG) Orchestration"` (aka **_intra-vpc separation_**).
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Settings** and click on the `"Manage"` button, inside the `"Security Group (SG) Orchestration"` field.

```{figure} images/lab10-orchestration.png
---
height: 300px
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

### 5.5 SG Orchestration Verification Using the Gatus App

Open the Gatus App on **_aws-us-east-2-spoke1-test1_** and verify the connectivity with **_aws-us-east-2-spoke1-test2_**.

```{figure} images/lab10-gatus81.png
---
align: center
---
Gatus
```

The ping to test2 will gradually fail!

```{note}
SSH traffic will also fail!
```

```{figure} images/lab10-gatus8786.png
---
align: center
---
Gatus
```

### 5.6 SG Orchestration Verification Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Relaunch the ping from **aws-us-east-2-spoke1-<span style='color:#479608'>test1</span>** towards **aws-us-east-2-spoke1-<span style='color:red'>test2</span>**. 

```{figure} images/lab10-pingtotest2fail.png
---
align: center
---
Ping fails
```

```{important}
This time the ping fails. You have achieved a complete separation between Smart Groups deployed in the same VPC in AWS US-EAST-2 region, thanks to the Security Group Orchestration carried out by the **Aviatrix Controller**.
```

```{note}
SSH traffic will also fail!
```

### 5.7 Verify SSH within bu1 Using the Gatus App

Open the Gatus App on **_aws-us-east-2-spoke1-test1_** and verify the SSH Traffic.

```{figure} images/lab10-gatus88.png
---
align: center
---
Gatus
```

**_aws-us-east-2-spoke1-test1_**  is unable to SSH into any other instances within bu1, as well as those in bu2.

### 5.8 Verify SSH within bu1 Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

SSH to the Private IP of the instance **_azure-west-us-spoke1-test1_** in Azure. Despite the fact that the instance is within the same Smart Group "bu1", the SSH will fail due to the absence of a dedicated rule that would permit SSH traffic within the Smart Group.

```{figure} images/lab10-sshfail.png
---
align: center
---
SSH fails
```

### 5.9 Add a rule that allows SSH in bu1

Create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newrule2.png
---
align: center
---
New rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

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

Click on `"Commit"` to enforce the new rule into the **Data Plane**.

```{figure} images/lab10-commitsshbu1.png
---
height: 300px
align: center
---
Commit
```

## 6. Verification after the intra-rule 

### 6.1 Verify SSH within bu1 Using the Gatus App

Open the Gatus App on **_aws-us-east-2-spoke1-test1_** and verify the SSH Traffic.

```{figure} images/lab10-gatus97.png
---
align: center
---
Gatus
```

**_aws-us-east-2-spoke1-test1_** can now SSH to **_azure-west-us-spoke1-test1_**

### 6.2 Verify SSH within bu1 Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Try once again to SSH to the Private IP of the instance **_azure-west-us-spoke1-<span style='color:red'>test1</span>_** in Azure in BU1.

This time the connection will be established, thanks to the new intra-rule.

```{figure} images/lab10-sshbu1ok.png
---
align: center
---
SSH ok
```

## 7. Logs

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and filter out based on the **intra-ssh-bu1** Rule!

```{figure} images/lab10-logsshbu187.png
---
height: 150px
align: center
---
Logs 
```

```{figure} images/lab10-logsshbu1.png
---
height: 150px
align: center
---
Logs 
```

The log above clearly indicates that the `"intra-ssh-bu1"` rule is successfully permitting SSH traffic within the Smart Group bu1.

Following the creation of this intra-rule, the topology with the permitted protocols will appear as follows:

```{figure} images/lab10-topologynew.png
---
height: 400px
align: center
---
New Topology
```

## 8.  Verification inside bu2

### 8.1 Verify ICMP traffic within bu2 Using Gatus App

Open the Gatus App on **_gcp-us-central1-spoke1-test1_** and verify the ICMP Traffic.

```{figure} images/lab10-gatus34.png
---
align: center
---
Gatus
```

Ping towards the **_azure-west-us-spoke2-test1_** and **_aws-us-east-2-spoke1-test2_** will work, because these two instance belongs to the same Smart Group bu2!

### 8.2 Verify ICMP traffic within bu2 Using SSH Client<span style='color:#33ECFF'>(BONUS)</span></summary>

SSH to the Public IP of the instance **_gcp-us-central1-spoke1-test1_**:

```{figure} images/lab10-sshtocentral.png
---
align: center
---
SSH to gcp-us-central1-spoke1-test1
```

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

### 8.3 Logs Verification

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and filter based on the _intra-icmp-bu2_ rule.

```{figure} images/lab10-bu23monitor.png
---
height: 150px
align: center
---
intra-icmp-bu2
```

```{figure} images/lab10-bu2monitor.png
---
height: 150px
align: center
---
Monitor
```

The logs above confirm that the **ICMP** protocol is permitted within the Smart Group bu2.

## 9. Inter-rule from bu2 to bu1

Create a new rule that allows **ICMP** <span style='color:green'>FROM</span></summary> bu2 <span style='color:green'>TO</span></summary> bu1.

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

Enforce this new rule into the Data Plane clicking on the `"Commit"` button.

### 9.1 Verify ICMP traffic from bu2 to bu1 Using the Gatus App

Open the Gatus App on **_azure-west-us-spoke2-test1_** and verify the ICMP Traffic.

```{figure} images/lab10-newcommit2.png
---
align: center
---
Gatus
```

All pings will be successful except those to the **aws EAST-1** region, as the lack of a **full-mesh** topology impacts connectivity.

### 9.2 Verify ICMP traffic from bu2 to bu1 Using SSH Client<span style='color:#33ECFF'>(BONUS)</span></summary>

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

**AWS US-EAST-1** region is not reachable yet!

```{figure} images/lab10-pingallokk.png
---
align: center
---
Ping fails
```

## 10. Logs Monitor

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

Filter out based on the **inter-icmp-bu2-bu1** Rule!

```{figure} images/lab10-monitorfresh.png
---
height: 150px
align: center
---
Monitor
```

The logs clearly show that the inter-rule is successfully allowing ICMP traffic from bu2 to bu1.

After establishing this inter-rule, the topology with all permitted protocols will be represented as follows:

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
From-To
```

The inter-rule is **Stateful**, meaning it allows the echo-reply generated from bu1 to reach the instance in bu2.

## 11. East-1 and the Multi-Tier Transit

### 11.1 Activation of the MTT

Let's now include the AWS region **US-EAST-1**.

This time, you need to permit ICMP traffic exclusively between the Smart Group **bu2** and the EC2 instance **_aws-us-east-1-spoke1-test2_**.

```{figure} images/lab10-newtopology3.png
---
height: 400px
align: center
---
New Topology
```

### 11.2 Verification of ICMP traffic between Azure and AWS Using Gatus App

Open the Gatus App on **_azure-west-us-spoke2-test1_** and verify the ICMP Traffic.

```{figure} images/lab10-gatus334.png
---
align: center
---
Gatus
```

Ping towards the **_aws-us-east-1-spoke1-test1_** and **_aws-us-east-1-spoke1-test2_** will NOT work.

### 11.3 Verification ICMP between Azure and AWS Using SSH Client<span style='color:#33ECFF'>(BONUS)</span></summary>

SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_**.

Ping the following instance:

- **aws-us-east-1-spoke1-test2** in AWS

```{figure} images/lab10-pingfails10.png
---
align: center
---
Ping
```

The ping fails!

## 12. Verification of the MTT

Let’s check the routing table of the Spoke Gateway **_azure-west-us-spoke2_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the gateway **_azure-west-us-spoke2_**

```{figure} images/lab10-spoke2azure.png
---
align: center
---
azure-west-us-spoke2
```

Then click on the `"Gateway Routes"` tab and check whether the destination route (i.e. **10.0.12.0/23**) is present in the routing table or not.

```{figure} images/lab10-gatewayroutes.png
---
align: center
---
Gateway Routes
```

```{figure} images/lab10-newjoe20.png
---
height: 300px
align: center
---
10.0.12.0
```

```{note}
The destination route is **NOT** present in the routing table because the Transit Gateway in the AWS US-EAST-1 region has only one peering connection with the Transit Gateway in the AWS US-EAST-2 region. As a result, the Controller will only install the routes from US-EAST-1 into the routing tables of the Gateways in AWS US-EAST-2, excluding the other Gateways in the MCNA. If you want to distribute the routes from the AWS US-EAST-1 region throughout the entire MCNA, you have <ins>two options</ins>:
```

- Enabling `"Full-Mesh"` on the Transit Gateways in **_aws-us-east1-transit_** VPC

    **OR**

- Enabling `"Multi-Tier Transit"`

Let’s enable the **MTT** feature and observe its behavior in action!

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
height: 300px
align: center
---
Border Gateway Protocol (BGP)
```

Repeat the previous action for the last Transit Gateway still without a BGP ASN configured properly:

- **azure-west-us-transit**: <span style='color:#479608'>ASN **64515**</span>

```{figure} images/lab10-newlab.png
---
align: center
---
azure-west-us-transit
```

```{note}
Both the **aws-us-east-2-transit** and **gcp-us-central1-transit** have already been configured with their ASNs during Lab 8!
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east-2-transit_**.

```{figure} images/lab10-mtt3.png
---
align: center
---
aws-us-east-2-transit
```

Navigate to the `"Settings"` tab, expand the `"General"` section, and activate `"Multi-Tier Transit"` by toggling the corresponding switch.

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

This time if you click on the `"Gateway Routes"` tab, you will be able to see the destination route, **10.0.12.0/23**, in **aws-us-east1-spoke1** VPC.

```{figure} images/lab10-mtt6.png
---
height: 300px
align: center
---
10.0.12.0/23
```

```{hint}
Click on `"+ 230 more"`
```

## 13. The last DCF rule

### 13.1 ICMP Verification  traffic between Azure and AWS Using Gatus App after enabling MTT

Open the Gatus App on **_azure-west-us-spoke2-test1_** and verify the ICMP Traffic.

```{figure} images/lab10-gatus334.png
---
align: center
---
Gatus
```

Pinging **_aws-us-east-1-spoke1-test1_** and **_aws-us-east-1-spoke1-test2_** is still not working.

### 13.2 ICMP Verification between Azure and AWS Using SSH Client after enabling MTT<span style='color:#33ECFF'>(BONUS)</span></summary>

- SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_**.

Ping the following instance:

- **aws-us-east-1-spoke1-test2** in AWS (refer to your personal POD portal for the private IP).

```{figure} images/lab10-mtt7.png
---
align: center
---
Ping
```

## 14. Final Considerations

Although there is now a valid route to the destination, thanks to the **MTT** feature, the pings are still failing.

```{warning}
The reason is that the EC2 instance **aws-us-east-1-spoke1-test2** has not been allocated to any Smart Groups yet.
```

### 14.1 Smart Group “east1”

Let’s create an additional Smart Group for the test instance **_aws-us-east-1-spoke1-test2_** in the US-EAST-1 region of AWS.

Go to **Copilot > Groups > SmartGroups** and click on  `"+ SmartGroup"` button.

```{figure} images/lab10-mttnew.png
---
align: center
---
New Smart Group
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>east1</span>
- **Virtual Machines/Name**: <span style='color:#479608'>aws-us-east-1-spoke1-test2</span>

```{figure} images/lab10-mtt9.png
---
align: center
---
Resource Selection
```

CoPilot indicates that there is only one instance that matches the condition:

- aws-us-east-1-spoke1-test2 in AWS

Make sure to click **Save** once you're done.

### 14.2 Create an inter-rule that allows ICMP from bu2 towards east1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create another rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-mtt8.png
---
align: center
---
New Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

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

### 14.3 Verify connectivity between bu2 and east1

#### 14.3.1 ICMP Verification  traffic between Azure and AWS Using Gatus App after enabled MTT

Open the Gatus App on **_azure-west-us-spoke2-test1_** and verify the ICMP Traffic to **_aws-us-east-1-spoke1-test2_**

```{figure} images/lab10-gatus345.png
---
align: center
---
Gatus
```

The ICMP traffic test will gradually turn green!

#### 14.3.2 ICMP Verification between Azure and AWS Using SSH Client after enabled MTT<span style='color:#33ECFF'>(BONUS)</span></summary>

- SSH to the Public IP of the instance **_azure-west-us-spoke2-test1_** and ping the private IP of the ec2-instance **_aws-us-east-1-spoke1-test2_**

```{figure} images/lab10-lastping.png
---
align: center
---
Ping
```

This time, the ping will be successful!

#### 14.3.3 Logs

Check the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

```{figure} images/lab10-reallylast.png
---
height: 200px
align: center
---
inter-icmp-bu2-east1 Logs
```

After creating both the previous inter-rule and the additional Smart Groups, the topology with all permitted protocols will appear as follows:

```{figure} images/lab10-newjoe.png
---
height: 400px
align: center
---
Final Topology
```

## 15. Spoke to Spoke Attachment

Now that you have enabled the Distributed Cloud Firewall, the owner of the **_azure-west-us-spoke2-test1_** VM would like to communicate directly with the nearby **_azure-west-us-spoke1-test1_** VM, avoding that the traffic generated from the VNet is sent to the NGFW, first.

```{figure} images/lab10-spoke2spoke01.png
---
align: center
---
No More NGFW
```

### 15.1 Creating a Spoke to Spoke Attachment

Go to **Copilot > Cloud Fabric > Gateways > Spoke Gateways**, locate the **_azure-west-us-spoke2_** gateway and click on the **`Manage Gateway Attachments`** icon on the right-hand side.

```{figure} images/lab10-spoke2spoke02.png
---
height: 200px
align: center
---
Manage Gateway Attachments
```

Select the **Spoke Gateway** tab, click on the `"+ Attachment"` button and then choose the **azure-west-us-spoke1** GW from the drop-down window.

```{figure} images/lab10-spoke2spoke03.png
---
align: center
---
azure-west-us-spoke2
```

```{figure} images/lab10-newspokeatt.png
---
align: center
---
Save
```

Do not forget to click on **Save**.

Now go to **CoPilot > Cloud Fabric > Topology** and check the new attachment between the two Spoke Gateways in Azure.

```{figure} images/lab10-spoke2spoke04.png
---
align: center
---
Spoke to Spoke Attachment
```

```{caution}
It will take approximately **2** minutes to reflect into the Dynamic Topology.
```

Let's check the **Routing Table** of the **_Spoke2_** in Azure.

Go to **CoPilot > Cloud Fabric > Gateways**, select the **azure-west-us-spoke2**, then select the **Gateways Routes** tab and search for the subnet **`192.168.1.0`** on the right-hand side.

```{figure} images/lab10-spoke2spoke05.png
---
align: center
---
azure-west-us-spoke2
```

You will notice that the destination is now reachable with a **lower** metric (50)!

```{figure} images/lab10-spoke2spoke06.png
---
height: 300px
align: center
---
Metric 50
```

The traffic generated from the **_azure-west-us-spoke2-test1_** VM will now prefer going through the Spoke-to-Spoke Attachment for communication with the Spoke1 VNet.

```{important}
The Aviatrix Cloud Fabric is very flexible and does not lock you in with solely a _Hub and Spoke_ Topology!
```

```{figure} images/lab10-spoke2spoke07.png
---
align: center
---
Spoke to Spoke
```

After completing this lab, the overall topology will appear as follows:

```{figure} images/lab10-lastdrawing.png
---
height: 400px
align: center
---
Full-Blown Aviatrix Solution
```