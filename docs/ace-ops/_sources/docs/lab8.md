# Lab 8 - Distributed Cloud Firewall

## 1. SCENARIO

In the previous lab, you initiated micro-segmentation by defining a SmartGroup and applying the `Aviatrix Cloud Firewall` rules. Right now, _East–West_ traffic is largely blocked, except for the policy that allows **BU1 Frontend** to SSH to both **BU1 DB** and **BU2 DB**.

```{figure} images/lab7-segmentation.png
---
height: 400px
align: center
---
Initial Topology
```

```{important}
From a routing perspective, there is a flat rouing domain, enabled by the `Connection Policy` applied in Lab 2.
```

This is the current list of Distributed Cloud Firewall policies.

```{figure} images/lab8-initialrule.png
---
height: 300px
align: center
---
Existing DCF rules
```

These are the **requirements** for this lab:

1) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU1 Analytics

2) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU2 Mobile App.

3) Create an <span style='color:lightgreen'>**intra-rule**</span> that allows BU1 Frontend and BU2 Mobile App to ping each other

4) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 Frontend to talk with BU2 Mobile App on TCP/80

5) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 Analytics to ping BU1 Frontend

6) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 DB to communicate with BU2 DB on TCP/22

## 2. CHANGE REQUEST

### 2.1 SmartGroups creation

Navigate to **CoPilot > Groups > SmartGroups**, and click  the `“+ SmartGroup”` button.

```{figure} images/lab8-SGnew10.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-ANALYTICS</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-gcp-us-east1-spoke1-bu1-analytics</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-gcp-us-east1-spoke1-bu1-analytics
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab8-smart41.png
---
align: center
---
Resource Selection
```

```{figure} images/lab8-smart411.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-gcp-us-east1-spoke1-bu1-analytics_**

Add another Smart Group by tapping the `“+ SmartGroup”` one more time.

```{figure} images/lab8-SGnew20.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-MOBILEAPP</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-aws-eu-west-1-spoke2-bu2-mobile-app</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-aws-eu-west-1-spoke2-bu2-mobile-app
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab8-smart51.png
---
align: center
---
Resource Selection
```

```{figure} images/lab8-smart511.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-aws-eu-west-1-spoke2-bu2-mobile-app_**

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/lab8-greenfield56661.png
---
align: center
---
New Rule
```

### 2.2 Distributed Cloud Firewall Policies

Let's begin defining the **DCF rules** to govern `East–West` traffic.

#### 2.2.1 Intra-rule between BU1 Frontend and BU2 Mobile App

The current SmartGroups status is depicted below. The first rule to create is an `intra-rule` involving both **BU1 Frontend** and **BU2 Mobile App**. To enforce this intra-rule, define an additional SmartGroup that can include both instances, for example by using the label `Region=eu-west-1`.

```{figure} images/lab71-segmentation.png
---
height: 400px
align: center
---
Current SmartGroup status
```

- Navigate to **CoPilot > Groups > SmartGroups** and click the `“+ SmartGroup”`.

```{figure} images/lab8-SGnew2090.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>EU-WEST-1</span>
- **Matches all conditions (AND)/Region**: <span style='color:#479608'>eu-west-1</span>

```{figure} images/lab8-smart3.png
---
align: center
---
Region = eu-west-1
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab8-smart5122.png
---
align: center
---
Resource Selection
```

```{figure} images/lab8-smart51132.png
---
align: center
---
Preview
```

The CoPilot shows that there are three instances that perfectly match the condition:

- **_ACE-FW_**
- **_ace-aws-eu-west-1-spoke1-bu1-frontend_**
- **_ace-aws-eu-west-1-spoke2-bu2-mobile-app_**

The new SmartGroup encompasses the two instances involved in establishing an _intra-rule_ between **BU1 Frontend** and **BU2 Mobile App**, as illustrated in the figure below.

```{figure} images/lab712-segmentation.png
---
height: 400px
align: center
---
Topology with the latest SmartGroup
```

```{important}
Instances can belong to multiple SmartGroups.
```
Enter the following parameters:

- **Name**: <span style='color:#479608'>intra-icmp-bu1frontend-bu2mobileapp</span>
- **Source Smartgroups**: <span style='color:#479608'>EU-WEST-1</span>
- **Destination Smartgroups**: <span style='color:#479608'>EU-WEST-1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
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

- SSH on the **BU1 Frontend** and try to SSH to any other instances in BU1.
  - SSH fails as expected.

```{figure} images/lab8-sshbu1.png
---
align: center
---
SSH fails within BU1
```

- Create an intra-rule that allows SSH **within** BU1 and then verify that SSH is permitted among BU1’s instances. 
  Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-rule01.png
---
height: 200px
align: center
---
Another intra-rule
```

- Retry to SSH to the BU1 Analytics from the BU1 Frontend; this time the operation will be accomplished!

```{figure} images/lab8-sshok.png
---
align: center
---
SSH is ok within BU1
```

```{figure} images/lab8-intrassh-bu1.png
---
height: 400px
align: center
---
Intra-SSH BU1
```

- Check the **Logs** generated after the action carried out before.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**
```

```{figure} images/lab8-monitornew.png
---
height: 100px
align: center
---
Monitor: Logs
```

- Now terminate the SSH session with the BU1 Analytics, typing `"exit"`, and issue the **ping** command towards the BU1 Analytics from the BU1 Frontend. The ping will not work!

```{figure} images/lab8-pingbu1.png
---
align: center
---
Ping will fail
```

- Check again the **Logs** from the `Monitor` section. The ICMP attempt hit the **_Explicit-Deny-Rule_**, whereby the action was denied!

```{figure} images/lab8-denyicmp.png
---
height: 100px
align: center
---
Monitor: Denied ICMP
```

- Create another intra-rule that allows ICMP **within** BU1 and then verify that ICMP is permitted among BU1’s instances. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-icmprulebu1.png
---
height: 200px
align: center
---
New intra-rule
```

- The ping will work this time thanks to this new fresh intra-rule!

```{figure} images/lab8-icmprulebu1ok.png
---
align: center
---
Ping will be ok!
```

- Once again, inspect the **Monitor** section to find out immediately the corresponding *logs* of the action carried out before.

```{figure} images/lab8-pingpermitted.png
---
height: 100px
align: center
---
Monitor: successful logs
```

```{figure} images/lab8-intrasshicmp-bu1.png
---
height: 400px
align: center
---
Intra-SSH-ICMP BU1
```

- Let's try to ping the BU2 Mobile App from the BU1 Frontend. The ping will fail due to the absence of an **inter-rule**.

```{figure} images/lab8-pingbu1bu2.png
---
align: center
---
Ping will fail
```

- This time create an inter-rule that allows ICMP **from** BU1 **towards** BU2. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-icmp-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-penultimaterule.png
---
height: 200px
align: center
---
DCF Rules List
```

- Retry to ping the BU2 Mobile App from the BU1 Frontend. The ping will be successful thanks to the inter-rule!

```{figure} images/lab8-penultimateping.png
---
align: center
---
Ping from BU1 to BU2
```

```{figure} images/lab8-intericmp-bu1bu2.png
---
height: 400px
align: center
---
Inter-ICMP BU1 to BU2
```

- Now, let's SSH to the Public IP of the **BU2 Mobile App** and then try to SSH to the Private IP of the **BU1 Frontend**. Of course, SSH will fail!

```{figure} images/lab8-bu2sshbu1.png
---
align: center
---
SSH from BU2 to BU1 will fail
```

- Create another inter-rule that allows SSH **from** BU2 **towards** BU1. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU2</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-lastrule.png
---
height: 200px
align: center
---
"inter-ssh-bu2-bu1" Rule
```

- Let's try to issue the SSH command from the BU2 Mobile App towards the BU1 Frontend. This time the SSH will work smoothly.

```{figure} images/lab8-lastssh.png
---
align: center
---
SSH works from BU2 to BU1
```

```{figure} images/lab8-interssh-bu2bu1.png
---
height: 400px
align: center
---
Inter-SSH BU2 to BU1
```

## 3. CHANGE REQUEST

- Now before completing the lab, remove the `Inspection Policy` from the Transit GW in AWS. 

The Aviatrix Distributed Cloud Firewall has been enabled across the whole multicloud infrastructure, therefore we can get rid of the bolted Firewall in AWS.

```{tip}
Go to **CoPilot > Security > FireNet > FireNet Gateways** and click on the **_ace-aws-eu-west-1-transit1_** GW, then click on `Policy`, select the two Spoke VPCs in AWS and choose the `Remove` action.

Traffic will be not be sent towards the FW anymore. Th DCF will perform the firewalling very close to the source!
```{figure} images/lab8-manageatt.png
---
align: center
---
FireNet cfg
```

```{figure} images/lab8-lastone.png
---
align: center
---
Remove the inspection Policy
```

Congratulations, you have completed all labs and created a nice set of DCF rules across your Multicloud infrastructure!
