# Lab 8 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` work.

 
## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall feature allows to create logical containers, called `Smart Groups`, that encompass instances that present similarities inside a VPC/VNet/VCN, and then it also allows to enforce rules (aka **_Distributed Cloud Firewalling Rules_**) within a Smart Group (i.e. the `intra-rule`) or among Smart Groups (i.e. the `inter-rule`).

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the <span style='color:lightgreen'>Green</span> domain and the <span style='color:lightblue'>Blue</span> domain.
```

All the Test instances have been deployed with the typical <ins>CSP tags</ins>. 

```{important}
The CSP tagging is the recommended method for defining the SmartGroups.
```

In this lab you are asked to achieve the following requirements among the instances deployed across the three CSPs:

- Create a Smart Group with the name `"bu1"` leveraging the tag `"environment"`.
- Create a Smart Group with the name `"bu2"` leveraging the tag `"environment"`.
- Create an `intra-rule` that allows ICMP traffic within bu1.
- Create an `intra-rule` that allows ICMP traffic within bu2.
- Create an `inter-rule` that allows ICMP traffic only from bu1 towards bu2.

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
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - **Existing Rule**: <span style='color:#33ECFF'>intra-icmp-bu1</span>
  
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

### 5.4. SSH to VM in bu2

SSH to the Public IP of the instance **_gcp-us-central1-spoke1-test1_**:

![lab10-sshtocentral](images/lab10-sshtocentral.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 5.5. Verify ICMP traffic within bu2

Ping the following instances:

- **aws-us-east2-spoke1-test1** in AWS
- **aws-us-east2-spoke1-test2** in AWS
- **azure-us-west-spoke1-test1** in Azure
- **azure-us-west-spoke2-test1** in Azure

According to the rules created before, only the ping towards the **azure-us-west-spoke2-test1** and **aws-us-east2-spoke1-test2** will work, because these two instance belongs to the same Smart Group bu2 as the instance from where you will be launching the ICMP packets.

![lab10-pingtestgcp](images/lab10-pingtestgcp.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

![lab10-bu2monitor](images/lab10-bu2monitor.png)
_Figure 88: Edit the Explicit-Deny-Rule_

The logs above confirm that the ICMP protocol is permitted within the SmartGroup bu2.
 
### 5.6. Inter-rule between bu1 and bu2

Create a new rule that allows ICMP between bu1 and bu2.

Go to **CoPilot > Security > Distributed Firewalling > Rules and click on the `"+ Rule"` button.

![lab10-newrule4](images/lab10-newrule4.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>inter-icmp-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - **Existing Rule**: <span style='color:#33ECFF'>intra-ssh-bu1</span>
  
Do not forget to click on **Save In Drafts**.

![lab10-interssh](images/lab10-interssh.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Enforce the this new rule in the data plane clicking on the `"Commit"` button.

![lab10-newcommit2](images/lab10-newcommit2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

SSH to the Public IP of the instance **_aws-us-east2-spoke1-test1_**.

Ping the following instances:
- **aws-us-east2-spoke1-test2** in AWS
- **gcp-us-central1-spoke1-test1** in GCP
- **azure-us-west-spoke1-test1** in Azure
- **azure-us-west-spoke2-test1** in Azure

Thit time all pings will be successful, thanks to the inter-rule applied between bu1 and bu2.

![lab10-missingping](images/lab10-lastdrawing.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Let's investigate the logs once again.

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**

![lab10-missingmonitor](images/lab10-lastdrawing.png)
_Figure 88: Edit the Explicit-Deny-Rule_

The logs clearly demonstrate that the inter-rule is successfully permitting ICMP traffic from bu1 to bu2.

After the creation of the previous inter-rule, this is how the topology with all the permitted protocols should look like.

![lab10-newtopology2](images/lab10-newtopology2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

```{note}
The last inter-rule works smoothly only because the ICMP traffic is generated from the bu1, however, if you SSH to any instances in the Smart Group bu2, the ICMP traffic towards bu1 will fail due to the direction of the inter-rule that was created before: **FROM** bu1 **TO** bu2 (please note the direction of the arrow in the drawing).
```

![lab10-direction](images/lab10-direction.png)
_Figure 88: Edit the Explicit-Deny-Rule_

The inter-rule is Stateful in the sense that it will permit the echo-reply generated from the bu2 to reach the instance in bu1.
 
## 6. East-1 and the Multi-Tier Transit

### 6.1 Activation of the MTT

Let’s now also involve the AWS region **US-EAST-1**.

This time, you have to allow the ICMP traffic between the Smart Group **bu2** and the ec2 instance **_aws-us-east1-spoke1-test2_**, solely.

![lab10-newtopology3](images/lab10-newtopology3.png)
_Figure 88: Edit the Explicit-Deny-Rule_

SSH to the Public IP of the instance **_azure-us-west-spoke2-test1_**.

Ping the following instance:

- **aws-us-east1-spoke1-test2** in AWS

![lab10-newtopology3](images/lab10-pingfails10.png)
_Figure 88: Edit the Explicit-Deny-Rule_

The ping fails, therefore, let’s check the routing table of the Spoke Gateway **_azure-us-west-spoke2_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the relevant gateway **_azure-us-west-spoke2_**

![lab10-spoke2azure](images/lab10-spoke2azure.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Then click on the `"Gateway Routes"` tab and check whether the destination route is present in the routing table or not.

![lab10-gatewayroutes](images/lab10-gatewayroutes.png)
_Figure 88: Edit the Explicit-Deny-Rule_

```{note}
The destination route is not inside the routing table, due to the fact that the Transit Gateway in AWS us-east-1 region has only one peering with the Transit Gateway in us-east-2, therefore the Controller will install the routes that belong to us-east-1 only inside the routing tables of the Gateways in AWS us-east-2, excluding the rest of the Gateways of the MCNA. If you want to distribute the routes from AWS us-east-1 region in the whole MCNA, you have two choices:
```

- Enabling `"Full-Mesh"` on the Transit Gateways in **_aws-us-east1-transit_** VPC

    **OR**

- Enabling `"Multi-Tier Transit"`

Let’s enable this time the MTT feature!

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east1-transit_**.

![lab10-mtt](images/lab10-mtt.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Go to `"Settings"` tab and expand the `"“Border Gateway Protocol (BGP)”` section and insert the AS number **64512** on the empty field related to the `"“Local AS Number”`, then click on **Save**.

![lab10-mtt2](images/lab10-mtt2.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Repeat the previous action for the remaning Transit Gateways:

- **aws-us-east2-transit**: <span style='color:#33ECFF'>ASN **64513**</span>
- **gcp-us-central1-transit**: <span style='color:#33ECFF'>ASN **64514**</span>
- **azure-us-west-transit**: <span style='color:#33ECFF'>ASN **64515**</span>

Go to C**oPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the Transit Gateway **_aws-us-east2-transit_**.

![lab10-mtt3](images/lab10-mtt3.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Go to `"Settings"` tab and expand the `"General"` section and activate the `"Multi-Tier Transit"`, turning on the corresponding knob. 

Then click on **Save**.

![lab10-mtt4](images/lab10-mtt4.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Let’s verify once again the routing table of the Spoke Gateway in **_azure-us-west-spoke2_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the relevant gateway **_azure-us-west-spoke2_**

![lab10-mtt5](images/lab10-mtt5.png)
_Figure 88: Edit the Explicit-Deny-Rule_

This time if you click on the `"Gateway Routes"` tab, you will be able to see the destination route in **aws-us-east1-spoke1** VPC.

![lab10-mtt6](images/lab10-mtt6.png)
_Figure 88: Edit the Explicit-Deny-Rule_

- SSH to the Public IP of the instance **_azure-us-west-spoke2-test1_**.

Ping the following instance:

- **aws-us-east1-spoke1-test2** in AWS

![lab10-mtt7](images/lab10-mtt7.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Although this time there is a valid route to the destination, thanks to the **MTT** feature, the pings fails. The reason is that the ec2-instance in **_aws-us-east1-spoke1_** VPC is not allocated to any Smart Groups yet.

### 6.2 Smart Group “east1”

Let’s create another Smart Group for the test instance in AWS us-east-1 region.

Go to **Copilot > SmartGroups** and click on  `"+ SmartGroup"` button.

![lab10-mtt8](images/lab10-mtt8.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Ensure these parameters are entered in the pop-up window `"Create New SmartGroup"`:

- **Name**: <span style='color:#33ECFF'>east1</span>
- **CSP Tag Key**: <span style='color:#33ECFF'>Name</span>
- **CSP Tag Value**: <span style='color:#33ECFF'>aws-us-east1-spoke1-test2</span>

![lab10-mtt9](images/lab10-mtt9.png)
_Figure 88: Edit the Explicit-Deny-Rule_

The CoPilot shows that there is just one single instance that matches the condition:

- **aws-us-east1-spoke1-test2** in AWS

Do not forget to click on **Save**.

### 6.3 Create an inter-rule that allows ICMP from bu2 towards east1

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create another rule clicking on the `"+ Rule"` button.

![lab10-mtt10](images/lab10-mtt10.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Ensure these parameters are entered in the pop-up window `"Create New Rule"`:

- **Name**: <span style='color:#33ECFF'>inter-icmp-bu2-east1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>bu2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>east1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Then click on **Save In Drafts**.

```{caution}
Please note the direction of this new inter-rule: **FROM** bu2 **TO** east1
```

![lab10-lastrule](images/lab10-lastrule.png)
_Figure 88: Edit the Explicit-Deny-Rule_

Now you can carry on with the last **commit**!

![lab10-lastcommit](images/lab10-lastcommit.png)
_Figure 88: Edit the Explicit-Deny-Rule_

### 6.4 Verify connectivity between bu2 and east1

- SSH to the Public IP of the instance **_azure-us-west-spoke2-test1_** and ping the private IP of the ec2-instance **_aws-us-east1-spoke1-test2_**

![lab10-lastping](images/lab10-lastping.png)
_Figure 88: Edit the Explicit-Deny-Rule_

This time the ping will be successful.

# Congratulations, you have deployed the full-blown Aviatrix solution!

![lab10-lastdrawing](images/lab10-lastdrawing.png)
_Figure 88: Edit the Explicit-Deny-Rule_