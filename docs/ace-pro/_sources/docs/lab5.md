# Lab 5 - HPE WITH ACTIVE MESH

## 1. Objective

In this lab, we will demonstrate Active/Active communication between the resources utilizing high performance encryption (*aka `Insane Mode`* - **large number of A/A tunnels**) between Aviatrix Gateways (Spoke-to-Transit & Transit-to-Transit).

## 2. High Performance Encryption and ActiveMesh

Encryption in Cloud could be a compliance, security or business requirement. Hybrid cloud connectivity and in the cloud communication is untrusted. Aviatrix **`HPE`** provides high performance end-end encryption from on-prem (private/public connections) to the cloud walled-gardens, between the regions and clouds. It can also help overcome native constructs routing scalability challenges.

Aviatrix **`ActiveMesh`** technology provides network resiliency, better convergence and high performance. Both Aviatrix gateways in transit and spoke VPC/VNet/VCNs forward traffic simultaneously. It helps enterprises in traffic engineering and provides deterministic next hop based on path-selection algorithm.

## 3. Topology

In this lab we will configure the pending attachment between the Spoke Gateways in aws-us-east-1-spoke1 and the Transit Gateways in aws-us-east-1-transit, and the peering between the Transit Gateways aws-us-east-1-transit and the Transit Gateways in aws-us-east-2-transit. The Gateways in AWS region us-east-1 are pre-configured with HPE (High Performance Encryption, also known as `Insane Mode`) and AWS us-east-2 region was configured in Lab 3.

```{figure} images/lab5-topology.png
---
height: 400px
align: center
---
Lab 5 Topology
```

```{note}
Please keep in mind that the Spoke Gateway in **azure-west-us-spoke2** VPC will still remain unattached in this lab!
```

## 4. High Performance Encryption Configuration
 
### 4.1. CoPilot View before starting

Go to **CoPilot > Cloud Fabric > Topology > Overview**

```{important}
Verify that **AWS US-EAST-1** region has a Transit gateway and a Spoke gateway that are not connected yet.
```

```{figure} images/lab5-topologycopilot.png
---
align: center
---
CoPilot view
```

### 4.2. Transit-Spoke Attachment

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **aws-us-east-1-spoke1** clicking on the pencil icon:

```{figure} images/lab5-editspoke.png
---
align: center
---
Edit Spoke US-East-1
```

Select the Transit Gateway **aws-us-east1-transit** from the drop-down window `"Attach To Transit Gateway"`, and then click on **Save**.

```{figure} images/lab5-editspoke2.png
---
align: center
---
Attachment
```

## 4.3. CoPilot View after Transit-Spoke Attachment

Go to **CoPilot > Cloud Fabric > Topology > Overview**

Verify that both **_aws-us-east-1-transit_** gateway and **_aws-us-east-1-spoke1_**  gateway are now connected. 

```{tip}
Wait a handful of minutes and then refresh the web page, in order to see the changes applied on the topology!
```

```{figure} images/lab5-copilotview.png
---
align: center
---
Attachment on the CoPilot
```

```{important}
Solid line = HPE connection

Dashed line = Non-HPE connection
```

## 4.4. Transit Peerings Configuration

Here you will configure Transit Peering between **aws-us-east-1** and **aws-us-east-2** regions.

Go back to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**

- **aws-us-east-1-transit** to **aws-us-east-2-transit**

Edit the Transit Gateway **aws-us-east-1-transit**, clicking on the pencil icon:

```{figure} images/lab5-peering.png
---
align: center
---
Edit Transit in US-EAST-1
```

Select the Transit Gateway **_aws-us-east-2-transit_** from the drop-down window `"Peer To Transit Gateways"`, and then click on **Save**.

```{figure} images/lab5-peering2.png
---
align: center
---
Peering
```

### 4.4.1. Transit Peerings Verification

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gateway **_aws-us-east-1-transit_**, select the `"Gateway Routes"` tab and check the route **10.0.1.0/24** for instance.

```{note}
It may take a minute or two to reflect here.
```

You will find out that the route 10.0.1.0/24 is reachable through **nine** connections with the **_aws-us-east-2-transit_**.

```{figure} images/lab5-hpe.png
---
align: center
---
HPE in action on the First Transit GW
```

Likewise, the same route is also reachable via other **nine** connections but through the Second Transit Gateway **_aws-us-east-2-transit-1_**:

```{figure} images/lab5-hpe2.png
---
align: center
---
HPE in action on the Second Transit GW
```

```{important}
The number of additional connections depend on the **_size_** of the Aviatrix Gateway.
```

At this point, this is how the overall topology would look like:

```{figure} images/lab5-topologyview.png
---
height: 400px
align: center
---
Logical Topology View
```

This is the topology view from CoPilot at this stage:

```{figure} images/lab5-peeringdrawing.png
---
align: center
---
CoPilot Topology View
```

```{caution}
The actual configuration of **`High Performance Encryption`**  on both the **_aws-us-east-1-transit_** and the **_aws-us-east-1-spoke1_** was done when the gateways were created before this lab.
```

## 5. High Performance Encryption Verification

### 5.1. CoPilot Verification of the VPC Peerings(Transit-Transit and Spoke-Transit)

HPE automatically creates an underlying VPC Peering attachment within AWS. Verify it on the CoPilot.

Go to **CoPilot > Networking > Connectivity > Native Peering**

```{figure} images/lab5-native0.png
---
align: center
---
Native Peerings
```

Click on any VPC peerings to expand its properties on the right side.

```{figure} images/lab5-native.png
---
align: center
---
Native Peerings Properties
```

## 5.2. CoPilot Verification of HPE

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gatewasy **_aws-us-east-1-transit_**, select the `"Interfaces"` tab and check the huge number of tunnel interfaces that HPE has instantiated. 

These tunnels are used with the Spoke Gateway aws-us-east-1-spoke1 and the Transit Gateway aws-us-east-2-transit, because HPE is also enabled on these gateways:

```{figure} images/lab5-ipip.png
---
align: center
---
Interface Stats
```

## 6. ActiveMesh

### 6.1. CoPilot Verification of ActiveMesh

Go to **CoPilot > Diagnostics > Cloud Routes > VPC/VNet Routes**

Click the filter button, select `“Name”` and enter **_aws-us-east-1-spoke1-Public-1-us-east-1a-rtb_** to filter by just that route table.

```{note}
The RFC 1918 summary routes points to the Aviatrix Spoke gateway for this routing table programmed by the Aviatrix Controller:
```

```{figure} images/lab5-filter.png
---
align: center
---
Filter
```

```{figure} images/lab5-summary.png
---
align: center
---
RFC1918 routes pointing towards the First Spoke GW
```

Now remove the previous filter and select **_aws-us-east-1-spoke1-Public-2-us-east-1b-rtb_**. 

```{note}
This time the RFC 1918 summary routes points to the Aviatrix **Second** Spoke gateway for this routing table programmed by once again the Aviatrix Controller:
```

```{figure} images/lab5-filter2.png
---
align: center
---
Filter
```

```{figure} images/lab5-summary2.png
---
align: center
---
RFC1918 routes pointing towards the Second Spoke GW
```

As you can see, Active/Active is achieved within a VPC as well. Each gateway is active on the Availability Zone where it resides.
 
## 6.2. Connectivity test of ActiveMesh (Pt.1)

Test that the EC2 instances in two subnets are pointing to two different routing tables. If one gateway goes down, the controller will switch the **ENI** of the available gateway in the routing table.

SSH into **both** EC2 test instances in **_aws-us-east-1-spoke1_** VPC (refer to your Pod assignment). 

These test instances are in separate AZs and their default gateways are two different Aviatrix Spoke gateways in their respective AZs.

Ping the EC2 test instance (10.0.1.100) in aws-us-east-2-spoke1 VPC. 

```{figure} images/lab5-new8.png
---
align: center
---
from US-EAST-1 to US-EAST-2
```

It will fail. **WHY?** Because we didn’t enable segmentation on **_aws-us-east-1-transit_** and associate **_aws-us-east-1-spoke1_** with the transit gateway in the appropriate network domain.
 
### 6.2.1 Enable Segmentation

Go to **CoPilot > Networking > Network Segmentation > Network Domains > Transit Gateways**

Enable Segmentation on **_aws-us-east-1-transit_**:

```{figure} images/lab5-enable.png
---
align: center
---
Enable Segmentation
```

### 6.2.2. Associate Aviatrix Spoke to the Network Domain

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Associate **_aws-us-east-1-spoke1_** with its transit in the <span style='color:lightgreen'>Green</span> network domain:

```{figure} images/lab5-green.png
---
align: center
---
Association
```

At this point, this is how the overall topology would look like:

```{figure} images/lab5-topologyview22.png
---
height: 400px
align: center
---
New Logical Topology View
```

## 6.3. Connectivity test of ActiveMesh (Pt.2)

Now from to the **_aws-us-east-1-spoke1-test1_** in AWS US-East1, launch the ping command towards the **_aws-us-east-2-spoke1-test1_** in AWS US-**East2**.

```{figure} images/lab5-new.png
---
align: center
---
First ping 
```

```{figure} images/lab5-ping1.png
---
align: center
---
ping from aws-us-east-1-spoke1-test1
```

Repeat the ping from the **_aws-us-east-1-spoke1-test2_** in AWS US-East1 towards **_aws-us-east-2-spoke1-test1_** in AWS US-**East2**.

```{figure} images/lab5-new2.png
---
align: center
---
second ping
```

```{figure} images/lab5-ping2.png
---
align: center
---
ping from aws-us-east-1-spoke1-test2
```

```{important}
Please keep **both** the ping sessions running recursively on your SSH client! Don't interrupt the ping.
```

To demonstrate ActiveMesh capability, you will shut down temporarily one of the spoke gateways and notice traffic converging to the other gateway.

Login to **AWS console**</a>. Refer to your pod info for login information (<ins>this screenshot is for Pod 149</ins>).

```{figure} images/lab5-newone.png
---
align: center
---
AWS URL and credentials
```

```{figure} images/lab5-awsconsole.png
---
align: center
---
AWS console
```

Change the region to **N. Virginia** and invoke **EC2** service.

```{figure} images/lab5-region.png
---
align: center
---
Change the region
```

Click on **Instances (running)**:

```{figure} images/lab5-running.png
---
align: center
---
Instances running
```

Search for **_aviatrix-aws-us-east-1-spoke1_** and then select **Instance state > Stop instance**

```{figure} images/lab5-stop.png
---
align: center
---
Stop the Instance
```

Confirm by clicking on **Stop** one more time.

```{figure} images/lab5-stop2.png
---
align: center
---
Confirm the stop
```

You will notice ping drops solely from **_aws-us-east-1-spoke1-test1_**. The traffic will reconverge to the spoke gateway in the other AZ, in about <ins>1 minute and 30 seconds to 2 minutes</ins>.

This shows how the Aviatrix Controller intelligently auto-heals the VPC routing.

```{figure} images/lab5-drop.png
---
align: center
---
Temporary disruption with FAST keepalive!
```

**Bonus Step**: `Restart` the Gateway from the AWS console and reverify the traffic flow. This time you will not notice any disruption: the traffic flow fill switch back to the aviatrix-aws-us-east-1-spoke1 GW.

```{figure} images/lab5-restart.png
---
align: center
---
Restart
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab5-finaltopo.png
---
height: 400px
align: center
---
Final Topology for Lab 5
```

## 7. FlightPath

 Go to **CoPilot > Diagnostics > AppIQ > FlightPath**

Use the following inputs:

- **Source**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Destination**: <span style='color:#479608'>aws-us-east-2-spoke1-test1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>443</span>
- **Interface**: <span style='color:#479608'>Private</span>

```{figure} images/lab5-flightpath.png
---
align: center
---
FlightPath config
```

This will provide an AppIQ report of how **_aws-us-east-1-spoke1-test1_** is connected with **_aws-us-east-2-spoke1-test1_** and display the path along with end-to-end latency.

```{note}
You might see some links still depicted in <span style='color:red'>**red**</span>, after having restarted the Spoke Gateway. Be patient and relaunch the report and you will get the same outcome as depicted below.
```

```{figure} images/lab5-flight2.png
---
align: center
---
FlightPath Report
```

Scroll down to get more details about:

- The latency between each pair of gateways
- Performance monitoring metrics of all gateways in the path
- FlowIQ between the two instances
- Security group checks
- NACL checks
- Routing tables 

You can also download the entire report in pdf format by clicking the **PDF icon** at the top right corner:

```{figure} images/lab5-download.png
---
align: center
---
FlightPath Report PDF
```

## Bonus questions
### Gateway Keepalive Templates

Experiment with <a href="https://docs.aviatrix.com/previous/documentation/latest/building-your-network/gateway-keepalives.html" target="_blank">GATEWAY TO CONTROLLER COMMUNICATION</a> and retest convergence times when bringing down a spoke gateway.

```{tip}
You can modify the <ins>Gateway to Controller Communication timer</ins> directly from the CoPilot. Go to **CoPilot > Cloud Fabric > Gateways > Settings > Gateway to Controller Communication** and change the `Keep Alive Speed` from **_medium_** (default) to **_fast_**. Then repeat the experiment carried out earlier.
```

```{figure} images/lab5-keepalive.png
---
align: center
---
Keep Alive Speed
```

### Transitive Routing

```{warning}
The test instances in **_aws-us-east-1-spoke1_** are not able to communicate with the test instances in GCP or in Azure.
```

You can verify this with the Gateway routing table on the **CoPilot > Diagnostics > Cloud Routes > Gateway Routes > aws-us-east-1-transit**. You will not see the GCP Spoke routes of **_172.16.1.0/24_**., for instance.

- Why is that?
What would be needed to make those routes visible?