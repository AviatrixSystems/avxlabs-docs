# Lab 5 - Routes Manipulation & NAT

## 1. SCENARIO

**BU1 Frontend** suddenly cannot communicate with **BU1 Analytics** or **BU1 DB**. The network team found that a disgruntled employee had compromised the connectivity between these workloads.

```{figure} images/lab5-topology.png
---
height: 400px
align: center
---
Lab 5 Topology
```

## 2. TROUBLESHOOT REQUEST

Let’s proceed with another troubleshooting request, using the same tools we used in the previous labs.

### 2.1 BU1 Analytics is not reachable - verifiy connectivity Using Gatus

### 2.2 BU1 Analytics is not reachable - verifiy connectivity Using the Diagnostic Tools

- Verify that the connectivity between **BU1 Frontend** and **BU1 Analytics** is actually broken.

  - Run the ping command from the `Diagnostic Tools` to the private IP address of the **BU1 Analytics**

```{figure} images/lab5-traceroute2812.png
---
align: center
---
Ping from the Spoke1
```

### 2.3 BU1 Analytics is not reachable - verifiy connectivity Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

-  Alternatively, establish an SSH connection to the **BU1 Frontend** and execute ping, traceroute, or SSH commands toward **BU1 Analytics**.

```{figure} images/lab5-pingfails.png
---
align: center
---
Ping fails
```

### 2.4 Routing Tables verification

- Check whether the **AWS Spoke1 GW** and the **GCP Spoke1 GW** have the relevant routes or not.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **Spoke1 Gateway in AWS > Gateway Routes** and filter by the remote route.

Repeat the same process from the GCP Spoke perspective!
```

```{figure} images/lab5-filter.png
---
height: 350px
align: center
---
Filter
```

The **Spoke1** in AWS does not have the destination route to reach the destination in GCP, based on the outcome above...

#### 2.4.1 Gateway Routes Filter

- Check what route is received by the **GCP Transit GW** from the **GCP Spoke1 GW**.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **_ace-gcp-us-east1-transit1_** Gateway in GCP **> Gateway Routes** and filter by the parameter depicted below.
```

- "**Next Hop Gateway** *contains* **gcp**"

```{figure} images/lab5-nexthop200.png
---
height: 350px
align: center
---
Next Hop
```

```{figure} images/lab5-nexthop.png
---
height: 350px
align: center
---
Malicious Route
```

Spoke1 in GCP is advertising an **invalid route** (`40.40.40.0/24`), while the legitimate route **172.16.211.0/24** has been withdrawn.

```{figure} images/lab5-nexthop99.png
---
height: 350px
align: center
---
Malicious Route
```

### 2.5 Misconfiguration in Settings

- Fix the issue checking the **Settings** (i.e., the *Route Manipulation* section) section on the **Spoke GW in GCP**.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **Spoke GW in GCP > Settings > Routing** and <ins>remove</ins> the CIDR 40.40.40.0/24 from the **Customize Spoke Advertised VPC/VNet CIDRs** section and then click on **SAVE**.
```

```{figure} images/lab5-customize.png
---
height: 350px
align: center
---
Customize Spoke Advertised VPC/VNet CIDRs
```

```{figure} images/lab5-customize100.png
---
height: 350px
align: center
---
Delete the malicious route
```

### 2.6 BU1 Analytics - final verification

Now that we’ve identified the problem location, let’s re-run the connectivity tests.

#### 2.6.1 BU1 Analytics is not reachable - final verification Using Gatus

#### 2.6.2 BU1 Analytics is not reachable - final verification Using the Diagnostic Tools

- From `Diagnostic Tools`, re-run ping to the private IP address of **BU2 MobileApp**.

```{figure} images/lab5-traceroute28123.png
---
align: center
---
Ping from the Spoke1
```

#### 2.6.3 BU1 Analytics is not reachable - final verification Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- If you prefer, please reevaluate connectivity by reissuing a ping from **BU1 Frontend** to **BU1 Analytics** via your SSH client.

```{figure} images/lab5-pingok.png
---
align: center
---
Ping is ok
```

### 2.7 BU1 DB is not reachable

- Now verify that the connectivity between **BU1 Frontend** and **BU1 DB** is actually broken.

#### 2.7.1 BU1 Analytics is not reachable - verifiy connectivity Using Gatus

#### 2.7.2 BU1 Analytics is not reachable - verifiy connectivity Using the Diagnostic Tools

- Run the traceroute command from the `Diagnostic Tools` to the private IP address of the BU1 DB.
  
```{tip}
Navigate to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select **_ace-aws-eu-west-1-spoke1_** Spoke GW and perform a **traceroute** to BU1 DB’s private IP.
```

```{figure} images/lab5-5hops.png
---
align: center
---
5 Hops...
```

```{important}
The results above show **five** hops. The last hop to respond is the Spoke1 Gateway in Azure, <ins>which means the BU1 DB did not respond to the traceroute</ins>.
```

#### 2.7.3 BU1 Analytics is not reachable - verifiy connectivity Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

  - Alternatively, establish an **SSH session** to **BU1 Frontend** and perform _ping, ssh and traceroute_ commands targeting **BU1 DB**.

```{figure} images/lab5-pingfails2.png
---
align: center
---
Ping is unsuccessful
```

```{figure} images/lab5-pingfails23.png
---
align: center
---
SSH is unsuccessful
```

```{figure} images/lab5-pingfails2314.png
---
align: center
---
Traceroute = 6 hops
```

```{important}
The results above show **six** hops. The last hop to respond is the Spoke1 Gateway in Azure, <ins>which means the BU1 DB did not respond to the traceroute</ins>.
```

#### 2.2.1 Packet Capture

You can duplicate the current CoPilot UI tab and run `Diagnostics Tools` in parallel: one tab to generate ICMP traffic, and another to capture packets.

- Navigate to **CoPilot > Diagnostics > Diagnostics Tools**. Select the **_ace-azure-east-us-spoke1_** gateway, then choose `Packet Capture` from the actions list, select the **eth0** interface and set the capture duration to **10 seconds**. Before clicking **RUN**, switch to the other tab to generate ICMP traffic.

```{figure} images/lab5-packetcapture000.png
---
align: center
---
Packet Capture
```

- On the other tab, navigate to **CoPilot > Diagnostics > Diagnostics Tools**. Select the **_ace-aws-eu-west-1-spoke1_** gateway, then choose **Ping** from the actions list, and enter the private IP address of **BU1-DB**. Before clicking RUN, <ins>first trigger the packet capture on the other tab</ins>, then execute the ping.

```{figure} images/lab5-packetcapture0001.png
---
align: center
---
PING
```

- Alternatively, you can generate the ICMP traffic using the SSH client, and keep capturing the packets from the **Diagnostics Tools**!

```{figure} images/lab5-pinfail.png
---
align: center
---
Ping to BU1 DB fails
```

You will only see **ICMP Echo Request** packets leaving from the LAN interface. Additionally, the source IP is entirely different from what you'd expect: instead of an IP from the legitimate CIDR 10.1.211.0/24, the source address belongs to the `50.50.50.0/24` range.

```{figure} images/lab5-packetcapture00.png
---
height: 350px
align: center
---
Packet Capture
```

- Fix the issue checking the **Routing** section (aka as _Routes Manipulation_ section) on the **Spoke1 GW in Azure**.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke1_** GW in Azure **> Settings > Network Address Translation (NAT)**, delete the NAT Rule and click on **SAVE**.
```

```{figure} images/lab5-deleterule.png
---
height: 400px
align: center
---
Deletion of the NAT rule
```

### 2.8  BU1 DB - final verification

#### 2.8.1 BU1 DB is not reachable - final verification Using Gatus

#### 2.8.2 BU1 DB is not reachable - final verification Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Run the ping command from the `Diagnostic tools` to the private IP address of the BU1 DB.

```{figure} images/lab5-traceroute2812141.png
---
align: center
---
Ping from the Spoke1
```

#### 2.8.3 BU1 DB is not reachable - final verification Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Alternatively, re-run the ping to **BU1 DB** from the SSH session established with the BU1 Frontend.

```{figure} images/lab5-pingok2.png
---
align: center
---
Ping is ok
```

## 3. CHANGE REQUEST

- Now that connectivity has been restored, the **BU2 DB** owner has also requested that you filter out the route **10.0.211.0/24** from the routing table of the Azure Spoke2 Gateway.

```{important}
The subnet `10.0.211.0/24` is advertised by the **OnPrem DC Router** towards the Aviatrix CNSF via the Site-to-Cloud connection.

```{figure} images/lab5-newrequest2.png
---
align: center
---
S2C route
```

- Check the **BGP Learned Routes**!

```{tip}
Navigate to **CoPilot > Diagnostics > Cloud Routes > BGP Info** and then click on the **3 dots** icon on the right-hand side of the screen and select the `"Show BGP Learned Routes"` option.
```

```{figure} images/lab5-bgpinfo.png
---
align: center
---
Show BGP Learned Routes
```

```{figure} images/lab5-bgpinfo100.png
---
height: 450px
align: center
---
10.0.211.0/24
```

You are required to keep this route visible (fully installed in the routing tables) on all the Spoke gateways, <ins>except for the Azure Spoke2 Gateway</ins>.

```{figure} images/lab5-newrequest.png
---
height: 400px
align: center
---
Topology
```

- Verify first the presence of the route inside the routing table of the **Azure Spoke2 GW**.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} images/lab5-newrequest3.png
---
height: 300px
align: center
---
Search for the remote route from the OnPrem DC
```

- Filter the route `10.0.211.0/24` in the `Settings` section (i.e., the *Routes Manipulation* section).

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Settings** tab.
Expand the **Routing** section and insert the subnet **10.0.211.0/24** inside the `"Exclude Learned CIDRs to Spoke VPC/VNet Route Table"` option.

Do not forget to click on **Save**.
```

```{figure} images/lab5-newrequest4.png
---
height: 450px
align: center
---
Exclude the route
```

- Confirm that the route has been removed from the routing table of the **Azure Spoke2 Gateway**.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} images/lab5-newrequest5.png
---
height: 450px
align: center
---
No Results found
```

The route has been successfully removed from the routing table!