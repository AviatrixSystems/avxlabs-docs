# Lab 9 - THREATIQ 

## 1. Objective

This lab will demonstrate how `ThreatIQ` works.
 
## 2. ThreatIQ Overview

Aviatrix gateways send NetFlow data to CoPilot. CoPilot uses this data in many ways. **FlowIQ** is one. **ThreatIQ** is another. ThreatIQ alerts you on Malicious IPs with bad reputations, but then can also apply an enforcement. These IPs are reported in the ThreatIQ database that CoPilot maintains.

```{important}
ThreatIQ protect all the Aviatrix Gateways
```

## 3. Topology

In this lab, we will deploy a `“PSF"` gateway in AWS US-EAST-1 region, to protect the public subnet.

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
align: center
---
PSF
```

Insert the following parameters:
- **Name**: <span style='color:#479608'>aws-us-east1-psf</span>
- **Account**: <span style='color:#479608'>aws-account</span>
- **Region**: <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC**: <span style='color:#479608'>aws-us-east1-spoke1</span>
- **Instance Size**: <span style='color:#479608'>t2.medium</span>
- **Attach to Unused Subnet**: <span style='color:#479608'>us-east-1a</span>
- **Route Table**: <span style='color:#479608'>aws-us-east1-spoke1-Public-1-us-east-1a-rtb</span>

Do not forget to click on **Save**.

```{figure} images/lab9-psfcreate.png
---
align: center
---
PSF template
```

```{warning}
Wait for about **8** minutes for the completion of the PSF deployment
```

```{figure} images/lab9-psfinprogress.png
---
align: center
---
PSF deployment in progress
```

### 4.2 RTB verification

- Click on the **PSF** gateway, select the **VPC/VNet Route Tables** and then inspect the **_aviatrix-Aviatrix-Filter-Gateway_** Route Table

```{figure} images/lab9-psfclick.png
---
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

Before enabling the blocking, on the far right side, ensure that the **ThreatGuard firewall rules order** is set to `Prepend`.

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

- Now test `ThreatGuard` by first issuing this command (make sure to enter **HTTPS**):

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

Set the **Time Period** to `"Last 60 Minutes"` and click on **Apply**.
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

Afterwards, click on **VIEW** on the right-hand side of the Timestamp.

```{note}
The IP shown in these screenshots  might not be deemed a threat when you read this. Please use the malicious IP provided by the instructor.
```

```{figure} images/lab9-view.png
---
align: center
---
View
```

```{figure} images/lab9-view2.png
---
align: center
---
Threat details
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
Go to **CoPilot > Security > ThreatIQ > Configuration** and turn on the toggle `"Block Threats"`.
```

```{figure} images/lab9-threatguard.png
---
align: center
---
ThreatGuard
```

By default, <ins>**all** VPCs are enabled for ThreatGuard</ins>, therefore click on **Save** to continue.


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

- Now try issuing the same curl command once again, from the test instance **_aws-us-east-1-spoke1-test1_**

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
align: center
---
Refresh button
```

- Click on **VIEW** under the column View Rules, on the **_aws-us-east-1-psf_** row:

```{figure} images/lab9-viewrules.png
---
align: center
---
Configuration VIEW
```

Filter based on the malicious IP (choose the **source IP** as parameter): you will find out that ThreatIQ applied the enforcement `"force-drop"`.

```{figure} images/lab9-force.png
---
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

After this lab, this is how the overall topology would look like:

```{figure} images/lab9-final.png
---
height: 400px
align: center
---
Final topology for Lab 9
```
