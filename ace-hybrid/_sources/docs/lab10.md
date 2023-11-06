# Lab 10 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` work.

 
## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall feature allows to create logical containers, called `Smart Groups`, that encompass instances that present similarities inside a VPC/VNet/VCN, and then they also allow to enforce rules (aka **_Distributed Cloud Firewalling Rules_**) within a Smart Group (i.e. the `intra-rule`) or among Smart Groups (i.e. the `inter-rule`).

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the <span style='color:lightgreen'>Green</span> domain and the <span style='color:lightblue'>Blue</span> domain.
```

All the Test instances have been deployed with the typical <ins>CSP tags</ins>. 

```{important}
The CSP tagging is the recommended method for defining the SmartGroups.
```

In this lab you are asked to achieve the following requirements among the instances deployed across the three CSPs:

- Create a Smart Group with the name `"bu1"` leveraging the tag `"environment"`
- Create a Smart Group with the name `"bu2"` leveraging the tag `"environment"`
- Create an `intra-rule` that allows ICMP traffic within bu1
- Create an `intra-rule` that allows ICMP traffic within bu2
- Create an `inter-rule` that allows ICMP traffic only from bu1 towards bu2

![lab10-initial](images/lab10-initial.png)
_Figure 88: Edit the Explicit-Deny-Rule_

## 3. Smart Group Creation

Create two Smart Groups and classify each Smart Group, leveraging the CSP tag `"environment"`:

- Assign the name `"bu1"` to the Smart Group **#1**.
- Assign the name `"bu2"` to the Smart Group **#2**.

### 3.1. Smart Group “bu1”

Go to **CoPilot > SmartGroups** and click on `"+ SmartGroup"`.

![lab10-smart2](images/lab10-smart2.png)
_Figure 88: +SmartGroup_

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>bu1</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>environment</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>bu1</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Resource Selection"`.

![lab10-smart3](images/lab10-smart3.png)
_Figure 88: +SmartGroup_

The CoPilot shows that there are two instances that perfectly match the condition:

- **aws-us-east2-spoke1-test1** in AWS
- **azure-us-west-spoke1-test1** in Azure

![lab10-smart4](images/lab10-smart4.png)
_Figure 88: +SmartGroup_

### 3.2. Smart Group “bu2”

Create another Smart Group clicking on the `"+ SmartGroup"` button.

![lab10-smart5](images/lab10-smart5.png)
_Figure 88: +SmartGroup_

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>bu2</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>environment</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>bu2</span>

Before clicking on **SAVE**, discover what instances match the condition, turning on the knob `"Resource Selection"`.

![lab10-smart6](images/lab10-smart6.png)
_Figure 88: +SmartGroup_

The CoPilot shows that there are three instances that match the condition:

- **aws-us-east2-spoke1-test2** in AWS
- **azure-us-west-spoke2-test1** in Azure
- **gcp-us-central1-spoke1-test1** in GCP

![lab10-smart7](images/lab10-smart7.png)
_Figure 88: +SmartGroup_

At this point, you have only created logical containers that do not affect the existing routing domain.

Let's verify that everything has been kept unchanged! Bear in mind that there is the `Greenfield-Rule` at the very top of your DCF rules list, whereby all kind of traffic will be permitted.

### 3.3. Connectivity verification (ICMP)

Open a terminal window and SSH to the public IP of the instance **aws-us-east2-spoke1-test1**, and from there ping the private IP of each other instances to verify that the connectivity within AWS, and from AWS to GCP and Azure has not been modified.

```{note}
Refer to your POD for the private IPs.
```

![lab10-ping](images/lab10-ping.png)
_Figure 88: +SmartGroup_

### 3.4.  Connectivity verification (SSH)

Verify also from the instance **aws-us-east2-spoke1-test1** that you can SSH to the private instance in AWS, to the instance in GCP and likewise to the other two instances in Azure.

```{note}
Refer to your POD for the private IPs.
```

![lab10-sshtoaws](images/lab10-sshtoaws.png)
_Figure 88: +SmartGroup_

![lab10-sshtogcp](images/lab10-sshtogcp.png)
_Figure 88: +SmartGroup_

![lab10-sshtoazure1](images/lab10-sshtoazure1.png)
_Figure 88: +SmartGroup_

![lab10-sshtoazure1](images/lab10-sshtoazure2.png)
_Figure 88: +SmartGroup_

The previous outcomes confirm undoubtetly that the connectivity is working smoothly, despite the creation of those two new Smart Groups.

## 4. Rules Creation
### 4.1. Establish a ZTN approach

First and foremost, let's move the `Explicit-Deny-Rule` at the very top of the list of your DCF rules.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default)**, click on the the "two arrows" icon on the righ-hand side of the `Explicit-Deny-Rule` and choose *`"Move Rule"`* at the very Top. Then click on **Save in Draft**.
```

![lab10-explicit](images/lab10-explicit.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Then **commit** your change!

![lab10-commit](images/lab10-commit.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 4.2. Create an intra-rule that allows ICMP inside bu1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

![lab10-newrule](images/lab10-newrule.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Insert the following parameters:

- **Name**: <span style='color:#33ECFF'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

![lab10-rule1](images/lab10-rule1.png)
_Figure 88: Edit the Explicit-Deny-Rule_

At this point, there would be one uncommitted rule at the very top, as depicted below.

![lab10-rule2](images/lab10-rule2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 4.2. Create an intra-rule that allows ICMP inside bu2

Create another rule clicking on the `"+ Rule"` button.

![lab10-rule3](images/lab10-rule3.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>intra-icmp-bu2</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>**Below**</span>
  - **Existing Rule**: <span style='color:#33ECFF'>**intra-icmp-bu1**</span>

Do not forget to click on **Save In Drafts**.

![lab10-rule4](images/lab10-rule4.png)
_Figure 88: Edit the Explicit-Deny-Rule_

At this point, you will have two new rules marked as `New`, therefore you can proceed and click on the **Commit** button.

![lab10-rule5](images/lab10-rule5.png)
_Figure 88: Edit the Explicit-Deny-Rule_

## 5. Verification

Afte the creation of the previous Smart Groups and Rules, this is how the topology with the permitted protocols should look like:

![lab10-topology2](images/lab10-topology2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 5.1. Verify SSH traffic from your laptop to bu1

SSH to the Public IP of the instance **aws-us-east2-spoke1-test1**.

![lab10-sshpod](images/lab10-sshpod.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 5.2. Verify ICMP within bu1 and from bu1 towards bu2

Ping the following instances from **aws-us-east2-spoke1-test1**:

- **gcp-us-central1-spoke1-test1** in GCP
- **azure-us-west-spoke1-test1** in Azure
- **azure-us-west-spoke2-test1** in Azure

According to the rules created before, only the ping towards the **azure-us-west-spoke1-test1** will work, because this instance belongs to the same Smart Group bu1 as the instance from where you will be launching ICMP packets.

![lab10-pingcheck](images/lab10-pingcheck.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Let's investigate the logs:

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

![lab10-pingcheck](images/lab10-monitor.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Now, let's try to ping the instance **aws-us-east2-spoke1-test2** from **aws-us-east2-spoke1-test1**. 

```{warning}
The instance **aws-us-east2-spoke1-test1** is in the same VPC. Although these two instances have been deployed in two distinct and separate VPCs, the communication will occur until you don't enable the `"intra-vpc separation"`.
```

![lab10-pingtotest2](images/lab10-pingtotest2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Go to **CoPilot > Security > Distributed Cloud Firewall > Settings** and click on the `"Manage"` button inside the `"Security Group (SG) Orchestration"` field.

![lab10-orchestration](images/lab10-orchestration.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Enable the SG orchestration on the **_aws-us-east2-spoke1_** VPC, then put a tick on the checkbox `"I understand the network impact"` and click on **Save**.

![lab10-orchestration](images/lab10-orchestration2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Relaunch the ping the instance towards **aws-us-east2-spoke1-test2** from **aws-us-east2-spoke1-test1**. 

![lab10-orchestration](images/lab10-pingtotest2fail.png)
_Figure 88: Edit the Explicit-Deny-Rule_

```{important}
This time the ping fails. You have achieved a complete separation between SmartGroups deployed in the same VPC in AWS US-EAST-2
```

### 5.3. Verify SSH within bu1

SSH to the Private IP of the instance **_azure-us-west-spoke1-test1_** in Azure. Despite the fact that the instance is within the same Smart Group "bu1", the SSH will fail due to the absence of a rule that would permit SSH traffic within the Smart Group.

![lab10-sshfail](images/lab10-sshfail.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 5.4. Add a rule that allows SSH in bu1

Create another rule clicking on the `"+ Rule"` button.

![lab10-newrule2](images/lab10-newrule2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Protocol**: <span style='color:#33ECFF'>TCP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>**Below**</span>
  - **Existing Rule**: <span style='color:#33ECFF'>**intra-icmp-bu2**</span>

Do not forget to click on **Save In Drafts**.

![lab10-sshbu1](images/lab10-sshbu1.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Click on `"Commit"` to enforce the new rule in the **Data Plane**.

![lab10-commitsshbu1](images/lab10-commitsshbu1.png)
_Figure 88: Edit the Explicit-Deny-Rule_

- Try once again to SSH to the Private IP of the instance **_azure-us-west-spoke1-test1_** in Azure in BU1.

This time the connection will be established, thanks to the new intra-rule.

![lab10-sshbu1ok](images/lab10-sshbu1ok.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

![lab10-logsshbu1](images/lab10-logsshbu1.png)
_Figure 88: Edit the Explicit-Deny-Rule_

From the log above is quite evident that the `"intra-ssh-bu1`" rule is permitting SSH traffic within the SmartGroup bu1, successfully.

After the creation of the previous intra-rule, this is how the topology with the permitted protocols should look like:

![lab10-topologynew](images/lab10-topologynew.png)
_Figure 88: Edit the Explicit-Deny-Rule_

5.4. SSH to VM in bu2

SSH to the Public IP of the instance gcp-us-central1-spoke1-test1: