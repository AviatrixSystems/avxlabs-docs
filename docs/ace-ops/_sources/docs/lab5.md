# Lab 5 - Routes Manipulation & NAT

## 1. SCENARIO

BU1 Frontend suddenly cannot communicate with BU1 Analytics or BU1 DB. The network team found that a disgruntled employee had compromised the connectivity between these workloads.

```{figure} images/lab5-topology.png
---
height: 400px
align: center
---
Lab 5 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between BU1 Frontend and BU1 Analytics is actually broken.
  - SSH to BU1 Frontend and carry out ping/traceroute/ssh commands towards BU1 Analytics.

```{figure} images/lab5-pingfails.png
---
align: center
---
Ping fails
```

- Check whether the AWS Spoke1 GW and the GCP Spoke1 GW have the relevant routes or not.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **Spoke1 Gateway in AWS > Gateway Routes** and filter by the remote route.

Repeat the same process from the GCP Spoke perspective!
```

```{figure} images/lab5-filter.png
---
height: 350px
align: center
---
Filter out
```

The **Spoke1** in AWS does not have the destination route to reach the destination in GCP, based on the outcome above...

- Check what route is received by the **GCP Transit GW** from the **GCP Spoke1 GW**.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **_ace-gcp-us-east1-transit1_** Gateway in GCP **> Gateway Routes** and filter by the parameter depicted below.
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

You will notice that the Spoke1 in GCP is advertising a **bogus/malicious** route (i.e. `40.40.40.0/24`), whereas the legit route **172.16.211.0/24** has been withdrawn!

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

- Relaunch the ping from **BU1 Frontend** towards **BU1 Analytics**.

```{figure} images/lab5-pingok.png
---
align: center
---
Ping is ok
```

- Now verify that the connectivity between **BU1 Frontend** and **BU1 DB** is actually broken.
  - SSH to BU1 Frontend and carry out ping/traceroute/ssh commands towards BU1 DB.

```{figure} images/lab5-pingfails2.png
---
align: center
---
Ping is unsuccessful
```

- Use **Diagnostics Tools** from the *Spoke1 Gateway* in AWS and launch a **traceroute** towards the BU1 DB in Azure.

```{tip}
Navigate to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_ace-aws-eu-west-1-spoke1_** Spoke GW and this time launch a **traceroute** towards the private IP address of the BU1 DB.
```

```{figure} images/lab5-5hops.png
---
align: center
---
5 Hops...
```

```{important}
The results above show **five** hops. The last hop to respond is the Spoke1 Gateway in Azure, which means the BU1 DB did not respond to the traceroute.
```

- Start a **Packet Capture** on the LAN interface (`eth0`) of the **_ace-azure-east-us-spoke1_** gateway, and filter the results to focus on **ICMP** traffic.

```{tip}
Navigate to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_ace-azure-east-us-spoke1_** Spoke GW and launch the packet capture on its eth0 interface.

This is the LAN interface within the VNet, where also the **VNet router** resides!
```


```{tip}
<ins>Open two tabs</ins>: on the first tab launch a packet capture on the **Azure Spoke GW1** on the LAN interface (`etho`), <ins>and after few seconds</ins>, start a ping from the **BU1 Frontend** towards **BU1 DB** (VM in Azure) and keep it running, although it will fail.

- Diminish the `Duration` to solely **10** seconds to speed up the capture!
- Filter out based on the `"ICMP"` protocol.
```

```{figure} images/lab5-pinfail.png
---
align: center
---
Ping to BU1 DB fails
```

```{figure} images/lab5-packetcapture00.png
---
height: 350px
align: center
---
Packet Capture
```

You will only see **ICMP Echo Request** packets leaving from the LAN interface. Additionally, the source IP is entirely different from what you'd expect: instead of an IP from the legitimate CIDR 10.1.211.0/24, the source address belongs to the `50.50.50.0/24` range.

- Fix the issue checking the **Routes Manipulation** section on the **Spoke1 GW in Azure**.

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

- Relaunch the ping from **BU1 Frontend** towards **BU1 DB**.

```{figure} images/lab5-pingok2.png
---
align: center
---
Ping is ok
```

## 3. CHANGE REQUEST

- Now that connectivity has been restored, the BU2 DB owner has also requested that you filter out the route **10.0.211.0/24** from the routing table of the Azure Spoke2 Gateway.

```{important}
The subnet `10.0.211.0/24` is advertised by the OnPrem DC Router towards the Aviatrix CNSF via the Site-to-Cloud connection.

```{figure} images/lab5-newrequest2.png
---
align: center
---
S2C route
```

- Check the BGP Learned Routes

```{tip}
Navigate to **CoPilot > Diagnostics > Cloud Routes > BGP Info** and then click on the **3 dots** icon on the right-hand side of the screen and select the `"Show BGP Learned Routes"` option.
```

```{figure} images/lab5-bgpinfo.png
---
height: 350px
align: center
---
Show BGP Learned Routes
```

```{figure} images/lab5-bgpinfo100.png
---
height: 350px
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

- Verify first the presence of the route inside the routing table of the Azure Spoke2 GW.

```{tip}
NAvigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} images/lab5-newrequest3.png
---
height: 300px
align: center
---
Search for the remote route from the OnPrem DC
```

- Filter out the route `10.0.211.0/24` in the `Settings` section (i.e., the *Route Manipulation* section).

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Settings** tab.
Expand the **Routing** section and insert the subnet **10.0.211.0/24** inside the `"Exclude Learned CIDRs to Spoke VPC/VNet Route Table"` option.

Do not forget to click on **Save**.
```

```{figure} images/lab5-newrequest4.png
---
height: 350px
align: center
---
Exclude the route
```

- Confirm that the route has been removed from the routing table of the Azure Spoke2 Gateway.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} images/lab5-newrequest5.png
---
height: 300px
align: center
---
No Results found
```

You can see that the route has been successfully removed from the routing table.