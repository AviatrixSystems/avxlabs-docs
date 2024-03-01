# Lab 3 - FireNet - Interface

## 1. SCENARIO

BU1 and BU2 were able to communicate with each other as of Friday EOB (i.e end of LAB2).

However, the network team received a complaint from BU1 Frontend Team that the connectivity with BU2 Mobile App was no longer working.

```{warning}
The traffic between the two VPCs in AWS is inspected by a **Firewall** deployed through the Aviatrix **FireNet** feature.

The **`Inspection Policy`** had been already applied on each Spoke VPC in AWS, at the launch of the PODs. The traffic generating from or going to either of the AWS VPCs will be sent to the Firewall by the Transit FireNet Gateway for carrying out the Deep Packet Inspection.

```{figure} images/lab3-policy2.png
---
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

  - SSH to BU1 Frontend and launch ping/ssh towards BU2 Mobile App.

```{figure} images/lab3-pingfails.png
---
align: center
---
Ping fails
```

- Check whether the relevant Spoke Gateways have the required routes installed on their routing tables or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select for example the gateway **ace-aws-eu-west-1-spoke1**  **> Gateway Routes** and search for the subnet **10.1.212.0/24**, where BU2 Mobile App resides.
```

```{figure} images/lab3-routecheck.png
---
align: center
---
Filtering out
```

From the outcome above, it is evident that Spoke1 in AWS has the destination route in its RTB. You can verify the inverse, inspecting the Routing Table of  the Spoke2 in AWS.

- Use **Diagnostics Tools** from the *Spoke1 Gateway* in AWS and try to ping/traceroute the instance behind the other spoke (i.e. BU2 Mobile App).

```{tip}
Retrieve the private IP address of the **BU2 Mobile App** from the Cloud Assets inventory, first!

Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for `"mobile"`, then retrieve the private IP of the interested EC2 instance.
```

```{figure} images/lab3-personalpod.png
---
align: center
---
Private IP of BU2 Mobile App
```

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools** then select the GW **ace-aws-eu-west-1-spoke1** and ping the private IP of BU2 Mobile App
```

```{figure} images/lab3-pingfailss.png
---
align: center
---
Ping is unsuccessful
```

From the outcome above you can notice that from the Spoke1 GW in AWS, the BU2 Mobile App is not reachable.

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

```{figure} images/lab3-unreachable.png
---
align: center
---
The FW is unreachable
```

Click on the **Firewall** tab! You will notice that the Firewall turns out being down.

```{important}
The ICMP health check is initiated every **5** seconds from the Aviatrix Transit FireNet Gateway. The 5-second setting is the default and cannot be changed.
```

```{figure} images/lab3-fwdown.png
---
align: center
---
The FW is down
```

- Apply a quick **workaround**. Before exploring the firewall configuration, exclude from the East-West Inspection, either of the CIDRs of the two AWS Spoke VPCs.

```{tip}
Go to **CoPilot > Security > FireNet > FireNet Gateways** and click on the **_ace-aws-eu-west-1-transit1_** GW.

Then click on **Settings** and insert the CIDR of the AWS **ace-aws-eu-west-1-spoke2** VPC on the field `"Exclude from East-West Inspection"`. 

Do not forget to click on **Save**.
```

```{figure} images/lab3-firenetgw.png
---
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

- Issue the ping towards the **BU2 Mobile App** from the **BU1 Frontend**. This time the connectivity will work!

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

- Now remove the workaround and explore the Firewall configuration!

```{tip}
Go to **CoPilot > Security > FireNet > FireNet Gateways**, click on the **_ace-aws-eu-west-1-transit1_** GW, then click on **Settings** and remove the CIDR `10.1.212.0/24`. 

Do not forget to click on **Save**.
```

```{figure} images/lab3-removeex.png
---
align: center
---
Remove the exemption
```

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
Go to **FW > Network > Interfaces** and click the LAN interface (port2) and fix the issue!
```

The port was disabled to simulate a software failure of the FW...

Double click on the greyed-out interface LAN (port2), or alternatively, select the interface and click on the **Edit** button.

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

Relaunch the ping from **BU1 Frontend** towards **BU2 Mobile App**, after having <ins>reactivated</ins> the LAN interface of the firewall.

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
Ping is ok!
```

```{important}
This time the traceroute outcome shows twice the IP address of the Transit FireNet GW (i.e. **.134** as shown in the screenshot above) along the path, due to the fact that the traffic is now diverted towards the Firewall that, after the completion of the Deep Packet Inspection, sends the permitted traffic back to the Transit FireNet GW.
```
