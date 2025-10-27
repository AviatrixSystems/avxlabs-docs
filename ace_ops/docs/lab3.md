# Lab 3 - FireNet - Interface

## 1. SCENARIO

BU1 and BU2 were able to communicate with each other as of Friday EOB (i.e end of LAB2).

However, the network team received a complaint from BU1 Frontend Team that the connectivity with BU2 Mobile App was no longer working.

```{warning}
The traffic between the two VPCs in AWS is inspected by a **Firewall** deployed through the Aviatrix **FireNet** feature.

The **`Inspection Policy`** had been already applied on each Spoke VPC in AWS, at the launch of the PODs. The traffic generating from or going to either of the AWS VPCs will be sent to the Firewall by the Transit FireNet Gateway for carrying out the Deep Packet Inspection.

```{figure} images/lab3-policy2.png
---
height: 300px
align: center
---
Inspection Policies on the FireNet Transit GW
```

```{figure} images/lab3-topology.png
---
height: 400px
align: center
---
Lab 3 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH to **BU1 Frontend** and run ping and SSH tests toward **BU2 Mobile App**.

```{figure} images/lab3-pingfails.png
---
align: center
---
Ping fails
```

- Check whether the relevant Spoke Gateways have the required routes installed on their routing tables or not.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select for example the gateway **ace-aws-eu-west-1-spoke1**  **> Gateway Routes** and search for the subnet **10.1.212.0/24**, where BU2 Mobile App resides.
```

```{figure} images/lab3-routecheck.png
---
height: 300px
align: center
---
Filtering out
```

From the above outcome, Spoke1 in AWS has the destination route in its RTB. You can verify the inverse by inspecting the Routing Table of Spoke2 in AWS.

- Use **Diagnostics Tools** from the *Spoke1 Gateway* in AWS and try to ping/traceroute the instance behind the other spoke (i.e. **BU2 Mobile App**).

```{tip}
Retrieve the private IP address of the **BU2 Mobile App** from the **Cloud Assets** inventory, first!

Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for `"mobile"`, then retrieve the private IP of the relevant EC2 instance.
```

```{figure} images/lab3-personalpod.png
---
height: 300px
align: center
---
Private IP of BU2 Mobile App
```

```{tip}
Navigate to **CoPilot > Diagnostics > Diagnostics Tools** then select the GW **ace-aws-eu-west-1-spoke1** and ping the private IP of BU2 Mobile App
```

```{figure} images/lab3-pingfailss.png
---
height: 300px
align: center
---
Ping is unsuccessful
```

From the outcome above you can notice that from the Spoke1 GW in AWS, the BU2 Mobile App is not reachable.

- Now try to ping both workloads (i.e. BU1 Frontend and BU2 Mobile App) from the **Transit** GW in AWS.

```{figure} images/lab3-bu1ping.png
---
height: 400px
align: center
---
Ping is ok from the Transit GW towards BU1 Frontend
```

```{figure} images/lab3-bu2ping.png
---
height: 400px
align: center
---
Ping is ok from the Transit GW towards BU2 Mobile App
```

From the outcome above, you can notice that the Transit GW in AWS can ping both BU1 Frontend and BU2 Mobile App, successfully.

- Let's check the **FireNet** section!

```{tip}
Navigate to **CoPilot > Security > FireNet**
```

```{figure} images/lab3-unreachable.png
---
height: 250px
align: center
---
The FW is unreachable
```

Open the **Firewall** tab; the firewall appears to be down.

```{important}
The ICMP health check is initiated every **5** seconds from the Aviatrix Transit FireNet Gateway. The 5-second setting is the default and cannot be changed.
```

```{figure} images/lab3-fwdown.png
---
height: 250px
align: center
---
The FW is down
```

- Apply a quick **workaround**. Before continuing the firewall troubleshooting, exclude from the East-West Inspection, either of the CIDRs of the two AWS Spoke VPCs.

```{tip}
Navigate to **CoPilot > Security > FireNet > FireNet Gateways** and click on the **_ace-aws-eu-west-1-transit1_** GW.

Then click on **Settings** and insert the CIDR of the AWS **ace-aws-eu-west-1-spoke2** VPC on the field `"Exclude from East-West Inspection"`. 

Do not forget to click on **Save**.
```

```{figure} images/lab3-firenetgw.png
---
height: 250px
align: center
---
The Transit FireNet GW
```

```{figure} images/lab3-excemption2.png
---
align: center
---
Exclude the CIDR
```

- Execute a ping from **BU1 Frontend** to **BU2 Mobile App** to verify connectivity; results should indicate a working connection.

```{figure} images/lab3-pingok2.png
---
align: center
---
Ping is ok
```

- Launch a **traceroute** towards the Private IP address of the BU2 Mobile App.

```{figure} images/lab3-traceroute.png
---
align: center
---
Traceroute is ok
```

```{important}
The traceroute outcome shows the whole path from the source, the BU1 Frontend, till the recipient, the BU2 Mobile App.

<ins>Undoubtedly, the Firewall is excluded from this path!</ins>
```

- <ins>Remove</ins> the _workaround_ and perform a thorough review of the Firewall configuration.


```{tip}
Navigate to **CoPilot > Security > FireNet > FireNet Gateways**, click on the **_ace-aws-eu-west-1-transit1_** GW, then click on **Settings** and remove the CIDR `10.1.212.0/24`. 

Do not forget to click on **Save**.
```

```{figure} images/lab3-removeex.png
---
height: 250px
align: center
---
Remove the exemption
```

### 2.1 Health-check mechanism failure

Continue troubleshooting using the packet capture feature on the interface of the transit gateway attached to the NGFW.

```{figure} images/lab3-ethernet01.png
---
height: 250px
align: center
---
eth-fn0 interface
```

- Maintain a continuous ping from the BU1 Frontend session on your SSH client to the private IP of the BU2 Mobile App.

```{figure} images/lab3-ethernet02.png
---
height: 250px
align: center
---
ping
```

```{important}
Refer always to your personal POD Portal for the IP addresses!
```

- Navigate to **CoPilot > Diagnostics > Diagnostics Tools**. Select the **_ace-aws-eu-west-1-transit1_** gateway, then choose **`Packet Capture`** from the available features. Select the `eth-fn0` interface, set the duration to **10** seconds, and click **Run**. When the results appear, you can filter by "icmp."


```{figure} images/lab3-ethernet03.png
---
height: 250px
align: center
---
packet capture
```

The observed traffic consists solely of ICMP echo requests issued by the Transit Gateway to the firewall’s LAN interface. The firewall is not producing echo replies; consequently, the Transit Gateway has no alternative but to drop the ICMP traffic generated by the BU1 Frontend, as the health-check mechanism has clearly failed.

### 2.2 NGFW Misconfiguration

- Log in to the FW (<ins>refer to your POD portal for the credentials</ins>) and explore the configuration.

```{note}
You can access the Firewall in two ways:
1) From the URL available on the **POD Portal**.
2) **CoPilot > Security > FireNet > Firewall** and click on the **URL**. 
```

```{figure} images/lab3-newportal.png
---
align: center
---
POD Portal
```

```{figure} images/lab3-url.png
---
height: 200px
align: center
---
FW URL from the FireNet section on the CoPilot
```

- Insert the credentials.

```{figure} images/lab3-fw.png
---
align: center
---
FW access
```

```{tip}
Navigate to **FW > Network > Interfaces** and click the LAN interface (_port2_) and fix the issue!
```

The port was disabled to simulate a software failure of the FW...

Double click on the greyed-out interface **LAN (port2)**, or alternatively, select the interface and click on the **Edit** button.

```{figure} images/lab3-fwint.png
---
align: center
---
FW interface
```

Scroll down until you see the `"Status"` button of the interface and finally reactivate it.

Of course, then click on **OK**.

```{figure} images/lab3-enableint.png
---
align: center
---
Reactivate the LAN port
```

Reattempt the ping from **BU1 Frontend** to **BU2 Mobile App** once the firewall’s LAN interface has been reactivated.

```{figure} images/lab3-pingworks.png
---
align: center
---
Ping is ok!
```

Relaunch also the traceroute from **BU1 Frontend** towards **BU2 Mobile App**.

```{figure} images/lab3-traceroute2.png
---
align: center
---
New Traceroute outcome
```

```{important}
This time, the traceroute shows the IP address of the Transit FireNet Gateway (e.g., **.143**, as in the example screenshot) twice along the path. This happens because the traffic is diverted to the firewall for _Deep Packet Inspection_, and after approval, the firewall sends the traffic back to the Transit FireNet Gateway.
```
