# Backbone Lab

## Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates solely within the **Azure** cloud environment. The company decided expaning their cloud infrastructure also within AWS, however, they are facing cloud skills gap, because they do not have experties in AWS yet. Moreover, the company is heavily using the native CSP transit solutions (e.g. Azure Route Server).

## Lab Objective

You, the newly appointed solutions architect, have been assigned the following task:

- Create a **_multicloud backbone architecture_** that works in every cloud.
- The solution should also provide **_hybrid connectivity_** towards the Data Center.

```{important}
**Azure Route Server** does not natively sypport connectivity to other Cloud Providers.

Aviatrix supports `Multicloud Transit`.
```

## Initial Set-up

1- Both the `Aviatrix Controller` and the `Aviatrix CoPilot` got already deployed in AWS, inside a dedicated MGMT VPC.

2- A pair of `Transit Gateways` got also deployed in Azure and attached to the existing **Azure Route Server**.

3- An `Aviatrix Secure Edge` got already deployed inside the on-prem Data Center.

```{figure} images/backbone-initial-topology.png
---
height: 400px
align: center
---
Initial Topology
```

## LAB Access Details

| **POD#** | **Copilot** |
|:----------:|:---------------:|
|      6     |       <a href="https://cplt.pod6.aviatrixlab.com" target="_blank">POD##</a>     |
|      7     |       <a href="https://cplt.pod7.aviatrixlab.com" target="_blank">POD##</a>     |
|      8     |       <a href="https://cplt.pod8.aviatrixlab.com" target="_blank">POD##</a>     |
|      9     |       <a href="https://cplt.pod9.aviatrixlab.com" target="_blank">POD##</a>     |
|      10     |       <a href="https://cplt.pod10.aviatrixlab.com" target="_blank">POD##</a>     |
|      11     |       <a href="https://cplt.pod11.aviatrixlab.com" target="_blank">POD##</a>     |
|      12     |       <a href="https://cplt.pod12.aviatrixlab.com" target="_blank">POD##</a>     |
|      13     |       <a href="https://cplt.pod13.aviatrixlab.com" target="_blank">POD##</a>     |
|      14     |       <a href="https://cplt.pod14.aviatrixlab.com" target="_blank">POD##</a>     |
|      15     |       <a href="https://cplt.pod15.aviatrixlab.com" target="_blank">POD##</a>     |
|      16     |       <a href="https://cplt.pod16.aviatrixlab.com" target="_blank">POD##</a>     |
|      17     |       <a href="https://cplt.pod17.aviatrixlab.com" target="_blank">POD##</a>     |
|      18     |       <a href="https://cplt.pod18.aviatrixlab.com" target="_blank">POD##</a>     |
|      19     |       <a href="https://cplt.pod19.aviatrixlab.com" target="_blank">POD##</a>     |
|      20     |       <a href="https://cplt.pod20.aviatrixlab.com" target="_blank">POD##</a>     |
|      21     |       <a href="https://cplt.pod21.aviatrixlab.com" target="_blank">POD##</a>     |
|      22     |       <a href="https://cplt.pod22.aviatrixlab.com" target="_blank">POD##</a>     |
|      23     |       <a href="https://cplt.pod23.aviatrixlab.com" target="_blank">POD##</a>     |
|      24     |       <a href="https://cplt.pod24.aviatrixlab.com" target="_blank">POD##</a>     |
|      25     |       <a href="https://cplt.pod25.aviatrixlab.com" target="_blank">POD##</a>     |
|      26     |       <a href="https://cplt.pod26.aviatrixlab.com" target="_blank">POD##</a>     |
|      27     |       <a href="https://cplt.pod27.aviatrixlab.com" target="_blank">POD##</a>     |
|      28     |       <a href="https://cplt.pod28.aviatrixlab.com" target="_blank">POD##</a>     |

## Access credentials

Username:

```bash
student
```

Password:

```bash
112!p94N4SXVPV
```

## LAB Pre-Req

Before starting building your backbone infrastructure, change the following **Fetch Timers** to their lowest value.

```{hint}
Go to **CoPilot > Settings > Resources > Task Server**
```

Ensure `Fetch GW Routes` and `Fetch VPC Routes` intervals are set to **“1 Second”** each and then click on **SAVE**.

```{figure} images/backbone-tasks.png
---
height: 400px
align: center
---
Initial Topology
```

Do not forget to click on the **COMMIT** button!

```{caution}
These are very aggressive settings. In a Production environment, you should not set these intervals that frequently!
```

## Task #1: Create an AWS TGW using the CoPilot

The Copilot provides an `AWS TGW (Transit Gateway) Network Orchestration` service, that allows to deploy the AWS TGW, avoiding to use the AWS Console.

```{figure} images/backbone-tgw01.png
---
height: 400px
align: center
---
AWS TGW - initial topology for task#1
```

Go to **CoPilot > Networking > Connectivity > AWS TGW** and click on the `"+ AWS TGW"` button.

```{figure} images/backbone-tgw02.png
---
height: 400px
align: center
---
AWS TGW section on the CoPilot
```

Ensure these parameters are entered in the pop-up window `"Create AWS TGW"`.

- **Name:** <span style='color:#479608'>AWS-NVirginia-TGW</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account </span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **AWS Side SAS Number:** <span style='color:#479608'>64512</span>

```{figure} images/backbone-tgw03.png
---
height: 400px
align: center
---
AWS TGW template
```

Do not forget to click on **SAVE**.

```{caution}
It will take roughly **2 minutes** for the Aviatrix Controller to completing the deployment of the AWS TGW.
```

```{figure} images/backbone-tgw04.png
---
align: center
---
Final Deployment outcome
```

## Task #2: Attach VPC to AWS TGW

Now that you have an AWS TGW, you need also to attach it to the existing **spoke-vpc**.

Click on the `AWS-NVirginia-TGW` element, select the **VPC** tab and then click on the `"Attach VPC"` button.

```{figure} images/backbone-tgw05.png
---
align: center
---
AWS NVirginia-TGW
```

Ensure these parameters are entered in the pop-up window `"Attach VPC to AWS TGW"`.

- **VPC:** <span style='color:#479608'>spoke-aws</span>
- **Network Domain:** <span style='color:#479608'>Default_Domain</span>

```{figure} images/backbone-tgw06.png
---
height: 400px
align: center
---
AWS NVirginia-TGW
```

Do not forget to click on **SAVE**.

```{caution}
It will take roughly **2 minutes** for the Aviatrix Controller to completing the attachment.
```

```{figure} images/backbone-tgw07.png
---
height: 350px
align: center
---
Attachment
```

## Task #3: Create an Aviatrix Transit VPC

Let's continue building up the cloud backbone, now you are asked to create a `Transit VPC`.

```{figure} images/backbone-tgw08.png
---
height: 400px
align: center
---
Initial Topology for Task#3
```

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets** and click on the `"+ VPC/VNet"` button.

```{figure} images/backbone-tgw09.png
---
align: center
---
Transit VPC
```

Ensure these parameters are entered in the pop-up window `"Create VPC/VNet"`.

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

```{note}
Wait few minutes for the completion of the task. Check the hourglass icon on the right-hand side of your Copilot.
```

## Task #4: Create both the Transit Gateways and the peering

Now it's time to deploy a pair of **`Transit GWs`** inside the VPC created on the previous task. In addition to this, you have also to establish the `peering` between the Transit GWs in Azure and the Transit GWs in AWS.

```{figure} images/backbone-tgw011.png
---
height: 400px
align: center
---
Inital Topology for Task #4
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the `"+ Transit Gateway"` button.

```{figure} images/backbone-tgw12.png
---
height: 400px
align: center
---
Transit Gateways section
```

Ensure these parameters are entered in the pop-up window `"Create Transit Gateway"`.

- **Name:** <span style='color:#479608'>transit-aws</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-1 (N. Virginia)</span>
- **VPC/VNet:** <span style='color:#479608'>transit-aws</span>
- **Instance Size:** <span style='color:#479608'>c6in.large</span>
- **High Performance Encryption:** <span style='color:#479608'>**ON**</span>
- **Peer To Transit Gateways:** <span style='color:#479608'>transit-azure</span>

then click on the `"+ Instance"` button!

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
The Aviatrix Controller will deploy two Transit Gateways and, at the same time, it will establish the peering with the predeployed Transit Gateways in Azure.
```

You can monitor the progress of the task going to **CoPilot > Monitor > Notifications > Tasks** and expanding the corresponding task, named `"Create transit gateway: transit-aws"`.

```{figure} images/backbone-tgw014.png
---
height: 400px
align: center
---
Task in progress
```

```{caution}
it will take roughly **10 minutes** for the Aviatrix Controller for completing this task, therefore, be patient!
```

Now go to **CoPilot > Cloud Fabric > Topology**, click on the `"Managed"` button for hiding all the unmanaged VPCs (i.e. VPCs without an Aviatrix GW) and then click on the `"Collapse all VPC/VNets"` button.

```{figure} images/backbone-tgw015.png
---
height: 400px
align: center
---
Dynamic Topology
```

You will notice the presence of the newly created **peering**.

## Task #5: Attach Transit Gateways to aws-tgw

Now let's attach the Transit GWs in AWS to the AWS TGW.

```{figure} images/backbone-tgw016.png
---
height: 400px
align: center
---
Initial Topology for task #5
```

Go to **CoPilot > Networking > Connectivity > AWS TGW** and click on the `"Attach Transit Gateway"` button.

```{figure} images/backbone-tgw017.png
---
align: center
---
"Attach Transit GW" button
```

Ensure this parameter is entered in the pop-up window `"Attach Transit Gateway to AWS-NVirginia-TGW"`.

- **Transit Gateway:** <span style='color:#479608'>transit-aws</span>

```{figure} images/backbone-tgw018.png
---
align: center
---
Attachment Template
```

Do not forget to click on **SAVE**.

```{caution}
it will take roughly **3 minutes** for the Aviatrix Controller for completing this task, therefore, be patient!
```

## Task #6: Configure BGP ASN

Now that you have established the peering between the two CSPs, you have achieved the **`Aviatrix Cloud Backbone`**.

Let's carry on with the final deployment of the connectivity to the on-prem DC.

Before configuring the attachments between the Secure Edge Gateway and the Transit Gateways, you have to ensure that the Transit Gateways's cluster earlier deployed in AWS is configured with a **`BGP AS number`**. <ins>This is a prerequisite for completing the Edge's deployment</ins>!

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **transit-aws** cluster!

```{figure} images/backbone-tgw019.png
---
height: 300px
align: center
---
transit-aws
```

Go to `"Settings"` tab and expand the `“Border Gateway Protocol (BGP)”` section and insert the AS number **64582** on the empty field related to the `"“Local AS Number”`, then click on **Save**.

```{figure} images/backbone-tgw020.png
---
align: center
---
BGP ASN
```

After this task, this is how the overall topology would look like.

```{figure} images/backbone-tgw021.png
---
height: 400px
align: center
---
Topology after task #6
```

## Task #7: Edge to Transits

Let's now attach the **Secure Edge Gateway** to the **Transit Gateways**.

```{figure} images/backbone-tgw022.png
---
height: 400px
align: center
---
The Hybrid Cloud through the Aviatrix Backbone
```

Go to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Transit Gateway Attachment"` button, on the right-hand side of the screen.

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

Fill in the attachment template using the following settings:

- **Transit Gateway**: <span style='color:#479608'>transit-aws</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>
- **High Performance Encryption**: <span style='color:#479608'>**ON**</span>
- **Number of Tunnels**: <span style='color:#479608'>**4**</span>

```{figure} images/backbone-tgw025.png
---
align: center
---
Template for aws
```

Before saving the template, click on the `"+ Attachment"` button and insert the following values in the second attachment template:

- **Transit Gateway**: <span style='color:#479608'>azure-aws</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>
- **High Performance Encryption**: <span style='color:#479608'>**ON**</span>
- **Number of Tunnels**: <span style='color:#479608'>**4**</span>

Now you can click on the **SAVE** button.

```{figure} images/backbone-tgw026.png
---
align: center
---
Template for azure
```

Now go to **CoPilot > Cloud Fabric > Hybrid Cloud** and click on the **aviarix-edge-1** instance.

```{figure} images/backbone-tgw027.png
---
align: center
---
aviatrix-edge-1
```

Now search for the subnet **`10.1.2.0`** in AWS. You will notice the presence of 4x IPSec tunnels towards the First Transit Gateway, likewise other 4x IPSec tunnels towards the Second Transit Gateway.

```{figure} images/backbone-tgw028.png
---
height: 400px
align: center
---
8x IPSec tunnels towards AWS
```

Now search for the subnet **`10.2.2.0`** in Azure. You will notice again 4x IPSec tunnels per each Transit Gateway!

```{figure} images/backbone-tgw029.png
---
height: 400px
align: center
---
8x IPSec tunnels towards Azure
```

You have successfully extended the Aviatrix solution to the on-prem Data Center, harnessing the **`High Performance Encryption`** functionality.

## Connectivity Test

Let's now verify the connectivity between Azuere, AWS and the on-prem DC.

Go to **CoPilot > Cloud Resources > Cloud Assets** and search for the **aws-instance** on the Search field, then retrieve its **Public** IP address!

```{figure} images/backbone-tgw030.png
---
height: 250px
align: center
---
aws-instance on the Cloud Assets
```

Now open an **SSH client** and establish a session towards the <ins>aws-instance</ins>.

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

Issue the following command to verify the connectivity towards the **_azure-instance_**:

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

Stop the command and try to ping also the **test-vm** inside the on-prem DC.

```{figure} images/backbone-tgw034.png
---
height: 400px
align: center
---
ping towards the test-vm inside the DC
```

Issue the following command to verify the connectivity towards the **_test-vm_**:

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

You have successfully verified the connectivity between the two CSPs and the on-prem DC by means of the Aviatrix Cloud Backbone.

Wait for 4-5 minutes and then go to **CoPilot > Monitor > FlowIQ** and on the `"Filters"` field click on thew **+** button and create the following condition:

```bash
Destination IP Address is 10.2.2.40
```

```{figure} images/backbone-tgw036.png
---
height: 400px
align: center
---
FlowIQ
```

```{figure} images/backbone-tgw037.png
---
height: 400px
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

## Conclusion

In this lab, you successfully designed and implemented a multi-cloud backbone architecture for ABC Healthcare, extending their cloud infrastructure across both Azure and AWS. Despite initial challenges, such as the lack of native connectivity between Azure Route Server and other cloud providers, you leveraged Aviatrix's multicloud transit capabilities to create an efficient and robust solution. Through sequential tasks, you deployed key infrastructure components, including AWS Transit Gateway, Aviatrix Transit VPCs, and Aviatrix Transit Gateways. As a result, ABC Healthcare now possesses a `scalable and high-performance multi-cloud network backbone`, poised to support its growing cloud initiatives with enhanced inter-cloud and hybrid connectivity.
