# Lab 4 - HPE WITH ACTIVE MESH

## 1. Objective

In this lab, we will demonstrate Active/Active communication between resources using **High-Performance Encryption** (also known as `Insane Mode` — which involves a **large number of Active/Active tunnels**) between Aviatrix Gateways.

## 2. High Performance Encryption and ActiveMesh

Encryption in the cloud can be a compliance, security, or business requirement. Hybrid cloud connectivity and in-cloud communication are considered untrusted. Aviatrix **`HPE`** provides high-performance encryption from on-premises (private/public connections) to cloud walled gardens, as well as between regions and clouds. It also helps address routing scalability challenges associated with native constructs.

Aviatrix **`ActiveMesh`** technology provides network resiliency, better convergence and high performance. Both Aviatrix gateways in transit and spoke VPC/VNet/VCNs forward traffic simultaneously. It helps enterprises in traffic engineering and provides deterministic next hop based on path-selection algorithm.

## 3. Topology

In this lab we will configure the pending attachment between the Spoke Gateways in _aws-us-east-1-spoke1_ and the Transit Gateway in _aws-us-east-1-transit_, and the peering between the Transit Gateway _aws-us-east-1-transit_ and the Transit Gateway in _aws-us-east-2-transit_. The Gateway in AWS region us-east-1 is pre-configured with HPE (High Performance Encryption, also known as `Insane Mode`) and AWS us-east-2 region was configured in Lab 3.

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

### 4.1 CoPilot View before starting

Go to **CoPilot > Cloud Fabric > Topology > Overview**

```{important}
Verify that **AWS US-EAST-1** region has a Transit gateway and a Spoke gateway that are not connected yet.
```

```{figure} images/lab5-topologycopilot.png
---
height: 400px
align: center
---
CoPilot view
```

### 4.2 Transit-Spoke Attachment

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **aws-us-east-1-spoke1** clicking on the pencil icon:

```{figure} images/lab5-editspoke.png
---
height: 200px
align: center
---
Edit Spoke US-East-1
```

Select the Transit Gateway **aws-us-east-1-transit** from the drop-down window `"Attach To Transit Gateway"`, and then click on **Save**.

```{figure} images/lab5-editspoke2.png
---
align: center
---
Attachment
```

### 4.3 CoPilot View after Transit-Spoke Attachment

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
**Solid** line = HPE connection

**Dashed** line = Non-HPE connection
```

## 5. Transit Peerings Configuration

Here you will configure Transit Peering between **aws-us-east-1** and **aws-us-east-2** regions.

Go back to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**

- **aws-us-east-1-transit** to **aws-us-east-2-transit**

Edit the Transit Gateway **aws-us-east-1-transit**, clicking on the pencil icon:

```{figure} images/lab5-peering.png
---
height: 200px
align: center
---
Edit Transit in US-EAST-1
```

Select the Transit Gateway **_aws-us-east-2-transit_** from the drop-down window `"Attach To Transit Gateways"`, and then click on **Save**.

```{figure} images/lab5-peering2.png
---
align: center
---
Peering
```

### 5.1 Transit Peerings Verification

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gateway **_aws-us-east-1-transit_**, select the `"Gateway Routes"` tab and check the route **10.0.1.0/24** for instance.

```{note}
It may take a minute or two to reflect here.
```

You will find out that the route 10.0.1.0/24 is reachable through **nine** connections with the **_aws-us-east-2-transit_**.

```{figure} images/lab5-hpe.png
---
align: center
---
HPE in action 
```

```{important}
The number of additional connections depends on the **_size_** of the Aviatrix Gateway.
```

At this point, this is how the overall topology will look like:

```{figure} images/lab5-topologyview.png
---
height: 400px
align: center
---
Logical Topology View
```

This is the current topology view from CoPilot:

```{figure} images/lab5-peeringdrawing.png
---
align: center
---
CoPilot Topology View
```

```{caution}
The actual configuration of **`High Performance Encryption`**  on both the **_aws-us-east-1-transit_** and the **_aws-us-east-1-spoke1_** was done when the gateways were created before this lab.
```

## 6. High Performance Encryption Verification

### 6.1 CoPilot Verification of the VPC Peerings(Transit-Transit and Spoke-Transit)

HPE automatically creates an underlying **VPC Peering attachment** within AWS. Verify it on the CoPilot.

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
height: 250px
align: center
---
Native Peerings Properties
```

### 6.2 CoPilot Verification of HPE

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the Transit Gateway **_aws-us-east-1-transit_**, select the `"Interfaces"` tab and check the huge number of tunnel interfaces that HPE has instantiated. 

These tunnels are used with the Spoke Gateway _aws-us-east-1-spoke1_ and the Transit Gateway _aws-us-east-2-transit_, because HPE is also enabled on these gateways:

```{figure} images/lab5-ipip.png
---
align: center
---
Interface Stats
```

## 7. ActiveMesh

### 7.1 CoPilot Verification of ActiveMesh

Go to **CoPilot > Diagnostics > Cloud Routes > VPC/VNet Routes**

Click the filter button, select `“Name”` and enter **_aws-us-east-1-spoke1-Public-1-us-east-1a-rtb_** to filter by just that route table.

```{note}
The **RFC 1918** summary routes points to the Aviatrix Spoke gateway for this routing table programmed by the Aviatrix Controller:
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
This time, the RFC 1918 summary routes point to the Aviatrix **Second** Spoke Gateway for this routing table, programmed once again by the Aviatrix Controller:
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

As you can see, Active/Active configuration is also achieved within a VPC. Each gateway is active in the Availability Zone where it resides.

```{important}
Don't forget to remove the previously applied filter!
```{figure} images/lab4-removefilter.png
---
height: 250px
align: center
---
Remove the filter
```

## 8. Connectivity Testing of ActiveMesh (Pt.1)

Verify that both the **EAST-1** region and **EAST-2** region in AWS have connectivity after applying the attachments.

### 8.1 Connectivity Testing Using the Gatus App

First and foremost, navigate to your POD Portal, locate the `Gatus widget`, and select both **_aws-us-east-1-spoke1-test1_** and **_aws-us-east-1-spoke1-test2_**.

Insert the credentials available on your POD Portal and then click on **"Sign in"**.

```{figure} images/lab4-gatus200.png
---
height: 400px
align: center
---
Istances
```

```{figure} images/lab4-gatusnew00.png
---
height: 400px
align: center
---
Istances
```

- Now check if these two instances in the **_aws-us-east-1-spoke1_** VPC have `ICMP` reachability to the instance **_aws-us-east-2-spoke1-test1_** in the **US-EAST-2** region.

```{figure} images/lab4-gatus202.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test1
```

```{figure} images/lab4-gatus201.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test2
```

From the outcome above, you can conclude that the <ins>connectivity is broken</ins>!

### 8.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you wish, you can also check the ICMP test using your SSH client.

- SSH into **both** EC2 test instances in the **_aws-us-east-1-spoke1_** VPC (refer to your POD assignment).

Ping the EC2 test instance (10.0.1.100) in **_aws-us-east-1-spoke1_** VPC.

```{figure} images/lab5-new8.png
---
align: center
---
From US-EAST-1 to US-EAST-2
```

```{figure} images/lab4-gatus203.png
---
height: 300px
align: center
---
Ping fails from aws-us-east-1-spoke1-test1
```

```{figure} images/lab4-gatus204.png
---
height: 300px
align: center
---
Ping fails from aws-us-east-1-spoke1-test2
```

```{caution}
It will fail. Why? Because we didn’t enable segmentation on **aws-us-east-1-transit** and associate **aws-us-east-1-spoke1** with the transit gateway in the correct network domain.
```

## 9. Enable Segmentation

Go to **CoPilot > Networking > Network Segmentation > Network Domains > Transit Gateways**

Enable Segmentation on **_aws-us-east-1-transit_**:

```{figure} images/lab5-enable.png
---
align: center
---
Enable Segmentation
```

### 9.2 Associate Aviatrix Spoke to the Network Domain

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Associate **_aws-us-east-1-spoke1_** with its transit in the <span style='color:lightgreen'>Green</span> network domain:

```{figure} images/lab5-green.png
---
align: center
---
Association
```

At this point, the overall topology will be as follows:

```{figure} images/lab5-topologyview22.png
---
height: 400px
align: center
---
New Logical Topology View
```

## 10. Connectivity Testing of ActiveMesh (Pt.2)

Verify that both **EAST-1** region and **EAST-2** region have perfect connectivity.

### 10.1 Connectivity Testing Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select both **_aws-us-east-1-spoke1-test1_** and **_aws-us-east-1-spoke1-test2_**. 

- Verify if these two instances in the **_aws-us-east-1-spoke1_** VPC have `ICMP` reachability to the instance **_aws-us-east-2-spoke1-test1_** in the **US-EAST-2** region.

```{figure} images/lab4-gatus301.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test1
```

```{figure} images/lab4-gatus302.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test2
```

Now the connectivity to the **EAST2** region is fine, thanks to the Network Segmentation!

### 10.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Launch your SSH session to the **_aws-us-east-1-spoke1-test1_** in AWS US-East 1, and then carry out the ping command to the **_aws-us-east-2-spoke1-test1_** in AWS US-East 2.

```{figure} images/lab5-new.png
---
align: center
---
Instances are now in the same Segment! 
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
Second ping
```

```{figure} images/lab5-ping2.png
---
align: center
---
ping from aws-us-east-1-spoke1-test2
```

```{important}
Please keep **both** ping sessions running continuously in your SSH client! Do not interrupt the pings.
```

## 11. Failover

Test that the EC2 instances in the two subnets deployed in the same VPC, **_aws-us-east-1-spoke1_**, are pointing to two different routing tables. If one gateway goes down, the controller will switch the **ENI** (_Elastic Network interface_) to the available gateway in the routing table.

To demonstrate the `Active Mesh` capability, you will shut down _temporarily_ one of the spoke gateways and notice traffic converging to the other gateway.

Before proceeding with the actions that you need to carry out on the AWS console, let's turn off the `Gateway Single AZ HA` functionality.

- Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east-1-spoke1_** cluster! Now select the **Settings** tab, then expand the **General** section and last but not least, turn off the _Gateway Single AZ HA_ knob! Do not forget to click on **Save**.

```{figure} images/lab5-activemeshha.png
---
align: center
---
Disable "Gateway Single AZ HA
"
```

```{caution}
The `Gateway Single AZ HA` feature enables the **Aviatrix Controller** to monitor the health of the gateway instance and restart the gateway instance if it becomes unreachable. 

<ins>Gateway Single AZ HA is enabled by default</ins>.

When Gateway Single AZ HA status is **On**, the Aviatrix Controller attempts to restart the gateway instance. When status is **Off**, Controller does **NOT** attempt to restart the gateway instance.
```

### 11.1 Connectivity Testing Using the Gatus App

Login to **AWS console**</a>. Refer to your pod info for login information.

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
height: 300px
align: center
---
Change the Region
```

```{figure} images/lab5-region00.png
---
height: 300px
align: center
---
Invoke the EC2 service
```

Click on **Instances (running)**.

```{figure} images/lab5-running.png
---
align: center
---
Instances running
```

Search for **_aviatrix-aws-us-east-1-spoke1_**, select the instance and then choose **Instance state > Stop instance**

```{figure} images/lab5-stop.png
---
height: 200px
align: center
---
Stop the Instance
```

Confirm by clicking on **Stop**, one more time.

```{figure} images/lab5-stop2.png
---
align: center
---
Confirm the stop
```

Now, check the `Gatus Dashboard` for the instance **_aws-us-east-1-spoke1-test1_** from your personal POD portal:

```{tip}
Diminish the ICMP timeout to **10** seconds to see the ICMP results that will be unsuccessful in real time.
```

```{figure} images/lab4-gatus401.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test1
```

```{tip}
The **Health Check** mechanism will be successful after approximately 1 minute and 30 seconds to 2 minutes. Please be patient!
```

```{figure} images/lab4-gatus402.png
---
height: 400px
align: center
---
Reconvergence in action
```

The ICMP test initiated from the **_aws-us-east-1-spoke1-test2_** instance will be successful because Spoke Gateway 2 is still operational, and traffic is continuously flowing through it.

```{figure} images/lab4-gatus405.png
---
height: 400px
align: center
---
aws-us-east-1-spoke1-test1
```

- Now restart the **Spoke Gateway #1**.

Go to the AWS Console, select the **_aviatrix-aws-us-east-1-spoke1_** Spoke Gateway, then click on the "Instance state" button and choose `"Start instance"`.

```{figure} images/lab4-gatus406.png
---
height: 200px
align: center
---
AWS Console
```

The traffic will be diverted back to Spoke Gateway **#1**; however, you will not experience any packet loss on the Gatus Dashboard.

The <ins>preemption</ins> is seamless!

```{figure} images/lab4-gatus420.png
---
height: 400px
align: center
---
Gatus from aws-us-east-1-spoke1-test1
```

### 11.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you wish, you can also verify the failover mechanism using the **SSH** Client.

Go back to your **AWS console**</a>. Refer to your pod info for login information.

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
height: 300px
align: center
---
Change the Region
```

```{figure} images/lab5-region00.png
---
height: 300px
align: center
---
Invoke the EC2 service
```

Click on **Instances (running)**.

```{figure} images/lab5-running.png
---
align: center
---
Instances running
```

Search for **_aviatrix-aws-us-east-1-spoke1_**, select the instance and then choose **Instance state > Stop instance**

```{figure} images/lab5-stop.png
---
height: 200px
align: center
---
Stop the Instance
```

Confirm by clicking on **Stop**, one more time.

```{figure} images/lab5-stop2.png
---
align: center
---
Confirm the stop
```

You will observe ping drops exclusively from **_aws-us-east-1-spoke1-test1_**. The traffic will reconverge to the spoke gateway in the other availability zone within approximately <ins>1 minute and 30 seconds to 2 minutes</ins>.

This shows how the **Aviatrix Controller** intelligently auto-heals the VPC routing, `reprogramming` the Routing Table of the VPC Router!

```{figure} images/lab5-drop.png
---
align: center
---
Temporary disruption with **FAST** keepalive!
```

```{figure} images/lab5-drop00.png
---
align: center
---
No Packets drop from the aws-us-east-1-spoke1-test2
```

- `Restart` the Gateway from the AWS console and reverify the traffic flow. This time you will notice any kind of disruption: the traffic flow fill switch back to the **aviatrix-aws-us-east-1-spoke1** GW.

```{figure} images/lab5-restart.png
---
height: 200px
align: center
---
Restart
```

Upon completing this lab, the overall topology will be as follows:

```{figure} images/lab5-finaltopo.png
---
height: 400px
align: center
---
Final Topology for Lab 4
```

## 12. Reprogramming of the VPC Router's Routing Table <span style='color:#33ECFF'>(BONUS)</span></summary>

If you want to see how the **Aviatrix Controller** reprograms the routing of VPC Route0 behind the scenes, you can redo the previous test by following these steps:


    Return to the AWS Console, invoke the EC2 service, and ensure that you are still in the us-east-1 region (i.e., N. Virginia).

```{figure} images/lab5-reprogram00.png
---
height: 400px
align: center
---
EC2 and N. Virginia
```

Now, scroll down through the navigation panel on the left-hand side and click on the **Network Interfaces** section!

```{figure} images/lab5-reprogram01.png
---
height: 400px
align: center
---
Network Interfaces
```

Click on the arrow icon to arrange the items in chronological order, and then identify the eth0 interface for both Spoke Gateway 1 and Spoke Gateway 2 (i.e., '-1').

```{figure} images/lab5-reprogram02.png
---
height: 400px
align: center
---
ENI
```

## 13. FlightPath

 Go to **CoPilot > Diagnostics > AppIQ > FlightPath**

Use the following inputs:

- **Source**: <span style='color:#479608'>aws-us-east-1-spoke1-test1</span>
- **Destination**: <span style='color:#479608'>aws-us-east-2-spoke1-test1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>443</span>
- **Interface**: <span style='color:#479608'>Private</span>

```{figure} images/lab5-flightpath.png
---
height: 250px
align: center
---
FlightPath config
```

This will provide a `FlightPath` report showing how **_aws-us-east-1-spoke1-test1_** is connected with **_aws-us-east-2-spoke1-test1_**, displaying the path along with end-to-end latency.

```{note}
After restarting the Spoke Gateway, you may notice some links still displayed in <span style='color:red'>**red**</span>. Please be patient and relaunch the report to see the same outcome as shown below.
```

```{figure} images/lab5-flight2.png
---
height: 400px
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

## 14. Final Tasks

### 14.1 Gateway Keepalive Templates

Experiment with <a href="https://docs.aviatrix.com/previous/documentation/latest/building-your-network/gateway-keepalives.html" target="_blank">GATEWAY TO CONTROLLER COMMUNICATION</a> and retest convergence times when bringing down a spoke gateway.

```{tip}
You can modify the <ins>Gateway to Controller Communication timer</ins> directly from the CoPilot. Go to **CoPilot > Cloud Fabric > Gateways > Settings > Gateway to Controller Communication** and change the `Keep Alive Speed` from **_medium_** (default) to **_fast_**. 

Then <ins>repeat</ins> the experiment carried out earlier!
```

```{figure} images/lab5-keepalive.png
---
height: 250px
align: center
---
Keep Alive Speed
```

### 14.2 Transitive Routing Question

```{warning}
The test instances in **_aws-us-east-1-spoke1_** are not able to communicate with the test instances in GCP or in Azure.
```

You can verify this in the Gateway Routing Table under **CoPilot > Diagnostics > Cloud Routes > Gateway Routes > aws-us-east-1-transit**. You will not see the GCP Spoke route of **_172.16.1.0/24_**, nor will you see any routes from Azure.

```{figure} images/lab5-keepalive100.png
---
height: 250px
align: center
---
aws-us-east-1-transit's rtb
```

- Why is that?
What would be needed to make those routes visible?

<details>
  <summary>Click here for a logical explanation: <span style='color:#33ECFF'>Hint!</span></summary>

- Only AWS US East <span style='color:red'>**2**</span> Transit, GCP US Central 1 Transit, and Azure West US Transit are connected in a **FULL-MESH** configuration.

```{important}
**FULL-MESH**:
A type of networking design in which each node in the system has a circuit that connects it to every other node. While full mesh does make multiple redundant connections, this design keeps traffic going even if one node fails.

Full-mesh design is useful in systems which are intransitive: A connects to B and B connects to C, but A cannot interact with C.
```

```{figure} images/lab4-fullmesh01.png
---
height: 400px
align: center
---
Full-mesh: physical
```

```{figure} images/lab4-fullmesh02.png
---
height: 400px
align: center
---
Full-mesh: logical
```

The Gateway in the **AWS US-EAST-1** region is exclusively peering with the Gateway in **AWS US-EAST-2**.

```{figure} images/lab4-fullmesh03.png
---
height: 400px
align: center
---
Single peering
```

The Aviatrix Controller will not install a route to a Transit Gateway that is not directly peering with the originating Transit Gateway. Therefore, each Transit Gateway must learn about networks from its originating Transit Gateway.

To avoid the limitations of full-mesh peering, which is not very scalable, you can utilize the [multi-tier transit](https://docs.aviatrix.com/documentation/latest/getting-started/platform-overview/aviatrix-glossary.html#multi-tier-transit) feature.

```{figure} images/lab4-fullmesh04.png
---
height: 400px
align: center
---
Intransitive behaviour without Full-Mesh config.
```

```{caution}
The Multi-Tier feature will be activated on **Lab 11**.
```

</details>
