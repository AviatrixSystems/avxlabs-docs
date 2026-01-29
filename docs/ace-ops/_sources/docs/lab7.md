# Lab 7 - Distributed Cloud Firewall

## 1. SCENARIO

In this lab, you will begin micro-segmentation by creating `SmartGroups` and applying `Aviatrix Cloud Firewall` rules.

```{figure} images/templab7-segmentation.png
---
height: 400px
align: center
---
Initial Topology
```

```{important}
From a routing perspective, there is a flat routing domain, enabled by the `Connection Policy` applied in Lab 2.
```

These are the **requirements** for this lab:

1) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU1 Frontend

1) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU2 Mobile App

2) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU1 Analytics

3) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU1 Mobile App.

4) Create a <span style='color:red'>**Smart Group**</span> that identifies the BU2 Mobile App.

5) Create an <span style='color:lightgreen'>**intra-rule**</span> that allows BU1 Frontend and BU2 Mobile App to ping each other

6) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 Frontend to talk with BU2 Mobile App on TCP/80

7) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 Analytics to ping BU1 Frontend

8) Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 DB to communicate with BU2 DB on TCP/22

9)  Create an <span style='color:orange'>**inter-rule**</span> that allows BU1 Frontend to ping with BU2 DB.

## 2. CHANGE REQUEST

Let’s begin by activating the Distributed Cloud Firewall.  

### 2.1 Distributed Cloud Firewall

To control and enforce policies within your CNSF, you must enable the `Distributed Cloud Firewall`.

#### 2.1.1 DCF - Activation

- Enable the **Distributed Cloud Firewall** feature.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall**, click `Begin Using Distributed Cloud Firewall`, and then click `Begin` on the next screen.
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

#### 2.1.2 Ad-hoc Greenfield-Rule

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
- **Log**: <span style='color:#479608'>**At Start & End**</span>
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

#### 2.1.3 ZTNA

Now let’s enable the `Zero Trust` control by creating an explicit deny rule, which must be placed above the greenfield rule.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**. 
```

- Clicking the `"+ Rule"` button.

```{figure} images/templab7-green53.png
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
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>**Deny**</span>
- **Place Rule**: <span style='color:#479608'>Above</span>
  - **Existing Rule**: <span style='color:#479608'>Greenfield-Rule</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/templab7-exdeall.png
---
align: center
---
ExplicitDenyAll
```

Click **Commit** to enforce the policies in the data plane.

```{figure} images/templab7-xdeall00.png
---
height: 300px
align: center
---
Commit
```

```{caution}
From this point forward, **East–West** traffic will be blocked. You’ll need to create ad hoc policies to control traffic within your CNSF.
```

### 2.2 SmartGroups creation

Let's create the Smart Groups:

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

Create another Smart Group by clicking the `“+ SmartGroup”` button.

```{figure} images/templab8-SGnew10.png
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

```{figure} images/templab8-SGnew20.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU2-MOBILEAPP</span>
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

The CoPilot shows that there is one instance that perfectly matches the condition::

- **_ace-aws-eu-west-1-spoke2-bu2-mobile-app_**

### 2.2 Distributed Cloud Firewall Policies

Let's begin defining the **DCF rules** to govern `East–West` traffic.

#### 2.2.1 Intra-rule between BU1 Frontend and BU2 Mobile App

The current SmartGroups status is depicted below. 

The first requirement states: “Create an intra-rule that allows BU1 Frontend and BU2 Mobile App to ping each other.”  

Accordingly, the first rule to create is an `intra-rule` that includes both **BU1 Frontend** and **BU2 Mobile App**. To enforce this intra-rule, define an additional SmartGroup that contains instances from both applications—for example, by selecting a shared attribute such as the label `Region=eu-west-1`.

```{figure} images/lab71-segmentation.png
---
height: 400px
align: center
---
Current SmartGroup status
```

- Navigate to **CoPilot > Groups > SmartGroups** and click the `“+ SmartGroup”`.

```{figure} images/templab8-SGnew2090.png
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

```{figure} images/templab8-greenfield56661.png
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
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/templab8-greenfield566612.png
---
align: center
---
intra-icmp-bu1frontend-bu2mobileapp
```

Click on **Commit**.

```{figure} images/templab8-greenfield5666123.png
---
align: center
---
Commit
```

#### 2.2.2 Verification Using Gatus

```{figure} images/templab3-4.28.diagnosticstools1190156.png
---
height: 400px
align: center
---
intra-icmp-bu1frontend-bu2mobileapp
```

#### 2.2.3 Verification Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

#### 2.2.4 Inter-rule between BU1 Frontend and BU2 Mobile App

The second policy is an inter-rule, requiring **BU1 Frontend** to communicate with **BU2 Mobile App** via TCP port 80.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/templab8-greenfield13.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-http-bu1frontend-bu2mobileapp</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-MOBILEAPP</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>80</span>
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/templab8-greenfieldjoe00.png
---
align: center
---
inter-http-bu1frontend-bu2mobileapp
```

Click on **Commit**.

```{figure} images/templab8-greenfieldcommit00.png
---
align: center
---
Commit
```

#### 2.2.5 Verification Using Gatus

```{figure} images/templab3-4.28.diagnosticstools119015688.png
---
height: 400px
align: center
---
intra-icmp-bu1frontend-bu2mobileapp
```

#### 2.2.6 Verification Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

#### 2.2.7 Inter-rule between BU1 Analytics and BU1 Frontend

The third policy is again an inter-rule, requiring **BU1 Analytics** to ping **BU1 Frontend**.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/templab8-greenfield14.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-icmp-bu1analytics-bu1frontend</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-ANALYTICS</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/templab8-greenfieldjoe01.png
---
align: center
---
inter-icmp-bu1analytics-bu1frontend
```

Click on **Commit**.

```{figure} images/templab8-greenfieldcommit01.png
---
align: center
---
Commit
```

#### 2.2.8 Verification Using Gatus

```{figure} images/templab3-4.28.diagnosticstools1190156881.png
---
height: 400px
align: center
---
inter-icmp-bu1analytics-bu1frontend
```

#### 2.2.9 Verification Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

#### 2.2.10 Inter-rule between BU1 DB and BU2 DB

The fourth policy is an inter-rule designed to permit **BU1 DB** to establish SSH access to **BU2 DB**.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/templab8-greenfield15.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-ssh-bu1db-bu2db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-DB</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/templab8-greenfieldjoe0100.png
---
align: center
---
inter-ssh-bu1db-bu2db
```

Click on **Commit**.

```{figure} images/templab8-greenfieldcommit022.png
---
align: center
---
Commit
```

#### 2.2.11 Verification Using Gatus

```{figure} images/templab3-4.28.diagnosticstools119015688122.png
---
height: 400px
align: center
---
inter-ssh-bu1db-bu2db
```

#### 2.2.12 Verification Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- SSH into the **BU1 Frontend** and generate ICMP traffic targeting the private IP address of the **BU1 Frontend**.

```{figure} images/lab712-intraruleinaction245.png
---
height: 200px
align: center
---
first SSH
```

```{figure} images/lab712-intraruleinaction24.png
---
height: 200px
align: center
---
second SSH
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. After completing the previous test, the logs for the `inter-ssh s-bu1db-bu2db` rule will appear.

```{figure} image s/lab712-monitor03.png
---
align: center
---
logs
```

This is how the overall topology would look after implementing the latest inter-rule.

```{figure} images/lab712-segmentation093.png
---
height: 400px
align: center
---
inter-rule
```

#### 2.2.5 Inter-rule between BU1 Frontend and BU1 DB

The final policy is an inter-rule designed to permit **BU1 Frontend** to ping **BU2 DB**.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, and click on the **"+ Rule"** button.

```{figure} images/lab8-greenfield16.png
---
height: 400px
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-icmp-bu1frontend-bu1db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab8-greenfieldjoe0101.png
---
align: center
---
inter-icmp-bu1frontend-bu1db
```

Click on **Commit**.

```{figure} images/lab8-greenfieldcommit023.png
---
align: center
---
Commit
```

- SSH into the **BU1 Frontend** and generate ICMP traffic targeting the private IP address of the **BU1 DB**.

```{figure} images/lab712-intraruleinactionping.png
---
height: 200px
align: center
---
ping
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. After completing the previous test, the logs for the `inter-icmp-bu1frontend-bu1db` rule will appear.

```{figure} images/lab712-monitor04.png
---
align: center
---
logs
```

This is how the overall topology would look after implementing the latest inter-rule.

```{figure} images/lab712-segmentation094.png
---
height: 400px
align: center
---
inter-rule
```

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
