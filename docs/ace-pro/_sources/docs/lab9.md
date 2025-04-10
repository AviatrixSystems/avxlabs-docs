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

```{caution}
Please pay close attention to select the routing table **1a-rtb**!
```

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

## 4. Generate traffic towards a Malicious IP

Now that there is a PSF Gateway on defending the Public Subnet, let's generate some traffic towards an IP with **Bad Reputation**.

### 4.1 Connectivity Testing Using the Gatus App

Go to your personal POD Portal, identify the status widget and then click on **_aws-us-east1-spoke1-test1_**.

```{figure} images/lab9-gatus010.png
---
height: 250px
align: center
---
Gatus
```

```{figure} images/lab9-gatus011.png
---
height: 250px
align: center
---
Malicious IP
```

The EC2 instance is already generating traffic towards a malicious IP address.

### 4.2 SSH to aws-us-east1-spoke1-test1 <span style='color:#33ECFF'>(BONUS)</span></summary>

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

- Issue this command (make sure to enter **HTTPS**):

```bash
curl https://<malicious-IP>
```

`<malicious-IP>`: **178.17.174.164**

```{figure} images/lab9-instancetest.png
---
align: center
---
Curl towards the malicious IP
```

## 5. Create a new SmartGroup

The traffic towards the malicious IP is permitted... Let's now enforce the `Threats Detection mechanism`!

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
- **Virtual Machines - Matches all conditions (AND)**: <span style='color:#479608'>Name</span>
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

## 6. Create two new DCF rules

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

## 7. Generate again traffic towards the "Bad Guy"

### 7.1 Deletion of the Greenfield-Rule = ZTNA

Now let's delete the **Greenfield-Rule**, such that the **ZTNA** can be restored in the Data Path!

- Click on the **three dots** icon on the right-hand side of the Greenfield-Rule entry and then choose the `"Delete Rule"` option.

Do not forget to click on **Commit**.

```{figure} images/lab66-newruledelete.png
---
height: 250px
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

### 7.2 Create a new WebGroup

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

### 7.3 Create a PSF-Allow-Rule

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

Before launching the connectivity test, the three rules related to the PSF must be enforced on the PSF gateway itself.

```{caution}
The `PSF` Gateway is a **_standalone Gateway_**: it is neither a Spoke nor a Transit.
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Settings** and enable the `"Enforcement on PSF Gateweays"` functionality.

```{figure} images/lab96-newrule333.png
---
height: 400px
align: center
---
Enforcement
```

### 8.1 Connectivity Testing Using the Gatus App

Go to your personal POD portal and click again on the click on **_aws-us-east-1-spoke1-test1_** instance.

```{figure} images/lab9-gatus010.png
---
height: 250px
align: center
---
Gatus
```

The traffic to the Malicious IP will gradually fail.

```{figure} images/lab9-gatus678.png
---
height: 400px
align: center
---
Malicious ip
```

However, you will gradually start to notice that traffic to **US EAST-2** will begin to fail.

```{figure} images/lab9-gatus622.png
---
height: 400px
align: center
---
US EAST-2
```

### 8.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

```{important}
Before proceeding with the verification from your SSH client, you need to re-establish the SSH connection with **_aws-us-east-1-spoke1-test1_**. After applying the previous DCF rules, the SSH session will be dropped. Although the SSH session will be established through the IGW, it will then span the Spoke GW, which will enforce the Distributed Cloud Firewall rules!
```

```{figure} images/lab9-igw.png
---
height: 250px
align: center
---
IGW and Spoke GW
```

#### 8.2.1 SSH to aws-us-east1-spoke1-test1

Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, then search for **aws-us-east-1-spoke1-test<span style='color:#33ECFF'>2</span></summary>**, retrieve its Public IP address and SSH in to the EC2 instance.

```{figure} images/lab9-igw04.png
---
height: 250px
align: center
---
SSH in to aws-us-east-1-spoke1-test2
```

- SSH from **_aws-us-east-1-spoke1-test2_** to **_aws-us-east-1-spoke1-test1_**, using this time the Private IP address of the test1 instance.

```{figure} images/lab9-igw05.png
---
height: 250px
align: center
---
SSH in to test1 from test2
```

```{important}
Is it possible to directly SSH into **_aws-us-east-1-spoke1-test1_** without first connecting to test2?
```

- Execute now the following commands:

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
curl https://178.17.174.164
```

```{figure} images/lab96-newrule302.png
---
align: center
---
Towards the Malicious IP
```

You will notice that the traffic towards the **IP with Bad Reputation** was blocked at the very first **SYN** and **SYN-ACK** packets!

## 9. Final Considerations

Now go to **CoPilot > Security > ThreatIQ**  section, then scroll down through the whole **Overview** section, click on the filter icon and filter out based on the Maliciuous IP: you can choose either **_Source_** or **_Destination_**!

```{figure} images/lab96-newrule308.png
---
align: center
---
Filter
```

```{figure} images/lab96-newrule323.png
---
align: center
---
Condition
```

Now click on the **VIEW** link on the right-hand side of the entry:

```{figure} images/lab96-newrule309.png
---
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

After completing this lab, the overall topology will be as follows:

```{figure} images/lab9-finaltopologyy.png
---
height: 400px
align: center
---
Final Topology for Lab 9
```