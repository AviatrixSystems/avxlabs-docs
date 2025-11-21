# Lab 9 - THREAT PREVENTION

This lab will demonstrate how the `Default ThreatGroup` is capable to prevent traffic from being sent to, or from, a set of **threat IPs**.

## 1. ThreatGroup Overview

The **Default ThreatGroup** can be used in DCF rules to ensure that traffic meeting the ThreatGroup criteria is blocked.

```{important}
The Default ThreatGroup is regularly updated with data from the **`Proofpoint Global Threat Database`**.
```

## 2. Topology

In this lab, we will deploy a `“PSF"` gateway in **us-east-1** to protect **_only_** the public subnet hosting the test1 instance.

```{figure} images/lab9-initialtopology.png
---
align: center
---
Lab 9 Initial Topology
```

## 3. PSF

The Aviatrix **Public Subnet Filtering** Gateway delivers _ingress_ and _egress_ security for AWS public subnets in which instances have public IP addresses.

### 3.1 Deploy the PSF

Navigate to **CoPilot > Cloud Fabric > Gateways > Specialty Gateways**, then click on the `“+ Gateway"` button and then choose the **Public Subnet Filtering Gateway**.

```{figure} images/lab9-psf.png
---
height: 300px
align: center
---
PSF
```

Enter the following parameters:
- **Name**: <span style='color:#479608'>aws-us-east-1-psf</span>
- **Account**: <span style='color:#479608'>aws-account</span>
- **Region**: <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC**: <span style='color:#479608'>aws-us-east1-spoke1</span>
- **Instance Size**: <span style='color:#479608'>c5.large</span>
- **Attach to Unused Subnet**: <span style='color:#479608'>us-east-1a</span>
- **Route Table**: <span style='color:#479608'>aws-us-east-1-spoke1-Public-1-us-east-1a-rtb</span>

Do not forget to click on **Save**.

```{caution}
Please ensure that routing table **1a-rtb** is selected.
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
The subnet with the PSF gateway is a **Public** subnet with a 0/0 route pointing to the IGW.

**No workload instances should be deployed in this subnet**.
```

- Validate the routing table we selected during the PSF Gateway deployment: **_aws-us-east-1-spoke1-Public-1-us-east-1a-rtb_**. The default route should point to the PSF Gateway; we’re verifying this RTB because the test instance’s subnet points to it.

```{figure} images/lab9-routetablepsf2.png
---
align: center
---
aws-us-east1-spoke1-rtb-public-a
```

## 4. Generate traffic towards a Malicious IP

Now that there is a **PSF** Gateway on defending the Public Subnet, let's generate some traffic towards an IP with **Bad Reputation**.

### 4.1 Connectivity Testing Using the Gatus App

Navigate to your personal POD Portal, identify the Gatus widget and then click on **_aws-us-east1-spoke1-test1_**. Scroll through the tests to find the one named 'psf'.

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

The EC2 instance is actively communicating with a **malicious IP** address.

### 4.2 SSH to aws-us-east1-spoke1-test1 <span style='color:#33ECFF'>(BONUS)</span></summary>

Retrieve the Public IP address of **_aws-us-east-1-spoke1-test1_** instance:

- Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, search for `aws-us-east-1-spoke1-test1` and then copy the **Public** IPv4 address!

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

The traffic to the malicious IP is permitted. Immediately enforce the `Threats Detection mechanism`.

Let's create another SmartGroup that can identify the **_aws-us-east-1-spoke1-test1_** instance that resides in the **US-EAST-1** region.

```{figure} images/lab9-routetablepsf234.png
---
align: center
---
aws-us-east-1-spoke1-Public-1-us-east-1a
```

Navigate to **CoPilot > Groups > SmartGroups** and then click on the `"+ SmartGroup"` button.

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

```{figure} images/lab9-smart00331.png
---
align: center
---
Name
```

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

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab9-newrule10.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>PSF-Deny-Rule-from-aws-us-east-1-spoke1-test1</span>
- **Source Groups**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Destination Groups**: <span style='color:#479608'>Deafult ThreatGroup</span>
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

Enter the following parameters:

- **Name**: <span style='color:#479608'>PSF-Deny-Rule-from-malicious-ips</span>
- **Source Groups**: <span style='color:#479608'>Deafult ThreatGroup</span>
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

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies (default tab)**, click the `trash icon` on the right side of the **Greenfield-Rule** entry to delete it.

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
height: 250px
align: center
---
Commit
```

```{figure} images/lab66-newruledeleted00.png
---
height: 250px
align: center
---
DCF Rules List
```

### 7.2 Create a new WebGroup

Let's create another **WebGroup** that will exactly match three domains:

1) www.nginx.com
2) www.ubuntu.com
3) www.aviatrix.com

Navigate to **CoPilot > Groups > WebGroups** and then click on the `"+ WebGroup"` button.

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
- **Domains/URLs**: <span style='color:#479608'>www.aviatrix.ai</span>

Do not forget to click on **Save**.

```{figure} images/lab6-webgroup200.png
---
align: center
---
WebGroup creation
```

### 7.3 Create the "PSF-Allow-Rule"

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab9-newrule100.png
---
align: center
---
New Rule
```

Enter the following parameters:

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

Prior to launching the connectivity test, ensure the three PSF rules are **enforced** on the PSF gateway.

```{caution}
The `PSF` Gateway is a **_standalone Gateway_**: it is neither a Spoke nor a Transit.
```

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Settings** and enable the `"Enforcement on PSF Gateweays"` functionality.

```{figure} images/lab96-newrule333.png
---
height: 400px
align: center
---
Enforcement
```

### 8.1 Connectivity Testing Using the Gatus App

Navigate to your personal POD portal and click again on the click on **_aws-us-east-1-spoke1-test1_** instance.

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

However, you will progressively see also failures in traffic routed to **US EAST-2**.

```{figure} images/lab9-gatus622.png
---
height: 400px
align: center
---
US EAST-2
```

### 8.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Let's verify that traffic to the malicious IP address has been successfully blocked, including via the SSH client.

```{caution}
If you accidentally close your SSH session and try to log in again to the **_aws-us-east-1-spoke1-test1_** public IP, the SSH connection will fail. With the PSF deployed and the Distributed Cloud Firewall Policy enforced, the PSF is an `Enforcement Security Point` like other Spoke Gateways. SSH from your laptop will be blocked unless you create a policy that allows SSH from your public IP.
```{figure} images/lab9-igw.png
---
height: 250px
align: center
---
Enforcement Security Point
```

If your SSH session on **_aws-us-east1-spoke1-test1_** is still up and responsive, go ahead with task **8.2.2**.

```{figure} images/lab9-igw456.png
---
height: 250px
align: center
---
SSH still ok
```

#### 8.2.1 SSH to aws-us-east1-spoke1-test1

Let's use the fastest approach to SSH back into the **_aws-us-east1-spoke1-test1_** instance, then from there connect to **_aws-us-east1-spoke1-test2_**.

```{important}
The **_aws-us-east1-spoke1-test2_** Public IP address is not protected by the _PSF gateway_, thus you can easily SSH to the aws-us-east1-spoke1-test2 instance first.

Once you have SSH access to the **_aws-us-east1-spoke1-test2_** instance, SSH to the private IP of **_aws-us-east1-spoke1-test1_**.
```

```{figure} images/lab9-igw32.png
---
height: 250px
align: center
---
IGW and Spoke GW
```

```{important}
**Is it possible to directly SSH into **_aws-us-east-1-spoke1-test1_** without first connecting to test2?**
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

- Navigate to this URL https://whatismyipaddress.com/ and copy your personal Public IP address.

```{figure} images/lab9-myip02.png
---
height: 400px
align: center
---
whatismyipaddress.com
```

- Navigate to **CoPilot > Groups > SmartGroups** and then click on the `"+ SmartGroup"` button.

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

- Navigate to **CoPilot > Security > Distributed Cloud Firewall** and create a new rule clicking on the `"+ Rule"` button:

```{figure} images/lab9-myip06.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>inter-myip-east1-test1-ssh</span>
- **Source Groups**: <span style='color:#479608'>My-IP</span>
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

- Retrieve the public IP address of the **aws-us-east-1-spoke1-test1** instance by navigating to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**. Search for _aws-us-east-1-spoke1-test1_, click on `"+ 1 more"` to display the public IP address, and then copy it to your clipboard.

```{figure} images/lab9-myip09.png
---
height: 200px
align: center
---
aws-us-east-1-spoke1-test1
```

- Open your **SSH client** and execute the following command, pasting the previously copied Public IP address. Use the password provided on your personal POD portal.

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
If you prefer not to attempt the hidden resolution, please proceed with the next task.
```

Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, then search for **aws-us-east-1-spoke1-test<span style='color:#33ECFF'>2</span></summary>**, retrieve its Public IP address and SSH in to the EC2 instance.

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

#### 8.2.2 Execution of the curl commands

- Execute now the following commands:

```bash
curl https://www.nginx.com
```

```bash
curl https://www.ubuntu.com
```

```bash
curl https://www.aviatrix.ai
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

Navigate to **CoPilot > Security > ThreatIQ**  section, then scroll down through the whole **Overview** section, click on the filter icon and filter out based on the Maliciuous IP (i.e., `178.17.174.164`): you can choose either **_Source_** or **_Destination_**!

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