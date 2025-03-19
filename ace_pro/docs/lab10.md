# Lab 10 - COSTIQ

This lab will demonstrate how `CostIQ` works.

## 1. CostIQ Overview

The CostIQ feature provides detailed traffic distribution analysis for your cost centers, including traffic flowing to shared-service resource hosts by Cloud Account, by Cost Center, by VPC/VNet, and by Gateway.

## 2. Implement CostIQ

- Define the following **Cost Centers** and **Shared Service**:

**<span style='color:orange'>COST CENTERS</span>**:

**AWS**:
- aws-us-east-1-spoke1
- aws-us-east-1-spoke2

**GCP**:
- gcp-us-central1-spoke1

**AZURE**:
- azure-west-us-spoke1
- azure-west-us-spoke2

**<span style='color:green'>SHARED SERVICE</span>**:

**NEW YORK DC**:
- workstation client "edge"

### 2.1 Enable CostIQ

Go to **Copilot > Settings > Configuration > License**, identify the CostIQ Add-on Feature and click on the `"Enable"` button.

```{figure} images/lab9-costiq123.png
---
align: center
---
Enable CostIQ
```

Go to **Copilot > Administration > Billing > CostIQ** and click on the `"Enable CostIQ"` button.

```{warning}
Refresh the web page, before proceeding with the next task!
```

Go to **Copilot > Monitor > CostIQ** and click on `"+ Cost Center"` and create the **AWS** Cost Center.

```{figure} images/lab9-costiq02.png
---
align: center
---
"+ Cost Center"
```

```{figure} images/lab9-costiq03.png
---
align: center
---
AWS
```

Repeat the action creating the remaining two Cost Centers: **GCP** and **Azure**, associating the corresponing Application VPCs/VNets.

```{figure} images/lab9-costiq04.png
---
align: center
---
GCP
```

```{figure} images/lab9-costiq05.png
---
align: center
---
AZURE
```

You should immediately get insights on how they have been utilized.

```{figure} images/lab9-costiq06.png
---
height: 150px
align: center
---
Cost Centers Overview
```

## 3. New York DC is the Shared Services

Now let's discover the **CIDRs** of **New York DC**:

- Go to **CoPilot > Diagnostics > Cloud Routes > BGP Info**, then click on the three dots icon and select the `"Show BGP Learned Routes"`.

```{figure} images/lab9-costiq10.png
---
align: center
---
Show BGP Learned Routes
```

You will find out that all the local subnets advertised by the DC belong to the cidr **10.40.0.0/16**.

```{figure} images/lab9-cidr.png
---
align: center
---
CIDR
```

Go back to **CoPilot > Monitor > CostIQ > Shared Services** and click on the `"+ Shared Service"` button.

```{figure} images/lab9-costiq12.png
---
align: center
---
"+ Shared Service"
```

Create the **Shared Service** based on the aforementioned requirements.

```{figure} images/lab9-costiq13.png
---
align: center
---
"+ Shared Service"
```

## 4. DCF Configuration

- Create a rule that permits **ICMP** traffic from the **Workstation Edge** to the **_aws-us-east-2-spoke1-test1_**.

### 4.1 Create a new SmartGroup for the WorkstationEdge

Go to **CoPilot > Groups** and click on the `"+ SmartGroup"` button.

```{figure} images/lab10-newsg.png
---
height: 400px
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>WorkstationEdge</span>
- **Virtual Machines**:
  - **Matches all conditions (AND)/Name**: <span style='color:#479608'>onprem-pod##-host-vm</span>

```{figure} images/lab10-newsg854.png
---
height: 400px
align: center
---
New SmartGroup
```

```{warning}
Do not select the value that ends with **"vm-1"**.
```{figure} images/lab10-newsg898.png
---
height: 400px
align: center
---
onprem-pod##-host-vm
```

```{figure} images/lab10-newsg01.png
---
height: 400px
align: center
---
WorkstationEdge SG
```

Do not forget to click on **Save**.

### 4.2 Create a new SmartGroup for aws-us-east-2-spoke1-test1

Click again on the `"+ SmartGroup"` button.

```{figure} images/lab10-newsg090.png
---
height: 400px
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>aws-us-east-2-spoke1-test1</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>aws-us-east-2-spoke1-test1</span>

```{figure} images/lab10-newsg0133.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test1 SG
```

Do not forget to click on **Save**.

### 4.3 Create a new DCF rule

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab10-newnew00.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>inter-icmp-edge-us-east2-test1</span>
- **Source Smartgroups**: <span style='color:#479608'>WorkstationEdge</span>
- **Destination Smartgroups**: <span style='color:#479608'>aws-us-east-2-spoke1-test1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab10-newnew0233.png
---
align: center
---
Destination SG
```

```{figure} images/lab10-newnew02.png
---
align: center
---
Create Rule
```

Click then on **Commit**.

## 5. Verification

### 5.1 Connectivity Testing Using Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select **_onprem-pod<span style='color:#33ECFF'>##</span></summary>-edge_**.

```{figure} images/lab10-podportal110.png
---
height: 400px
align: center
---
Gatus
```

The traffic generated from the Workstation Edge will gradually turn green!

```{figure} images/lab10-podportal111.png
---
height: 300px
align: center
---
WorkstationEdge
```

### 5.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you kept running the ping on the Workstation Edge's terminal, then you should see both the relative traffic and the absolute one from any Cost Centers towards the Shared Service.

```{figure} images/lab9-ping.png
---
align: center
---
Ping from the Wortkstation "Edge"
```

### 5.3 Cost Center and Shared Service

Go to **CoPilot > Billing & Cost > CostIQ > Shared Services** and click on **NEW YORK DC** to expand the visibility of the percentage of traffic between the Shared Services and the Cost Center AWS. This traffic is permitted thanks to the DCF rule previously created.

```{figure} images/lab9-counter.png
---
height: 350px
align: center
---
From the Cost Center towards the Shared Service
```

After this lab, this is how the overall topology will look like:

```{figure} images/lab9-final.png
---
height: 400px
align: center
---
Final topology for Lab 10
```

