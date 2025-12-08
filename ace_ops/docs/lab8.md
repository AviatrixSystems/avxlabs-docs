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

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/lab8-greenfield56661.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>intra-icmp-bu1frontend-bu2mobileapp</span>
- **Source Smartgroups**: <span style='color:#479608'>EU-WEST-1</span>
- **Destination Smartgroups**: <span style='color:#479608'>EU-WEST-1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab8-greenfield566612.png
---
align: center
---
intra-icmp-bu1frontend-bu2mobileapp
```

Click on **Commit**.

```{figure} images/lab8-greenfield5666123.png
---
align: center
---
Commit
```

- SSH to the **BU1 Frontend** and ping the private IP address of the **BU2 Mobile App**.

```{figure} images/lab712-intraruleinaction00.png
---
height: 400px
align: center
---
intra-rule in action
```

- Now repeat the test: SSH into the **BU2 Mobile App** and ping the private IP of the **BU1 Frontend** to confirm that both instances can generate an ICMP Echo request.

```{figure} images/lab712-intraruleinaction01.png
---
height: 400px
align: center
---
intra-rule in action
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. After completing the previous tests, the logs for the `intra-icmp-bu1frontend-bu2mobileapp` rule will appear.

```{figure} images/lab712-monitor00.png
---
align: center
---
logs
```

This is how the overall topology would look after implementing the latest intra-rule.

```{figure} images/lab712-segmentation090.png
---
height: 400px
align: center
---
intra-rule
```

#### 2.2.2 Inter-rule between BU1 Frontend and BU2 Mobile App

The second policy is an inter-rule, requiring **BU1 Frontend** to communicate with **BU2 Mobile App** via TCP port 80.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/lab8-greenfield13.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-http-bu1frontend-bu2mobileapp</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-MOBILEAPP1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>80</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab8-greenfieldjoe00.png
---
align: center
---
inter-http-bu1frontend-bu2mobileapp
```

Click on **Commit**.

```{figure} images/lab8-greenfieldcommit00.png
---
align: center
---
Commit
```

- SSH into the **BU1 Frontend** and generate a curl command targeting the private IP address of the **BU2 Mobile App**.

```{figure} images/lab712-intraruleinaction22.png
---
height: 200px
align: center
---
inter-rule in action
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. After completing the previous test, the logs for the `inter-http-bu1frontend-bu2mobileapp` rule will appear.

```{figure} images/lab712-monitor01.png
---
align: center
---
logs
```

This is how the overall topology would look after implementing the latest inter-rule.

```{figure} images/lab712-segmentation091.png
---
height: 400px
align: center
---
inter-rule
```

#### 2.2.3 Inter-rule between BU1 Analytics and BU1 Frontend

The third policy is again an inter-rule, requiring **BU1 Analytics** to ping **BU1 Frontend**.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/lab8-greenfield14.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-icmp-bu1analytics-bu1frontend</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-MOBILEAPP1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>80</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab8-greenfieldjoe01.png
---
align: center
---
inter-icmp-bu1analytics-bu1frontend
```

Click on **Commit**.

```{figure} images/lab8-greenfieldcommit01.png
---
align: center
---
Commit
```

- SSH into the **BU1 Analytics** and generate ICMP traffic targeting the private IP address of the **BU1 Frontend**.

```{figure} images/lab712-intraruleinaction23.png
---
height: 200px
align: center
---
inter-rule in action
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. After completing the previous test, the logs for the `inter-icmp-bu1analytics-bu1frontend` rule will appear.

```{figure} images/lab712-monitor02.png
---
align: center
---
logs
```

This is how the overall topology would look after implementing the latest inter-rule.

```{figure} images/lab712-segmentation092.png
---
height: 400px
align: center
---
inter-rule
```

#### 2.2.4 Inter-rule between BU1 DB and BU2 DB

## 3. CHANGE REQUEST

- Now before completing the lab, remove the `Inspection Policy` from the Transit GW in AWS. 

The Aviatrix Distributed Cloud Firewall is now enabled across the multicloud infrastructure, so we can **retire** the NGFW that was in place.

```{tip}
Navigate to **CoPilot > Security > FireNet > FireNet Gateways**. Open the **_ace-aws-eu-west-1-transit1_** GW, go to **Policy**, select the two AWS spoke VPCs, and click Remove. Traffic will not be sent to the firewall anymore, **and the Aviatrix DCF will perform firewalling near the source**.

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

Congratulations, you have completed all labs and created a nice set of DCF rules across your Hybrid-cloud infrastructure!
