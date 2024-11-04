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

1- You have already deployed both the `Aviatrix Controller` and the `Aviatrix CoPilot`, inside AWS.

2- You have also deployed a pair of `Transit Gateways` in Azure and attached them to the existing Azure Route Server.

3- You have also deployed an `Aviatrix Secure Edge` inside the on-prem Data Center.

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
|      1     |       <a href="https://cplt.pod1.aviatrixlab.com" target="_blank">POD1</a>     |
|      2     |       <a href="https://cplt.pod2.aviatrixlab.com" target="_blank">POD2</a>      |
|      3     |       <a href="https://cplt.pod3.aviatrixlab.com" target="_blank">POD3</a>      |
|      4     |       <a href="https://cplt.pod4.aviatrixlab.com" target="_blank">POD4</a>      |
|      5     |       <a href="https://cplt.pod5.aviatrixlab.com" target="_blank">POD5</a>     |
|      6     |       <a href="https://cplt.pod6.aviatrixlab.com" target="_blank">POD6</a>      |
|      7     |       <a href="https://cplt.pod7.aviatrixlab.com" target="_blank">POD7</a>      |
|      8     |       <a href="https://cplt.pod8.aviatrixlab.com" target="_blank">POD8</a>      |
|      9     |       <a href="https://cplt.pod9.aviatrixlab.com" target="_blank">POD9</a>     |
|      10     |       <a href="https://cplt.pod10.aviatrixlab.com" target="_blank">POD10</a>      |

## Access credentials

Username:

```bash
student
```

Password:

```bash
1012fw633#SYTY3
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

## Task #1: Create an AWS TGW using the CoPilot

The Copilot provides an `AWS TGW (Transit Gateway) Network Orchestration` service, that allows to deploy the AWS TGW avoiding to use the AWS Console.

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
height: 250px
align: center
---
Attachment
```

## Task #3: Create an Aviatrix Transit VPC

Let's continue building the cloud backbone, now you are asked to create the `Transit VPC`.

```{figure} images/backbone-tgw08.png
---
align: center
---
Initial Topology for Task#3

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

Now it's time to deploy a pair of **`Transit GWs`** inside the VPC created on the previous task. In addition to this, you have also to establish the peering between the Transit GWs in Azure and the Transit GWs in AWS.

```{figure} images/backbone-tgw011.png
---
align: center
---
Inital Topology for Task #4
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the `"+ Transit Gateway"` button.

```{figure} images/backbone-tgw12.png
---
align: center
---
Transit Gateways section
```

### Deploy Aviatrix Spoke GW

- The public IP address will be different (Public EIP automatically allocated by CSP)
- The Subnet CIDR could be different (automatically picked up by Aviatrix Controller)
- Region: us-east-1

![Spoke](images/egress_spoke_gw.png)

Check the Egress setting. The Egress traffic is going through the AWS NAT GW.

![Egress](images/egress_egress.png)

### Enable spoke GW to become the Egress GW

1. Click +Local Egress on VPC/VNets.
2. In the Add Local Egress on VPC/VNets dialog, select the VPC/VNets on which to enable Local Egress.
3. Click Add.

[Read more at Aviatrix Documentation](https://docs.aviatrix.com/copilot/latest/network-security/index.html)

![Local](images/egress_add_local.png)

Add Local Egress on VPC/VNets
Adding Egress Control on VPC/VNet changes the default route on VPC/VNet to point to the Spoke Gateway and enables SNAT. Egress Control also requires additional resources on the Spoke Gateway.VPC/VNets

Now the diagram should look like the following:

![Vpc](images/egress_vpc.png)

## Conclusion

By bringing Aviatrix Secure Egress into play, our healthcare provider shored up their defenses, dropped the high costs, and eliminated the visibility black hole courtesy of the AWS NAT Gateway. Sensitive patient data is safe, and the provider’s reputation will be secured.
Remember, Aviatrix Secure Egress is your go-to for a secure, cost-effective solution for managing internet-bound traffic. Need more help? Our support team’s got your back.
