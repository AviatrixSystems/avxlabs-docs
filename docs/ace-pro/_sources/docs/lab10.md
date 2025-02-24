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

Go to **Copilot > Billing & Cost > CostIQ** and click on the `"Enable CostIQ"` button.

```{figure} images/lab9-costiq.png
---
height: 250px
align: center
---
Enable CostIQ
```

```{figure} images/lab9-costiq001.png
---
height: 300px
align: center
---
Enable CostIQ
```

Now click on `"+ Cost Center"` and create the **AWS** Cost Center aforementioned.

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

Let's move on the **Shared Services** tab and click on `"+ Shared Service"`.

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

## 4. Verification

### 4.1 Connectivity Testing Using Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select **_onprem-pod<span style='color:#33ECFF'>##</span></summary>-edge_**.

```{figure} images/lab10-podportal110.png
---
height: 400px
align: center
---
Gatus
```

```{figure} images/lab10-podportal111.png
---
height: 300px
align: center
---
aws-us-east-2-spoke1-test1
```

### 4.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you kept running the ping on the Workstation Edge's terminal, then you should see both the relative traffic and the absolute one from any Cost Centers towards the Shared Service.

```{figure} images/lab9-ping.png
---
align: center
---
Ping from the Wortkstation "Edge"
```

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

