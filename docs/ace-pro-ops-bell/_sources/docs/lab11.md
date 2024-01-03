# Lab 11 - ROUTES MANIPULATION & NAT

## 1. SCENARIO

BU1 is suddenly unable to communicate with both BU1 Analytics and BU1 DB.

The network team discovered a disgruntled employee that jeopardized the connectivity between the involved workloads.

```{figure} ../../ace_ops/docs/images/lab5-topology.png
---
height: 400px
align: center
---
Lab 5 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between BU1 Frontend and BU1 Analytics is actually broken.
  - SSH to BU1 Frontend and carry out ping/traceroute/ssh commands towards BU1 Analytics.

```{figure} ../../ace_ops/docs/images/lab5-pingfails.png
---
align: center
---
Ping fails
```

- Check whether the AWS Spoke1 GW and the GCP Spoke1 GW have the relevant routes or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **Spoke1 Gateway in AWS > Gateway Routes** and filter out based on the remote route.

Repeat the same operation from the GCP's Spoke perspective!
```

```{figure} ../../ace_ops/docs/images/lab5-filter.png
---
align: center
---
Filter out
```

The **Spoke1** in AWS does not have the destination route to reach the destination in GCP, based on the outcome above...

- Check what route is received by the **GCP Transit GW** from the **GCP Spoke1 GW**.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **_ace-gcp-us-east1-transit1_** Gateway in GCP **> Gateway Routes** and filter based on the parameter depicted below.
```

- "**Next Hop Gateway** *contains* **gcp**"

```{figure} ../../ace_ops/docs/images/lab5-nexthop.png
---
align: center
---
Next Hop
```

You will notice that the Spoke1 in GCP is advertising a **bogus/malicious** route (i.e. `40.40.40.0/24`), whereas the legit route **172.16.211.0/24** has been withdrawn!

- Fix the issue checking the **Route Manipulation** section on the **Spoke GW in GCP**.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **Spoke GW in GCP > Settings > Routing** and <ins>remove</ins> the CIDR 40.40.40.0/24 from the **Customize Spoke Advertised VPC/VNet CIDRs** section and then click on **SAVE**.
```

```{figure} ../../ace_ops/docs/images/lab5-customize.png
---
align: center
---
Customize Spoke Advertised VPC/VNet CIDRs
```

- Relaunch the ping from **BU1 Frontend** towards **BU1 Analytics**.

```{figure} ../../ace_ops/docs/images/lab5-pingok.png
---
align: center
---
Ping is ok
```

- Now verify that the connectivity between **BU1 Frontend** and **BU1 DB** is actually broken.
  - SSH to BU1 Frontend and carry out ping/traceroute/ssh commands towards BU1 DB.

```{figure} ../../ace_ops/docs/images/lab5-pingfails2.png
---
align: center
---
Ping is unsuccessful
```

- Use **Diagnostics Tools** from the *Spoke1 Gateway* in AWS and launch a **traceroute** towards the BU1 DB in Azure.

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_ace-aws-eu-west-1-spoke1_** Spoke GW and this time launch a **traceroute** towards the private IP address of the BU1 DB.
```

```{figure} ../../ace_ops/docs/images/lab5-5hops.png
---
align: center
---
5 Hops...
```

```{important}
The outcome above shows **5** Hops. The last Hop that has responded to the traceroute is the Spoke1 GW in Azure, therefore the BU1 DB has not responded to the traceroute!</ins>
```

- Launch a **Packet Capture** on the LAN interface (`eth0`) of the **_ace-azure-east-us-spoke1_** GW and filter out the outcome based on **ICMP**.

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_ace-azure-east-us-spoke1_** Spoke GW and launch the packet capture on its eth0 interface.

This is the LAN interface within the VNet, where also the **VNet router** resides!
```


```{tip}
<ins>Open two tabs</ins>: on the first tab launch a packet capture on the **Azure Spoke GW1** on the LAN interface (`etho`), <ins>and after few seconds</ins>, launch a ping from the **BU1 Frontend** towards **BU1 DB** (VM in Azure) and keep it running, although it will fail.

- Diminish the `Duration` to solely **10** seconds to speed up the capture!
- Filter out based on the `"ICMP"` protocol.
```

```{figure} ../../ace_ops/docs/images/lab5-pinfail.png
---
align: center
---
Ping to BU1 DB fails
```

```{figure} ../../ace_ops/docs/images/lab5-packetcapture00.png
---
align: center
---
Packet Capture
```

You will only notice **ICMP Echo Request** packets going out from the LAN interface. Moreover the Source IP is completely different from the expected source IP: instead of an IP from the CIDR 10.1.211.0/24 (the legit source CIDR), the source IP is an address from the `50.50.50.0/24` CIDR!

- Fix the issue checking the **Routes Manipulation** section on the **Spoke1 GW in Azure**.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke1_** GW in Azure **> Settings > Network Address Translation (NAT)**, delete the NAT Rule and click on **SAVE**.
```

```{figure} ../../ace_ops/docs/images/lab5-deleterule.png
---
align: center
---
Deletion of the NAT rule
```

- Relaunch the ping from **BU1 Frontend** towards **BU1 DB**.

```{figure} ../../ace_ops/docs/images/lab5-pingok2.png
---
align: center
---
Ping is ok
```

## 3. CHANGE REQUEST

- Now that you have restored the connectivity, the BU2 DB owner has also asked you to filter out the route **10.0.211.0/24** from the routing table of the Azure Spoke2 GW.

```{important}
The subnet `10.0.211.0/24` is advertised by the OnPrem DC Router towards the Aviatrix MCNA through the Site2Cloud connection.

```{figure} ../../ace_ops/docs/images/lab5-newrequest2.png
---
align: center
---
S2C route
```

- Check the BGP Learned Routes

```{tip}
Go to **CoPilot > Diagnostics > Cloud Routes > BGP Info** and then click on the **3 dots** icon on the right-hand side of the screen and select the `"Show BGP Learned Routes"` option.
```

```{figure} ../../ace_ops/docs/images/lab5-bgpinfo.png
---
height: 400px
align: center
---
New request: remove the route
```

You are asked to maintain visible this route (therefore fully installed in the routing tables) on all the Spoke gateways, <ins>except the Azure Spoke2 GW!</ins>

```{figure} ../../ace_ops/docs/images/lab5-newrequest.png
---
height: 400px
align: center
---
New request: remove the route
```

- Verify first the presence of the route inside the routing table of the Azure Spoke2 GW.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} ../../ace_ops/docs/images/lab5-newrequest3.png
---
align: center
---
Remote route from the OnPrem DC
```

- Filter out the route 10.0.211.0/24 from the `Settings` section (i.e. *Route Manipulation* section).

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Settings** tab.
Expand the **Routing** section and insert the subnet **10.0.211.0/24** inside the `"Exclude Learned CIDRs to Spoke VPC/VNet Route Table"` option.

Do not forget to click on **Save**.
```

```{figure} ../../ace_ops/docs/images/lab5-newrequest4.png
---
align: center
---
Exclude the route
```

- Verify that the route has been withdrawn from the routing table of the Azure Spoke2 GW.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **_ace-azure-east-us-spoke2_** Gateway and then select the **Gateway Routes** tab.

Now search for `10.0.211.0/24`.
```

```{figure} ../../ace_ops/docs/images/lab5-newrequest5.png
---
align: center
---
Route
```

You can notice that route has been removed from the routing table, successfully.