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

Your responsibility is to conduct a POC/POV in your lab environment and demonstrate how your company can leverage the Aviatrix Cloud Perimeter Solution to address this pain point. You should deploy the Aviatrix Secure Egress solution using the Aviatrix Spoke Gateway to protect internet-bound traffic more effectively than the AWS NAT Gateway.

<ins>The Zero Trust policy should allow only the specified domains and block all other FQDNs</ins>.

Please note that the lab provides only some of the steps needed to complete this exercise. If you encounter any difficulties, you are encouraged to consult the documentation at <a href="https://docs.aviatrix.com" target="_blank">docs.aviatrix.com</a>.

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

- Before you begin building your multicloud infrastructure, please adjust the **fetch timers** in CoPilot.

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

These are the requirements for this hands-on activity. Please follow each step carefully. However, if you get stuck, feel free to jump to **_Section #7_** for a thorough solution.

### 5.1 Deploy the Aviatrix Spoke Gateway

- Create a single Aviatrix Spoke Gateway in the AWS **us-east-1** region within the VPC named `“egress-vpc”`. You may assign any name you prefer.

- The Spoke Gateway instance size should be **t3a.small**.

- The Spoke gateway must be attached to the **egress-vpc-public-us-east-1a** subnet

### 5.2 Create the WebGroups

- Create **two** WebGroups that match the domains listed on **_Section #3_**.

```{hint}
The WebGroup section will become enabled and visible in CoPilot once you activate the **Distributed Cloud Firewall** service (i.e., the Aviatrix Cloud Firewall).
```

### 5.3 Create an "editable" ExplicitDenyAll rule above the Greenfield-Rule

- Create a rule named **ExplicitDenyAll**.

- Enable **Logging** for the rule  

- Position the rule **above** _Greenfield-Rule_

```{caution}
The pre-defined ExplicitDenyAll rule is not editable; therefore, logging cannot be activated for it!
```

### 5.4 Create a SmartGroup to classify traffic passing through the private subnets

- You need to create a SmartGroup that includes all EAST-WEST traffic.

```{hint}
Use the RFC1918 routes!
```

### 5.5 Create a DCF rule that permits HTTP traffic from any private subnets, using the corresponding WebGroup.


### 5.6 Create a DCF rule that permits HTTPs traffic from any private subnets, using the corresponding WebGroup.




- The last DCF rule is a zero-trust rule
- Rule 100 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-https` web group
- Rule 0 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-http` web group

![DCF](images/egress_dcf_rules.png)

### 5.2 Create rfc1918 SmartGroup

![Group](images/egress_groups.png)

![rfc1918](images/egress_rfc1918.png)

### 5.3 Create WebGroup to Define FQDN Allowed to Access Internet

![WebGroup](images/egress_create_group.png)

![Edit Group](images/egress_edit_group.png)

![Polling](images/egress_polling.png)

### 5.4 Deploy Aviatrix Spoke GW

- The public IP address will be different (Public EIP automatically allocated by CSP)
- The Subnet CIDR could be different (automatically picked up by Aviatrix Controller)
- Region: us-east-1

![Spoke](images/egress_spoke_gw.png)

Check the Egress setting. The Egress traffic is going through the AWS NAT GW.

![Egress](images/egress_egress.png)

### 5.5 Enable spoke GW to become the Egress GW

1. Click +Local Egress on VPC/VNets.
2. In the Add Local Egress on VPC/VNets dialog, select the VPC/VNets on which to enable Local Egress.
3. Click Add.

[Read more at Aviatrix Documentation](https://docs.aviatrix.com/copilot/latest/network-security/index.html)

![Local](images/egress_add_local.png)

Add Local Egress on VPC/VNets
Adding Egress Control on VPC/VNet changes the default route on VPC/VNet to point to the Spoke Gateway and enables SNAT. Egress Control also requires additional resources on the Spoke Gateway.VPC/VNets

Now the diagram should look like the following:

![Vpc](images/egress_vpc.png)

## 6. Conclusion

By implementing Aviatrix Secure Egress, our healthcare provider strengthened their security posture, reduced costs, and eliminated the visibility gaps associated with AWS NAT Gateway. Patient data remains protected, and the provider’s reputation is safeguarded.
Remember, Aviatrix Secure Egress is your trusted solution for secure and cost-effective management of internet-bound traffic. Need assistance? Our support team is here to help.

## 7. Lab Resolution

<details>
  <summary>
Click here to view the complete walkthrough for the lab resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

### Task 5.1 resolution

- Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateway**, then click on the `"+ Spoke Gateway"` button.

Make sure these parameters are entered in the `"Create Spoke Gateway"` pop-up window.

```{note}
You need to deploy one single instance!
```

- **Name:** <span style='color:#479608'>Choose your favorite name</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-2 (Ohio)</span>
- **VPC ID:** <span style='color:#479608'>aws-us-east-2-spoke1 (Make sure you don't select aws-us-east-2-**transit** VPC)</span>
- **Instance Size:** <span style='color:#479608'>t2.medium</span>
- **High Performance Encryption:** <span style='color:#479608'>**Off**</span>
- **Attach to Subnet:** <span style='color:#479608'>10.0.1.96/27 - aws-us-east-2-spoke1-Public-1-us-east-2a</span>
- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

Click **SAVE**.

```{figure} images/lab-resegress01.png
---
align: center
---
Create Spoke Gateway in AWS
```

- Now, review the Dynamic Topology by navigating to **CoPilot > Cloud Fabric > Topology**. Here, you can see how the topology will appear after the Gateway deployment. Please keep in mind that it may take a few additional minutes for the changes to be reflected, so kindly be patient.

```{figure} images/lab-resegress02.png
---
align: center
---
height: 400px
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

### Task 5.2 resolution

This task includes an important tip: you must first activate the DCF service before proceeding with the creation of WebGroups.

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

### Task 5.3 resolution

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
align: center
---
Commit
```

### Task 5.4 resolution

This task requires you to create an ad-hoc SmartGroup, leveraging the three famous Summary Routes:
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







</details>