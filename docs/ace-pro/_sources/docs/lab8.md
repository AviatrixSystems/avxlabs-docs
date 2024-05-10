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
- **Authetication Method**: <span style='color:#479608'>PSK</span>
- **Pre-Shared Key**: <span style='color:#479608'>[**Refer to your Pod assignment for the CSR Public IP - Lab8 section**]</span>
- **Remote Gateway IP**: <span style='color:#479608'>[**Refer to your Pod assignment for the CSR Public IP - Lab8 section**]</span>
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

### 4.2. Site2Cloud Connection - StrongSwan's private IP

The StrongSwan has been alreayd pre-configured with the IPSec commands. Nevertheless, you need to fetch its private IP address!

```{note}
The StrongSwan router is not acting as an actual branch router because it is being NAT'd by an **AWS IGW**. For that purpose, you need to specify that the **`Remote Identifier`** of the IKE tunnel is the private IP of the CSR, not the public IP.
```

To find out the **Private IP** of the StrongSwan router, SSH as `student` to the on-premises router (same as the Remote Peer IP above) and issue the command `ip addr show`. 

```{tip}
The Public IP of the StrongSwan router is retrievable via `"dig"` command, resolving its DNS symbolic name. Please refer to the previous section.
```

The Private IP that you need to copy is that one assigned to the second interface, the **_ens#_** interface!

```{figure} images/lab8-giga.png
---
align: center
---
ip addr show
```

Use the Private IP of the **ens#** interface. It would be something in 172.16.1.0/24, such as 172.16.1.176, in the above output.

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)** and click on the `GCP-to-OnPremPartner` connection.

```{figure} images/lab8-s2craw.png
---
align: center
---
S2C node
```

```{note}
The connection will show up in red, which in turn means, it is not established yet.
```

Then click on the `"Settings"` tab, expand the `"General"` section and paste the Private IP on the `"Remote Gateway Identifier"` field, as depicted below. 

```{note}
You will find the public IP address. Please delete it and paste the private IP address (i.e. ens# interrface).
```

```{figure} images/lab8-delete.png
---
align: center
---
Delete the existing Public IP address
```

Do not forget to click on **Save**.

```{figure} images/lab8-settings.png
---
align: center
---
Remote Gateway Identifier
```

After doing so, click on the refres button

```{figure} images/lab8-up.png
---
align: center
---
Connection is up/up
```

## 5. S2C - Verification

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**

```{tip}
Click on the **refresh** button to see the stratus icon changing from red to green.

The status will be reflected on the screen in about **1 minute**. 
```

```{figure} images/lab8-refresh.png
---
align: center
---
Connection is up/up also on the CoPilot
```

- Verify the newly created tunnel is up (might take a few seconds once configuration is applied on the CSR (alternatively you can just click on the `Refresh` button).

Go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)**

Filter out based on the GCP Cloud, expand all the VPCs and you will see the new S2C connection with the remote OnPrem-Partner site!

```{figure} images/lab8-onprem.png
---
align: center
---
OnPrem-Partner site
```

Now go back to your SSH terminal, and from the on-premises router’s console, enter a ping sourced from the **_GigabitEthernet1_** interface to the test instance in GCP (**_gcp-us-central1-spoke1-test1_**) as follows: 

```bash
ping 192.168.200.100 source gi1
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

```{important}
You will have to relaunch the ping command once again from the CSR router and click on **Run** on the CoPilot for capturing the `Active Sessions`! 
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

You can notice both the AS numbers of each side of the connection and the /30 prefix length used in the underlay.

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

### 6.1. Peering between Edge and the Transit

Let's establish a peering between the Aviatrix Edge device and the pair of Transit Gateways in **US-EAST-2**. 

In the Topology depicted below, you will notice that there is a workstation named "edge" attached to the LAN router. Once the peerings has been established, you will launch your ping from that client!

```{figure} images/lab8-edge8.png
---
height: 400px
align: center
---
Peerings not established yet!
```

First and foremost, you have to configure a **BGP ASN** on the **_aws-us-east-2-transit_** GWs cluster!

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_**.

```{figure} images/lab8-edge12.png
---
align: center
---
Peerings not established yet!
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section and insert the AS number **64513** on the empty field related to the `“Local AS Number”`, then click on **Save**.

```{figure} images/lab8-edge13.png
---
align: center
---
Peerings not established yet!
```

Now it's time to establish the peering! 

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

Fill in the peering template using the following settings:

- **Transt Gateway**: <span style='color:#479608'>aws-us-east-2-transit</span>
- **Connecting Edge Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over Private Network**: <span style='color:#479608'>**OFF**</span>
- **High Performance Encryption**: <span style='color:#479608'>**ON**</span>
- **Number of Tunnels**: <span style='color:#479608'>4</span>

Do not forget to click on **Save**.

```{figure} images/lab8-edge11.png
---
align: center
---
Peering creation template
```

Wait for a bunch of seconds for the Aviatrix Controller to establish the peering and then a message will pop up confirming that the operation has been accomplished, successfully!

```{figure} images/lab8-edge14.png
---
align: center
---
Peering created
```

Let's verify the presence of the peering previously created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

```{figure} images/lab8-edge15.png
---
align: center
---
New peering
```

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** cluster.

```{figure} images/lab8-edge16.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Connections"` tab and then click on `"Transit-Edge Peering"`. You will notice this additional tab that confirms the presence of a peering between the Transit GW in the cloud and the Edge running in the DC!

```{figure} images/lab8-edge17.png
---
align: center
---
Transit-Edge Peering
```

This is how the Topology would look like after the creation of the peering.

```{figure} images/lab8-edge18.png
---
height: 400px
align: center
---
Peerings established!
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
This time you will notice that the Edge device is advertising all the MCNA CIDRs to the LAN router! Those routes got installed into the Edge deviceby the **Aviatrix Controller** after the peering establishment!
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

```{figure} images/lab8-edge21.png
---
align: center
---
Connectivity test
```

The ping will be successful, this means that you have extended the Aviatrix MCNA to your on-prem DC, that ultimately can now be considered an additional VPC!
```{figure} images/lab8-edge22.png
---
align: center
---
Connectivity test
```

### 6.3. Edge: FlowIQ

* Use <span style='color:#FF0000'>**FlowIQ**</span> from the Aviatrix CoPilot, <ins> for inspecting the NetFlow Data.

```{tip}
Go to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based  on the `"Destination IP Address"` **10.0.1.100** (i.e. **_aws-us-east-2-spoke1-test1_**).

Do not forget to click on **Apply**.
```

```{figure} images/lab8-edge24.png
---
align: center
---
FlowIQ Filter
```

```{caution}
If you do not get immediately any results, wait for a couple of minutes and apply again the filter!
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

After this lab, this is how the overall topology would look like:

```{figure} images/lab8-edge25.png
---
height: 400px
align: center
---
Final Topology for Lab 8
```