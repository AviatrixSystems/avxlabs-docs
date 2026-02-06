# Lab 3 - FireNet - Interface

## 1. SCENARIO

BU1 and BU2 were able to communicate with each other as of Friday EOB (i.e end of LAB2).

However, the network team received a complaint from BU1 Frontend Team that the connectivity with BU2 Mobile App was no longer working.

```{warning}
The traffic between the two VPCs in AWS is inspected by a **Firewall** deployed through the Aviatrix **FireNet** feature.

The **`Inspection Policy`** has already been applied to each Spoke VPC in AWS at POD launch. Traffic to or from either AWS VPC is routed to the Firewall through the Transit FireNet Gateway for Deep Packet Inspection.

- Navigate to **CoPilot > Security > FireNet**, click on the _ace-aws-eu-west-1-transit1_ gateway and inspect the `"Policy"` tab.
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
Let’s dive into troubleshooting to figure out exactly where things are going wrong.

### 2.1 Verify connectivity Using Gatus

Access the **BU1 Frontend** Dashboard. You’ll notice East–West traffic is entirely blocked, impacting communication with the BU2 Mobile App and all other instances.

```{figure} images/lab3-4.28.diagnosticstools11901.png
---
height: 400px
align: center
---
All types of traffic are blocked
```

### 2.2 Verify connectivity using the Diagnostic Tools

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**. Select the **_ace-aws-eu-west-1-spoke1_** gateway (the gateway in front of the source endpoint). First, `ping` the private IP address of **BU2-MobileApp**, then perform a `traceroute` to that address.

```{tip}
Retrieve the private IP address of the **BU2 Mobile App** from the **Cloud Assets** inventory, first!

Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for `"mobile"`, then retrieve the private IP of the relevant EC2 instance.
```

```{figure} images/lab23-cp000.png
---
height: 300px
align: center
---
BU2-MobileApp is unreachable
```

```{figure} images/lab23-cp001.png
---
height: 400px
align: center
---
Traceroute outcome
```

```{important}
The traceroute results show a surprising outcome: **0** hops.
```

- Now try to ping both workloads (i.e. BU1 Frontend and BU2 Mobile App) from the **Transit** GW in AWS.

```{figure} images/lab3-bu1ping.png
---
height: 400px
align: center
---
Ping is successful from the Transit Gateway to BU1 Frontend.
```

```{figure} images/lab3-bu2ping.png
---
height: 300px
align: center
---
Ping is successful from the Transit Gateway to BU2 Mobile App.
```

From the outcome above, you can notice that the Transit GW in AWS can ping both BU1 Frontend and BU2 Mobile App, successfully.

### 2.3 Verify connectivity Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH into **BU1 Frontend**, run ping tests to **BU2-Mobile-App**, initiate an SSH session to **BU2-Mobile-App**, and run also a traceroute to **BU2-Mobile-App**.

```{tip}
Private IP addresses can be found in the `Cloud Assets` section.
```

```{figure} images/lab3-pingfails.png
---
align: center
---
Ping fails
```

```{figure} images/lab3-sshfails00.png
---
align: center
---
SSH fails
```

```{figure} images/lab3-sshfails01.png
---
align: center
---
Traceroute outcome
```

```{important}
The traceroute results show a surprising outcome: **1** hop. Only the Spoke gateway acting as the default gateway is responding to the traceroute.
```

### 2.4 Verify the Routing

- Check whether the relevant Spoke Gateways have the required routes installed on their respective routing tables or not.

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

```{tip}
Confirm that the **_ace-aws-eu-west-1-spoke2_** gateway includes a route to **10.1.211.0/24** in its routing table.
```

### 2.5 FireNet

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
height: 350px
align: center
---
The FW is down
```

#### 2.5.1 FireNet Workaround

- Apply a quick **workaround**. Before continuing the firewall troubleshooting, exclude from the East-West Inspection, either of the CIDRs of the two AWS Spoke VPCs.

```{tip}
Navigate to **CoPilot > Security > FireNet** and click on the **_ace-aws-eu-west-1-transit1_** GW.

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

#### 2.5.2 FireNet - Verification using Gatus

Open the Gatus Dashboard for **BU1 Frontend**. You’ll notice almost immediately that ICMP traffic and TCP traffic to ports 80 and 22 for the BU2 Mobile App have been restored, thanks to the workaround.

```{figure} images/lab3-4.28.diagnosticstools1190156.png
---
height: 400px
align: center
---
All kind fo traffic is blocked
```

#### 2.5.3 FireNet - Verification using the Diagnostic Tools

- Run `traceroute` directly from Spoke1 in EU-WEST-1, the gateway in the same VPC as the **Frontend** instance.

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**. Select the **_ace-aws-eu-west-1-spoke1_** gateway, then run traceroute to the private IP address of the **BU2 Mobile App** instance.

```{figure} images/lab3-tracemobileapp.png
---
align: center
---
Traceroute is ok
```

```{important}
The traceroute outcome shows the whole path from the source, the BU1 Frontend, till the recipient, the BU2 Mobile App.

<ins>Undoubtedly, the Firewall is excluded from this path!</ins>
```

#### 2.5.4 FireNet - Verification Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Alternatively, you can execute a `ping` from **BU1 Frontend** to **BU2 Mobile App** to verify connectivity; results should indicate a working connection.

```{figure} images/lab3-pingok2.png
---
align: center
---
Ping is ok
```

- Initiate a `traceroute` to the private IP address of the BU2 Mobile App.

```{figure} images/lab3-traceroute.png
---
align: center
---
Traceroute is ok
```

```{important}
The firewall is indeed excluded.
```

### 2.6 Discarding the workaround

Having identified the root cause and applied the workaround, we should remove the workaround and investigate the firewall-side configuration.

- <ins>Remove</ins> the _workaround_ and perform a thorough review of the Firewall configuration.

```{tip}
Navigate to **CoPilot > Security > FireNet**, click on the **_ace-aws-eu-west-1-transit1_** GW, then click on **Settings** and remove the CIDR `10.1.212.0/24`. 

Do not forget to click on **Save**.
```

```{figure} images/lab3-removeex.png
---
height: 250px
align: center
---
Remove the exemption
```

### 2.7 Health-check mechanism failure

Continue troubleshooting by using the packet capture feature on the transit gateway interface connected to the NGFW.

```{figure} images/lab3-ethernet01.png
---
height: 250px
align: center
---
eth-fn0 interface
```

#### 2.7.1 Connectivity test Using the Diagnostic Tools

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**, select the **_ace-aws-eu-west-1-spoke1_** gateway, and generate a ping to the **BU2 Mobile App** private IP address.

```{tip}
To capture packets on the Transit Gateway, duplicate the tab and start a packet capture on the **eth-fn0** interface <ins>before generating the ICMP traffic</ins>.
```

```{figure} images/lab23-cp111.png
---
height: 300px
align: center
---
Packet Capture
```

```{figure} images/lab23-cp112.png
---
height: 300px
align: center
---
Ping
```

```{figure} images/lab23-cp113.png
---
height: 300px
align: center
---
Packet Capture outcome
```

```{important}
You will observe only ICMP Echo Request packets initiated by the Transit Gateway (i.e., from its `eth-fn0` interface). You can reasonably conclude:

1) ECHO request packets generated by the Spoke1 Gateway are not visible on the LAN interface.

2) ECHO reply packets generated by the BU2 Mobile App are not visible on the LAN interface.

3) ECHO reply packets generated by the NGFW are not visible on the LAN interface.
```

#### 2.7.2 Connectivity test Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Alternatively, maintain an open SSH session on the **BU1 Frontend** and generate ICMP Echo requests to the private IP address of the **BU2 Mobile App**.

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

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**. Select the **_ace-aws-eu-west-1-transit1_** gateway, then choose **`Packet Capture`** from the available features. Select the `eth-fn0` interface, set the duration to **10** seconds, and click **Run**. When the results appear, you can filter by **"icmp"**.

```{figure} images/lab3-ethernet03.png
---
height: 250px
align: center
---
packet capture
```

The observed traffic consists solely of ICMP echo requests issued by the Transit Gateway to the firewall’s LAN interface. The firewall is not producing echo replies; consequently, the Transit Gateway has no alternative but to drop the ICMP traffic generated by the BU1 Frontend, as the health-check mechanism has clearly failed.

### 2.8 NGFW issue remediation <span style='color:#33ECFF'>(BONUS)</span></summary>

In the next two sections, you will be asked to access the Fortinet firewall to resolve the configuration issue. You will be prompted to accept a self-signed certificate. If you are using a corporate laptop, please proceed with task **2.8.1** (<span style='color:#33ECFF'>Guacamole Client</span>); otherwise, proceed with task **2.8.2**.

#### 2.8.1 NGFW Misconfiguration <span style='color:#33ECFF'>(BONUS)</span></summary>

Accessing the NGFW will require accepting a self-signed certificate. In the Personal POD Portal, navigate to the POD Portal, identify the `Jumpbox button`, click it, and enter your credentials to access the Guacamole client.

```{figure} images/lab3-newportal909.png
---
align: center
---
Jumpbox
```

```{figure} images/lab3-newportal9091.png
---
align: center
---
Credentials
```

```{figure} images/lab3-newportal90911.png
---
align: center
---
Browser
```

```{figure} images/lab3-newportal909112.png
---
align: center
---
NGFW URL
```

#### 2.8.2 NGFW Misconfiguration

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

### 2.9 Final Verification

Let's proceed to the final verification after the firewall problem is resolved.

#### 2.9.1 Verify connectivity Using Gatus

Reopen the **BU1 Frontend** Gatus Dashboard. All traffic flows have been restored.

```{figure} images/lab3-4.28.diagnosticstools1190189.png
---
height: 400px
align: center
---
All types of traffic have been restored
```

#### 2.9.2 Verify connectivity Using the Diagnostic Tools

- Rerun ping and traceroute from **_ace-aws-eu-west-1-spoke1_** (`Diagnostics > Diagnostic Tools`) to **BU2-MobileApp**.

```{figure} images/lab3-traceroute280.png
---
align: center
---
New Traceroute outcome from the Spoke1
```

```{figure} images/lab3-traceroute28089.png
---
align: center
---
New Ping outcome from the Spoke1
```

You have successfully fixed the issue and restored connectivity.

#### 2.9.3 Verify connectivity Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

Reattempt the ping from **BU1 Frontend** to **BU2 Mobile App** once the firewall’s LAN interface has been reactivated.

```{figure} images/lab3-pingworks.png
---
align: center
---
Ping is ok!
```

- From the **BU1 Frontend** SSH session (still open), rerun the traceroute to **BU2 Mobile App**.

```{figure} images/lab3-traceroute2.png
---
align: center
---
New Traceroute outcome
```

```{important}
This time, the traceroute shows the IP address of the Transit FireNet Gateway (e.g., **.186**, as in the example screenshot) twice along the path. This happens because the traffic is diverted to the firewall for _Deep Packet Inspection_, and after approval, the firewall sends the traffic back to the Transit FireNet Gateway.
```
