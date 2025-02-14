# Lab 2 - TRANSIT NETWORKING

## 1. Objective

Build Transit Network in Azure, GCP and AWS using Aviatrix Multicloud Transit `Hub-and-Spoke Topology`.

In this lab, we will use Aviatrix CoPilot to connect three major clouds, i.e. Azure, GCP and AWS. The workloads in VPCs/VNets, in all three clouds, must communicate without manual configuration on the native consoles.

## 2. Multicloud Connectivity Overview

Enterprises are relying increasingly on multiple clouds (multicloud) providers. However, setting up the connectivity between those providers is difficult. Moreover, maintaining and monitoring the tunnels is time-consuming and cumbersome to troubleshoot.

Aviatrix simplifies this by providing simple, point-and-click tunnel creation between cloud providers.

Additionally, Aviatrix gives you a single, centralised location from which to troubleshoot and monitor your connections.

## 3. Topology

In this lab, as shown in the topology below, we will configure the grey Aviatrix gateways, the grey attachments between Transit-Spoke and the grey peerings between Transit-Transit, but only in the following regions:

- **AWS**: us-east-2
- **GCP**: us-central-1
- **AZURE**: west-us (only spoke1)

The rest of the topology has been preprovisioned to save time, including the test instances/VMs and the **Edge** inside the on-prem DC.

```{figure} images/lab2-topology.png
---
height: 400px
align: center
---
Initial pre-provisioned topology
```

### 3.1. Preliminary Monitoring Check Using the Gatus APP

All of the pre-deployed instances are running [Gatus](https://gatus.io/) and attempting to connect to each other on various ports (e.g. icmp and tcp:80/443). 

You can access the Gatus' dashboard directly from your POD Portal.

```{figure} images/lab2-gatus00.png
---
height: 400px
align: center
---
Gatus from the POD Portal
```

You can select any instances from the three CSP environments, and you will find that the connectivity is indeed broken. This is because all the involved VPCs are currently isolated, as you have not yet created the MCNA.

- Select the EC2 instance "**_aws-us-east-2-spoke1-test1_**" and then enter the credentials to access the Gatus App!

```{figure} images/lab2-gatus01.png
---
height: 350px
align: center
---
aws-us-east-2-spoke1-test1 
```

```{figure} images/lab2-gatus02.png
---
height: 350px
align: center
---
aws-us-east-2-spoke1-test1 on the logical Topology
```

Insert the credentials available on your POD Portal and then click on **"Sign in"**.

```{figure} images/lab2-gatus03.png
---
height: 400px
align: center
---
Credentials
```

After signing into the Gatus App, you'll observe that two protocols are currently in use: `ICMP` and `SSH`.

The only item showing a green status is `aws-us-east-2-spoke1-test2`. 

Both ICMP and SSH are functioning perfectly here. This success is attributed to the fact that this communication occurs within the same VPC (**`intra-VPC traffic`**), meaning it does not traverse other VPCs, regions, or cloud service providers!

```{figure} images/lab2-gatus04.png
---
height: 400px
align: center
---
ICMP
```

```{figure} images/lab2-gatus05.png
---
height: 400px
align: center
---
SSH
```

```{figure} images/lab2-intravpc.png
---
height: 400px
align: center
---
Normal intra-vpc traffic 
```

```{important}
Check the **Gatus App** status for the remaining instances as well.
```

### 3.2. Preliminary Monitoring Check Using an SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Alternatively, if you want an additional verification, you can also access all the istances using an SSH client.

- Go to your POD Portal and search for the widget assigned to the **_aws-us-east-2-spoke1-test1_** instance within the LAB2 and LAB3 sections. Then copy on the clipboard the <ins>symbolic public DNS name</ins> of the istance!

```{figure} images/lab2-gatus06.png
---
height: 400px
align: center
---
DNS name
```

Now, open your SSH client and enter the command to access the EC2 instance using your credentials!

```bash
ssh student@...
```

```{figure} images/lab2-gatus07.png
---
height: 400px
align: center
---
SSH client
```

You can also access your instances using also their **Public IP addresses**! You can retrieve them through your CoPilot, specifically from two sections:

1) `Cloud Assets`
2) `Topology`

#### 3.2.1 Cloud Assets (to retrieve IP addresses)

- Go to **CoPilot > Cloud Resources > Cloud Assets** and filer based on the keyword `"test"`.
- Identify the instance **_aws-us-east-2-spoke1-test1_** and copy its public ip address and use it with your SSH client.

```{figure} images/lab2-sshclient10.png
---
height: 400px
align: center
---
Search the test instances
```

```{figure} images/lab2-sshclient11.png
---
height: 400px
align: center
---
Public IP address
```

```{important}
Within the Cloud Assets, you can also retrieve the **Private IP addresses** for EAST-WEST traffic.
```

Now access the EC2 instance using its public IP address:

```{figure} images/lab2-sshclient20.png
---
height: 400px
align: center
---
SSH with public IP address
```

#### 3.2.2 Topology (to retrieve IP addresses)

- Navigate to **CoPilot > Topology > Overview (Default Tab)** and enter **_aws-us-east-2-spoke1-test1_** in the search field located on the right-hand side.

```{figure} images/lab02-searchfield.png
---
height: 400px
align: center
---
Topology
```

Now, navigate to the Properties window, locate the Public IP address of the instance, and copy it to your clipboard!

```{figure} images/lab02-searchfield00.png
---
height: 400px
align: center
---
Topology
```

```{important}
In the Properties window under the Topology section of your CoPilot, you can also access the **Private IP addresses** for EAST-WEST traffic.
```

### 3.3 Verification Using Your SSH Client 

Now that you have logged in to the **_aws-us-east-2-spoke1-test1_** successfully, you can issue your ping commands towards the **Private IP address** of the other instances!

```{figure} images/lab2-sshclient00.png
---
height: 400px
align: center
---
SSH client
```

```{figure} images/lab2-pingfails.png
---
height: 400px
align: center
---
Ping outcomes
```

```{tip}
Verify also the **SSH** protocol
```

## 4. Aviatrix CoPilot

Now let's access the Aviatrix Centralized Management Node, the CoPilot.

Go to your POD portal, click on the `"Open Copilot"` button and insert the credentials!

```{figure} images/lab2-gatus06.png
---
height: 400px
align: center
---
POD Portal
```

```{figure} images/lab2-gatus07.png
---
height: 400px
align: center
---
CoPilot UI
```

```{note}
The test VPCs/VNet you created in **Lab 1** will not be used in other labs.

For sake of semplicity, in this lab, the Transits will NOT be deployed in pairs. As a best practice, Aviatrix recommends always to deploy two Transits to ensure High Availability.
```

The CoPilot *Dashboard* should look something like this:

```{figure} images/lab2-dashboard.png
---
height: 400px
align: center
---
Dashboard
```

- Before starting building your multicloud infrastructure, adjust the **fetch timers** on the CoPilot.

```{hint}
Go to **CoPilot > Settings > Resources > Task Server**
```

Ensure that `Fetch Topology`, `Fetch Instances`, `Fetch GW Routes` and `Fetch VPC Routes` intervals are set to **“1 Second”** each and then click on **SAVE**.

```{figure} images/lab2-timer.png
---
height: 400px
align: center
---
Task Server
```

```{caution}
The order of the **_Task Servers_** on the screenshot above might be different on your CoPilot
```

```{figure} images/lab2-fetchtopology.png
---
align: center
---
Fetch Topology
```

```{figure} images/lab2-fetchinstances.png
---
align: center
---
Fetch Instances
```

```{figure} images/lab2-fetchgw.png
---
align: center
---
Fetch GW
```

```{figure} images/lab2-fetchvpc.png
---
align: center
---
Fetch VPC
```

Afterwards, click on **Commit**.

```{figure} images/lab2-commit.png
---
height: 400px
align: center
---
Commit
```

```{note}
These are very aggressive settings. In a Production environment, you should not set these intervals that frequently!
```

## 4. Initial configuration

Go to **CoPilot > Dashboard** and check the `Gateways Health` either of the Spoke GW Clusters or the Transit GW Clusters. 

```{figure} images/lab2-dashboardgw.png
---
align: center
---
Transit, Spoke & Edge
```

When you begin this lab, you should have **six** Gateway Clusters (i.e. 3x Transit GWs clusters, 2x Spoke GWs clusters and 1x Edge cluster) in your pod.

- **3x** <span style='color:green'>Transit GW Clusters</span>
- **2x** <span style='color:orange'>Spoke GW Clusters</span>
- **1x** <span style='color:violet'>Edge GW Cluster</span>

If you go to **CoPilot > Cloud Fabric > Gateways > Overview (default tab)**, you will notice that the number of Transit Gateways is set to **three** indeed, whereas the number of Spoke Gateways is set to **two**.

```{figure} images/lab2-cluster.png
---
align: center
---
These are Clusters of GWs
```

This view within the Cloud Fabric section does not indicate the exact number of gateways but it refers to the number of **_clusters_**, per each type of gateway.

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and you will find out that there are a total of **Three** Transit Gateways (Public IPs may differ):

```{figure} images/lab2-clustertransit.png
---
height: 300px
align: center
---
Transit GWs Clusters
```

```{note}
You can deploy up to maximum **two** Transit Gateways per each Transit VPC/VNet/VCN.
```

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and explore the list of the available Spoke gateways. You can find out that there are a total of **three** Spoke Gateways (once again, the Public IPs may differ):

```{figure} images/lab2-clusterspoke.png
---
height: 250px
align: center
---
Spoke GWs Clusters
```

You can notice that the cluster in AWS comprises two Spoke Gateways, whereas in Azure there is just a single Spoke Gateway. This is a valid deployment. The number of Spoke Gateways that you should deploy per each Spoke VPC/VNet/VCN is dictated by your architecture design.

```{note}
You can deploy up to maximum **fifteen** Spoke Gateways per each Spoke VPC/VNet/VCN.
```

### 4.1. Aviatrix Transit Gateways

In this section, you will experience the power and simplicity of the Aviatrix platform by deploying (i.e. creating) 4 gateways:

 - **Transit gateway in AWS US East 2:** <span style='color:#479608'>aws-us-east-2-transit</span>
 
 - **Spoke gateway in AWS US East 2:** <span style='color:#479608'>aws-us-east-2-spoke1</span>
- **Spoke gateway in Azure West US:** <span style='color:#479608'>azure-west-us-spoke1</span>
- **Spoke gateway in GCP US Central 1:** <span style='color:#479608'>gcp-us-central1-spoke1</span>

```{warning}
Please pay close attention to each step, as a misconfiguration could result in **20+** minutes of lost time! 
```

- Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways > + Transit Gateway**

```{figure} images/lab2-transitbutton.png
---
height: 350px
align: center
---
+Transit Gateway
```

Deploy Aviatrix Transit Gateways in AWS **_East-2_** region. To save time, Aviatrix Transit Gateways in Azure, GCP and AWS east-1 region have already been pre-deployed in pairs for this lab.

### 4.1.1.Transit Gateway in AWS US-EAST-2

Ensure these parameters are entered in the pop-up window `"Create Transit Gateway"`.

- **Name:** <span style='color:#479608'>aws-us-east-2-transit</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account </span>
- **Region:** <span style='color:#479608'>us-east-2 (Ohio)</span>
- **VPC ID:** <span style='color:#479608'>aws-us-east-2-transit (Make sure you don't select aws-us-east-2-**spoke1** VPC)</span>
- **Instance Size:** <span style='color:#479608'>c6in.large</span>
- **High Performance Encryption:** <span style='color:#479608'>**On**</span>

- **Attach to Subnet:** <span style='color:#479608'>us-east-2a</span>

- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

```{note}
As soon as you select High Performance Encryption, **/26** subnets will appear in the drop-down window.
```

Click **SAVE**.

```{figure} images/lab2-transitcreation.png
---
align: center
---
Create Transit Gateway
```

```{note}
Only one Transit Gateway will be deployed in VPC **aws-us-east2-transit**.
```

You will immediately get a message as follows.

```{figure} images/lab2-gwmessage.png
---
align: center
---
Gateway deployment in progress
```

You may check the status of the gateway creation in the top right corner by expanding the **task icon**.

```{figure} images/lab2-taskicon.png
---
align: center
---
Task icon
```

This action will instantiate the Transit Gateway with the following name:

- **aws-us-east-2-transit**

Meanwhile the deployment is happening, you may proceed to the next section of this lab guide to deploy your Spoke gateways.

### 4.2. Aviatrix Spoke Gateways

Navigate to the tab immediately to the right, which is `Spoke Gateways`.

This is **CoPilot > Cloud Fabric > Gateways > Spoke Gateways > + Spoke Gateway**.

```{figure} images/lab2-spokecreate.png
---
height: 350px
align: center
---
+Spoke Gateway
```

### 4.2.1. Spoke Gateway in AWS

Ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{note}
Only one Spoke Gateway will be deployed in VPC **aws-us-east2-spoke1**.
```

- **Name:** <span style='color:#479608'>aws-us-east-2-spoke1</span>
- **Cloud:** <span style='color:#479608'>AWS (Standard)</span>
- **Account:** <span style='color:#479608'>aws-account</span>
- **Region:** <span style='color:#479608'>us-east-2 (Ohio)</span>
- **VPC ID:** <span style='color:#479608'>aws-us-east-2-spoke1 (Make sure you don't select aws-us-east-2-**transit** VPC)</span>
- **Instance Size:** <span style='color:#479608'>t2.medium</span>
- **High Performance Encryption:** <span style='color:#479608'>**Off**</span>
- **Attach to Subnet:** <span style='color:#479608'>10.0.1.96/27 - aws-us-east-2-spoke1-Public-1-us-east-2a</span>
- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

Click **SAVE**.

```{figure} images/lab2-spokeinaws.png
---
align: center
---
Create Spoke Gateway in AWS
```

While the gateway is being created, you may proceed to the next section.

### 4.2.2. Spoke Gateway in Azure

Repeat the previous steps for Azure, click on the button `"+ Spoke Gateway"` and ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{note}
Only one Spoke Gateway will be deployed in VNet **azure-west-us-spoke1**.
```

- **Name:** <span style='color:#479608'>azure-west-us-spoke1</span>
- **Cloud:** <span style='color:#479608'>Azure (Global)</span>
- **Account:** <span style='color:#479608'>azure-account</span>
- **Region:** <span style='color:#479608'>West US</span>
- **VNet:** <span style='color:#479608'>azure-west-us-spoke1 (Make sure you don't select azure-west-us-**spoke2** VPC)</span>
- **Instance Size:** <span style='color:#479608'>Standard_B2ms</span>
- **High Performance Encryption:** <span style='color:#479608'>**Off**</span>
- **Attach to Subnet:** <span style='color:#479608'>192.168.1.0/28 - azure-west-us-spoke1-Public-gateway-subnet-1</span>
- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

```{warning}
Make sure you <ins>do not select the subnets that begins with az-1, az-2, or az-3</ins>. It is Aviatrix's recommended practice to deploy gateways in subnets with 'gateway' in their name, whereas workloads in subnets that do not have 'gateway' in their name).
```

```{figure} images/lab2-rightsubnet.png
---
align: center
---
Subnet selection
```

Do not forget to click on **SAVE**.

```{figure} images/lab2-spokeinazure.png
---
align: center
---
Spoke GW in Azure
```

While the gateway is being created, you may proceed to the next section.

### 4.2.3. Spoke Gateway in GCP

Repeat the previous steps for GCP. Ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{warning}
Only one Spoke Gateway will be deployed in VPC **gcp-us-central1-spoke1**.
```

- **Name:** <span style='color:#479608'>gcp-us-central1-spoke1</span>
- **Cloud:** <span style='color:#479608'>GCP</span>
- **Account:** <span style='color:#479608'>gcp-account</span>
- **VPC:** <span style='color:#479608'>gcp-us-**central1**-spoke1 (Make sure you don't select gcp-us-**west2**-spoke1 VPC)</span>
- **Instance Size:** <span style='color:#479608'>n1-standard-1</span>
- **High Performance Encryption:** <span style='color:#479608'>**Off**</span>
- **Attach to Subnet:** <span style='color:#479608'>gcp-us-central1-spoke1-sub1</span>
- **Zone:** <span style='color:#479608'>172.16.1.0/24 - us-central1-a</span>
- **Public IP:** <span style='color:#479608'>Allocate New Static Public IP</span>

Click **SAVE**.

```{figure} images/lab2-spokeingcp.png
---
align: center
---
Spoke GW in GCP
```

```{caution}
You can see the progress of gateway deployment at any time by expanding the task icon in the top right corner of the CoPilot.

<ins>Bear in mind that the gateways' creation process can take several minutes to complete, therefore please be patient!</ins>
```

```{figure} images/lab2-inprogress2.png
---
align: center
---
Deployment in progress
```

Once all gateways have been created, confirm from **CoPilot > Cloud Fabric > Gateways > Overview (default TAB)** the presence of a total of **nine** GWs Clusters!

```{figure} images/lab2-14gws.png
---
align: center
---
Dashboard
```

After created the Transit gateway in AWS  US-EAST-1 region and the single Spoke gateways in each cloud, this is how the topology would look like.

```{figure} images/lab2-temptopology.png
---
height: 400px
align: center
---
Overview of the new topology
```

## 4.3. Explore the Cloud Fabric

Go to **CoPilot > Cloud Fabric > Topology > Overview (default tab)**, then click on  the `"Managed"` button to only showing the Managed VPCs!

```{figure} images/lab2-newuitopo.png
---
height: 400px
align: center
---
Managed VPCs and Unmanaged VPCs
```

```{note}
**Managed VPC** = Indicates an Aviatrix gateway is running in the VPC/VNet.

**Unmanaged VPC** = Indicates no Aviatrix gateways exist in the VPC/VNet.
```

Now you can click on the `"Collapse all VPC/VNets"` button on the bottom right-hand side, as depicted below.

```{figure} images/lab2-collapse.png
---
height: 400px
align: center
---
Collapse button
```

```{figure} images/lab2-topologyoverview.png
---
align: center
---
VPC circles
```

```{important}
The inner circle represents the Transit Gateway VPCs, whereas the outer one represents the Spoke Gateway VPCs. 

<ins>It is clearly shown at this stage that the spokes are not connected to the transits yet</ins>. 
```

In addition, you can explore the components of any of the gateways in terms of subnets and Virtual Machines that reside within the VPC/VNet.
 
## 4.4 Aviatrix Spoke to Transit Gateways Attachments

In this section you are going to attach the Aviatrix Spoke Gateways created above in each cloud, to their corresponding Aviatrix Transit Gateways.
 
### 4.4.1. Spoke to Transit Attachment in AWS

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **_aws-us-east-2-spoke1_**, clicking on the pencil icon:

```{figure} images/lab2-spokeinawstotransit.png
---
height: 300px
align: center
---
Attachment for AWS
```

Select the Transit Gateway **_aws-us-east-<span style='color:violet'>2</span>-transit_** (do not select the **_aws-us-east-1-transit_**) from the drop-down window `"Attach To Transit Gateway"`, and then click on **Save**.

```{figure} images/lab2-editspokeinaws.png
---
align: center
---
Edit Spoke in AWS
```

You will see immediately a message informing that the updating is in progress.

```{figure} images/lab2-immediatemessage.png
---
align: center
---
Update in progress
```

### 4.4.2 Spoke to Transit Attachment in Azure

- **azure-west-us-spoke1** to **azure-west-us-transit**

Edit the Spoke Gateway **_azure-west-us-spoke1_** (not the **spoke2**), clicking on the pencil icon.

```{figure} images/lab2-editspokeinazure.png
---
height: 300px
align: center
---
Edit spoke in Azure
```

Select the Transit Gateway **_azure-west-us-transit_** from the drop-down window `"Attach To Transit Gateway"`, and then click on **Save**.

```{figure} images/lab2-editazure.png
---
align: center
---
Attachment in Azure
```

### 4.4.3. Spoke to Transit Attachment in GCP

- **gcp-us-central1-spoke1** to **gcp-us-central1-transit**

Edit the Spoke Gateway **_gcp-us-central-spoke1_**, clicking on the pencil icon:

```{figure} images/lab2-editspokeingcp.png
---
height: 300px
align: center
---
Edit spoke in GCP
```

Select the Transit Gateway **_gcp-us-central1-transit_** from the drop-down window `"Attach To Transit Gateway"`, and then click on **Save**.

```{figure} images/lab2-editgcp.png
---
align: center
---
Attachment in GCP
```

Look for these three confirmations through the task icon, before proceeding.

```{figure} images/lab2-confirmation.png
---
align: center
---
Confirmations
```

At this point, after **attaching** Spoke Gateways to their respective Transit Gateways, this is what the overall topology would look like.

```{figure} images/lab2-topologywithattachments.png
---
height: 400px
align: center
---
New state of the Dynamic Topology
```

```{note}
The Spoke Gateway azure-west-us-**spoke2** will be attached to its Transit Gateway in a subsequent lab, likewise the Spoke Gateways in AWS **us-east-1** will be attached to the Transit Gateway in the same region only in a subsequent lab.
```

## 4.5. CoPilot Verification of Spoke-Transit Attachments

Go to **CoPilot > Cloud Fabric > Topology > Overview**

Now, verify the spoke-transit attachments. You can see the dotted lines connecting the 3 spoke gateways to their respective transits.

```{tip}
Be patient, it will take some minutes before seeing the changes reflected into the topology. Refresh the web page to see the change reflected on the map!
```

```{figure} images/lab2-attachment.png
---
align: center
---
Attachments
```

```{figure} images/lab2-expandedtopology1.png
---
align: center
---
Expanded Topology
```

Click on the `"Legend"` button to figure out what those icons represent.

```{important}
Dashed line = Default IPSec tunnel

Solid line = HPE IPSec tunnel
```

```{figure} images/lab2-hpe.png
---
align: center
---
Legend
```

## 4.6. Multicloud Transit Peerings

In this section you are going to establish the peerings among the Aviatrix Transit Gateways.

```{warning}
Transit peering is **_bidirectional_**. 

You do not need to configure peering in the opposite direction.
```

Go back to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**
 
### 4.6.1. AWS and Azure

- **aws-us-east-2-transit** to **azure-west-us-transit**

Edit the Transit Gateway **_aws-us-east-2-transit_**, clicking on the pencil icon:

```{figure} images/lab2-edittransitinaws.png
---
height: 300px
align: center
---
Edit Transit in AWS
```

Select the Transit Gateway **_azure-west-us-transit_** from the drop-down window `"Peer To Transit Gateways"`, and then click on **Save**.

```{figure} images/lab2-peeringawsazure.png
---
align: center
---
Peering AWS-Azure
```

### 4.6.2 Azure and GCP

- **azure-west-us-transit** to **gcp-us-central1-transit**

Edit the Transit Gateway **_azure-west-us-transit_**, clicking on the pencil icon:

```{figure} images/lab2-edittransitinazure.png
---
height: 300px
align: center
---
Edit Transit in Azure
```

Select the Transit Gateway **_gcp-us-central1-transit_** from the drop-down window `"Peer To Transit Gateways"`, and then click on **Save**.

```{figure} images/lab2-peeringazuregcp.png
---
align: center
---
Peering Azure-GCP
```

### 4.6.3. GCP and AWS

- **gcp-us-central1-transit** to **aws-us-east-2-transit**

Edit the Transit Gateway **_gcp-us-central1-transit_**, clicking on the pencil icon:

```{figure} images/lab2-editgcp2.png
---
height: 300px
align: center
---
Edit Transit in GCP
```

Select the Transit Gateway **_aws-us-east-2-transit_** (<ins>not the east-1 !</ins>) from the drop-down window `"Peer To Transit Gateways"`, and then click on **Save**.

```{figure} images/lab2-peeringgcpaws.png
---
align: center
---
Peering GCP-AWS
```

At this point, this is what the overall topology would look like:

```{figure} images/lab2-peeringtopology.png
---
height: 400px
align: center
---
New Topopology state after Peerings deployment
```

```{note}
Please pay close attention that the following pending elements will be completed only in a subsequent lab:

- Attachment between **aws-us-east-1-spoke1** and **aws-us-east-1-transit**
- Peering between **aws-us-east-1-transit** and **aws-us-east-2-transit**
- Attachment between **azure-west-us-spoke2** and **azure-west-us-transit**
- Attachment between **aws-us-east-2-transit** and **on-prem-dc-edge**
```

## 5. Verification
 
### 5.1. Verification of Transit Peerings on CoPilot (Cloud Fabric)

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gateway **_aws-us-east-2-transit_**, then select the `Attachments"` tab and finally select the `"Transit-Transit Peering"` tab: you will see **one** connection per each peering, that correspond to the `two IPSec tunnels`.

```{figure} images/lab2-verification.png
---
align: center
---
Verification
```

### 5.2. Verification of Transit Peerings on CoPilot (Topology)

Go to **CoPilot > Cloud Fabric > Topology > Overview**

```{note}
Refresh the web page!
```

Verify transit-transit peering. You can now see the dotted lines in the inner circle representing the configured full mesh peering between the three transits.

```{figure} images/lab2-peering.png
---
align: center
---
Peerings
```

```{figure} images/lab2-expanded2.png
---
align: center
---
Expanded Topology
```

### 5.3. Route Info DB

Route Info DB is akin to the *Routing Information Base (RIB)*. It will provide the overall routing information of a Transit Gateway known by the CoPilot.

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the Transit Gateway **_aws-us-east-2-transit_**:

```{figure} images/lab2-transitaws.png
---
align: center
---
Explore Transit in AWS
```

Then select the `"Route DB"` tab. 
Pay special attention to `“Best Routes”`, its prefixes, type and metric value:

```{figure} images/lab2-rib.png
---
align: center
---
Route DB
```

## 6. Final Connectivity Testing

Verify that each test instance can ping each other.

### 6.1 Connectivity Test Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select **_aws-us-east-2-spoke1-test1_** as an example. 

Then, sign in to the Gatus app.

```{figure} images/lab2-gastusfinal00.png
---
align: center
---
ICMP
```

```{figure} images/lab2-gastusfinal01.png
---
align: center
---
SSH
```

You will observe that both ICMP and SSH tests have successfully completed for all three instances:

1) **_aws-us-east-2-spoke1-test2_** (enabled by the normal `intra-VPC traffic`)

2) **_gcp-us-central1-spoke1-test1 _**(enabled by the `Aviatrix MCNA`)

3) **_azure-west-us-spoke1-test1_** (enabled by the `Aviatrix MCNA`)

```{important}
Review the Gatus App status for the other instances in Azure and GCP.
```

### 6.1 Connectivity Test Using the SSH Client

If you're not satisfied with the Gatus dashboard, you can also use your personal `SSH client` to perform the connectivity tests!

- Open three terminal windows to SSH to the **public IPs** of the 3 spoke **test instances/VMs** in each cloud.

Then ping the **private** IPs of each other VMs to test the Multi-Cloud connectivity. 

<ins>Refer to your pod portal for the public/private IPs or retrieve them from the **_Cloud Assets/Topology_**</ins>.

```{important}
Refresh the web page, to see the changes reflected into your CoPilot's topology!
```

```{note}
**`POD PORTAL`**:

Please note that both the public DNS names and private IP addresses of the **test** instances can be retrieved from your personal POD portal.
```{figure} images/lab2-newpic.png
---
align: center
---
POD Portal info
```

```{note}
**`TOPOLOGY (CoPilot > Cloud Fabric > Topology)`**:

Explore the **_aws-us-east-2-spoke1_** VPC and select the instance **_aws-us-east-2-spoke1-test1_** with the **EC2 instance logo**, then check its properties: you will be able to fetch both Public and Private IP addresses.

```{figure} images/lab2-newpic3.png
---
height: 400px
align: center
---
Test Instance Properties

```

```{note}
Do not select the instance with the **Aviatrix logo**!

<ins>You can't SSH to any Aviatrix GWs !</ins>


```{figure} images/lab2-newpic2.png
---
align: center
---
Different Logos
```


- SSH into aws-us-**east-2**-spoke1-<span style='color:red'>test1</span> (ssh student@public_ip)
- SSH into azure-west-us-**spoke1**-<span style='color:red'>test1</span> (ssh student@public_ip)
- SSH into gcp-us-central1-spoke1-<span style='color:red'>test1</span> (ssh student@public_ip)

Run ping from the AWS instance to verify connectivity to Azure and GCP:

```{figure} images/lab2-pingfromaws.png
---
align: center
---
Ping from AWS
```

Run ping from Azure VM to verify connectivity to AWS and GCP:

```{figure} images/lab2-pingfromazure.png
---
align: center
---
Ping from Azure
```

Run ping from GCP VM to verify connectivity to Azure and AWS:

```{figure} images/lab2-pingfromgcp.png
---
align: center
---
Ping from GCP
```