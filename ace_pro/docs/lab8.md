# Lab 8 - SECURE HIGH-PERFORMANCE DATACENTER EDGE
This lab will demonstrate the functionalities of the `Edge Gateway`...

```{important}
Edge Gateway = An Aviatrix gateway that enables connectivity to edge locations such as data centers, co-locations, remote sites, provider locations, branch offices, and retail stores.
```

## 1. General Objectives

You are tasked with connecting the on-premises data center in New York to your Aviatrix **CNSF**. An **Aviatrix Edge** device has already been provisioned and registered with the existing **Aviatrix Controller**.

The Aviatrix Secure High-Performance Datacenter Edge solution empowers network administrators to provide seamless cloud connectivity without compromise. This robust solution offers encrypted, line-rate performance and supports single-region, multi-region, or multi-cloud redundancy options. Additionally, it ensures comprehensive visibility and troubleshooting capabilities throughout the entire network.

## 2. Initial set-up

Now let's connect the `Aviatrix Edge` to the existing **Cloud Native Security Fabric (CNSF)**. 

First and foremost let's explore the **BGP Map** that describes the connectivity established through the BGPoverLAN.

Navigate to **CoPilot > Diagnostics > Cloud Routes > BGP info**. Click on the three dots icon and select the `"Show BGP Map"` option.

```{figure} images/lab8-edge3.png
---
height: 200px
align: center
---
CoPilot BGP Map
```

You can notice both the AS numbers of each side of the connection and the **/30** subnet used in the underlay.

```{figure} images/lab8-edge4.png
---
align: center
---
BGPoverLAN inside the On-Prem DC
```

Close the BGP Map, then click the 'three-dot' icon again and select `"Show BGP Learned Routes"`.

```{figure} images/lab8-edge5.png
---
align: center
---
Show BGP Learned Routes
```

The LAN router is advertising **225** routes to the Aviatrix Edge.

```{figure} images/lab8-edge6.png
---
align: center
---
225 Routes
```

If you also review the `"Show BGP Advertised Routes"` output, you'll notice that Aviatrix Edge is not advertising any routes, as it is not yet connected to the CNSF.

```{figure} images/lab8-edge7.png
---
align: center
---
No routes advertised by the Edge yet
```

### 2.1. Attachment between Edge and the Transit

Let's establish a peering connection between the Aviatrix Edge device and the Transit Gateway in **US-EAST-2**. 

In the topology below, you'll see a workstation named `Edge` connected to the LAN router. Once the connection is established, you can initiate a ping from this client to verify connectivity.

```{figure} images/lab8-edge8.png
---
align: center
---
Peerings not established yet!
```

First and foremost, you have to configure a **BGP ASN** on the **_aws-us-east-2-transit_** GW!

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_**.

```{figure} images/lab8-edge12.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section and insert the AS number **64513** on the empty field related to the `“Local AS Number”`, then click on **Save**.

```{figure} images/lab8-edge13.png
---
align: center
---
BGP ASN
```

Now it's time to establish the attachment! 

Navigate to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Gateway Attachments"` button, on the right-hand side of the screen.

```{figure} images/lab8-edge922.png
---
height: 200px
align: center
---
Manage Gateway Attachment
```

Click on the `"+Attachment"` button.

```{figure} images/lab8-edge10.png
---
align: center
---
Transit Gateway Attachment
```

Fill in the attachment template using the following settings:

- **Transit Gateway**: <span style='color:#479608'>aws-us-east-2-transit</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>
- **Number of Tunnels**: <span style='color:#479608'>**Custom:**</span>
  - <span style='color:#479608'>**4**</span>


Do not forget to click on **Save**.

```{figure} images/lab8-edgeveryverynew02.png
---
align: center
---
Manage Gateway Attachment
```

```{caution}
Please make sure to select `"Attach Over: Public Network"`. 

If you accidentally choose "_Attach Over: Private Network_", the connection will not be established!
```{figure} images/lab8-edgeveryverynew.png
---
align: center
---
Attach over PUBLIC Network
```

Please wait a few moments for the Aviatrix Controller to establish the attachment. Once completed, a confirmation message will appear, indicating that the operation was successful!

```{figure} images/lab8-edge14.png
---
align: center
---
Peering created
```

Let's verify the presence of the previously created attachment in the topology:

Navigate to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

```{figure} images/lab8-edge15.png
---
align: center
---
New attachment
```

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** cluster.

```{figure} images/lab8-edge16.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Attachments"` tab and then click on `"Transit-Spoke Peering"`. 

You will observe an attachment established between the Transit Gateway in the cloud and the **Edge** device operating in the data center!

```{figure} images/lab8-edge17.png
---
align: center
---
Transit-Spoke Edge Peering
```

```{note}
Please be patient; it may take a few minutes for the connection to show as **'Up'**.
```

Here’s how the topology will appear following the establishment of the attachment.

```{figure} images/lab8-edge18.png
---
align: center
---
Attachment established!
```

```{important}
Once again, the **Edge** device facilitates the extension of all Aviatrix functionalities to the remote data center!
```

## 3. Network Domain Association

Let's associate the Edge connection with one of the existing Network Domains.

Navigate to **CoPilot > Networking > Network Segmentation > Network Domains** and edit, for instance, the **Green** domain. Select the **`on-prem-edge`** connection and do not forget to click on **Save**!

```{figure} images/lab8-edge19.png
---
height: 400px
align: center
---
Network Domain Association
```

You have successfully expanded **Network Segmentation** across the data center.

```{figure} images/lab8-newjoe.png
---
align: center
---
The DC is now another VPC
```

Let's explore again the Cloud Routes section!

Navigate to **CoPilot > Diagnostics > Cloud Routes > BGP info** and click on the three dots icon and select the `"Show BGP Advertised Routes` option.

```{important}
This time you will notice that the Edge device is advertising all the MCNA CIDRs to the LAN router! Those routes got installed into the Edge device by the **Aviatrix Controller**, after the attachment got established!
```

```{figure} images/lab8-edge20.png
---
align: center
---
BGP Advertised Routes
```

## 4. Edge: Connectivity Test

Let's initiate a connectivity test from the 'Edge' workstation located inside the Data Center in New York.

```{figure} images/lab8-newjoe2.png
---
align: center
---
BGP Advertised Routes
```

Navigate to your personal POD portal, scroll down untill your reach the **Lab 7 and 8** section and click on the `"Open Workstation"` button.

```{figure} images/lab8-edgenew.png
---
height: 250px
align: center
---
Workstation Edge access from the POD Portal
```

Next, enter the credentials provided in the POD Portal.

```{figure} images/lab8-newjoe3.png
---
align: center
---
Workstation Edge credentials
```

You will land on the Desktop of the Workstation Edge and from here launch the `LX Terminal`.

```{figure} images/lab8-newjoe4.png
---
align: center
---
LX Terminal
```

Execute a ping to the private IP address of the **aws-us-east-2-spoke1-test1** instance (**10.0.1.100**).

```{figure} images/lab8-edge22.png
---
align: center
---
Target for the connectivity test
```

The ping will be successful, this means that you have extended the Aviatrix CNSF to your on-prem DC, that ultimately can now be considered as just an additional VPC!

```{note}
Do NOT forget to exit from the previous session used to log in to the **on-prem-partner1 router** (i.e. Lab 7)
```{figure} images/lab8-edge67.png
---
height: 250px
align: center
---
exit
```

```{figure} images/lab8-edge30.png
---
align: center
---
Ping
```

## 5. Edge: FlowIQ

* Use <span style='color:#FF0000'>**FlowIQ**</span> within Aviatrix CoPilot to analyze and inspect NetFlow data.

```{tip}
Navigate to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based  on the `"Destination IP Address"` **10.0.1.100** (i.e. **_aws-us-east-2-spoke1-test1_**).

Do not forget to click on **Apply**.
```

```{figure} images/lab8-plus.png
---
align: center
---
Create the filter
```

```{figure} images/lab8-edge24.png
---
align: center
---
FlowIQ Filter
```

```{caution}
It may take about <ins>**5** minutes</ins> for flow data to show up on the CoPilot UI, therefore please wait for a bit
and then click on the **`"Refresh Data"`** button!
```{figure} images/lab8-refresh2.png
---
align: center
---
Refresh
```

Then scroll a little bit and check the `"Flow Exporters"` widget, then from the drop-down menu select the **`"Aviatrix Gateway"`** widget: you will see the list of all the Aviatrix Gateways involved along the path.

```{figure} images/lab8-newjoe6.png
---
align: center
---
Widget
```

```{figure} images/lab8-flowiq.png
---
align: center
---
Aviatrix Gateway
```

```{note}
On the **Aviatrix Gateway** widget, the very first gateway from the list is the gateway with the highest traffic (in KibiBytes).
```

## 6. Edge: "It's more than a Spoke GW"

The Aviatrix Edge device is capable to be connected to multiple Transit Gateways, simultaneously, thus <ins>the Edge device is regarded much more than a classic Spoke gateway</ins>.

Let's connect the Edge device also to the Transit Gateway in **US-Central-1** in **GCP**.

```{figure} images/lab8-edgedouble.png
---
align: center
---
New Attachment towards GCP
```

Once again, you have to configure a **BGP ASN** on the **_gcp-us-central1-transit_** GW first, before deploying any new attachments.

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_gcp-us-central1-transit_**.

```{figure} images/lab8-edgedouble5.png
---
align: center
---
gcp-us-central1-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section and insert the AS number **64514** on the empty field related to the `“Local AS Number”`, then click on **Save**.

```{figure} images/lab8-edgedouble6.png
---
align: center
---
BGP ASN
```

Now you are ready to proceed with the rest of the configuration on the Edge section!

Navigate to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Gateway Attachments"` button, on the right-hand side of the screen.

```{figure} images/lab8-edgedouble221.png
---
height: 200px
align: center
---
Manage Gateway Attachment
```

Now click on the `"+ Attachment"` button.
You will notice the existing attachment (grayedout) with the Transit Gateway in AWS US-East-2.

```{figure} images/lab8-edgedouble3.png
---
align: center
---
New Attachment
```

Fill in the attachment template using the following settings:

- **Transit Gateway**: <span style='color:#479608'>gcp-us-central1-transit</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>

```{caution}
The **High Performance Encryption** option is not visible in this case, because the  Aviatrix Controller is aware that the GCP Transit is already configured with _**HPE=off**_
```

Do not forget to click on **Save**.

```{figure} images/lab8-attachment01.png
---
align: center
---
Edge Attachment Template
```

Wait for **1 minute** for the Aviatrix Controller to establish the attachment between the Edge and the GCP Transit Gateway. 

Once the operation is completed you will be notified!

```{figure} images/lab8-edgedouble9.png
---
align: center
---
Notification
```

Let's verify the presence of the new attachment previously created on the Topology. 

Navigate to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

```{figure} images/lab8-edgedouble10.png
---
align: center
---
Topology
```

```{caution}
The Aviatrix Edge Spoke Gateway can be attached to multiple Transit Gateways!
```

### 6.1 Connectivity Testing

Now let's verify the connectivity beetween  **aws-us-east-2-spoke1-test1** in AWS  and **gcp-us-central1-spoke1-test1** in GCP

#### 6.1.1 Connectivity Testing Using the Diagnostics Tools

Navigate to your POD Portal, locate the `Gatus widget`, and select both  **_aws-us-east-2-spoke1-test1_** and **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab8-podportal110.png
---
height: 400px
align: center
---
Gatus
```

```{figure} images/lab8-podportal111.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test1
```

```{figure} images/lab8-podportal112.png
---
height: 400px
align: center
---
gcp-us-central1-spoke1-test1
```

The two instances can communicate without any issues.

```{caution}
If the two instances are unable to communicate, check the DCF section, the Network Segmentation section, and the Hybrid Cloud section, as you may have overlooked some necessary configurations in these areas.
```

Navigate to **CoPilot > Diagnostics > Diagnostics Tools**, select the **_aws-us-east-2-spoke1_** Gateway, choose the `Traceroute` command and enter the private IP address of **_gcp-us-central1-spoke1-test1_** (i.e., `172.16.1.100`).

```{figure} images/lab8-podportal113.png
---
height: 200px
align: center
---
Traceroute
```

The traffic is indeed predictable:

1) **First hop**: Transit Gateway in AWS US-EAST-2
2) **Second hop**: Transit Gateway in GCP US-CENTRAL-1
3) **Third hop**: Spoke GAteway in GCP US-CENTRAL-1
4) **Forth hop**: gcp-us-central1-spoke1-test1 VM

```{figure} images/lab8-podportal114.png
---
height: 400px
align: center
---
Hop by Hop
```

#### 6.1.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Now, let's **SSH** on the EC2 _**aws-us-east-2-spoke1-test1**_  and then launch the command _traceroute_ towards the VM _**gcp-us-central1-spoke1-test1**_ in GCP

```{figure} images/lab8-edgedouble21.png
---
align: center
---
aws-us-east-2-spoke1-test1 
```

- Install the **`inetutils-traceroute`** package, typing the following command:

```bash
sudo apt install inetutils-traceroute
```

```{caution}
You will be asked to type the student's password!
```

```{figure} images/lab8-edgedouble22.png
---
align: center
---
inetutils-traceroute
```

When you see this pop-up message, just click on the **Enter** button on your keyboard!

```{figure} images/lab8-popup.png
---
align: center
---
confirm
```

Now type the traceroute command towards the test VM in GCP:

```bash
traceroute 172.16.1.100
```

```{figure} images/lab8-edgedouble23.png
---
align: center
---
Traceroute
```

The traceroute will reveal that the destination is exactly **5** hops away.

```{figure} images/lab8-edgedouble25.png
---
align: center
---
5 hops
```

## 7. AS-PATH Prepend

Let's harness the **as-path prepend** feature for manipulating the traffic. 

```{important}
The routes exchanged between transit gateways are considered `BGP-like routes`. This is because the Aviatrix Controller, which orchestrates SDN routing, utilizes mechanisms similar to those of BGP for routing decisions. As a result, these routes exhibit attributes akin to BGP routes—for example, each transit has its own `AS PATH`, which is used for the best path selection process. However, it's important to remember that the control plane within the CNSF is fundamentally based on `SDN` (Software-Defined Networking).
```

The objective of this task is to define a **Primary** path through the Edge device, whereas the path between the Transit gateways will be used as a **Backup** path.

```{figure} images/lab8-primary.png
---
align: center
---
Primary and Backup
```

Let's first check the `Route DB` of the **_aws-us-east-2-transit_** GW.

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the **_aws-us-east-2-transit_** Gateway.

```{figure} images/lab8-primary01.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Route DB"` tab, then on right-hand side  type **172.16.1.0** on the **Search** field.

```{figure} images/lab8-primary03.png
---
height: 250px
align: center
---
Route DB
```

```{figure} images/lab8-primary02.png
---
align: center
---
1 AS Path length
```

From the **_aws-us-east-2-transit_** perspective, the destination route `172.16.1.0` is reachable via a single AS (i.e., 64514).

Now let's apply the route manipulation. Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW.

```{figure} images/lab8-edgedouble30.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Settings"` tab, expand the `"Border Gateway Protocol (BGP)"` section, and in the `AS Path Prepend` widget, choose the `gcp-us-central1-transit-peering` connection. Then, enter the AS number **64513** <ins>twice</ins>.

Of course, then click on **Save**.

```{figure} images/lab8-edgedouble31.png
---
align: center
---
as-path prepend
```

Let's repeat the same kind of configuration on the **GCP** Transit GW.

Navigate to **CoPilot > Cloud fabric > Gateways > Transit Gateways** and click on the **_gcp-us-central1-transit_** GW.

```{figure} images/lab8-edgedouble32.png
---
align: center
---
gcp-us-central1-transit
```

Select the `"Settings"` tab, expand the `"Border Gateway Protocol (BGP)"` section, and in the `AS Path Prepend` widget, choose the `aws-us-east-2-transit-peering` connection. Then, enter the AS number **64514** <ins>twice</ins>.

Click on **Save** to apply the change!

```{figure} images/lab8-edgedouble33.png
---
align: center
---
as-path prepend
```

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW, then select the `"Route DB"` tab and then once again, on the right-hand side, type **172.16.1.0** inside the Search field. The AS Path Length will be **3**, caused by the `AS-Path prepend` route manipulation.

```{figure} images/lab8-path.png
---
height: 300px
align: center
---
As path length = 3
```

## 8. Final Verification

Now, let's launch again the traceroute towards **172.16.1.100**.

### 8.1 Traceroute through the Spoke Gateway

Navigate to **CoPilot > Diagnostics > Diagnostics Tools**, select the **aws-us-east-2-spoke1** Gateway, choose the `Traceroute` command and insert the private IP address of **_gcp-us-central1-spoke1-test1_** (i.e. 172.16.1.100).

```{figure} images/lab8-podportal113.png
---
height: 200px
align: center
---
Traceroute
```

### 8.2 Traceroute through the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Launch the traceroute again to **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab8-almostdone.png
---
align: center
---
traceroute
```

### 8.3 Transitive Routing

The traceroute still shows the Transit peering between AWS and GCP as the preferred path, despite the correct application of the `as-path prepend` earlier.

There is another option that needs to be enabled in order to complete this lab.

Navigate to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `Edge` gateway.

```{figure} images/lab8-almostdone02.png
---
align: center
---
Edge
```

Select the `"Settings"` Tab and then expand the `"Routing"` section, and then enable the `Transitive Routing` toggle. Do not forget to click on **Save**.

```{figure} images/lab8-almostdone03.png
---
align: center
---
edge
```

## 9. Verification of the Transitive Routing

### 9.1 Connectivity Testing Using the Diagnostics Tools

Navigate to **CoPilot > Diagnostics > Diagnostics Tools**, select the **_aws-us-east-2-spoke1_** Gateway, choose the `Traceroute` command and insert the private IP address of **_gcp-us-central1-spoke1-test1_** (i.e., **172.16.1.100**).

```{figure} images/lab8-podportal123.png
---
height: 250px
align: center
---
Traceroute
```

The traceroute reveals **five** hops.

```{figure} images/lab8-almostdone055.png
---
align: center
---
5 Hops 
```

### 9.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Let's relaunch the traceroute towards **172.16.1.100** from the **_aws-us-east-2-spoke1-test1_**.

```{figure} images/lab8-almostdone04.png
---
align: center
---
traceroute
```

The traceroute reveals **six** hops.

```{figure} images/lab8-almostdone05.png
---
align: center
---
6 Hops
```

## 10. Final Considerations

Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** Gateway, then select the `"Route DB"` tab and then once again, on the right-hand side, type `172.16.1.0` inside the **Search** field.

This time, the AS Path length will be equal to **2**.

<ins>The best path is via the Edge!</ins>

```{figure} images/lab8-2path.png
---
height: 250px
align: center
---
As path length = 2
```

Upon completing this lab, here is the overall topology you can expect to see:

```{figure} images/lab8-edge25.png
---
height: 400px
align: center
---
Final Topology for Lab 8
```