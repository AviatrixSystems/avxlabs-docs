# Lab 2 - TRANSIT NETWORKING

## 1. Objective

Build Transit Network in Azure, GCP and AWS using Aviatrix Multicloud Transit hub and spoke model.

In this lab, we will use Aviatrix CoPilot to connect three major clouds, i.e. Azure, GCP and AWS. The workloads in VPCs/VNets in all three clouds must communicate without manual configuration on the native consoles.
 
## 2. Multicloud Connectivity Overview

Enterprises are relying increasingly on multiple clouds (multicloud) providers. However, setting up the connectivity between those providers is difficult. Moreover, maintaining and monitoring the tunnels is time-consuming and cumbersome to troubleshoot.

Aviatrix simplifies this by providing simple, point-and-click tunnel creation between cloud providers.

Additionally, Aviatrix gives you a single, centralised location from which to troubleshoot and monitor your connections.
 
## 3. Topology

In this lab, as shown in the topology below, we will configure the grey Aviatrix gateways, the grey attachments between Transit-Spoke and the grey peerings betweenTransit-Transit, but only in the following regions:

- **AWS**: us-east-2
- **GCP**: us-central-1
- **AZURE**: us-west (only spoke1)

The rest of the topology has been pre-deployed to save time, including the test instances/VMs.

![lab2-topology](images/lab2-topology.png)
_Figure 20: Initial pre-provisioned topology_

```{note}
The test VPCs/VNet you created in Lab 1 will not be used in other labs.

In this lab, the Transits will be deployed in pairs, but the Spokes will not.
```

The CoPilot *dashboard* should look something like this:

![dashboard](images/lab2-dashboard.png)
_Figure 21: Dashboard_

- Before starting building your multicloud infrastructure, adjust the fetch timers on the CoPilot.

```{hint}
Go to **CoPilot > Settings > Resources > Task Server**
```

Ensure that `Fetch Topology`, `Fetch Instances`, `Fetch GW Routes` and `Fetch VPC Routes` intervals are set to **“1 Second”** each and then click on **SAVE**.

![Task server](images/lab2-timer.png)
_Figure 22: Task Server_

![Fetch topology](images/lab2-fetchtopology.png)
_Figure 23: Fetch Topology_

![Fetch instances](images/lab2-fetchinstances.png)
_Figure 24: Fetch Instances_

![Fetch gw](images/lab2-fetchgw.png)
_Figure 25: Fetch GW_

![Fetch vpc](images/lab2-fetchvpc.png)
_Figure 26: Fetch VPC_

Afterwards, click on **Commit**.

![commit](images/lab2-commit.png)
_Figure 27: Commit_

```{hint}
Ensure to `Refresh` the web page to see the changes applied, successfully.
```

```{note}
These are very aggressive settings. In a Production environment, you should not set these intervals that frequently!
```

## 4. Initial configuration

Go to **CoPilot > Dashboard** and click on the widget called `"Aviatrix Gateways"`. When you begin this lab, you should have **nine** gateways in your pod:

![dashboard gw](images/lab2-dashboardgw.png)
_Figure 28: Commit_

Nevertheless, if you go to **CoPilot > Cloud Fabric > Gateways > Overview (default tab)**, you will notice that the number of Transit Gateways is set to **three**, whereas the number of Spoke Gateways is set to **two**:

![cluster](images/lab2-cluster.png)
_Figure 29: Cluster_

This view within the Cloud Fabric section does not indicate the exact number of gateways but it refers to to the number of clusters, per each type of gateway.

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and expand the three drop-down lists. You can find out that there are a total of **six** Transit Gateways (Public IPs may differ):

![cluster transit gws](images/lab2-clustertransit.png)
_Figure 30: Transit GWs Clusters_

Furthermore, you can notice that the name of the cluster matches exactly the name of the first Transit Gateway, whereas the name of the second Transit Gateway is similar to the name of the first gateway but it has appended a "-hagw"; this is imposed by the "aviatrix_transit_gateway" Terraform resource.

```{note}
You can deploy up to maximum **two** Transit Gateways per each Transit VPC/VNet/VCN.
```

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and expand the unique drop-down list. You can find out that there are a total of **three** Spoke Gateways (once again, the Public IPs may differ):

![cluster spoke gws](images/lab2-clusterspoke.png)
_Figure 31: Transit GWs Clusters_

You can notice that the cluster in AWS comprises two Spoke Gateways, whereas in Azure there is just a single Spoke Gateway. This is a valid deployment. The number of Spoke Gateways that you should deploy per each Spoke VPC/VNet/VCN is dictated by your architecture design.

```{note}
You can deploy up to maximum **fifteen** Spoke Gateways per each Spoke VPC/VNet/VCN.
```

### 4.1. Aviatrix Transit Gateways

In this section, you will experience the power and simplicity of the Aviatrix platform by deploying (i.e. creating) 5 gateways:

 - **First Transit gateway in AWS US East 2:** <span style='color:#33ECFF'>aws-us-east2-transit</span>
 - **Second Transit gateway in AWS US East 2:** <span style='color:#33ECFF'>aws-us-east2-transit-1</span>
 - **Spoke gateway in AWS US East 2:** <span style='color:#33ECFF'>aws-us-east2-spoke1</span>
- **Spoke gateway in Azure West US:** <span style='color:#33ECFF'>azure-us-west-spoke1</span>
- **Spoke gateway in GCP US Central 1:** <span style='color:#33ECFF'>gcp-us-central1-spoke1</span>

```{warning}
Please pay close attention to each step, as a misconfiguration could result in **20+** minutes of lost time! 
```

- Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways > + Transit Gateway**

![cluster spoke gws](images/lab2-transitbutton.png)
_Figure 32: +Transit Gateway_

Deploy Aviatrix Transit Gateways in AWS east-2 region. To save time, Aviatrix Transit Gateways in Azure, GCP and AWS east-1 region have already been pre-deployed in pairs for this lab.

### 4.1.1.Transit Gateway in AWS US-EAST-2

Ensure these parameters are entered in the pop-up window `"Create Transit Gateway"`.

- **Name:** <span style='color:#33ECFF'>aws-us-east2-transit</span>
- **Cloud:** <span style='color:#33ECFF'>AWS (Standard)</span>
- **Account:** <span style='color:#33ECFF'>aws-account</span>
- **Region:** <span style='color:#33ECFF'>us-east-2 (Ohio)</span>
- **VPC ID:** <span style='color:#33ECFF'>aws-us-east2-transit (Make sure you don't select aws-us-east2-**spoke** VPC)</span>
- **Instance Size:** <span style='color:#33ECFF'>c5n.large</span>
- **High Performance Encryption:** <span style='color:#33ECFF'>On</span>

Click on `+ Instance` (Make sure you deploy also the Second Transit Gateway, in a different subnet).

- **Attach to Subnet (row 1):** <span style='color:#33ECFF'>us-east-2a</span>
- **Attach to Subnet (row 2):** <span style='color:#33ECFF'>us-east-2b</span>
- **Public IP:** <span style='color:#33ECFF'>Allocate New Static Public IP</span>

```{note}
As soon as you select High Performance Encryption, /26 subnets will appear in the drop-down window.
```

Click **SAVE**.

![create transit gw](images/lab2-transitcreation.png)
_Figure 33: Create Transit Gateway_

You will immediately get a message as follows.

![gw message creation](images/lab2-gwmessage.png)
_Figure 34: Gateway deployment in progress_

You may check the status of the gateway creation in the top right corner by expanding the task icon.

![gw message creation](images/lab2-taskicon.png)
_Figure 35: Task icon_

This action will instantiate two Transit Gateways with the following names:

1. **aws-us-east2-transit**
2. **aws-us-east2-transit-1**

```{note}
The second gateway will receive **"-1"** appended to its name, in order to differentiate it from the first gateway.
```

Meanwhile the deployment is happening, you may proceed to the next section of this lab guide to deploy your Spoke gateways.

### 4.2. Aviatrix Spoke Gateways

Navigate to the tab immediately to the right, which is `Spoke Gateways`. 

This is **CoPilot > Cloud Fabric > Gateways > Spoke Gateways > + Spoke Gateway**.

![Spoke GW button](images/lab2-spokecreate.png)
_Figure 36: +Spoke Gateway_

### 4.2.1. Spoke Gateway in AWS

Ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{note}
Only one Spoke Gateway will be deployed in VPC aws-us-east2-spoke1.
```

- **Name:** <span style='color:#33ECFF'>aws-us-east2-spoke1</span>
- **Cloud:** <span style='color:#33ECFF'>AWS (Standard)</span>
- **Account:** <span style='color:#33ECFF'>aws-account</span>
- **Region:** <span style='color:#33ECFF'>us-east-2 (Ohio)</span>
- **VPC ID:** <span style='color:#33ECFF'>aws-us-east2-spoke1 (Make sure you don't select aws-us-east2-**transit** VPC)</span>
- **Instance Size:** <span style='color:#33ECFF'>t2.medium</span>
- **High Performance Encryption:** <span style='color:#33ECFF'>Off</span>
- **Attach to Subnet:** <span style='color:#33ECFF'>us-east-2a</span>
- **Public IP:** <span style='color:#33ECFF'>Allocate New Static Public IP</span>

Click **SAVE**.

![Spoke GW button](images/lab2-spokeinaws.png)
_Figure 37: Create Spoke Gateway in AWS_

While the gateway is being created, you may proceed to the next section.

### 4.2.2. Spoke Gateway in Azure

Repeat the previous steps for Azure, click on the button `"+ Spoke Gateway"` and ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{note}
Only one Spoke Gateway will be deployed in VNet azure-us-wes-spoke1.
```

- **Name:** <span style='color:#33ECFF'>azure-us-west-spoke1</span>
- **Cloud:** <span style='color:#33ECFF'>Azure (Global)</span>
- **Account:** <span style='color:#33ECFF'>azure-account</span>
- **Region:** <span style='color:#33ECFF'>West US</span>
- **VNet:** <span style='color:#33ECFF'>azure-us-west-spoke1 (Make sure you don't select azure-us-west-**spoke2** VPC)</span>
- **Instance Size:** <span style='color:#33ECFF'>Standard_B1ms</span>
- **High Performance Encryption:** <span style='color:#33ECFF'>Off</span>
- **Attach to Subnet:** <span style='color:#33ECFF'>azure-us-west-spoke1-Public-gateway-subnet-1</span>
- **Public IP:** <span style='color:#33ECFF'>Allocate New Static Public IP</span>

```{warning}
Make sure you <ins>do not select the subnets that begins with az-1, az-2, or az-3</ins>. It is Aviatrix's recommended practice to deploy gateways in subnets with 'gateway' in their name, whereas workloads in subnets that do not have 'gateway' in their name).
```

![Subnet selection](images/lab2-rightsubnet.png)
_Figure 38: Subnet selection_

Do not forget to click on **SAVE**.

![Spoke in Azure](images/lab2-spokeinazure.png)
_Figure 39: Spoke GW in Azure_

While the gateway is being created, you may proceed to the next section.
 
### 4.2.3. Spoke Gateway in GCP

Repeat the previous steps for GCP. Ensure these parameters are entered in the pop-up window `"Create Spoke Gateway"`.

```{warning}
Only one Spoke Gateway will be deployed in VPC gcp-us-central1-spoke1.
```

- **Name:** <span style='color:#33ECFF'>gcp-us-central1-spoke1</span>
- **Cloud:** <span style='color:#33ECFF'>GCP</span>
- **Account:** <span style='color:#33ECFF'>gcp-account</span>
- **VPC:** <span style='color:#33ECFF'>gcp-us-central1-spoke1</span>
- **Instance Size:** <span style='color:#33ECFF'>n1-standard-1</span>
- **High Performance Encryption:** <span style='color:#33ECFF'>Off</span>
- **Attach to Subnet:** <span style='color:#33ECFF'>gcp-us-central1-spoke1-sub1</span>
- **Zone:** <span style='color:#33ECFF'>us-central1-a</span>
- **Public IP:** <span style='color:#33ECFF'>Allocate New Static Public IP</span>

Click **SAVE**.

![Spoke in GCP](images/lab2-spokeingcp.png)
_Figure 40: Spoke GW in GCP_

```{caution}
You can see the progress of gateway deployment at any time by expanding the task icon in the top right corner of the CoPilot.
```

![In progress](images/lab2-inprogress2.png)
_Figure 41: Deployment in progress_

Once all gateways have been created, confirm from **CoPilot > Dashboard** that 14 gateways exist in your environment:

![14 gws](images/lab2-14gws.png)
_Figure 42: Dashboard_

After creating the Transit gateways pair in AWS and the Spoke gateways in each cloud, this is how a portion of the topology would look like.

![temp topology](images/lab2-temptopology.png)
_Figure 43: Overview of the new topology state_

## 4.3. Explore the Cloud Fabric

Go to **CoPilot > Cloud Fabric > Topology > Overview (default tab)**.

![legend](images/lab2-topologyoverview.png)
_Figure 44: VPC circles_

![expand](images/lab2-expand.png)
_Figure 45: Expand all the VPCs_

```{important}
The inner circle represents the Transit Gateway VPCs, and the outer one represents the Spoke Gateway VPCs. <ins>It is clearly shown at this stage that the spokes are not connected to the transits yet</ins>. 
```

In addition, you can explore the components of any of the gateways in terms of subnets and Virtual Machines that reside within the VPC/VNet.
 
## 4.4 Aviatrix Spoke to Transit Gateways Attachments

In this section you are going to attach the Aviatrix Spoke Gateways created above in each cloud, to their corresponding Aviatrix Transit Gateways.
 
### 4.4.1. Spoke to Transit Attachment in AWS

Go to CoPilot > Cloud Fabric > Gateways > Spoke Gateways and edit the Spoke Gateway aws-us-east2-spoke1, clicking on the pencil icon:

![attachment for aws](images/lab2-spokeinawstotransit.png)
_Figure 46: Attachment for AWS_

Select the Transit Gateway **_aws-us-east2-transit_** from the drop-down window from the `"Attach To Transit Gateway"` field, and then click on **Save**.

![edit spoke in aws](images/lab2-editspokeinaws.png)
_Figure 47: Edit Spoke in AWS_

You will see immediately a message informing that the updating is in progress.

![immediate message](images/lab2-immediatemessage.png)
_Figure 48: Update in progress_

### 4.4.2 Spoke to Transit Attachment in Azure

- **azure-us-west-spoke1** to **azure-us-west-transit**

Edit the Spoke Gateway **_azure-us-west-spoke1_** (not the **spoke2**), clicking on the pencil icon.

![edit spoke in azure](images/lab2-editspokeinazure.png)
_Figure 49: Edit spoke in Azure_

Select the Transit Gateway **_azure-us-west-transit_** from the drop-down window from the `"Attach To Transit Gateway"` field, and then click on **Save**.

![edit azure](images/lab2-editazure.png)
_Figure 50: Attachment in Azure_

### 4.4.3. Spoke to Transit Attachment in GCP

- **gcp-us-central1-spoke1** to **gcp-us-central1-transit**

Edit the Spoke Gateway **_gcp-us-central-spoke1_**, clicking on the pencil icon:

![edit spoke in gcp](images/lab2-editspokeingcp.png)
_Figure 51: Edit spoke in GCP_

Select the Transit Gateway **_gcp-us-central1-transit_** from the drop-down window from the `"Attach To Transit Gateway"` field, and then click on **Save**.

![edit GCP](images/lab2-editgcp.png)
_Figure 52: Attachment in GCP_

Look for these three confirmations through the task icon, before proceeding.

![confirmations](images/lab2-confirmation.png)
_Figure 53: Confirmations_

At this point, after **attaching** Spoke Gateways to their respective Transit Gateways, this is what the overall topology would look like.

![new state of topology](images/lab2-topologywithattachments.png)
_Figure 54: New state of the topology_

```{note}
The Spoke Gateway azure-us-west-**spoke2** will be attached to its Transit Gateways in a subsequent lab, likewise the Spoke Gateways in AWS **us-east-1** will be attached to the Transit Gateways in the same region only in a subsequent lab.
```

## 4.5. CoPilot Verification of Spoke-Transit Attachments

Go to **CoPilot > Cloud Fabric > Topology > Overview**

Now, verify the spoke-transit attachments. You can see the dotted lines connecting the 3 spoke gateways to their respective transits.

```{tip}
Be patient, it will take some minutes before seeing the changes reflected into the topology. Refresh the page to see the change reflected on the map!
```

![attachment](images/lab2-attachment.png)
_Figure 55: Attachments_

![expanded](images/lab2-expandedtopology1.png)
_Figure 55: Expanded Topology_

## 4.6. Multicloud Transit Peerings

In this section you are going to establish the peerings among the Aviatrix Transit Gateways.

```{tip}
Transit peering is bi-directional. You do not need to configure peering in the opposite direction.
```

Go back to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**
 
### 4.6.1. AWS and Azure

- **aws-us-east2-transit** to **azure-us-west-transit**

Edit the Transit Gateway **_aws-us-east2-transit_**, clicking on the pencil icon:

![edit transit in aws](images/lab2-edittransitinaws.png)
_Figure 56: Edit Transit in AWS_

Select the Transit Gateway **_azure-us-west-transit_** from the drop-down window from the `"Peer To Transit Gateways"` field, and then click on **Save**.

![peering aws-azure](images/lab2-peeringawsazure.png)
_Figure 56: Peering AWS-Azure_

### 4.6.2 Azure AND GCP

- **azure-us-west-transit** to **gcp-us-central1-transit**

Edit the Transit Gateway **_azure-us-west-transit_**, clicking on the pencil icon:

![edit azure](images/lab2-edittransitinazure.png)
_Figure 57: Edit Transit in Azure_

Select the Transit Gateway **_gcp-us-central1-transit_** from the drop-down window from the `"Peer To Transit Gateways"` field, and then click on **Save**.

![peering azure-gcp](images/lab2-peeringawsazure.png)
_Figure 58: Peering Azure-GCP_

### 4.6.3. GCP and AWS

- **gcp-us-central1-transit** to **aws-us-east2-transit**

Edit the Transit Gateway **_gcp-us-central1-transit_**, clicking on the pencil icon:

![edit transit in gcp](images/lab2-editgcp2.png)
_Figure 59: Edit Transit in GCP_

Select the Transit Gateway **_aws-us-east2-transit_** (<ins>not the east1 !</ins>) from the drop-down window from the `"Peer To Transit Gateways"` field, and then click on **Save**.

![peering GCP-AWS](images/lab2-peeringgcpaws.png)
_Figure 60: Peering GCP-AWS_

At this point, this is what the overall topology would look like:

![peering topology](images/lab2-peeringtopology.png)
_Figure 60: New Topopology state after Peerings deployment_

```{note}
Please pay close attention that the following pending elements will be completed only in a subsequent lab:

- Attachment between **aws-us-east1-spoke1** and **aws-us-east1-transit**
- Peering between **aws-us-east1-transit** and **aws-us-east2-transit**
- Attachment between **azure-us-west-spoke2** and **azure-us-west-transit**
```

## 5. Verification
 
### 5.1. Verification of Transit Peerings on CoPilot(Cloud Fabric)

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gateway **_aws-us-east2-transit_**, then select the `"Connections"` tab and finally select the `"Transit-Transit Peering"` sub-tab:

![verification](images/lab2-verification.png)
_Figure 61: Verification_

### 5.2. Verification of Transit Peerings on CoPilot (Topology)

Go to **CoPilot > Cloud Fabric > Topology > Overview**

```{note}
Refresh the web page!
```

Verify transit-transit peering. You can now see the dotted lines in the inner circle representing the configured full mesh peering between the three transits.

![peerings](images/lab2-peering.png)
_Figure 62: Peerings_

![expanded2](images/lab2-expanded2.png)
_Figure 63: Expanded Topology_

### 5.3. Route Info DB

Route Info DB is similar to Routing Information Base (RIB). It will provide the overall routing information of a Transit Gateway known by the CoPilot

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the Transit Gateway **_aws-us-east2-transit_**:

![explore transit in aws](images/lab2-transitaws.png)
_Figure 64: Explore Transit in AWS_

Then select the "Route DB" tab. 
Pay special attention to `“Best Routes”`, its prefixes, type and metric value:

![rib](images/lab2-rib.png)
_Figure 65: Route DB_

## 5.4. Connectivity

Verify each test instance can ping each other.

Open three terminal windows to SSH to the public IPs of the 3 spoke test instances/VMs in each cloud.

Then ping the private IPs of each other to test the Multi-Cloud connectivity. <ins>Refer to your pod portal for the private IPs or retrieve the private IPs from the topology</ins>.

- SSH into aws-us-**east2**-spoke1-test1 (ssh student@public_ip)
- SSH into azure-us-west-spoke1-test1 (ssh student@public_ip)
- SSH into gcp-us-central1-spoke1-test1 (ssh student@public_ip)

Run ping from the AWS instance to verify connectivity to Azure and GCP:

![ping from aws](images/lab2-pingfromaws.png)
_Figure 66: Ping from AWS_

Run ping from Azure VM to verify connectivity to AWS and GCP:

![ping from azure](images/lab2-pingfromazure.png)
_Figure 67: Ping from Azure_

Run ping from GCP VM to verify connectivity to Azure and AWS:

![ping from gcp](images/lab2-pingfromgcp.png)
_Figure 68: Ping from GCP_

## 6. Bonus Question

- What resources are created in the respective clouds when deploying the transit gateway?