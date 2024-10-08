# Lab - Hints

## 1 - Initial Topology

This is the current topology of a multicloud enterprise that has Data Center in **US-East-1** and utilizes Equinix fabric as a Co-location facility in **US-West-1** and **EU** region.

MPLS is heavily used for any East-West traffic between these regions across multicloud environment.

```{figure} images/cbhc-origtopology.jpeg
---
height: 400px
align: center
---
Initial Topology
```

### 1.1 - Quick Knowledge Quiz

<span style='color:#479608'>Q1.</span> Which native cloud services providers resources are used to onboard on-premise resources?

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

## 2 - Dynamic Topology

Please login to Aviatrix Copilot and navigate to topology page by leveraging search bar as shown below.

```{figure} images/copilot_topology_search.png
---
align: center
---
Using the Search Bar
```

The following topology will appear on your Copilot

```{tip}
Click on the "Managed" button for only showing the managed VPCs

```{figure} images/managed-vpc.png
---
align: center
---
Managed VPCs
```

```{figure} images/topology.png
---
align: center
---
Cloud Backbone Topology
```

### 2.1 - Quick Knowledge Quiz

<span style='color:#479608'>Q2.</span> Where is the Aviatrix Edge deployed?

```{hint}
Check **CoPilot > Cloud Fabric > Edge**
```

<span style='color:#479608'>Q3.</span> To which Aviatrix cloud resources Aviatrix Edge is associated with?

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

## 3 - Networking & Security Infrastructure

### 3.1 - Transit VPC/VNET/VCN

Let's verify:

**Cloud Resources > Cloud Assets > VPC/VNets Subnets**

```{figure} images/cloudassets.png
---
align: center
---
Cloud Backbone Transit VPC/VNET/VCN
```

### 3.2 - Transit Gateways

Let's verify Aviatrix Transit Gateways and their peerings.

**Cloud Fabric > Gateways > Transit Gateways** then click on any Gateways Clusters and expand the "Connections" Tab!

```{figure} images/transitgws.png
---
align: center
---
Transit Gateways
```

```{figure} images/transitgws2.png
---
align: center
---
Transit Gateways Peerings
```

### 3.3 Connectivity between Aviatrix Cloud Backbone & Native CSP Constructs

Verify connectivity between Aviatrix Cloud Backbone and native CSP constructs.

**Networking > Connectivity > External Connections (S2C)**

You will notice different tunnel types due to CSP limitations.

```{figure} images/natives2c.png
---
align: center
---
Connectivity with Native CSP Constructs
```

### 3.4 - Quick Knowledge Quiz

<span style='color:#479608'>Q4.</span> How AWS, Azure and GCP transits are peered with each other?

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

<span style='color:#479608'>Q5.</span> Was Aviatrix Cloud Backbone established using Standard or High Performance Encryption?

```{hint}
Check **CoPilot > Cloud Fabric > Topology > Legend**
```

## 4 - Traffic Routing

Navigate to the Cloud Routes menu within Copilot by leveraging the search bar as shown below.

```{figure} images/copilot_cloudroutes_search.png
---
align: center
---
Search Bar - Cloud Routes
```

The following screen will appear:

```{figure} images/gatewayroutes.png
---
align: center
---
Cloud Routes
```

Verify Backbone components are receiving the routes across Multicloud environment including Equinix.

### 4.1 - Quick Knowledge Quiz

<span style='color:#479608'>Q6.</span> What routes GCP Transit GW is receiving from the Equinix Fabric?

```{hint}
Check **CoPilot > Diagnostics > Cloud Routes > Gateway Routes** and loot at the *backbone-gcp*'s RTB
```

## 5 - Data Plane Verification

Since this is a brownfield environment where this customer just started using  Aviatrix Cloud Backbone for East-West communication and doesn't have Aviatrix Spoke Gateways in any application VPCS/VNETS/VCNS, you won't be able to find the application EC2/VM on the dynamic topology map. 

To find the application EC2/VM <ins>Public IP address</ins>, go to:

**Cloud Assets > Virtual Machines**

```{figure} images/vm.png
---
align: center
---
Virtual Machines
```

SSH from your laptop to the instance **_aws-instance_**, using your favourite SSH client:

Alternatively, you can use the following command: 

```bash
ssh student@aws.pod6.aviatrixlab.com
```

```{note}
**username**: *student*

**password**: *1056ES!73qJfR9*
```

Ping GCP VM's private IP from the AWS instance to test the backbone:

```{figure} images/ping.png
---
align: center
---
Ping Verification
```

Access all the application instances/VMs using your laptop's browser using the following links:

http://aws.pod6.aviatrixlab.com/

http://azure.pod6.aviatrixlab.com/

http://gcp.pod6.aviatrixlab.com/

```{figure} images/browser.png
---
align: center
---
Browser Verification
```

<span style='color:#479608'>Q7.</span> For Data Plane Verification, how to find the public and private IP address of Instances deployed in CSP's?

## 6 - Troubleshooting

### 6.1 - Available Paths

Verify how many paths are available for an application to communicate between AWS and GCP, which one is the best path and why?

<span style='color:#479608'>Q8.</span> Visually from the dynamic topology map in Copilot, verify how many paths are available for an application to communicate between AWS and GCP?

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

#### 6.1.1 **Diagnostics > AppIQ > FlightPath**

```{figure} images/fp1.png
---
align: center
---
AppIQ
```

```{figure} images/fp2.png
---
align: center
---
FlightPath
```

#### 6.1.2 Latency Monitor

**Monitor > Traffic & Latencies > Gateways**

```{figure} images/latency.png
---
align: center
---
Latency Monitor
```

<span style='color:#479608'>Q9.</span> Which Path is best for an application to communicate between AWS and GCP?

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

## 7 - Conclusion

In this lab, we have successfully verified and troubleshooted Aviatrix Cloud Backbone, including Equinix colocation.
