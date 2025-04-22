# Backbone Lab

## 1. Scenario

ABC Healthcare, a leading healthcare provider, operates exclusively within the **Azure** cloud environment. The company has decided to expand its cloud infrastructure to include AWS; however, it is facing a skills gap, as it lacks expertise in AWS. Additionally, ABC Healthcare heavily relies on native cloud service provider (CSP) transit solutions, such as Azure Route Server.

## 2. Lab Objective

As the newly appointed solutions architect, you have been assigned the following tasks:

- Design a **_`multicloud backbone architecture`_** that operates seamlessly across all cloud platforms.

- Ensure the solution provides **_`hybrid connectivity`_** to the data center.

```{important}
**Azure Route Server** does not natively support connectivity to other cloud providers, whereas Aviatrix supports `Multicloud Transit`.
```

## 3. Initial Set-up

1- Both the `Aviatrix Controller` and the `Aviatrix CoPilot` have already been deployed in AWS, within a dedicated _management_ VPC.

2- A pair of `Transit Gateways` has also been deployed in Azure and connected to the existing **Azure Route Server**.

3- An `Aviatrix Secure Edge` has already been deployed in the on-premises data center.

```{figure} images/backbone-initial-topology.png
---
height: 400px
align: center
---
Initial Topology
```

## 4. Aviatrix CoPilot

Now, let's access the Aviatrix Centralized Management Node, the **CoPilot**.

Navigate to your POD portal, click on the `"Copilot"` button, and enter your credentials.

```{figure} images/lab-backbone01.png
---
height: 300px
align: center
---
POD Portal
```

```{figure} images/lab-backbone02.png
---
height: 400px
align: center
---
CoPilot UI - Welcome page
```

- Before building your backbone infrastructure, adjust the following **Fetch Timers** to their lowest values.

```{hint}
Go to **CoPilot > Settings > Resources > Task Server**
```

Please set both the `Fetch GW Routes` and `Fetch VPC Routes` intervals to **“1 Second”** each, then click **SAVE**.

```{hint}
Click the _pencil_ icon to edit the timers.
```

```{figure} images/backbone-tasks.png
---
height: 400px
align: center
---
Initial Topology
```

Remember to click the **COMMIT** button!

```{caution}
These settings are quite aggressive; in a production environment, it's advisable to avoid setting these intervals so frequently!
```

## 5. Gatus

All pre-deployed instances are running [Gatus](https://gatus.io/) and are attempting to connect with each other using various protocols, including **ICMP** and **TCP** on ports 1433, 3306, 443, 5000, 50100.

You can access the Gatus dashboard directly from your POD Portal.

```{figure} images/lab-gatus00.png
---
height: 400px
align: center
---
Gatus from the POD Portal
```

You can select any of the three pre-deployed instances in AWS, Azure, and the on-premises data center, and you will find that the connectivity is indeed broken. This is because all the involved VPCs are currently isolated, as you have not yet created the `MCNA` (i.e. MultiCloud Network Architecture).

## 6. Create an AWS TGW using the CoPilot

The CoPilot provides an `AWS TGW (Transit Gateway) Network Orchestration` service, allowing you to deploy AWS Transit Gateways seamlessly without relying on the AWS Console.

```{figure} images/backbone-tgw01.png
---
height: 400px
align: center
---
AWS TGW 
```

- Navigate to **CoPilot > Networking > Connectivity > AWS TGW** and click on the `"+ AWS TGW"` button.

```{figure} images/backbone-tgw02.png
---
height: 400px
align: center
---
AWS TGW section on the CoPilot
```

Please make sure to enter these parameters in the `"Create AWS TGW"` pop-up window.

- **Name:** <span style='color:#479608'>AWS-NVirginia-TGW</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account </span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **AWS Side SAS Number:** <span style='color:#479608'>64512 (_default value_)</span>

```{figure} images/backbone-tgw03.png
---
height: 400px
align: center
---
AWS TGW template
```

Do not forget to click on **SAVE**.

```{caution}
It will take approximately **2 minutes** for the Aviatrix Controller to complete the deployment of the AWS TGW.
```

```{figure} images/backbone-tgw04.png
---
align: center
---
Final Deployment outcome
```

## 7. Attach VPC to AWS TGW

Now that you have an AWS TGW, you need also to attach it to the existing **spoke-vpc**.

Click on the `AWS-NVirginia-TGW` element, select the **VPC** tab and then click on the `"Attach VPC"` button.

```{figure} images/backbone-tgw05.png
---
align: center
---
AWS NVirginia-TGW
```

Make sure to enter these parameters in the `"Attach VPC to AWS TGW"` pop-up window.

- **VPC:** <span style='color:#479608'>spoke-aws</span>
- **Network Domain:** <span style='color:#479608'>Default_Domain (_default setting_)</span>

```{figure} images/backbone-tgw06.png
---
height: 400px
align: center
---
AWS NVirginia-TGW
```

Do not forget to click on **SAVE**.

```{caution}
It will take roughly **2 minutes** for the Aviatrix Controller to complete the attachment.
```

```{figure} images/backbone-tgw07.png
---
height: 200px
align: center
---
Attachment
```

## 8. Create an Aviatrix Transit VPC

Let's continue fortifying the cloud infrastructure. Your next task is to create a `Transit VPC`.

```{figure} images/backbone-tgw08.png
---
height: 400px
align: center
---
Dedicated Transit VPC
```

- Navigate to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets** and click on the `"+ VPC/VNet"` button.

```{figure} images/backbone-tgw09.png
---
align: center
---
Transit VPC
```

Please ensure these parameters are entered in the `"Create VPC/VNet"` window.

- **Name:** <span style='color:#479608'>transit-aws</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC CIDR:** <span style='color:#479608'>10.10.0.0/16</span>
- **VPC Function:** <span style='color:#479608'>Transit + FireNet</span>

```{figure} images/backbone-tgw010.png
---
align: center
---
VPC Template
```

Do not forget to click on **Save**.

```{note}
Wait a few minutes for the task to complete. Check the _hourglass icon_ on the right side of your Copilot.
```

## 9. Create both the Transit Gateways and the peering

Now it's time to deploy a pair of **`Transit Gateways`** within the VPC you set up earlier. You'll also need to establish `peering` between the Transit Gateways in Azure and those in AWS.

```{figure} images/backbone-tgw011.png
---
height: 400px
align: center
---
Transit Gateways and the Peering
```

- Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the `"+ Transit Gateway"` button.

```{figure} images/backbone-tgw12.png
---
height: 400px
align: center
---
Transit Gateways section
```

Make sure to enter these parameters in the `"Create Transit Gateway"` pop-up window.

- **Name:** <span style='color:#479608'>transit-aws</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC/VNet:** <span style='color:#479608'>transit-aws</span>
- **Instance Size:** <span style='color:#479608'>c6in.large</span>
- **High Performance Encryption:** <span style='color:#479608'>**ON**</span>
- **Peer To Transit Gateways:** <span style='color:#479608'>transit-azure</span>

Then click on the `"+ Instance"` button!

**Instance-1**:
- **Attach to Subnet:** <span style='color:#479608'>us-east-1a</span>

**Instance-2**:
- **Attach to Subnet:** <span style='color:#479608'>us-east-1b</span>

```{figure} images/backbone-tgw013.png
---
align: center
---
Transit GW Template
```

Do not forget to click on **SAVE**.

```{note}
The Aviatrix Controller will deploy two Transit Gateways and set up peering with the pre-deployed Transit Gateways in Azure at the same time.
```

You can track the progress of the task by going to **CoPilot > Monitor > Notifications > Tasks** , then expanding the task labeled `"Create transit gateway: transit-aws"`.

```{figure} images/backbone-tgw014.png
---
align: center
---
Task in progress
```

```{caution}
It will take approximately **10 minutes** for the Aviatrix Controller to complete this task, so please be patient!
```

- Now, navigate to **CoPilot > Cloud Fabric > Topology**. Click the `"Managed"` button to hide all unmanaged VPCs (i.e., VPCs without an Aviatrix Gateway), and then click the `"Collapse all VPC/VNets"` button.

```{figure} images/backbone-tgw015.png
---
height: 400px
align: center
---
Dynamic Topology
```

You will see the newly created **peering**.

## 10. Attach Transit Gateways to aws-tgw

Now let's attach the Transit Gateways in AWS to the AWS TGW.

```{figure} images/backbone-tgw016.png
---
height: 400px
align: center
---
Attachment
```

- Navigate to **CoPilot > Networking > Connectivity > AWS TGW**, select the _AWS-NVirginia-TGW_ instance and then click the `"Attach Transit Gateway"` button.

```{figure} images/backbone-tgw017.png
---
align: center
---
"Attach Transit GW" button
```

Ensure that you enter this parameter in the pop-up window titled `"Attach Transit Gateway to AWS-NVirginia-TGW"`.

- **Transit Gateway:** <span style='color:#479608'>transit-aws</span>

```{figure} images/backbone-tgw018.png
---
align: center
---
Attachment Template
```

Remember to click **Save**!

```{caution}
It will take approximately **3 minutes** for the Aviatrix Controller to complete this task, so please be patient!
```

## 11. Configure BGP ASN

Now that you’ve established the peering between the two cloud service providers, you’ve successfully created the **`Aviatrix Cloud Backbone`**.

Let’s proceed with the final deployment of connectivity to the on-premises data center.

Before configuring the attachments between the Secure Edge Gateway and the Transit Gateways, ensure that the Transit Gateway cluster previously deployed in AWS is configured with a **`BGP AS number`**. <ins>This is a prerequisite for completing the Edge deployment</ins>!

- Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the `transit-aws` cluster!

```{figure} images/backbone-tgw019.png
---
height: 300px
align: center
---
transit-aws
```

Go to the `"Settings"` tab, expand the `“Border Gateway Protocol (BGP)”` section, and enter the AS number **64582** in the field labeled `"“Local AS Number”`. Then, click **Save**.

```{figure} images/backbone-tgw020.png
---
align: center
---
BGP ASN
```

After completing this task, here’s how the overall topology will appear.

```{figure} images/backbone-tgw021.png
---
height: 400px
align: center
---
Topology after task #6
```

## 12. Edge to Transits

Let's now connect the **Secure Edge Gateway** to the **Transit Gateways**.

```{figure} images/backbone-tgw022.png
---
height: 400px
align: center
---
The Hybrid Cloud through the Aviatrix Backbone
```

- Navigate to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Transit Gateway Attachment"` button located on the right side of the screen.

```{figure} images/backbone-tgw023.png
---
height: 200px
align: center
---
Manage Gateway Attachment
```

Click on the `"+Attachment"` button.

```{figure} images/backbone-tgw024.png
---
align: center
---
+Attachment
```

Fill out the attachment template with the following settings:

- **Transit Gateway**: <span style='color:#479608'>transit-aws</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>
- **Number of Tunnels**: <span style='color:#479608'>**Custom**</span>
  - 4

```{figure} images/backbone-tgw025.png
---
align: center
---
Template for aws
```

```{caution}
Please make sure to select `"Attach over: Public Network"`. 

If you accidentally choose "_Attach over: Private Network_", the connection will not be established!
```{figure} images/edgeveryverynew.png
---
align: center
---
Attach over PUBLIC Network
```

Before saving the template, click the `"+ Attachment"` button and enter the following values in the second attachment template:

- **Transit Gateway**: <span style='color:#479608'>azure-aws</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>
- **High Performance Encryption**: <span style='color:#479608'>**ON**</span>
- **Number of Tunnels**: <span style='color:#479608'>**4**</span>

Once completed, click the **Save** button.

```{figure} images/backbone-tgw026.png
---
align: center
---
Template for azure
```

- Navigate to **CoPilot > Cloud Fabric > Hybrid Cloud** and select the **aviarix-edge-1** instance.

```{figure} images/backbone-tgw027.png
---
align: center
---
aviatrix-edge-1
```

Navigate to the Gateway Routes tab and search for the subnet **`10.1.2.0`** in AWS. You should see four IPSec tunnels connected to the First Transit Gateway, along with another four IPSec tunnels connected to the Second Transit Gateway.

```{figure} images/backbone-tgw028.png
---
height: 400px
align: center
---
8x IPSec tunnels towards AWS
```

Now, search for the subnet **`10.2.2.0`** in Azure. You will observe four IPSec tunnels for each Transit Gateway once again.

```{figure} images/backbone-tgw029.png
---
height: 400px
align: center
---
8x IPSec tunnels towards Azure
```

You have successfully extended the Aviatrix solution to the on-premises Data Center by leveraging the **`High Performance Encryption`**  feature.

## 13. Connectivity Test

Let's now verify the connectivity between Azure, AWS, and the on-premises Data Center.

Navigate to **CoPilot > Cloud Resources > Cloud Assets** and search for the **aws-instance** on the Search field, then retrieve its **Public** IP address!

```{figure} images/backbone-tgw233.png
---
height: 400px
align: center
---
click on +1 more
```

```{figure} images/backbone-tgw030.png
---
height: 200px
align: center
---
aws-instance on the Cloud Assets
```

Open an **SSH client** and establish a session with the <ins>aws-instance</ins>.

```{figure} images/backbone-tgw031.png
---
align: center
---
SSH
```

Now let's ping the **Private** IP address of the <ins>azure-instance</ins>

```{figure} images/backbone-tgw032.png
---
height: 400px
align: center
---
azure-instance
```

Run the following command to verify connectivity to the **_azure-instance_**:

```bash
ping 10.2.2.40 
```

```{figure} images/backbone-tgw033.png
---
height: 200px
align: center
---
ping towards azure-instance
```

Stop the command, then attempt to ping the **test-vm** inside the on-premises Data Center.

```{figure} images/backbone-tgw034.png
---
height: 400px
align: center
---
ping towards the test-vm inside the DC
```

Run the following command to verify connectivity to the **_test-vm_**:

```bash
ping 10.40.251.29
```

```{figure} images/backbone-tgw035.png
---
height: 400px
align: center
---
ping towards the test-vm inside the DC
```

You have successfully verified the connectivity between the two CSPs and the on-premises Data Center via the Aviatrix Cloud Backbone.

Wait 4-5 minutes, then navigate to **CoPilot > Monitor > FlowIQ**. In the `"Filters"` field, click the **+** button to create the following condition:

```bash
Destination IP Address is 10.2.2.40
```

```{figure} images/backbone-tgw036.png
---
height: 150px
align: center
---
FlowIQ
```

```{figure} images/backbone-tgw037.png
---
height: 150px
align: center
---
Condition
```

```{important}
`FlowIQ` provides a very interesting service that helps you gaining a very rich and deep visibility into your Traffic Flows.

For all network traffic moving across your Aviatrix-managed network, Aviatrix gateways capture `metadata` for all traffic traversing their links. 
```

```{figure} images/backbone-tgw038.png
---
height: 400px
align: center
---
FlowIQ outcome
```

## 14. Conclusion

In this lab, you successfully designed and implemented a multi-cloud backbone architecture for ABC Healthcare, integrating their cloud infrastructure across Azure and AWS. Despite initial challenges, such as the absence of native connectivity between Azure Route Server and other cloud providers, you utilized Aviatrix’s multi-cloud transit capabilities to develop an efficient and resilient solution. Through a series of guided tasks, you deployed key infrastructure components, including AWS Transit Gateway, Aviatrix Transit VPCs, and Aviatrix Transit Gateways. As a result, ABC Healthcare now has a `scalable and high-performance multi-cloud network backbone`, well-equipped to support its expanding cloud initiatives with improved inter-cloud and hybrid connectivity.
