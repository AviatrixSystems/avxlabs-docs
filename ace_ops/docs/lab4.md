# Lab 4 - FireNet - Routes

## 1. SCENARIO

BU1 and BU2 were able to communicate as end of Lab3.

Unfortunately, the network team received another complaint from BU1 Frontend Team that BU2 Mobile App was no longer reachable.

```{figure} images/lab4-topology.png
---
height: 400px
align: center
---
Lab 4 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH to BU1 Frontend and launch ping/ssh to BU2 Mobile App.

```{figure} images/lab4-pingunsucc.png
---
align: center
---
Ping fails
```

- Check whether both **AWS Spoke1** and **AWS Spoke2** have the required routes installed in their RTBs or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **ace-aws-eu-west-1-spoke1** gateway for instance, and filter out based on the remote route.
```

```{figure} images/lab4-filter.png
---
align: center
---
Filter out
```

From the outcome above, once again it is evident that Spoke1 in AWS has the destination route in its RTB.

Try to verify the inverse, checking the destination route from the **AWS Spoke2** side.

- This time launch the **Diagnostics Tools** directly from the *`Topology`* section.
- From the *Spoke1 Gateway* in AWS, try to ping/traceroute the instance behind the other spoke (i.e. BU2 Mobile App).

```{tip}
Go to **CoPilot > Cloud Fabric > Topology**

Expand the **_ace-aws-eu-west-1-spoke1_** VPC, clicking on its corresponding icon, select the **_ace-aws-eu-west-1-spoke1_** Spoke GW and then click on the `Tools` button within the **Properties** window and click on `Gateway Diagnostics`.

<ins>Repeat this operation</ins>: from the **_ace-aws-eu-west-1-spoke2_** Spoke GW, try to ping the private IP address of the BU1 Frontend too.
```{figure} images/lab4-mapdiag1.png
---
align: center
---
Diagnostics Tools from the Topology
```

```{figure} images/lab4-pingfails2.png
---
align: center
---
Ping fails from ace-aws-eu-west-1-spoke1 towards BU2 Mobile App
```

```{figure} images/lab4-pingfails.png
---
align: center
---
Ping fails from ace-aws-eu-west-1-spoke2 towards BU1 Frontend
```

- Now try to ping both workloads (i.e. BU1 Frontend and BU2 Mobile App) from the **Transit** GW in AWS.

```{figure} images/lab3-bu1ping.png
---
align: center
---
Ping is ok from the Transit GW towards BU1 Frontend
```

```{figure} images/lab3-bu2ping.png
---
align: center
---
Ping is ok from the Transit GW towards BU2 Mobile App
```

From the outcome above, you can notice that the Transit GW in AWS can ping both BU1 Frontend and BU2 Mobile App, successfully.

- Let's check the **FireNet** section!

```{tip}
Go to **CoPilot > Security > FireNet**
```

```{figure} images/lab4-firewallok.png
---
align: center
---
The FW is UP
```

```{note}
This time the FW is responding properly to the **Health Check** mechanism, therefore we can't blame the Security Team.

The Firewall is reachable from the Transit FireNet GW!
```

- Keep running the ping command from the **BU1 Frontend** towards the **BU2 Mobile App**, although it will fail!

```{figure} images/lab4-keepit.png
---
align: center
---
Keep the ping running...
```

- Now let's launch a *packet capture* on the `eth2` interface of the Transit FireNet GW.

```{important}
The **eth2** interface is hooked up to the **LAN** interface of the Firewall, within the **_transit1-dmz-firewall_** subnet!
```

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**, select the **_ace-aws-eu-west-1-transit1_** GW, then select the `Packet Capture` tool.

```{figure} images/lab4-packet1.png
---
align: center
---
Packet Capture
```


Now click on the `"Column filter"` icon, and **uncheck** the following parameters (you are going to collect **_ICMP traffic_**, therefore you do not need to get the values for all the parameters!):

  - <span style='color:#33ECFF'>Uncheck Source Port</span>
  - <span style='color:#33ECFF'>Uncheck Dest Port</span>
  - <span style='color:#33ECFF'>Uncheck Flags</span>
  - <span style='color:#33ECFF'>Uncheck Length</span>
  
```{figure} images/lab4-packet2.png
---
align: center
---
Restrict the visibility of the Packet Capture to the required parameters
```

Choose the `"eth2"` interface and diminish the `"Duration"` of the packet capture to **10** seconds!

Then click on **Run**.

```{figure} images/lab4-packet3.png
---
align: center
---
Only ICMP Echo Request packets have been captured...
```

```{important}
You will notice only **ECHO Request** packets captured on the eth2 interface of the Transit FireNet GW.

The FW is not forwarding the traffic back! The **ECHO Reply** packets <ins>get dropped at the FW</ins>!
```

- Let's check the `Vendor Integration`!

```{tip}
Go to **CoPilot > Security > FireNet > Firewall**, click on the **_ACE-FW_** firewall.
```

```{figure} images/lab4-vendor1.png
---
align: center
---
10.0.0.0/8 is not installed in the RTB of the FW
```

```{note}
You will discover immediately that one of the three **RFC1918 routes** had been removed (i.e. `10.0.0.0/8`).
```

- Click on the `"Sync Routes to Firewall"` button, to carry out the **Vendor Integration** and the **Aviatrix Controller** will reinject the missing route into the routing table of the Firewall!

```{figure} images/lab4-vendor2.png
---
align: center
---
10.0.0.0/8 has been successfully reinjected into the FW's rtb!
```

- Relaunch the ping from **BU1 Frontend** towards **BU2 Mobile App**.

```{figure} images/lab4-pingworks.png
---
align: center
---
Ping is ok
```
