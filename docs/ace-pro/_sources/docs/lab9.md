# Lab 9 - THREATIQ & COSTIQ

## 1. Objective

This lab will demonstrate how `ThreatIQ` works.
 
## 2. ThreatIQ Overview

Aviatrix Gateways send NetFlow data to CoPilot. CoPilot uses this data in many ways. **FlowIQ** is one. **ThreatIQ** is another. ThreatIQ alerts you on Malicious IPs with bad reputations, but then can also apply an enforcement. These IPs are reported in the ThreatIQ database that CoPilot maintains.

```{important}
ThreatIQ protect all the Aviatrix Gateways and it relies on a well-known database, provided by **`Proofpoint`**.
```

## 3. Topology

In this lab, we will deploy a `“PSF"` gateway in AWS **US-EAST-1** region, to protect the public subnet.

```{figure} images/lab9-initialtopology.png
---
align: center
---
Lab 9 Initial Topology
```

## 4. PSF
### 4.1 Deploy the PSF

Go to **CoPilot > Cloud Fabric > Gateways > Specialty Gateways**, then click on the `“+Gateway"` button and then choose the **Public Subnet Filtering Gateway**.

```{figure} images/lab9-psf.png
---
height: 400px
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

```{note}
The subnet with the PSF gateway is a Public Subnet with 0/0 pointing to IGW. No workload instances should be deployed in this subnet.
```

- Verify one more routing table that we selected while deploying the PSF Gateway: **_aws-us-east1-spoke1-Public-1-us-east-1a-rtb_**. You can notice that the default route is pointing towards the PSF Gateway (we are verifying this rtb because the test instance’s subnet points to this rtb).

```{figure} images/lab9-routetablepsf2.png
---
align: center
---
aws-us-east1-spoke1-rtb-public-a
```

## 5. Enable ThreatIQ 

Navigate to **CoPilot > Security > ThreatIQ > Configuration**

Click on **Send Alert**:

```{figure} images/lab9-sendalert.png
---
align: center
---
Enable ThreatIQ
```

Then click on **Notification Settings**.

```{figure} images/lab9-notification.png
---
align: center
---
Notification Settings
```

Now click on the `"+ Email Address"` button.

```{figure} images/lab9-email.png
---
align: center
---
Email
```

Choose an **alias**, insert your **personal email** and then click on **Save**:

```{figure} images/lab9-email2.png
---
align: center
---
Alias and Personal Email
```

Navigate back to **CoPilot > Security > ThreatIQ > Configuration**

Click again on **Send Alert**:

```{figure} images/lab9-sendalert2.png
---
height: 300px
align: center
---
Send Alert Settings
```

Select the alias that was previously created from the drop-down window `"recipients"` and then click on **Save**.

```{figure} images/lab9-addrecipient.png
---
align: center
---
Add Recipient(s)
```

From this point onwards, if you enter a valid email address, you will receive email notifications about **ThreatIQ** alerts.

Before enabling the blocking,  ensure that the **ThreatGuard firewall rules order** is set to `Prepend` on the right-hand side.

```{figure} images/lab9-advanced.png
---
align: center
---
Prepend
```

### 5.1 Generate traffic towards the "Bad Guy"

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
```

Navigate back to **CoPilot > Security > ThreatIQ > Overview**

```{note}
**Wait for about 4-5 minutes**, before proceeding with the next action. 

Set the **Time Period** to `"Last 24 Hours"` and click on **Apply**.
```

```{figure} images/lab9-custom.png
---
align: center
---
Overview
```

You should see the IP in the table at the bottom. You can filter based on the destination IP address (insert the malicious IP address):

```{figure} images/lab9-threat.png
---
align: center
---
Threats
```

```{figure} images/lab9-threat2.png
---
align: center
---
Filter
```

```{tip}
If you do not get any outcomes, kindly change the Time Period to `"Last 60 Minutes"`.
```{figure} images/lab9-trigger.png
---
align: center
---
Trigger the outcome
```

Afterwards, click on **VIEW** on the right-hand side of the Timestamp.

```{note}
The IP shown in these screenshots  might not be deemed a threat when you read this. Please use the malicious IP provided by the instructor.
```

```{figure} images/lab9-view.png
---
height: 150px
align: center
---
View
```

Then select **Threat Summary** and pinpoint the metadata "tag" to determine how ThreatIQ has classified this IP.

```{figure} images/lab9-tor.png
---
align: center
---
"tag"
```

## 5. Enforcement

- Enable **Block Threats**

```{tip}
Go to **CoPilot > Security > ThreatIQ > Configuration**, click on the arrow to show the hidden section and then turn on the toggle `"Block Threats"`.
```

```{figure} images/lab9-hiddenview.png
---
height: 150px
align: center
---
Click on the arrow!
```

```{figure} images/lab9-threatguard.png
---
align: center
---
ThreatIQ - Automatic Enforcement
```

By default, <ins>**all** VPCs are enabled for ThreatIQ</ins>, therefore click on **Save** to continue.

```{figure} images/lab9-vpc.png
---
align: center
---
Select VPC
```

Then, click **CONFIRM**.

```{figure} images/lab9-confirm.png
---
align: center
---
Confirm
```

### 5.1. Automatic enforcement: "force-drop"

- Now wait for about <ins>8 minutes</ins>!
- Then issue the same curl command once again, from the test instance **_aws-us-east-1-spoke1-test1_**

```{caution}
Do not issue the curl command immediately! Wait for 8 minutes as suggested above.
```

```{figure} images/lab9-failed.png
---
align: center
---
Curl fails
```

Navigate to  **CoPilot > Security > ThreatIQ > Configuration**

```{note}
The CoPilot UI frequently changes, and what you see below may differ from your experience. 
```

- Click on the **refresh** button under.
  
```{figure} images/lab9-refresh.png
---
height: 150px
align: center
---
Refresh button
```

- Click on **VIEW** under the column View Rules, on the **_aws-us-east-1-psf_** row:

```{figure} images/lab9-viewrules.png
---
height: 150px
align: center
---
View Rules
```

Filter based on the malicious IP (choose the **Source IP** as parameter): you will find out that ThreatIQ applied the enforcement `"force-drop"`.

```{figure} images/lab9-force.png
---
height: 150px
align: center
---
Filter on Source IP
```

**ThreatIQ** has successfully blocked the malicious IP!

```{warning}
Before ending this lab, remove your email from the notification list!
```

Navigate to **CoPilot > Monitor > Notifications > Alerts Configuration**

Click on the pencil icon for editing the configured alert named `"ThreatIQ Alert"`:

```{figure} images/lab9-notification2.png
---
height: 300px
align: center
---
Edit ThreatIQ Alert
```

- Remove the recipient that is identified based on the alias that you chose before, then click on **Save**.

ThreatIQ will immediately stop sending the alerts to your personal email:

```{figure} images/lab9-joe.png
---
align: center
---
Stop alerts
```

## 6. CostIQ

Before completing this lab, let's enable `CostIQ` and define the following **Cost Centers** and **Shared Service**.

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


Go to **Copilot > Billing & Cost > CostIQ** and click on the `"Enable CostIQ"` button.

```{figure} images/lab9-costiq.png
---
height: 150px
align: center
---
Enable CostIQ
```

```{figure} images/lab9-costiq001.png
---
height: 150px
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

Let's move on the **Shared Service** tab and click on `"+ Shared Service"`.

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
