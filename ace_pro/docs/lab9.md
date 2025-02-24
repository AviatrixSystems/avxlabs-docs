# Lab 9 - THREAT PREVENTION

This lab will demonstrate how the `DCF ThreatGroup` is capable to prevent traffic from being sent to, or from, a set of **threat IPs**.

## 1. ThreatGroup Overview

The **Default ThreatGroup** can be used in DCF rules to ensure that traffic meeting the ThreatGroup criteria is blocked.

```{important}
The Default ThreatGroup is regularly updated with data from the **`Proofpoint Global Threat Database`**.
```

## 2. Topology

In this lab, we will deploy a `“PSF"` gateway in AWS **US-EAST-1** region, to protect SOLELY the public subnet where the test1 instance resides.

```{figure} images/lab9-initialtopology.png
---
align: center
---
Lab 9 Initial Topology
```

## 3. PSF
**Public Subnet Filtering** Gateway = An Aviatrix gateway that provides ingress and egress security for AWS public subnets where instances have public IP addresses.

### 3.1 Deploy the PSF

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

### 3.2 RTB verification

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

## 4.0 Generate traffic towards a Malicious IP

Now that there is a PSF Gateway on defending the Public Subnet, let's generate some traffic towards an IP with **Bad Reputation**.

### 4.1 SSH to aws-us-east1-spoke1-test1

Retrieve the Public IP address of **_aws-us-east-1-spoke1-test1_** instance:

- Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, search for `aws-us-east-1-spoke1-test1` and then copy the **Public** IPv4 address!

```{figure} images/lab9-newsg010.png
---
height: 250px
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

<ins>`Wait for the instructor to provide a malicious IP`</ins>. 

Let's call it `<malicious-IP>`. 

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

The traffic will be permitted... Let's now enforce the `Threats detection mechanism`!

```{note}
The IP shown in these screenshots  might not be deemed a threat when you read this. 

<ins>Please use the malicious IP provided by the instructor</ins>.
```

## 5.0 Create a new SmartGroup 

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
align: center
---
aws-us-east-1-spoke1-test1 SmartGroup
```

Do not forget to click on **Save**.

```{figure} images/lab9-smart003.png
---
height: 250px
align: center
---
SmartGroups List
```

## 6.0 Create two new DCF rules

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab9-newrule10.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>PSF-Deny-Rule-from-aws-us-east-1-spoke1-test1</span>
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

Now before committing, create another DCF rule for blocking also the traffic sourced from **any Malicious IP addresses** towards the **aws-us-east-1-spoke1-test1** instance.

Create a new rule clicking on the `"+ Rule"` button:

```{figure} images/lab911-new33.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>PSF-Deny-Rule-from-malicious-ips</span>
- **Source Groups**: <span style='color:#479608'>DeafultThreatGroup</span>
- **Destination Groups**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Enforcement**: <span style='color:#479608'>**On**</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

Do not forget to click on **Save In Drafts**. 

```{figure} images/lab96-newrule44.png
---
align: center
---
PSF-Deny-Rule-from-malicious-ips
```

Do not forget now to **Commit** your new rules!

```{figure} images/lab96-newrule-commit.png
---
height: 350px
align: center
---
Commit the new rules
```

```{important}
These two rules will protect the `bi-directional communication`: traffic will be blocked if **aws-us-east-1-spoke1-test1** will try to reach any **Malcious IPs** (by _ProofPoint's DB_), and likewise traffic will be blocked if any **Malicious IPs** (by _ProofPoint's DB_) will try to reach the **aws-us-east-1-spoke1-test1** instance.
```

Explore the content of the `Default ThreatGroup`: 

- Go to **CoPilot > Groups > ThreatGroups** and click on **Default ThreatGroup** and look at the ProofPoint Malicious IP addresses DB!

```{figure} images/lab96-newrule12.png
---
height: 400px
align: center
---
Default ThreatGroup
```

```{note}
`ProofPoint` sends its new malicious IP addresses DB to the CoPilot every **30 minutes**. 
```

## 7.0 Generate again traffic towards the "Bad Guy"

Now let's delete the **Greenfield-Rule**, such that the **ZTNA** can be restored in the Data Path!

- Click on the **three dots** icon on the right-hand side of the Greenfield-Rule entry and then choose the `"Delete Rule"` option.

Do not forget to click on **Commit**.

```{figure} images/lab66-newruledelete.png
---
height: 350px
align: center
---
Deletion of the Greenfield-Rule
```

```{figure} images/lab66-newruledeleted.png
---
height: 400px
align: center
---
Commit
```

```{figure} images/lab66-newruledeleted00.png
---
height: 400px
align: center
---
DCF Rules List
```

### 7.1 Create a new SmartGroup

Let's create another **WebGroup** that will exactly match three domains:

1) www.nginx.com
2) www.ubuntu.com
3) www.aviatrix.com

Go to **CoPilot > Groups > WebGroups** and then click on the `"+ WebGroup"` button.

```{figure} images/lab9-smart100.png
---
height: 400px
align: center
---
New WebGroup
```

Create the new **_WebGroup_** with the following parameters:

- **Name**: <span style='color:#479608'>Allowed-Public-Domains</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'>www.nginx.com</span>
- **Domains/URLs**: <span style='color:#479608'>www.ubuntu.com</span>
- **Domains/URLs**: <span style='color:#479608'>www.aviatrix.com</span>

Do not forget to click on **Save**.

```{figure} images/lab6-webgroup200.png
---
align: center
---
WebGroup creation
```

### 7.2 Create a PSF-Allow-Rule

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab9-newrule100.png
---
align: center
---
New Rule
```

Insert the following parameters

- **Name**: <span style='color:#479608'>PSF-Allow-Rule</span>
- **Source Groups**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Destination Groups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>Allowed-Public-Domains</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Enforcement**: <span style='color:#479608'>**On**</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab911-new100.png
---
align: center
---
Saving the new Rule
```

Click on the **Commit** button!

```{figure} images/lab96-newrule201.png
---
height: 400px
align: center
---
Commit
```

## 8. Connectivity Tests

Now before launching the connectivity test, those three rules related to the PSF must be enforced on the PSF gateway itself.

```{caution}
The `PSF` Gateway is a **_standalone Gateway_**: it is neither a Spoke nor a Transit.
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Settings** and enable the `"Enforcemente on PSF Gateweays"` functionality.

```{figure} images/lab96-newrule333.png
---
height: 400px
align: center
---
Commit
```

From your **SSH** client, issue the following commands from the **aws-us-east1-spoke1-test1** instance:

```bash
curl https://www.nginx.com
```

```bash
curl https://www.ubuntu.com
```

```bash
curl https://www.aviatrix.com
```

```{figure} images/lab96-newrule301.png
---
align: center
---
Outcomes from the curl commands
```

Now issue again the curl command towards the **malicious IP** address that was earlier provided by the Trainer!

```bash
curl https://<malicious-IP>
```

```{figure} images/lab96-newrule302.png
---
align: center
---
Towards the Malicious IP
```

You will notice that the traffic towards the **IP with Bad Reputation** was blocked at the very first **SYN** and **SYN-ACK** packets!

Now go to **CoPilot > Security > ThreatIQ**  section, then scroll down through the whole **Overview** section, click on the filter icon and filter out based on the Maliciuous IP: you can choose either **_Source_** or **_Destination_**!

```{figure} images/lab96-newrule308.png
---
height: 400px
align: center
---
Filter
```

```{figure} images/lab96-newrule309.png
---
align: center
---
Condition
```

Now click on the **VIEW** link on the right-hand side of the entry:

```{figure} images/lab96-newrule310.png
---
height: 400px
align: center
---
View link
```

Last but not least, explore the `Threat Summary` tab to find out how ProofPoint classified that IP address!

```{figure} images/lab96-newrule311.png
---
align: center
---
Condition
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab9-finaltopologyy.png
---
height: 400px
align: center
---
Final Topology for Lab 9
```