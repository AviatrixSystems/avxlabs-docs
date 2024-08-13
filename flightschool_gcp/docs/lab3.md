# Lab 3 - Expanding to multi-cloud

Lab time: ~30 minutes

**_Scenario_**:  The business has decided to go multicloud! Some apps simply run better in other clouds, and why put all eggs in one basket? During this exercise we will deploy in GCP an Aviatrix Transit VPC, Transit Gateway and attach the existing spoke gateways to the gcp transit. We will also peer Azure and GCP and test out the connectivity.

The Spoke VPCs and gateways have already been created to save you time. The process is very similar to deploying the transit gateway.

```{figure} images/lab-3.png
---
height: 400px
align: center
---
Current Topology
```

## Lab 3.1 - Deploy the GCP transit VPC

### Description

Let's go ahead and create the Transit VPC in GCP using the topology builder in Copilot.

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

|                    |             |
| ------------------ | ----------- |
| **Cloud Provider** | GCP         |
| **Account**        | gcp-account |

```{figure} images/lab-3-topology-builder.png
---
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
| **VPC Name**            | gcp-transit         |
| **Subnet Name**         | gcp-transit         |
| **Region**              | us-west1            |
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

In this exercise we are going to launch the Aviatrix Transit Gateway in the newly created Transit VPC in GCP.

### Validate

* Enter the following fields:

|                          |                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------- |
| **Transit Gateway VPC**  | gcp-transit                                                                       |
| **Gateway Name**         | gcp-transit                                                                       |
| **Gateway Size**         | n1-standard-1                                                                     |
| **Instance 1 (Primary)** | 10.101.40.0/23~us-west1~gcp-transit + Allocate New EIP                            |
| **Zone**                 | us-west1-a                                                                        |
| **Transit Peers**        | Click "Select Transit Gateways"" and add the **azure-transit** as a peer and save |

```{figure} images/lab3-transit-config.png
---
align: center
---
Create Aviatrix Transit Gateway
```

```{figure} images/lab3-transit-peering.png
---
align: center
---
Configure transit peering
```

* Hit **Save** to save the transit gateway settings and return to the topology builder overview.
* You now get the option to either generate your changes as Terraform code, or deploy directly from the UI. Choose and click **Deploy**.

```{figure} images/lab3-overview.png
---
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

* Go to **_Cloud Fabric -> Gateways -> Transit Gateways_** and click on **_gcp-transit_**.
* Open the settings pane and enable and save **Connected Transit** under the General settings.

```{figure} images/lab3-connected-transit.png
---
align: center
---
Enable connected transit
```

### Expected Results

Now that we have the gcp-transit deployed, peered to azure-transit and have enabled connected transit, we should see these changes reflected in our topology view. (under **_Cloud Fabric -> Topology_**)

```{figure} images/lab3-result.png
---
align: center
---
Topology view
```

## Lab 3.3 - Attach Spoke Gateways to Aviatrix Transit Gateway

### Description

Now that we have our gcp-transit set up, we need to establish connectivity between the GCP spoke gateways and the gcp-transit gateway. The spoke gateways have been pre-deployed to save some time, but the process is much the same as the transit gateway you have just deployed.

### Validate

* Go to **_Cloud Fabric -> Gateways -> Spoke Gateways_**
* Click the edit button (pencil) behind the **_gcp-prod_** gateway.

```{figure} images/lab3-edit-spoke.png
---
align: center
---
Edit spoke
```

* Change the attachment from empty to **_gcp-transit_** and hit save.

```{figure} images/lab3-attach-spoke.png
---
align: center
---
Attach Spoke Gateway to Transit
```

* Repeat the process for the **_gcp-shared_** spoke gateway.

### Expected Results
The GCP spoke gateways should now be attached to the gcp-transit. Within a few minutes, we should see these changes reflected in our topology view. (under **_Cloud Fabric -> Topology_**)

```{figure} images/lab3-copilot-spoke-attach.png
---
align: center
---
CoPilot Topology with Attached Spokes
```

* If your topology does not show the connection to the spokes, refresh the page after a minute or so.

## Lab 3.4 - Enable network segmentation on the GCP transit

### Description

Just like we enabled network segmentation on the azure-transit before, we want to extend this segmentation to GCP and enable it on the gcp-transit as well.

### Validate

* In the Copilot, go to **_Networking -> Network Segmentation -> Network Domains_** and click the **_Transit Gateways_** button.
* In the transit gateway pane, enable segmentation for **gcp-transit**.

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

For our GCP spokes, we want to create network domains and associations, just like we did for the Azure spokes before.

### Validate

In CoPilot, go to **_Networking -> Network Segmentation -> Network Domains_**

Use the **_+ Network Domains_** button to add the following domains and associations:

| Domain     | Association |
| :--------- | :---------- |
| GCP-Shared | gcp-shared  |
| GCP-Prod   | gcp-prod    |

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

We want the development team in the office, to be able to access GCP-Shared.

### Validate

* Edit the **_Office_** network domain under **_Networking -> Network Segmentation -> Network Domains_** in Copilot.

```{figure} images/lab3-edit-network-domain.png
---
align: center
---
Edit network domain
```

* Now add the **_GCP-Shared_** domain as a connected domain.

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

## Lab 3.7 - Test Connectivity to GCP Spokes

### Description

The GCP Spokes should be connected to the GCP Transit so now we can check connectivity to the spokes from the office.

### Validate

* Open the Remote Access Server and open the **RDP - Client** (or navigate to `https://client.pod[#].aviatrixlab.com`). This is the on-prem Host.
* Open up the `Min` browser on the RDP desktop
* Navigate to `http://localhost`

> Do we have access to gcp-shared and azure-build?

### Expected Results

* As you can see, you now have access to azure-build and gcp-shared.

![Screenshot](images/lab3-office-access.png)  
_Fig. Office Access_

> Were these connections succesful?

### Expected Results

From the office, you should be able to reach gcp-shared, but not gcp-prod.

## Lab 3 Summary

* GCP has been onboarded
* You deployed a VPC and an Aviatrix Transit Gateway in GCP
* You connected all of the Spoke VPCs to the Transit
* You created a Multicloud Transit network, connecting GCP and Azure
* You are able to reach the GCP environment from On-Prem, via the Azure Transit
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
