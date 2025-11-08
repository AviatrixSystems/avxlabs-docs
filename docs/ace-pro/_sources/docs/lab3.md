# Lab 3 - NETWORK SEGMENTATION

## 1. Objective

Implement network segmentation between workloads across multicloud and on-premises environments using network domains.

```{note}
Network Segmentation will be extended to on-premises in the Site2Cloud lab.
```

## 2. Network Segmentation Overview

Enterprises can define their `Network Domains` (aka *segments*) and group VNets/VPCs/VCNs with similar security policies without requiring firewall solutions.

Aviatrix *`Cloud Native Security Fabric (CNSF)`* helps enterprises create customized segments and onboard branches, partners, and customers within their respective segments, ensuring that no partner can communicate with each other unless explicitly permitted.

## 3. Topology

In this lab, we will use Aviatrix CoPilot to enable `Network Segmentation` in Azure, AWS and GCP, in order to segregate VPC/VNet with similarities. At this point, there is a flat routing domain and the communication among the three CSPs occurs through the Transit Core Backbone layer.

You will create two segments: **Green** and **Blue**.

Green VPC resources in AWS and Azure can communicate with each other, while access is restricted to Blue VPC resources in GCP. Later, we will ease this restriction with a `Connection Policy`: Blue and Green segments will be able to communicate with others as well.

```{figure} images/lab3-topology.png
---
height: 400px
align: center
---
Topology for Lab 3
```

## 4. Configuration

### 4.1 Aviatrix Transit Gateways

Navigate to **CoPilot > Networking > Network Segmentation > Network Domains > Transit Gateways**:

```{figure} images/lab3-enabletransit.png
---
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

### 4.2 Network Domains

Navigate to **CoPilot > Networking > Network Segmentation > Network Domains > + Network Domain**

```{figure} images/lab3-networkdomain.png
---
align: center
---
Network Domain Creation
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

### 5.1 CoPilot Verification

Navigate to **CoPilot > Networking > Network Segmentation > Network Domains**

Verify the segments and the associations as shown below:

```{figure} images/lab3-verification.png
---
align: center
---
Associations verification
```

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the Transit Gateway **_aws-us-east-2-transit_**:

```{figure} images/lab3-exploretransit.png
---
align: center
---
Select Transit in US-East-2
```

Then navigate to the `"Route DB"` tab and examine how routes are allocated to the dedicated routing table, which defines a distinct new Routing Domain (e.g., **Blue**, **Green**).
Then select the `"Route DB"` tab and inspect the routing table of the network domain **Green**, likewise the routing table of the network domain **Blue**:

```{figure} images/lab3-routedb1012.png
---
align: center
---
Route DB
```



Navigate to **CoPilot > Networking > Network Segmentation > Overview > Logical View**

The nodes depicted in the Logical View represent spokes and site2cloud instances. Hover over a node to highlight reachability. Nodes are grouped by colored arcs representing network domains. At this time, only the spoke gateways in Azure and AWS (i.e. Green Network Domain) are connected:

```{figure} images/lab3-logicalview.png
---
height: 400px
align: center
---
Logical View
```

## 6. Connectivity Testing after Applying the Network Segmentation

Verify that you have applied the separation between the two segments.

### 6.1 Connectivity Testing Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select **_aws-us-east-2-spoke1-test1_** as an example. 

```{important}
You should already be signed in to the **Gatus** app.
```

Eventually, both the **ICMP** check and the **SSH** check will begin to fail gradually, towards the instance **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab3-gatus00.png
---
height: 400px
align: center
---
ICMP Test towards gcp-us-central1-spoke1-test1 fails 
```

```{figure} images/lab3-gatus456.png
---
height: 400px
align: center
---
SSH Test towards gcp-us-central1-spoke1-test1 fails 
```

```{important}
Please note that the connectivity tests are currently set to 5 minutes. If you'd like faster results, you can reduce the interval to as little as **10 seconds**!
```{figure} images/lab3-gatus01.png
---
height: 400px
align: center
---
ICMP timeout
```

### 6.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

## 7. Connection Policy

Navigate to **CoPilot > Networking > Network Segmentation > Network Domains**

Click the pencil icon to edit. You can either select the Green domain or the Blue domain.

```{important}
The connection policy is always bidirectional!
```

```{figure} images/lab3-editnd.png
---
height: 250px
align: center
---
Edit Blue
```

Select the appropriate option from the **`"Connect to Network Domain"`** pull-down menu (Green shown here). Then click **Save**:

```{figure} images/lab3-applycp.png
---
align: center
---
Apply the Connection Policy
```

### 7.1 Verification of Connection Policy

Navigate to **CoPilot > Networking > Network Segmentation > Overview > Logical View**

Now you will see that the spoke gateways in all three clouds are connected. This is because the Blue and Green Network Domains are directly connected:

```{figure} images/lab3-cpnew.png
---
align: center
---
Logical View with the connection policy
```

## 8. Final Connectivity Testing after Applying the Connection Policy

Verify that you have successfully **merged** both Network Domains. Retest the connectivity; you should now have end-to-end connectivity across the multicloud environment.

```{caution}
AWS **US-EAST-1** and Azure **WEST-US Spoke 2** are not yet attached!
```{figure} images/lab3-unattached.png
---
height: 400px
align: center
---
Unattached
```

### 8.1 Connectivity Testing Using the Gatus App

Once again, navigate to your POD Portal, locate the `Gatus widget`, and select **_aws-us-east-2-spoke1-test1_** as an example.

The ICMP test and the SSH test for the instance **_gcp-us-central1-spoke1-test1_** should work this time.

```{figure} images/lab3-gatus100.png
---
height: 400px
align: center
---
ICMP test
```

```{figure} images/lab3-gatus188.png
---
height: 400px
align: center
---
SSH test
```

### 8.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you're not satisfied with the Gatus dashboard, you can also use your personal `SSH client` to perform the connectivity tests!

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

After this lab, the overall topology will appear as follows:

```{figure} images/lab3-finaltopology.png
---
height: 400px
align: center
---
Final topology for Lab 3
```