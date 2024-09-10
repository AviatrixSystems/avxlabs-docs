# Lab 3 - Expanding to multi-cloud

Lab time: ~30 minutes

**_Scenario_**:  The business has decided to go multicloud! Some apps simply run better in other clouds, and why put all eggs in one basket? During this exercise we will deploy in AWS an Aviatrix Transit VPC, Transit Gateway and attach the existing spoke gateways to the aws transit. We will also peer Azure and AWS and test out the connectivity.

The Spoke VPCs and gateways have already been created to save you time. The process is very similar to deploying the transit gateway.

```{figure} images/lab-3.png
---
height: 400px
align: center
---
Current Topology
```

## Lab 3.1 - Deploy the AWS transit VPC

### Description

Let's go ahead and create the Transit VPC in AWS using the topology builder in Copilot.

### Validate

* Log in to Aviatrix Copilot
* Scroll down on the left-hand pane to **_Cloud Fabric -> Topology -> Builder_**
* Click on "Acknowledge".
  
```{figure} images/lab3-ack.png
---
align: center
---
Current Topology
```

* Click **Select Cloud Region** and enter the following fields:

|                    |                          |
| ------------------ | ------------------------ |
| **Cloud Provider** | AWS                      |
| **Account**        | aws-account              |
| **Region**         | eu-central-1 (Frankfurt) |

```{figure} images/lab-3-topology-builder.png
---
height: 400px
align: center
---
Set up the topology builder
```

* Hit **Save** to start the configuration session.
* Click The "+" sign next to transit, to add a new transit.

```{figure} images/lab3-transit.png
---
align: center
---
New Transit
```

* Enter the following fields:

|                         |                     |
| ----------------------- | ------------------- |
| **Transit Gateway VPC** | Create New VPC/VNet |
| **VPC Name**            | aws-transit         |
| **VPC CIDR**            | 10.pod[#].40.0/23   |

```{figure} images/lab3-vpc.png
---
align: center
---
Create New VPC
```

```{figure} images/lab3-newvpc.png
---
align: center
---
Template for the new VPC
```

For the VPC CIDR, replace **pod[#]** with your pod number. For example, if your pod number is 150, pod[#] should be replaced with 150, so the complete CIDR will become 10.150.40.0/23. The topology builder will automatically create all the required public and private subnets, IGW, and routing tables.

Hit **Save**, and the VPC will be created.

```{caution}
Wait a few minutes for the **Aviatrix Controller** to create the new VPN and the additional native cloud constructs (i.e. IGW, Subnets, Routing Tables).
```
  
### Expected Results

You created a VPC with multiple subnets directly in the Aviatrix GUI.

## Lab 3.2 - Deploy the Aviatrix Transit Gateway

### Description

In this exercise we are going to launch the Aviatrix Transit Gateway in the newly created Transit VPC in AWS.

### Validate

* Enter the following fields:

|                          |                                                         |
| ------------------------ | ------------------------------------------------------- |
| **Transit Gateway VPC**  | aws-transit                                             |
| **Gateway Name**         | aws-transit                                             |
| **Gateway Size**         | t3.micro                                                |
| **Instance 1 (Primary)** | aws-transit-Public-1-eu-central-1a + Allocate New EIP   |
| **Transit Peers**        | Click "Select Transit Gateways"" and add the **azure-transit** as a peer and save |

```{figure} images/lab3-transit-config.png
---
height: 400px
align: center
---
Create Aviatrix Transit Gateway
```

```{figure} images/lab3-transit-peering.png
---
height: 400px
align: center
---
Configure transit peering
```

* Hit **Save** to save the transit gateway settings and return to the topology builder overview.
* You now get the option to either generate your changes as Terraform code, or deploy directly from the UI. Choose and click **Deploy**.

```{figure} images/lab3-overview.png
---
height: 300px
align: center
---
Overview
```

* As you can see, the topology builder will now start to deploy the transit gateway. Wait for this to finish.

```{caution}
It will take about 5-10 minutes, therefore please be patient.
```

```{figure} images/lab3-deploy.png
---
align: center
---
Deploy
```

> **Important:** By default, routes are not propagated between spokes attached to the transit gateway. We have to enable a setting on our transit gateway to allow this to happen.  

* Go to **_Cloud Fabric -> Gateways -> Transit Gateways_** and click on **_aws-transit_**.
* Open the settings pane and enable and save **Connected Transit** under the General settings.

```{figure} images/lab3-connected-transit.png
---
align: center
---
Enable connected transit
```

### Expected Results

Now that we have the aws-transit deployed, peered to azure-transit and have enabled connected transit, we should see these changes reflected in our topology view. (under **_Cloud Fabric -> Topology_**)

```{figure} images/lab3-result.png
---
height: 400px
align: center
---
Topology view
```

## Lab 3.3 - Attach Spoke Gateways to Aviatrix Transit Gateway

### Description

Now that we have our aws-transit set up, we need to establish connectivity between the AWS spoke gateways and the aws-transit gateway. The spoke gateways have been pre-deployed to save some time, but the process is much the same as the transit gateway you have just deployed.

### Validate

* Go to **_Cloud Fabric -> Gateways -> Spoke Gateways_**
* Click the edit button (pencil) behind the **_aws-prod_** gateway. 

```{figure} images/lab3-edit-spoke.png
---
height: 200px
align: center
---
Edit spoke
```

* Change the attachment from empty to **_aws-transit_** and hit save.

```{figure} images/lab3-attach-spoke.png
---
align: center
---
Attach Spoke Gateway to Transit
```

* Repeat the process for the **_aws-shared_** spoke gateway!

```{figure} images/lab3-aws-shared.png
---
align: center
---
Attach Spoke Gateway to Transit
```

### Expected Results
The AWS spoke gateways should now be attached to the aws-transit. Within a few minutes, we should see these changes reflected in our topology view. (under **_Cloud Fabric -> Topology_**)

```{figure} images/lab3-copilot-spoke-attach.png
---
align: center
---
CoPilot Topology with Attached Spokes
```

* If your topology does not show the connection to the spokes, refresh the page after a minute or so.

## Lab 3.4 - Enable network segmentation on the AWS transit

### Description

Just like we enabled network segmentation on the azure-transit before, we want to extend this segmentation to AWS and enable it on the aws-transit as well.

### Validate

* In the Copilot, go to **_Networking -> Network Segmentation -> Network Domains_** and click the **_Transit Gateways_** button.
* In the transit gateway pane, enable segmentation for **aws-transit**.

```{figure} images/lab2-enable-segmentation.png
---
align: center
---
Transit Segmentation
```

```{figure} images/lab3-enable-segmentation-3.png
---
align: center
---
Enable Segmentation
```

### Expected Results

Segmentation is now enabled on both gateways.

## Lab 3.5 - Create Network Domains

### Description

For our AWS spokes, we want to create network domains and associations, just like we did for the Azure spokes before.

### Validate

In CoPilot, go to **_Networking -> Network Segmentation -> Network Domains_** 

Use the **_+ Network Domains_** button to add the following domains and associations:

| Domain     | Association |
| :--------- | :---------- |
| AWS-Shared | aws-shared  |
| AWS-Prod   | aws-prod    |

### Expected Results

After adding the network domains and associations, you should see the following in Copilot:

```{figure} images/lab3-network-domains.png
---
align: center
---
Add network domains
```

## Lab 3.6 - Create a connection policy

### Description

We want the development team in the office, to be able to access AWS-Shared.

### Validate

* Edit the **_Office_** network domain under **_Networking -> Network Segmentation -> Network Domains_** in Copilot.

```{figure} images/lab3-edit-network-domain.png
---
align: center
---
Edit network domain
```

* Now add the **_AWS-Shared_** domain as a connected domain.

```{figure} images/lab3-add-network-domain-policy.png
---
align: center
---
Add network domain connection
```

### Expected Results

* The network domains should now be connected to each other, as shown on the screenshot below.
* Another great place to visualize the connectivity between network domains is **_Networking -> Network Segmentation-> Overview_**.

```{figure} images/lab3-add-connected-network-domain-result.png
---
align: center
---
Add network domain connection
```

```{figure} images/lab3-network-domain-overview.png
---
align: center
---
Network domain overview
```

## Lab 3.7 - Test Connectivity to AWS Spokes

### Description

The AWS Spokes should be connected to the AWS Transit so now we can check connectivity to the spokes from the office.

### Validate

* Open the Remote Access Server and open the **RDP - Client** (or navigate to `https://client.pod[#].aviatrixlab.com`). This is the on-prem Host.
* Open up the `Min` browser on the RDP desktop
* Navigate to `http://localhost`

> Do we have access to aws-shared and azure-build?

### Expected Results

* As you can see, you now have access to azure-build and aws-shared.

```{figure} images/lab3-office-access.png
---
align: center
---
Office Access
```

> Were these connections succesful?

### Expected Results

From the office, you should be able to reach aws-shared, but not aws-prod.

## Lab 3 Summary

* AWS has been onboarded
* You deployed a VPC and an Aviatrix Transit Gateway in AWS
* You connected all of the Spoke VPCs to the Transit
* You created a Multicloud Transit network, connecting AWS and Azure
* You are able to reach the AWS environment from On-Prem, via the Azure Transit
* Again, not a single route table entry needed to be touched on the CSP side
* You have end to end visibility into all network traffic
* All connectivity is private and encrypted across the entirety of the Aviatrix Cloud Fabric - from on-prem and within and between the clouds

```{figure} images/end-lab-3.png
---
height: 400px
align: center
---
Topology after Lab 3
```
