# Lab 3 - NETWORK SEGMENTATION

## 1. Objective

Implement network segmentation between workloads across multicloud and on-premises environments using network domains.

```{note}
Network Segmentation will be extended to on-premises in the Site2Cloud lab.
```

## 2. Network Segmentation Overview

Enterprises can define their network domains (aka *segments*) and group VNets/VPCs/VCNs with similar security policies without requiring firewall solutions.

Aviatrix transit and spokes architecture help (aka *Hub & Spoke* architecture) enterprises create customised segments and onboard branches, partners and customers in their respective segments so no partner can communicate with each other unless desired.

## 3. Topology

In this lab, we will use Aviatrix CoPilot to enable `Network Segmentation` in Azure, AWS and GCP, in order to segregate VPC/VNet with similarities. At this point, there is a flat routing domain and the communication among the three CSPs occurs through the Transit Core Backbone layer.

Green VPC resources in AWS and Azure can communicate with each other, while access is restricted to Blue VPC resources in GCP. Later, we will ease this restriction with a `Connection Policy`: Blue and Green segments will be able to communicate with others as well.

```{figure} images/lab3-topology.png
---
height: 400px
align: center
---
Topology for Lab 3
```

## 4. Configuration

### 4.1. Aviatrix Transit Gateways

Go to **CoPilot > Networking > Network Segmentation > Network Domains > Transit Gateways**:

```{figure} images/lab3-enabletransit.png
---
height: 400px
align: center
---
Enable the feature
```

Enable all three Aviatrix Transit Gateways in Azure, GCP and AWS (<ins>**us-east-2** only for now</ins>) for network segmentation as shown below:

```{figure} images/lab3-enabletransit2.png
---
align: center
---
Enable Segmentation on the relevant Transit GWs
```

## 4.2 Network Domains

Go to **CoPilot > Networking > Network Segmentation > Network Domains > + Network Domain**

```{figure} images/lab3-networkdomain.png
---
height: 400px
align: center
---
Network Domain creation
```

Create **two** network domains (Green and Blue) and associate them to their respective Spokes as follows:

-  <span style='color:lightgreen'>Green</span>- azure-west-us-spoke1 (do not select azure-west-us-**spoke2**)
- <span style='color:lightgreen'>Green</span> - aws-us-east-2-spoke1 (do not select aws-us-**east-1**-spoke1)
- <span style='color:lightblue'>Blue</span> - gcp-us-central1-spoke1

Click on **Save** after creating each Network Domain.

```{figure} images/lab3-green.png
---
align: center
---
Green network domain
```

```{figure} images/lab3-blue.png
---
align: center
---
Blue network domain
```

This is what the lab topology looks like after enabling network segmentation:

```{figure} images/lab3-topologywithnd.png
---
height: 400px
align: center
---
Topology with Network Domains
```

## 5. Verification of Segment Attachments

### 5.1. CoPilot Verification

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Verify the segments and the associations as shown below:

```{figure} images/lab3-verification.png
---
align: center
---
Associations verification
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the Transit Gateway **_aws-us-east-2-transit_**:

```{figure} images/lab3-exploretransit.png
---
align: center
---
Select Transit in US-East-2
```

Then select the `"Gateway Routes"` tab and inspect the routing table of the network domain **Green**, likewise the routing table of the network domain **Blue**:

```{figure} images/lab3-exploregreen.png
---
align: center
---
Explore Green
```

```{figure} images/lab3-exploreblue.png
---
align: center
---
Explore Blue
```

Go to **CoPilot > Networking > Network Segmentation > Overview > Logical View**

The nodes depicted in the Logical View represent spokes and site2cloud instances. Hover over a node to highlight reachability. Nodes are grouped by colored arcs representing network domains. At this time, only the spoke gateways in Azure and AWS (i.e. Green Network Domain) are connected:

```{figure} images/lab3-logicalview.png
---
align: center
---
Logical View
```

Open **three** terminal windows and SSH to the test instances/VMs in each cloud and ping the **private** IPs of each other to test the Multicloud connectivity (Refer to pod info).

Azure and AWS resources will ping each other, but neither will be able to access GCP VM, since GCP spoke is in a different segment (Blue).

**AWS**:

SSH into **_aws-us-east-2-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-ping1.png
---
align: center
---
Ping test from AWS
```

**Azure**:

SSH into **_azure-west-us-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-ping2.png
---
align: center
---
Ping test from Azure
```

**GCP**:

SSH into **_gcp-us-central1-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-ping3.png
---
align: center
---
Ping test from GCP
```

## 6. Connection Policy

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Click the pencil icon to edit. You can either select the Green domain or the Blue domain.

```{important}
The connection policy is always bidirectional!
```

```{figure} images/lab3-editnd.png
---
align: center
---
Edit Blue
```

Select the appropriate option from the **`"Connect to Network Domain"`** pull-down menu (Green shown here). Then click **Save**:

```{figure} images/lab3-applycp.png
---
height: 400px
align: center
---
Apply the connection polic
```

### 6.1. Verification of Connection Policy

Go to **CoPilot > Networking > Network Segmentation > Overview > Logical View**

Now you will see that the spoke gateways in all three clouds are connected. This is because the Blue and Green Network Domains are directly connected:

```{figure} images/lab3-cpnew.png
---
align: center
---
Logical View with the connection policy
```

Retest the connectivity; now you will have end-to-end connectivity across the multicloud environment.

**AWS**:

SSH into **_aws-us-east-2-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-newtest.png
---
align: center
---
New Test from AWS
```

**Azure**:

SSH into **_azure-us-west-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-newtest2.png
---
align: center
---
New Test from Azure
```

**GCP**: 

SSH into **_gcp-us-central1-spoke1-test1_** (ssh student@public_ip)

```{figure} images/lab3-newtest3.png
---
align: center
---
New Test from GCP
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab3-finaltopology.png
---
height: 400px
align: center
---
Final topology for Lab 3
```