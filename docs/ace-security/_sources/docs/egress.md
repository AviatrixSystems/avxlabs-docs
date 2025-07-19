# Aviatrix Cloud Firewall Lab

## 1. Initial Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates exclusively within the AWS cloud environment. For internet egress traffic, they utilize AWS NAT Gateway. While the NAT Gateway effectively handles IP address translation, it is not intended to serve as a comprehensive security solution. This limited security capability has raised concerns among ABC Healthcare’s management regarding data privacy and HIPAA compliance.

### 1.1 CSP Native NAT Gateway Limitations

The current AWS NAT Gateway doesn’t provide the necessary visibility and logging capabilities, and it’s also very expensive due to 2 cents per GB egress data processing charges. ABC Healthcare’s average monthly egress traffic is around 500 TB.
The native solution also lacks visibility, is cost-prohibitive, and doesn’t support zero trust architecture—putting sensitive patient data and the healthcare provider’s reputation at risk.

## 2. Data Exfiltration Attack

To add to the challenges, ABC Healthcare recently experienced a data exfiltration attack, causing substantial disruptions, reputational harm, and a decline in its stock value. As a consequence, the company dismissed its previous cloud network architects.

### 2.1 You are the Newly Hired Cloud Networking Architect

As the newly appointed architect, your task is to secure this traffic using the `Aviatrix Cloud Firewall` service.
The Aviatrix Cloud Firewall is a key component of the `Cloud Native Security Fabric`. Unlike _traditional monolithic firewalls_ that are rigidly embedded within the network, the Aviatrix Cloud Firewall is a distributed security solution designed specifically for cloud environments. It enables the enforcement of security policies very close to the source of traffic, providing granular control and reducing the attack surface. This architecture allows organizations to implement consistent, scalable, and flexible security policies across multi-cloud and hybrid cloud deployments, ensuring comprehensive protection without sacrificing agility.

Your objective is to implement a solution that enhances visibility, offers comprehensive logging, and complies with regulatory requirements—all while being cost-effective and efficient.

## 3. ZTNA and the allowed domains

Your primary responsibility is to conduct a comprehensive Proof of Concept (POC) or Proof of Value (POV) within your laboratory environment. The goal is to demonstrate how your organization can effectively leverage the Aviatrix Cloud Native Security Fabric to address this specific challenge.

As part of this demonstration, you should deploy the Aviatrix Cloud Firewall, a feature integrated into the Aviatrix Spoke Gateway, to enhance the security posture of internet-bound traffic. This approach offers a more efficient, scalable, and robust solution compared to traditional options such as the AWS NAT Gateway. By showcasing this deployment, you will highlight the advantages of the Aviatrix platform in providing seamless security, improved performance, and simplified management for cloud network security needs.

<ins>The Zero Trust policy must permit only the specified domains listed below and block all other FQDNs:</ins>

1) **Policy #1**
   - name: `allowed-internet-http` 
   - domains:
     - *.ubuntu.com

2) **Policy #2**
   - name: `allowed-internet-https` 
   - domains:
     - *.alibabacloud.com
     - *.microsoft.com
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

```{figure} images/lab-egress033.png
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

Please set the intervals for both `Fetch GW Routes` and `Fetch VPC Routes` to **“1 Second”** each, and then click on **SAVE** to apply the changes.

```{figure} images/lab-egress04.png
---
height: 400px
align: center
---
Task Server
```

```{caution}
Please note that the order of the **_Task Server_** timers displayed in the screenshot above may vary in your CoPilot interface. The arrangement can differ depending on your configuration or updates, but the functionality and roles of the timers remain the same.
```

```{figure} images/lab-egress055.png
---
align: center
---
Fetch Instances
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
These are very **aggressive** settings. In a Production environment, <ins>you should not set these intervals that frequently!</ins>
```

## 5. Lab Objectives

The purpose of this hands-on activity is to guide you through a series of tasks designed to enhance your understanding and practical skills. Please carefully follow each step outlined below to ensure a smooth and effective learning experience. As you progress, take your time to understand each action and its purpose.

```{important}
If you encounter any difficulties or feel unsure at any point, don’t hesitate to consult the `solution section` at the very end of each task, where you'll find comprehensive solutions and troubleshooting guidance!
```

### 5.1 Initial Topology
The initial network topology consists of two Virtual Private Clouds (VPCs) located in the **US-EAST-1** region. The first VPC is dedicated to hosting both the `Aviatrix Controller` and `Aviatrix CoPilot`, which are essential for managing and monitoring the cloud network environment. The second VPC hosts an EC2 instance named `"aws-instance"`, deployed within a subnet associated with a private routing table. This routing table intentionally lacks a default route pointing to the Internet Gateway (IGW), ensuring the instance remains private and isolated from direct internet access.

To enhance security and control over outbound traffic—while avoiding reliance on the cloud provider’s native NAT Gateway—you plan to deploy an Aviatrix Spoke Gateway within the network. Additionally, you will activate the `Aviatrix Cloud Firewall` service to enforce security policies effectively, providing granular egress control and threat protection at the network edge. This setup not only improves security posture but also offers greater flexibility and centralized management for your multi-cloud environment.

```{figure} images/lab-topology00.png
---
height: 400px
align: center
---
Initial Topology
```

```{caution}
The `aws-gatus` EC2 instance is deployed  within a public subnet.  This instance provides access to the **Gatus** Dashboard, where you can monitor the type of traffic generated by the `aws-instance`. From the POD Portal, access the Gatus Dashboard by entering the required credentials. You will immediately notice that the aws-instance is generating traffic to both allowed and prohibited domains.
```{figure} images/lab-topology0033.png
---
height: 400px
align: center
---
Gatus
```

```{figure} images/lab-topology0034.png
---
height: 400px
align: center
---
all green
```

```{important}
The first entry,  `"aviatrix | www.aviatrix.com"`,  pertains to connectivity originating from the public subnet where the aws-gatus instance is deployed. 
Please keep in mind that during your connectivity tests and policy enforcement, this entry will continue to remain green.
```{figure} images/lab-topology00347.png
---
align: center
---
all green
```

## 6. Tasks
Let's begin deploying the Aviatrix Cloud Firewall.

### 6.1 Deploy the Aviatrix Spoke Gateway

- Create a single Aviatrix **Spoke** Gateway in the AWS **“US-EAST-1“** region within the VPC named `“egress-vpc”`. You may assign any name you prefer.

- The Spoke Gateway instance size should be `t3.medium`.

- The Spoke gateway must be attached to the `egress-vpc-public-us-east-1a` subnet

```{figure} images/spokegateway-security01.png
---
height: 400px
align: center
---
Spoke Gateway
```

```{important}
Please note that within the egress-VPC, there is a pre-deployed EC2 instance named **aws-instance** that is actively generating network traffic. This instance is deployed inside a private subnet, ensuring it remains isolated from direct internet access. 

In addition, there is another EC2 instance named **aws-gatus**, which is deployed within a public subnet. This instance provides access to the Gatus Dashboard, enabling monitoring and visualization of network metrics. 

Understanding the placement and roles of these instances is important for interpreting traffic flow and security configurations within the environment.
```

```{caution}
Once you have initiated the deployment of the **Spoke** Gateway, please be patient, as it may take several minutes. You can monitor the progress by clicking on the **hourglass** icon in the top right corner of the CoPilot.
```{figure} images/spokegateway-security88.png
---
height: 400px
align: center
---
Spoke Gateway Deployment
```

<details>
  <summary>
Click here to view the complete Task 6.1 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

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
- **Instance Size:** <span style='color:#479608'>t3.medium</span>
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

```{important}
Click on the VPC icon labeled `"egress-vpc"` to expand and view the various elements contained within it. This will allow you to see the associated subnets, EC2 instances, and other resources that are part of this Virtual Private Cloud, providing a clearer understanding of its structure and the relationships between its components.
```{figure} images/lab-resegress8077.png
---
height: 400px
align: center
---
egress-vpc
```

```{caution}
- The **aws-instance** EC2 was pre-provisioned at the launch of the POD and is automatically generating traffic within the **private** subnet directed toward the Internet Public Zone.

- The **aws-gatus** EC2 was pre-provisioned at the launch of the POD and is deployed within a **public** subnet. This instance provides access to the Gatus Dashboard, allowing users to monitor the traffic generated by the aws-instance EC2.
```

```{figure} images/lab-resegress8079.png
---
height: 400px
align: center
---
aws-instance and aws-gatus
```

</details>

### 6.2 Create the WebGroups

- Create **two** WebGroups that correspond to the domains listed in **_Section #3_**.

```{hint}
The WebGroup section will become enabled and visible in CoPilot once you activate the **Distributed Cloud Firewall** service (i.e., _the Aviatrix Cloud Firewall_).
```

<details>
  <summary>
Click here to view the complete task 6.2 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

This task includes an important tip: you must first activate the Distributed Cloud Firewall service before proceeding with the creation of WebGroups.

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on `"Enable Distributed Cloud Firewall"`.
Then, click on `"Begin"`.

```{figure} images/lab-resegress044.png
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
+WebGroup
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
height: 250px
align: center
---
WebGroup section
```

</details>

### 6.3 Enabling the ZTNA

After enabling the Distributed Cloud Firewall, you must activate the **Zero Trust Network Access** methodology by installing an explicit deny rule positioned above the Greenfield-Rule.

- Create a rule named **ExplicitDenyAll**.

- Enable **Logging** for the rule  

- Position the rule **above** _Greenfield-Rule_

```{caution}
Once you enable the Distributed Cloud Firewall, the Aviatrix Controller will automatically inject the `Greenfield-Rule`, allowing all types of traffic. This indicates that the Aviatrix Cloud Firewall begins operating in Deny-List model.
```

```{tip}
The `DefaultDenyAll` cannot be positioned above the Greenfield Rule, nor can you enable logging on it. Therefore, you need to create an ad-hoc explicit deny rule to fulfill this requirement!
```

<details>
  <summary>
Click here to view the complete task 6.3 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

```{figure} images/lab-resegress100.png
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

```{figure} images/lab-resegress122.png
---
height: 200px
align: center
---
Commit
```

</details>

### 6.4 Create a SmartGroup to categorize and manage traffic originating from private subnets.

- The SmartGroup should include all **East-West** traffic, ensuring comprehensive coverage of internal data flows within the private networks.

```{hint}
Use the **RFC1918** address ranges and the Resource Type **IPs / CIDRs**.
```

<details>
  <summary>
Click here to view the complete task 6.4 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

This task requires you to create an ad-hoc SmartGroup, utilizing the three well-known _Summary Routes_:
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

Navigate to **CoPilot > Groups > SmartGroups** and click on the `"+ SmartGroup"` button.

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
height: 300px
align: center
---
SmartGroups section
```

</details>

### 6.5 Enable the Cloud Egress Security on the "egress-vpc"

- Activate the `cloud egress security` control to ensure that all private routing tables within the _egress-vpc_ update their next hop from the AWS NAT Gateway to the previously deployed Aviatrix Spoke Gateway,  which incorporates the Aviatrix Cloud Firewall service. This change will enhance security and traffic control for outbound internet access.

<details>
  <summary>
Click here to view the complete task 6.5 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

In this task, you are required to enable the `Egress`.

- Before enabling Egress Control, review the current routes in the private routing tables of the egress VPC.

- Navigate to **CoPilot > Diagnostics > Cloud Routes > VPC/VNets Routes** and examine the private routing tables.

```{figure} images/lab-resegress1734.png
---
height: 400px
align: center
---
Current Status of the Private Routing Tables
```

```{important}
Currently, two private routing tables have a default route pointing to the **AWS NAT Gateway**.
```

- Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and click on the `"Enable Local Egress on VPC/VNets"` button located on the right side of the **egress-vpc** entry. After that, select "egress-vpc" from the drop-down menu and click **Add** to complete the configuration.

```{figure} images/lab-resegress177.png
---
height: 400px
align: center
---
Enable
```

```{figure} images/lab-resegress1777.png
---
height: 400px
align: center
---
Select the egress-vpc
```

After a few seconds, click on the refresh button located at the bottom right side. You will notice that under the Point of Egress column, the value has changed from `"Native Cloud Egress"` to `"Local Egress"`.

```{figure} images/lab-resegress17777.png
---
height: 400px
align: center
---
Local Egress
```

- Once Cloud Egress Security is enabled, access the Gatus Dashboard from your personal POD Portal. You will observe that all traffic starts to fail, turning **red**, as a result of the ExplicitDenyRule applied above the Greenfield rule.

```{figure} images/lab-resegress1756.png
---
height: 400px
align: center
---
Turning red
```

```{important}
To reduce the timer duration, adjust the polling timer by selecting 10 seconds in the bottom-left corner.
```{figure} images/lab-resegress17567.png
---
height: 400px
align: center
---
10seconds
```

</details>

### 6.6 Create a Distributed Cloud Firewall rule that allows HTTP traffic from all private subnets by utilizing the appropriate WebGroup

- This Distributed Cloud Firewall rule should strictly permit **HTTP** traffic originating from any subnets associated with a private routing table to access the internet. The rule should specifically target the domains included in the _allowed-internet-http_ WebGroup, ensuring controlled and secure outbound web access.

<details>
  <summary>
Click here to view the complete Task 6.6 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

This task involves creating a Distributed Cloud Firewall rule and associating it with the WebGroup you previously established.

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
Save in Drafts
```

</details>

### 6.7 Create a Distributed Cloud Firewall rule that allows HTTPS traffic from all private subnets by utilizing the appropriate WebGroup

- This Distributed Cloud Firewall rule should exclusively permit **HTTPS** traffic originating from any subnets associated with a private routing table to access the internet. The rule should specifically target the domains listed in the _allowed-internet-https_ WebGroup, ensuring secure and controlled outbound connections.

<details>
  <summary>
Click here to view the complete Task 6.7 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

This task involves creating an additional Distributed Cloud Firewall rule and attaching it to the WebGroup you previously established. This will help you define specific security policies and ensure consistent enforcement across your network resources.

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

### 6.8 Verify that the Monitor section within the Egress area is effectively safeguarding the private subnet

- Once the <ins>permitted</ins> Distributed Cloud Firewall rules are enforced on the Spoke Gateway and integrated into the Aviatrix Cloud Firewall, you should observe **logs** displaying traffic to both the allowed domains and the prohibited domains (which will be dropped).

```{figure} images/lab-topology01.png
---
height: 400px
align: center
---
Final Topology
```

<details>
  <summary>
Click here to view the complete Task 6.8 resolution: <span style='color:#33ECFF'>[disclose the RESOLUTION]</span></summary>

This is the final task—simply review the logs in the **Egress** section for verification.

- Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)**, then select _egress-vpc_ from the drop-down menu in the `VPC/VNets` field.

```{figure} images/lab-resegress24.png
---
height: 200px
align: center
---
Monitor
```

You will immediately notice the allowed domains, thanks to the two Distributed Cloud Firewall (DCF) rules you configured, which permit traffic to these destinations. At the same time, you will see attempts to access prohibited domains, which are subsequently denied in accordance with the `Zero Trust` approach enabled by the Aviatrix Cloud Firewall. This setup effectively demonstrates how the security policies are enforced in real-time, ensuring only authorized traffic is permitted while malicious or unauthorized access is promptly blocked.

```{figure} images/lab-resegress25.png
---
height: 400px
align: center
---
Logs
```

</details>

## 7. Conclusion

By implementing the `Aviatrix Cloud Firewall`, our healthcare provider enhanced their security posture, reduced costs, and closed visibility gaps previously associated with the AWS NAT Gateway. Patient data remains protected, and the provider’s reputation is preserved.  

Remember, the `Aviatrix Cloud Firewall` is your trusted solution for secure and cost-effective management of internet-bound traffic.