# Aviatrix Cloud Firewall Lab

## 1. Initial Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates exclusively within the AWS cloud environment. For internet egress traffic, they utilize AWS NAT Gateway. While the NAT Gateway effectively handles IP address translation, it is not intended to serve as a comprehensive security solution. This limited security capability has raised concerns among ABC Healthcare’s management regarding data privacy and HIPAA compliance.

### 1.1 CSP Native NAT Gateway Limitations

The current AWS NAT Gateway doesn’t provide the necessary visibility and logging capabilities, and it’s also very expensive due to 2 cents per GB egress data processing charges. ABC Healthcare’s average monthly egress traffic is around 500 TB.
The native solution also lacks visibility, is cost-prohibitive, and doesn’t support zero trust architecture—putting sensitive patient data and the healthcare provider’s reputation at risk.

## 2. Data Exfiltration Attack

To add to the challenges, ABC Healthcare recently experienced a data exfiltration attack, causing substantial disruptions, reputational harm, and a decline in its stock value. As a consequence, the company dismissed its previous cloud network architects.

### 2.1 You are the Newly Hired Cloud Networking Architect

As the newly appointed architect, your task is to secure this traffic using the `Aviatrix Cloud Firewall` service. Your objective is to implement a solution that enhances visibility, offers comprehensive logging, and complies with regulatory requirements—all while being cost-effective and efficient.

## 3. ZTNA and the allowed domains

Your responsibility is to conduct a Proof of Concept (POC) or Proof of Value (POV) within your lab environment, demonstrating how your company can effectively leverage the Aviatrix Cloud Perimeter Solution to address this specific challenge. You should deploy the Aviatrix Cloud Firewall using the Aviatrix Spoke Gateway to enhance the security of internet-bound traffic, providing a more efficient and robust solution compared to the AWS NAT Gateway.

<ins>The Zero Trust policy must permit only the specified domains listed below and block all other FQDNs:</ins>

- `allowed-internet-http` domains
  - *.ubuntu.com
- `allowed-internet-https` domains
  - *.alibabacloud.com
  - azure.microsoft.com
  - aws.amazon.com
  - *.amazonaws.com
  - *.aviatrix.com
  - aviatrix.com
  - cloud.google.com
  - *.docker.com
  - *.docker.io
  - www.oracle.com

## 4. Aviatrix CoPilot

Now, let's access the Aviatrix Centralized Management Node, the **CoPilot**.

Navigate to your POD portal, click on the `"Copilot"` button, and enter your credentials.

```{figure} images/lab-egress01.png
---
height: 300px
align: center
---
POD Portal
```

```{figure} images/lab-egress02.png
---
height: 400px
align: center
---
CoPilot UI - Welcome page
```

The CoPilot *Dashboard* should look something like this:

```{figure} images/lab-egress03.png
---
height: 400px
align: center
---
Dashboard
```

- Before you begin building your `Aviatrix Cloud Firewall`, please adjust the **fetch timers** in CoPilot.

```{hint}
Navigate to **CoPilot > Settings > Resources > Task Server**
```

Set the intervals for `Fetch GW Routes` and `Fetch VPC Routes` to **“1 Second”** each, then click on **SAVE**.

```{figure} images/lab-egress04.png
---
height: 400px
align: center
---
Task Server
```

```{caution}
The order of the **_Task Servers_** on the screenshot above might be different on your CoPilot
```

```{figure} images/lab-egress05.png
---
align: center
---
Fetch GW Routes
```

```{figure} images/lab-egress06.png
---
align: center
---
Fetch VPC Routes
```

After that, click on **Commit**.

```{figure} images/lab-egress07.png
---
height: 400px
align: center
---
Commit
```

```{note}
These are very aggressive settings. In a Production environment, you should not set these intervals that frequently!
```

## 5. Lab Objectives

The purpose of this hands-on activity is to guide you through a series of tasks designed to enhance your understanding and practical skills. Please carefully follow each step outlined below to ensure a smooth and effective learning experience. As you progress, take your time to understand each action and its purpose.

```{important}
If you encounter any difficulties or feel unsure at any point, don’t hesitate to consult the `solution section` at the very end of each task, where you'll find comprehensive solutions and troubleshooting guidance!
```

### 5.1 Initial Topology
The initial topology consists of two VPCs in ****US-EAST-1** region: one dedicated to hosting both the `Aviatrix Controller` and the `Aviatrix CoPilot`, and another hosting an EC2 instance named `"aws-instance"` deployed within a subnet associated with a private routing table (i.e., _a routing table without a default route pointing to the IGW_). To ensure secure egress access and control—without using the cloud provider's native NAT gateway—you plan to deploy an Aviatrix Spoke Gateway and activate the `Aviatrix Cloud Firewall` service.

```{figure} images/lab-topology00.png
---
height: 400px
align: center
---
Initial Topology
```

## 6. Tasks
Let's begin deploying the Aviatrix Cloud Firewall.

### 6.1 Deploy the Aviatrix Spoke Gateway

- Create a single Aviatrix Spoke Gateway in the AWS **US-EAST-1** region within the VPC named `“egress-vpc”`. You may assign any name you prefer.

- The Spoke Gateway instance size should be `t3a.small`.

- The Spoke gateway must be attached to the `egress-vpc-public-us-east-1a` subnet

```{important}
Please note that within the egress-vpc, there is a pre-deployed EC2 instance named **aws-instance** that is actively generating traffic.
```

<details>
  <summary>
Click here to view the complete Task 6.1 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>
### Task 6.1 resolution

- Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateway**, then click on the `"+ Spoke Gateway"` button.

Make sure these parameters are entered in the `"Create Spoke Gateway"` pop-up window.

```{note}
You need to deploy one single instance!
```

- **Name:** <span style='color:#479608'>Choose your favorite name</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC ID:** <span style='color:#479608'>egress-vpc</span>
- **Instance Size:** <span style='color:#479608'>t3a.small</span>
- **High Performance Encryption:** <span style='color:#479608'>**Off**</span>
- **Attach to Subnet:** <span style='color:#479608'>10.1.2.32/28 - egress-vpc-public-us-east-1a</span>
- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

Click **Save**.

```{figure} images/lab-resegress01.png
---
align: center
---
Create Spoke Gateway in AWS
```

- Now, review the `Dynamic Topology` by navigating to **CoPilot > Cloud Fabric > Topology**. Here, you can see how the topology will appear after the Gateway deployment. Please keep in mind that it may take a few additional minutes for the changes to be reflected, so kindly be patient.

```{figure} images/lab-resegress02.png
---
align: center
height: 400px
---
Topology
```

```{important}
If you do not visualize the same topology, do not forget to click on the `"Managed"` button, to keeping hidden the _Unmanaged_ VPCs.
```{figure} images/lab-resegress03.png
---
height: 400px
align: center
---
Managed VPCs
```

```{figure} images/lab-resegress80.png
---
height: 400px
align: center
---
aws-instance
```

```{caution}
The **aws-instance** was pre-provisioned at the launch of the POD and is automatically generating traffic.
```

</details>

### 6.2 Create the WebGroups

- Create **two** WebGroups that match the domains listed on **_Section #3_**.

```{hint}
The WebGroup section will become enabled and visible in CoPilot once you activate the **Distributed Cloud Firewall** service (i.e., __the Aviatrix Cloud Firewall_).
```

<details>
  <summary>
Click here to view the complete task 6.2 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>
### Task 6.2 resolution

This task includes an important tip: you must first activate the Distributed Cloud Firewall service before proceeding with the creation of WebGroups.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on `"Enable Distributed Cloud Firewall"`.
Then, click on `"Begin"`.

```{figure} images/lab-resegress04.png
---
align: center
---
Begin using Distributed Cloud Firewall
```

```{figure} images/lab-resegress05.png
---
align: center
---
Begin
```

- Navigate to **CoPilot > Groups > WebGroups** and click on the `"+ WebGroup"` button.

```{figure} images/lab-resegress06.png
---
height: 400px
align: center
---
Begin
```

- Create the first **_WebGroup_** with the following parameters:

  - **Name**: <span style='color:#479608'>allowed-internet-http</span>
  - **Type**: <span style='color:#479608'>Domains</span>
  - **Domains/URLs**: <span style='color:#479608'>*.ubuntu.com</span>

Do not forget to click on **Save**.

```{figure} images/lab-resegress07.png
---
align: center
---
WebGroup creation
```

- Let's proceed to create the second **_WebGroup_** using the following parameters:

  - **Name**: <span style='color:#479608'>allowed-internet-https</span>
  - **Type**: <span style='color:#479608'>Domains</span>
  - **Domains/URLs**: <span style='color:#479608'>*.alibabacloud.com</span>
  - **Domains/URLs**: <span style='color:#479608'>azure.microsoft.com</span>
  - **Domains/URLs**: <span style='color:#479608'>aws.amazon.com</span>
  - **Domains/URLs**: <span style='color:#479608'>*.amazonaws.com</span>
  - **Domains/URLs**: <span style='color:#479608'>*.aviatrix.com</span>
  - **Domains/URLs**: <span style='color:#479608'>aviatrix.com</span>
  - **Domains/URLs**: <span style='color:#479608'>cloud.google.com</span>
  - **Domains/URLs**: <span style='color:#479608'>*.docker.com</span>
  - **Domains/URLs**: <span style='color:#479608'>*.docker.io</span>
  - **Domains/URLs**: <span style='color:#479608'>www.oracle.com</span>

Do not forget to click on **Save**.

```{figure} images/lab-resegress08.png
---
align: center
---
WebGroup #2
```

```{figure} images/lab-resegress09.png
---
height: 400px
align: center
---
WebGroup section
```

</details>

### 5.4 Create an "editable" ExplicitDenyAll rule above the Greenfield-Rule

- Create a rule named **ExplicitDenyAll**.

- Enable **Logging** for the rule  

- Position the rule **above** _Greenfield-Rule_

```{caution}
The pre-defined ExplicitDenyAll rule is not editable; therefore, logging cannot be activated for it!
```

<details>
  <summary>
Click here to view the complete task 6.4 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>
### Task 6.2 resolution


### Task 5.4 resolution

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab-resegress10.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>ExplicitDenyAll</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab-resegress11.png
---
align: center
---
ExplicitDenyAll 
```

Do not forget to click on **Commit**.

```{figure} images/lab-resegress12.png
---
height: 250px
align: center
---
Commit
```

</details>

### 5.5 Create a SmartGroup to classify traffic passing through the private subnets

- You need to create a SmartGroup that includes all EAST-WEST traffic.

```{hint}
Use the RFC1918 routes!
```

### 5.6 Enable the Local Egress on the egress-vpc VPC

- Activate the `local egress` service so that any private routing tables within the _egress-vpc_ will receive a default route pointing to the Spoke Gateway.

### 5.7 Create a DCF rule that permits HTTP traffic from any private subnets, using the corresponding WebGroup

- This Distributed Cloud Firewall rule should exclusively allow **HTTP** traffic originating from any subnets linked to a private routing table to access the internet, specifically targeting the domains listed in the _allowed-internet-http_ WebGroup.

<details>
  <summary>
Click here to view the complete Task 5.7 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

### Task 5.7 resolution

This task involves creating a Distributed Cloud Firewall rule and attaching the WebGroup you previously created.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab-resegress19.png
---
height: 400px
align: center
---
+Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>Choose your favorite name</span>
- **Source Smartgroups**: <span style='color:#479608'>rfc1918</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>**allowed-internet-http**</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>80</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab-resegress20.png
---
align: center
---
Saving the new Rule
```

</details>

### 5.8 Create a DCF rule that permits HTTPs traffic from any private subnets, using the corresponding WebGroup

- This Distributed Cloud Firewall rule should exclusively allow **HTTPS** traffic originating from any subnets linked to a private routing table to access the internet, specifically targeting the domains listed in the _allowed-internet-https_ WebGroup.

<details>
  <summary>
Click here to view the complete Task 5.8 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

### Task 5.8 resolution

This task involves creating another Distributed Cloud Firewall rule and attaching the WebGroup you previously created.

- Before clicking the **Commit** button, click again the `"+ Rule"` button.

```{figure} images/lab-resegress21.png
---
height: 250px
align: center
---
+Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>Choose your favorite name</span>
- **Source Smartgroups**: <span style='color:#479608'>rfc1918</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>**allowed-internet-https**</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>443</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab-resegress22.png
---
align: center
---
Saving the new Rule
```

Now you can click on **Commit**.

```{figure} images/lab-resegress23.png
---
height: 250px
align: center
---
Commit
```

</details>

### 5.9 Verify that the Monitor section in the Egress area is effectively protecting the private subnet

- After enabling the Aviatrix Cloud Firewall, you should see logs reflecting traffic to the permitted domains. All other traffic will be denied.

```{figure} images/lab-topology01.png
---
height: 400px
align: center
---
Final Topology
```

<details>
  <summary>
Click here to view the complete Task 6.9 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

### Task 6.9 resolution

This is the final task—simply review the logs in the **Egress** section for verification.

- Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)**, then select _egress-vpc_ from the drop-down menu in the `VPC/VNets` field.

```{figure} images/lab-resegress24.png
---
height: 300px
align: center
---
Monitor
```

You will immediately observe the allowed domains, thanks to the two DCF rules you created, as well as the prohibited domains that were accessed but subsequently denied.

```{figure} images/lab-resegress25.png
---
height: 400px
align: center
---
Logs
```
</details>

## 6. Conclusion

By implementing the `Aviatrix Cloud Firewall`, our healthcare provider enhanced their security posture, reduced costs, and closed visibility gaps previously associated with the AWS NAT Gateway. Patient data remains protected, and the provider’s reputation is preserved.  

Remember, the `Aviatrix Cloud Firewall` is your trusted solution for secure and cost-effective management of internet-bound traffic.


### Task 5.5 resolution

This task requires you to create an ad-hoc SmartGroup, utilizing the three well-known _Summary Routes_:
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

- Navigate to **CoPilot > Groups > SmartGroups** and click on the `"+ SmartGroup"` button.

```{figure} images/lab-resegress13.png
---
height: 400px
align: center
---
SmartGroup
```

Now, click on the arrow icon  inside the `"+ Resource Type"` button and select `"IP / CIDRs"`.

```{figure} images/lab-resegress14.png
---
height: 400px
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>rfc1918</span>
- **IPs / CIDRs**: <span style='color:#479608'>10.0.0.0/8</span>
- **IPs / CIDRs**: <span style='color:#479608'>172.16.0.0/12</span>
- **IPs / CIDRs**: <span style='color:#479608'>192.168.0.0/16</span>

Before clicking on **SAVE**, delete the empty `"Virtual Machines"` additional condition.

```{figure} images/lab-resegress15.png
---
height: 400px
align: center
---
New SG
```

```{figure} images/lab-resegress16.png
---
height: 400px
align: center
---
SmartGroups section
```

### Task 5.6 resolution

In this task, you are required to enable the `Local Egress`.

- Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and click on the `"Enable Local Egress on VPC/VNets"` button.

```{figure} images/lab-resegress17.png
---
height: 400px
align: center
---
Egress
```

- Now, select the _egress-vpc_ and then click on **Add**.

```{figure} images/lab-resegress18.png
---
height: 400px
align: center
---
egress-vpc with local Egress
```

</details>

### Task 5.7 resolution

This task involves creating a Distributed Cloud Firewall rule and attaching the WebGroup you previously created.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab-resegress19.png
---
height: 400px
align: center
---
+Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>Choose your favorite name</span>
- **Source Smartgroups**: <span style='color:#479608'>rfc1918</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>**allowed-internet-http**</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>80</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab-resegress20.png
---
align: center
---
Saving the new Rule
```

</details>