# Lab 9 - THREATIQ & COSTIQ

## 1. Objective

This lab will demonstrate how `ThreatIQ` and `CostIQ` work.
 
## 2. ThreatIQ Overview

Aviatrix Gateways send NetFlow data to CoPilot. CoPilot uses this data in many ways. **FlowIQ** is one. **ThreatIQ** is another. ThreatIQ alerts you on Malicious IPs with bad reputations, but then can also apply an enforcement. These IPs are reported in the ThreatIQ database that CoPilot maintains.

```{important}
ThreatIQ protect all the Aviatrix Gateways and it relies on a well-known database, provided by **`Proofpoint`**.
```

## 3. Topology

In this lab, we will deploy a `“PSF"` gateway in AWS **US-EAST-1** region, to protect SOLELY the public subnet where the test1 instance resides.

```{figure} images/lab9-initialtopology.png
---
align: center
---
Lab 9 Initial Topology
```

## 4. PSF
### 4.1 Deploy the PSF

Go to **CoPilot > Cloud Fabric > Gateways > Specialty Gateways**, then click on the `“+ Gateway"` button and then choose the **Public Subnet Filtering Gateway**.

```{figure} images/lab9-psf.png
---
height: 300px
align: center
---
PSF
```

Insert the following parameters:
- **Name**: <span style='color:#479608'>aws-us-east-1-psf</span>
- **Account**: <span style='color:#479608'>aws-account</span>
- **Region**: <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC**: <span style='color:#479608'>aws-us-east1-spoke1</span>
- **Instance Size**: <span style='color:#479608'>t2.medium</span>
- **Attach to Unused Subnet**: <span style='color:#479608'>us-east-1a</span>
- **Route Table**: <span style='color:#479608'>aws-us-east1-spoke1-Public-1-us-east-1a-rtb</span>

Do not forget to click on **Save**.

```{figure} images/lab9-new.png
---
align: center
---
PSF template
```

```{warning}
Wait for about **8** minutes for the completion of the PSF deployment.
```

```{figure} images/lab9-psfinprogress.png
---
height: 200px
align: center
---
PSF deployment in progress
```

### 4.2 RTB verification

- Click on the **PSF** gateway, select the **VPC/VNet Route Tables** and then inspect the **_aviatrix-Aviatrix-Filter-Gateway_** Route Table

```{figure} images/lab9-psfclick.png
---
height: 150px
align: center
---
PSF deployed
```

```{figure} images/lab9-routetablepsf.png
---
align: center
---
PSF rtb
```

```{caution}
The subnet with the PSF gateway is a **Public** Subnet with 0/0 pointing to IGW. 

**No workload instances should be deployed in this subnet**.
```

- Verify one more routing table that we selected while deploying the PSF Gateway: **_aws-us-east1-spoke1-Public-1-us-east-1a-rtb_**. You can notice that the default route is pointing towards the PSF Gateway (we are verifying this rtb because the test instance’s subnet points to this rtb).

```{figure} images/lab9-routetablepsf2.png
---
align: center
---
aws-us-east1-spoke1-rtb-public-a
```

## 5.0 Generate traffic towards a Malicious IP

Now that there is a PSF Gateway on defending the Public Subnet, let's generate some traffic towards an IP with Bad Reputation.

### 5.1 SSH to aws-us-east1-spoke1-test1

Retrieve the Public IP address of **_aws-us-east-1-spoke1-test1_** instance:

- Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, search for `aws-us-east-1-spoke1-test1` and then copy the **Public** IPv4 address!

```{figure} images/lab9-newsg010.png
---
align: center
---
Public IP address
```

```{figure} images/lab9-newsg011.png
---
align: center
---
SSH session
```

Wait for the instructor to provide a malicious IP. Let's call it `<malicious-IP>`. 

```{important}
<ins>Note down this IP address!</ins>
```

- Issue this command (make sure to enter **HTTPS**):

```bash
curl https://<malicious-IP>
```

```{figure} images/lab9-instancetest.png
---
align: center
---
Curl towards the malicious IP
```

The traffic will be permitted... Let's now enforce the `ThreatIQ mechanism`!

```{note}
The IP shown in these screenshots  might not be deemed a threat when you read this. Please use the malicious IP provided by the instructor.
```

## 6.0 Create a new SmartGroup 

Let's create another SmartGroup that can identify the **_aws-us-east-1-spoke1-test1_** instance that resides in the **US-EAST-1** region.

```{figure} images/lab9-routetablepsf234.png
---
align: center
---
aws-us-east-1-spoke1-Public-1-us-east-1a
```

Go to **CoPilot > Groups > SmartGroups** and then click on the `"+ SmartGroup"` button.

```{figure} images/lab9-smart001.png
---
height: 150px
align: center
---
New SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **CSP Tag Key**: <span style='color:#479608'>Name</span>
- **CSP Tag Value**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>

```{figure} images/lab9-smart002.png
---
height: 150px
align: center
---
aws-us-east-1-spoke1-test1 SmartGroup
```

Do not forget to click on **Save**.

```{figure} images/lab9-smart003.png
---
height: 150px
align: center
---
SmartGroups List
```

## 7.0 Create a new DCF rule

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab9-newrule10.png
---
align: center
---
New Rule
```

Insert the following parameters

- **Name**: <span style='color:#479608'>PSF-Rule</span>
- **Source Groups**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Destination Groups**: <span style='color:#479608'>DeafultThreatGroup</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Enforcement**: <span style='color:#479608'>**On**</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab911-new.png
---
align: center
---
Saving the new Rule
```

Click on the **Commit** button!

```{figure} images/lab96-newrule11.png
---
height: 150px
align: center
---
PSF-Rule
```

```{important}
The **`Default ThreatGroup`** can be used in DCF rules to ensure that traffic meeting the ThreatGroup criteria is blocked. When traffic triggers that rule, its DCF rule references are shown on the **Groups > ThreatGroups** tab.

The Default ThreatGroup is regularly updated with data from the Proofpoint Global Threat Database.
```

- Explore the content of the `Default ThreatGroup`: go to **CoPilot > Groups > ThreatGroups** and click on Default ThreatGroup and look at the ProofPoint Malicious IP addresses DB!

```{figure} images/lab96-newrule12.png
---
height: 150px
align: center
---
PSF-Rule
```

## 8.0 Generate again traffic towards the "Bad Guy"

Now delete the **Greenfield-Rule**: 

- Click on the **three dots** icon on the right-hand side of the Greenfield-Rule entry and then choose the `"Delete Rule"` option.

Do not forget to click on **Commit**.

```{figure} images/lab66-newruledelete.png
---
height: 150px
align: center
---
Deletion of the Greenfield-Rule
```

```{figure} images/lab66-newruledeleted.png
---
height: 150px
align: center
---
DCF Rules List
```

Wait for the instructor to provide a malicious IP. Let's call it `<malicious-IP>`. 

```{important}
<ins>Note down this IP address!</ins>
```

SSH to the EC2 instance **_aws-us-east1-spoke1-test1_**

- Now test `ThreatIQ` by first issuing this command (make sure to enter **HTTPS**):

```bash
curl https://<malicious-IP>
```

```{figure} images/lab9-instancetest.png
---
align: center
---
Curl towards the malicious IP


























# Lab 9 - COSTIQ

## 1. Objective

This lab will demonstrate how `CostIQ` works.

## 2. CostIQ

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

After this lab, this is how the overall topology would look like:

```{figure} images/lab9-final.png
---
height: 400px
align: center
---
Final topology for Lab 9
```

