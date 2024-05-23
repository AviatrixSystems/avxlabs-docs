# Lab 8 - SITE2CLOUD & EDGE

## 1. General Objectives

The objective of this lab is to resolve an IP address overlap between an on-premises partner and the cloud. You will be using the **Site2Cloud Mapped NAT** feature to achieve this. 

Moreover, you are also asked to interconnect the on-prem DC in New York to your MCNA. An **Aviatrix Edge** device has already been provisioned and it got already registered to the existing Aviatrix Controller.
 
## 2. Site2Cloud Overview

Let's start with the deployment of the S2C.

Site2Cloud builds an encrypted connection between two sites over the Internet in an easy-to-use and template-driven manner. Its workflow is similar to any CSP’s virtual private network workflow.

On one end of the tunnel is an Aviatrix Gateway; on the other could be an on-premises router, firewall or another public cloud VPC/VNet that Aviatrix Controller does not manage.
 
## 3. Topology

In this lab, you will first achieve Site2Cloud connectivity to a `StrongSwan Router`, emulating an on-premises branch router.

In this lab, you will work with a typical overlapping IP addresses scenario depicted in the drawing below:

```{figure} images/lab8-topology.png
---
align: center
---
Lab 8 Initial Topology
```

## 4. Configuration

### 4.1. Site2Cloud Connection (Cloud to On-Prem)

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**. Here you will immediately notice the presence of an existing S2C connection. 

This is the connection established between an **Aviatrix Edge** device deployed in the remote DC in New York and a LAN Router.

```{figure} images/lab8-edge1.png
---
align: center
---
Existing S2C connection
```

```{figure} images/lab8-edge2.png
---
align: center
---
BGPoverLAN
```

The S2C connection with Edge will be configured on the subsequent task. Now let's configure the S2C connection with the partner site!

```{figure} images/lab8-partner.png
---
align: center
---
S2C between Partner and GCP
```

Click on the `"+ External Connection"` button and let's create a new connection from scratch:

```{figure} images/lab8-s2c.png
---
align: center
---
S2C creation
```

Create a connection from Cloud (GCP) to an on-prem Partner site, using the following settings on the `"Add External Connection"` window:

- **Name**: <span style='color:#479608'>GCP-to-OnPremPartner</span>
- **External Device**: <span style='color:#479608'>Static Route-Based (Mapped)</span>
- **Local Gateway**: <span style='color:#479608'>gcp-us-central1-spoke1</span>
- **Real Local Subnet CIDR(s)**: <span style='color:#479608'>172.16.1.0/24</span>
- **Virtual Local Subnet CIDR(s)**: <span style='color:#479608'>192.168.200.0/24</span>
- **Remote Gateway Type**: <span style='color:#479608'>Generic</span>
- **Real Remote Subnet CIDR(s)**: <span style='color:#479608'>172.16.1.0/24</span>
- **Virtual Remote Subnet CIDR(s)**: <span style='color:#479608'>192.168.100.0/24</span>
- **IKEv2**: <span style='color:#479608'>**ON**</span>
- **Authetication Method**: <span style='color:#479608'>PSK</span>
- **Pre-Shared Key**: <span style='color:#479608'>[**Refer to your Pod assignment for the StrongSwan PSK - Lab8 section**]</span>
- **Remote Gateway IP**: <span style='color:#479608'>[**Refer to your Pod assignment for the StrongSwan Public IP - Lab8 section**]</span>
- **Local Gateway Instance**: <span style='color:#479608'>gcp-us-central1-spoke1</span>

Then click on **Save**.

```{figure} images/lab8-newone.png
---
align: center
---
DNS Name of the StrongSwan router
```

```{note}
Use the command “**dig strongswan.pod#.aviatrixlab.com +short**” <ins>from your personal laptop terminal</ins> to resolve the symbolic public name of the **on-prem-partner1 StrongSwan router** and retrieve the <ins>REMOTE GATEWAY PUBLIC IP address</ins>, as depicted in the example below.
```

<ins>Replace the **#** symbol with your POD number!</ins>

The example is referring to POD #150 (please issue the command based on your POD number).

```{figure} images/lab8-newdns.png
---
align: center
---
Dig command in action
```

```{tip}
For **Windows OS** you can use the command `"nslookup"`:

**nslookup strongswan.pod#.aviatrixlab.com**

```{figure} images/lab8-nslookup.png
---
align: center
---
Nslookup 
```

This is how your template would look like.

```{figure} images/lab8-s2ctemplate.png
---
align: center
---
S2C template
```

```{caution}
The configuration template will grey out after clicking on Save. Be patient and wait for the Aviatrix Controller to complete the deployment. <ins>This will create the first side of the connection, on the GCP Spoke GW</ins>.
```

Since On-Prem-Partner1 uses the overlapping IP space, we will utilise the Aviatrix Mapped NAT feature and use two virtual subnets.

- **192.168.100.0/24** will be used for the cloud to reach on-prem resources, and 1:1 NAT will be used.

- **192.168.200.0/24** will be used from on-prem to reach cloud resources. 

For example, gcp-us-central1-test1 (172.16.1.100) will be reached at 192.168.200.100 due to 1:1 NAT.

### 4.2 Site2Cloud Connection - StrongSwan's configuration

Now you have to complete the IPSec configuration of the StrongSwan router.

SSH to the StrongSwan router. You can either use the public IP address that you retrieved before using the dig/nslookup command or you can use its dns name available on your personal POD portal.

```{figure} images/lab8-personalpod.png
---
align: center
---
StrongSwan's DNS name
```

```{figure} images/lab8-strong.png
---
align: center
---
SSH
```

Now you need to **edit** the StrongSwan's router configuration file. 
Check the content of the following file: `/etc/swanctl/swanctl.conf`

Type the following command:

```bash
cat /etc/swanctl/swanctl.conf
```

You will need to replace the string **`REPLACE_WITH_SPOKE_GW_PUBLIC_IP`** with the public IP address of the GCP spoke gateway in 3 places in `/etc/swanctl/swanctl.conf`

```{figure} images/lab8-replace.png
---
align: center
---
cfg file
```

Let's retrieve the Public IP address assigned to the GCP Spoke Gateway.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and then identify the GCP Spoke Gateway and **copy** its Public IP address.

```{figure} images/lab8-gcppublic.png
---
align: center
---
Public IP address
```

Now go back on the SSH session established with the StrongSwan router and edit the cfg file.

Type the following command:

```bash
sudo su -
```

```bash
vim /etc/swanctl/swanctl.conf
```

```{figure} images/lab8-sudo.png
---
align: center
---
sudo + vim
```

```{figure} images/lab8-insert.png
---
align: center
---
Insert mode
```

Delete the string **`REPLACE_WITH_SPOKE_GW_PUBLIC_IP`** and paste the Public IP address of the Spoke Gateway

```{figure} images/lab8-deleteandcopy.png
---
align: center
---
editing
```

When you are done with the change, type `"ESC"` and then `":x"` in order to exit the Insert mode and save the configuration successfully!

Now you need to type the following two commands:

```bash
swanctl --load-all
```

```bash
swanctl --initiate --child gcp_tun1
```

```{figure} images/lab8-swan.png
---
align: center
---
swanctl daemon
```

## 5. S2C - Verification

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**

```{tip}
Click on the **refresh** button to see the status icon changing from red to green.

The status will be reflected on the screen in about **1 minute**. 
```

```{figure} images/lab8-refresh.png
---
align: center
---
Connection is up
```

Go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)**

Filter out based on the **GCP** Cloud, expand all the VPCs and you will see the new S2C connection with the remote OnPrem-Partner site!

```{figure} images/lab8-onprem.png
---
align: center
---
OnPrem-Partner site
```

Now go back to your SSH terminal, and from the on-premises router’s console (i.e. StrongSwan), issue the following command to verify the connectivity with the **_gcp-us-central1-spoke1-test1_**:

```bash
ping 192.168.200.100 
```

```{tip}
Keep the ping running recursively!
```

```{figure} images/lab8-pingok.png
---
align: center
---
Ping ok
```

Then go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)** and click on the icon of the Spoke Gateway **_gcp-us-central1-spoke1_**, click on `Tools` and then click on `Gateway Diagnostics`.

```{figure} images/lab8-diag.png
---
align: center
---
Gateway Diagnostics
```

Choose the `“Active Sessions”` option and in the Search field type `“icmp”` and then click on **Run**. You will notice the subnets involved (i.e. real and virtual subnets) in the Mapped NAT.

```{figure} images/lab8-active2.png
---
align: center
---
Active Sessions
```

After completing the S2C connection, this is what the overall lab topology would look like:

```{figure} images/lab8-finaltopology.png
---
height: 400px
align: center
---
The Topology with the new S2C connection
```
## 6. Edge

Now let's connect the `Aviatrix Edge` to the existing MCNA. 

First and foremost let's explore the BGP Map that describes the connectivity established through the BGPoverLAN.

Go to **CoPilot > Diagnostics > Cloud Routes > BGP info** and click on the three dots icon and select the `"Show BGP Map"` option.

```{figure} images/lab8-edge3.png
---
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

Close the BGP Map and then click again on the threee dots icon and this time select the `"Show BGP Learned Routes"`.

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

If you check also the `"Show BGP Advertised Routes"` outcome, you will notice that the Aviatrix Edge is not advertising any routes, because it is not connected to the MCNA yet!

```{figure} images/lab8-edge7.png
---
align: center
---
No routes advertised by the Edge yet
```

### 6.1. Attachment between Edge and the Transit

Let's establish a peering between the Aviatrix Edge device and the Transit Gateway in **US-EAST-2**. 

In the Topology depicted below, you will notice that there is a workstation named "edge" attached to the LAN router. Once the attachment has been established, you will launch your ping from that client, for the connectivity verification!

```{figure} images/lab8-edge8.png
---
align: center
---
Peerings not established yet!
```

First and foremost, you have to configure a **BGP ASN** on the **_aws-us-east-2-transit_** GW!

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_**.

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

Go to **CoPilot > Cloud Fabric > Edge > Edge Gateways** and click on the three dots icon beside the Edge device entry and then click on `"Manage Transit Gateway Attachment"`.

```{figure} images/lab8-edge9.png
---
align: center
---
Manage Transit Gateway Attachment
```

Click on the `"+ Transit Gateway Attachment"` button.

```{figure} images/lab8-edge10.png
---
align: center
---
Transit Gateway Attachment
```

Fill in the attachment template using the following settings:

- **Transt Gateway**: <span style='color:#479608'>aws-us-east-2-transit</span>
- **Connecting Edge Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over Private Network**: <span style='color:#479608'>**OFF**</span>
- **High Performance Encryption**: <span style='color:#479608'>**OFF**</span>

Do not forget to click on **Save**.

```{figure} images/lab8-edge11.png
---
align: center
---
Attachment creation template
```

Wait for a bunch of seconds for the Aviatrix Controller to establish the attachment and then a message will pop up confirming that the operation has been accomplished, successfully!

```{figure} images/lab8-edge14.png
---
align: center
---
Peering created
```

Let's verify the presence of the attachment previously created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

```{figure} images/lab8-edge15.png
---
align: center
---
New attachment
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** cluster.

```{figure} images/lab8-edge16.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Connections"` tab and then click on `"Transit-Edge Peering"`. You will notice this additional tab that confirms the presence of an attachment between the Transit GW in the cloud and the Edge running in the DC!

```{figure} images/lab8-edge17.png
---
align: center
---
Transit-Edge Peering
```

This is how the Topology would look like after the creation of the attachment.

```{figure} images/lab8-edge18.png
---
align: center
---
Attachment established!
```

The Edge devices allows to extend all the Aviatrix functionalities to the remote DC!

### 6.2. Network Domain Association

Let's assocciate the Edge connection to any of the existing Network Domains.

Go to **CoPilot > Networking > Network Segmentation > Network Domains** and edit, for instance, the **Green** domain. Select the on-prem-edge connection and do not forget to click on **Save**!

```{figure} images/lab8-edge19.png
---
align: center
---
Network Domain Association
```

Let's explore again the Cloud Routes section!
Go to **CoPilot > Diagnostics > Cloud Routes > BGP info** and click on the three dots icon and select the `"Show BGP Advertised Routes` option.

```{important}
This time you will notice that the Edge device is advertising all the MCNA CIDRs to the LAN router! Those routes got installed into the Edge device by the **Aviatrix Controller**, after the attachemnt got established!
```

```{figure} images/lab8-edge20.png
---
align: center
---
BGP Advertised Routes
```

### 6.3. Edge: Connectivity Test

Let's launch a connectivity test. 

**SSH** to the "edge" VM client that is behind the LAN router and then ping the **_aws-us-east-2-spoke1-test1_** EC2 in US-EAST-2.

```{figure} images/lab8-edgenew.png
---
align: center
---
Connectivity test
```

Once again, refer to your personal POD for retrieving the DNS symbolic name of the client attached to the LAN router!

```{figure} images/lab8-edgenew2.png
---
align: center
---
Connectivity test
```

```{figure} images/lab8-edge22.png
---
align: center
---
Target for the connectivity test
```

The ping will be successful, this means that you have extended the Aviatrix MCNA to your on-prem DC, that ultimately can now be considered an additional VPC!

```{figure} images/lab8-edge30.png
---
align: center
---
Ping
```

### 6.3. Edge: FlowIQ

* Use <span style='color:#FF0000'>**FlowIQ**</span> from the Aviatrix CoPilot, <ins> for inspecting the NetFlow Data.

```{tip}
Go to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based  on the `"Destination IP Address"` **10.0.1.100** (i.e. **_aws-us-east-2-spoke1-test1_**).

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
It may take about **5** minutes for flow data to appear in the CoPilot UI, therefore please wait for a bit
and then click on the **`"Refresh Data"`** button!
```{figure} images/lab8-refresh2.png
---
align: center
---
Refresh
```

Then scroll a little bit and check the `"Flow Exporters"` widget, then from the drop-down menu select the **`"Aviatrix Gateway"`** widget: you will see the list of all the Aviatrix Gateways involved along the path.

```{figure} images/lab8-flowiq.png
---
align: center
---
FlowIQ
```

```{note}
On the **Aviatrix Gateway** widget, the very first gateway from the list is the gateway with the highest traffic (in KibiBytes).
```

### 6.4. Edge: "It's more than a Spoke GW""

The Aviatrix Edge device is capable to be connected to multiple Transit Gateways, simultaneously, thus <ins>the Edge device is regarded much more than a classic Spoke gateway</ins>.

Let's connect the Edge device also to the Transit Gateway in **US-Central-1** in **GCP**.

```{figure} images/lab8-edgedouble.png
---
align: center
---
New Attachment towards GCP
```

Once again, you have to configure a **BGP ASN** on the **_gcp-us-central1-transit_** GW first, before deploying any new attachments.

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_gcp-us-central1-transit_**.

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

Go to **CoPilot > Cloud Fabric > Edge > Edge Gateways** and click on the three dots icon beside the Edge device entry and then click on `"Manage Transit Gateway Attachment"`.

```{figure} images/lab8-edgedouble2.png
---
align: center
---
Manage Transit Gateway Attachment
```

Now click on the `"+ Transit Gateway Attachment"` button.
You will notice the existing attachment (grayout) with the Transit Gateway in AWS US-East-2.

```{figure} images/lab8-edgedouble3.png
---
align: center
---
New Attachment
```

Fill in the attachment template using the following settings:

- **Transt Gateway**: <span style='color:#479608'>gcp-us-central1-transit</span>
- **Connecting Edge Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over Private Network**: <span style='color:#479608'>**OFF**</span>
- **High Performance Encryption**: <span style='color:#479608'>**OFF**</span>

Do not forget to click on **Save**.

```{figure} images/lab8-attachment01.png
---
align: center
---
Edge Attachment Template
```

Wait for 1 minute for the Aviatrix Controller to establish the attachment between the Edge and the GCP Transit Gateway. Once the operation is completed you will be notified!

```{figure} images/lab8-edgedouble9.png
---
align: center
---
Notification
```

Let's verify the presence of the new attachment previously created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

```{figure} images/lab8-edgedouble10.png
---
align: center
---
Topology
```

#### 6.4.1 Edge: As-Path Prepend

Now, let's SSH on the EC2 _aws-us-east-2-spoke1-test1_  and then launch the command traceroute towards the VM _gcp-us-central1-spoke1-test1_ in GCP

```{figure} images/lab8-edgedouble21.png
---
align: center
---
aws-us-east-2-spoke1-test1 
```

Install the **`inetutils-traceroute`** package, typing the following command:

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
apt install 
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

Let's harness the **as-path prepend** feature for manipulating the traffic. 

```{important}
The routes exchanged between transit gateways are considered `BGP-like routes`! This is because the Aviatrix Controller orchestrating the SD routing, also has to use a mechanism for the routing decision, and therefore these routes seem BGP routes, indeed they have some attributes similar to the attributes used with BGP routes. For instance, each Transit has its own `AS PATH`, and this is used for the best path selection process. Nevertheless, bear in mind that the control plane within the MCNA is based on `SDN` (Software Defined Networking).
```

The objective of this task is to define a **Primary** path through the Edge device, whereas the path between the Transit gateways will be used as a **Backup** path.

```{figure} images/lab8-primary.png
---
align: center
---
Primary and Backup
```

Let's first check the `Route DB` of the **_aws-us-east-2-transit_** GW.

Go to **CoPilot > Cloud Fabric > gateways > Transit Gateways** and select the **_aws-us-east-2-transit_** Gateway.

```{figure} images/lab8-primary01.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Route DB"` tab, then on right-hand side  type **172.16.1.0** on the Search field.

```{figure} images/lab8-primary03.png
---
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

From the **_aws-us-east-2-transit_** perspective, the destination route `172.16.1.0` is far just one single AS (i.e. 64514)

Now let's apply the route manipulation. Go to **CoPilot > Cloud fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW.

```{figure} images/lab8-edgedouble30.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section, then under the `AS Path Prepend` widget,  select the `gcp-us-central1-transit-peering` connection and type **twice** the AS number 64513. 

Of course, then click on **Save**.

```{figure} images/lab8-edgedouble31.png
---
align: center
---
as-path prepend
```

Let's repeat the same kind of configuration on the **GCP** Transit GW.

Go to **CoPilot > Cloud fabric > Gateways > Transit Gateways** and click on the **_gcp-us-central1-transit_** GW.

```{figure} images/lab8-edgedouble32.png
---
align: center
---
gcp-us-central1-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section, then under the `AS Path Prepend` widget select the `aws-us-east-2-transit-peering` connection and type **twice** the AS number 64514. 

Click on **Save** to apply the change!

```{figure} images/lab8-edgedouble33.png
---
align: center
---
as-path prepend
```

Now go to **CoPilot > Cloud fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW, then select the `"Route DB"` tab and then once again, on the right-hand side, type **172.16.1.0** inside the Search field. This time the AS Path Length will turn out being equal to 3, due to to the route manipulation that harnessed the `as-path prepend` feature.

```{figure} images/lab8-path.png
---
align: center
---
As path length = 3
```

Now, let's launch again the traceroute towards 172.16.1.100 from the **_aws-us-east-2-spoke1-test1_**.

```{figure} images/lab8-almostdone.png
---
align: center
---
traceroute
```

The traceroute is still showing the Transit peering as the preferred path, although the `as-path prepend` was correctly applied earlier. 

There is another option that needs to be enabled in order to complete this lab. Go to **CoPilot > Cloud Fabric > Edge > Edge Gateways** and click on the `Edge` device.

```{figure} images/lab8-almostdone02.png
---
align: center
---
edge
```

Select the `"Settings"` tab and then expand the `"Routing"` section, then turn on the knob `Transitive Routing`.

```{figure} images/lab8-almostdone03.png
---
align: center
---
edge
```

Let's relaunch the traceroute towards 172.16.1.100 from the **_aws-us-east-2-spoke1-test1_**.

```{figure} images/lab8-almostdone04.png
---
align: center
---
traceroute
```

```{figure} images/lab8-almostdone05.png
---
align: center
---
6 Hops
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab8-edge25.png
---
height: 400px
align: center
---
Final Topology for Lab 8
```