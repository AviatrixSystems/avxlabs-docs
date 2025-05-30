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
height: 400px
align: center
---
Gatus
```

```{figure} images/lab9-gatus011.png
---
height: 400px
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

Before proceeding, create an additional DCF rule to block traffic originating from **any Malicious IP addresses** targeting the **aws-us-east-1-spoke1-test1** instance.

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
These two rules will ensure `bi-directional communication`: traffic from **aws-us-east-1-spoke1-test1**1 to **Malcious IPs** (as identified by ProofPoint’s database) will be blocked, and likewise, traffic from **Malicious IPs** to **aws-us-east-1-spoke1-test1** will also be prevented.
```

```{note}
`ProofPoint` sends its new malicious IP addresses DB to the CoPilot every **30 minutes**. 
```

## 7. Generate again traffic towards the "Bad Guy"

### 7.1 Deletion of the Greenfield-Rule = ZTNA

Let's delete the **Greenfield-Rule** to restore **ZTNA** in the Data Path.

Click the `trash icon` on the right side of the Greenfield-Rule entry to delete it.

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

### 7.3 Create the "PSF-Allow-Rule"

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
Before verifying from your SSH client, please re-establish the SSH connection to **_aws-us-east-1-spoke1-test1_**. Note that applying the previous DCF rules will terminate the current SSH session. Although the connection is made through the IGW, it will then route via the Spoke Gateway, where the Distributed Cloud Firewall rules will be enforced.
```

```{figure} images/lab9-igw.png
---
height: 250px
align: center
---
IGW and Spoke GW
```

#### 8.2.1 SSH to aws-us-east1-spoke1-test1

First, SSH into **_aws-us-east1-spoke1-test1_**, then from there, SSH into **_aws-us-east1-spoke1-test2_**.

```{figure} images/lab9-igw32.png
---
height: 250px
align: center
---
IGW and Spoke GW
```

```{important}
Is it possible to directly SSH into **_aws-us-east-1-spoke1-test1_** without first connecting to test2?
```

<details>
  <summary>Click here to reveal the answer to the previous question: <span style='color:#33ECFF'>Hint!</span></summary>

If you want to SSH directly into the **aws-us-east-1-spoke1-test1** instance, you need to create an ad-hoc DCF rule that allows the public IP address of your laptop to access this instance.

```{figure} images/lab9-myip01.png
---
height: 400px
align: center
---
From your laptop to aws-us-east-1-spoke1-test1
```

- Go to this URL https://whatismyipaddress.com/ and copy your personal Public IP address.

```{figure} images/lab9-myip02.png
---
height: 400px
align: center
---
whatismyipaddress.com
```

- Go to **CoPilot > Groups > SmartGroups** and then click on the `"+ SmartGroup"` button.

```{figure} images/lab9-myip03.png
---
align: center
---
New SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>My-IP</span>

- Then click on the `"+ Resource Type"` button and choose IPs / CIDRs.

```{figure} images/lab9-myip04.png
---
align: center
---
Resource Type: IP
```

- Now paste the public IP address that you copied from the website _whatismyipaddress.com_. Before clicking **Save**, make sure to remove the Virtual Machines condition as shown below.

```{figure} images/lab9-myip05.png
---
align: center
---
Smart Group My-IP
```

- Go to **CoPilot > Security > Distributed Cloud Firewall** and create a new rule clicking on the `"+ Rule"` button:

```{figure} images/lab9-myip06.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>inter-myip-east1-test1-ssh</span>
- **Source Groups**: <span style='color:#479608'>My-IPp</span>
- **Destination Groups**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Enforcement**: <span style='color:#479608'>**On**</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab9-myip07.png
---
align: center
---
inter-myip-east1-test1-ssh
```

Click on the **Commit** button!

```{figure} images/lab9-myip08.png
---
height: 400px
align: center
---
Commit
```

- Now, retrieve the public IP address of the **aws-us-east-1-spoke1-test1** instance. Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for _aws-us-east-1-spoke1-test1_. Click on `"+ 1 more"` to reveal the public IP address, then copy it to your clipboard.

```{figure} images/lab9-myip09.png
---
height: 200px
align: center
---
aws-us-east-1-spoke1-test1
```

- Now, open your SSH client and carry out the following command, pasting the Public IP address previously copied, then use the password available on your personal POD portal

```bash
ssh student@...
```

```{figure} images/lab9-myip10.png
---
align: center
---
SSH to aws-us-east-1-spoke1-test1
```
</details>

```{caution}
If you do not wish to try the hidden resolution, please proceed with the task ahead.
```

Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, then search for **aws-us-east-1-spoke1-test<span style='color:#33ECFF'>2</span></summary>**, retrieve its Public IP address and SSH in to the EC2 instance.

```{figure} images/lab9-igw04.png
---
height: 400px
align: center
---
SSH in to aws-us-east-1-spoke1-test2
```

- SSH from **_aws-us-east-1-spoke1-test2_** to **_aws-us-east-1-spoke1-test1_**, using this time the Private IP address of the test1 instance.

```{figure} images/lab9-igw05.png
---
height: 400px
align: center
---
SSH in to test1 from test2
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

Now go to **CoPilot > Security > ThreatIQ**  section, then scroll down through the whole **Overview** section, click on the filter icon and filter out based on the Maliciuous IP (i.e., `178.17.174.164`): you can choose either **_Source_** or **_Destination_**!

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