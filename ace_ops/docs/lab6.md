# Lab 6 - Site2Cloud

## 1. SCENARIO#1

ACE’s OnPrem Data Center has recently hired a new network engineer.

You have been engaged for activating the <span style='color:orange'>**“Route Approval”**</span> feature in order to protect the Aviatrix **CNSF** from _unauthorized BGP advertisements_.

```{figure} images/lab6-topology.png
---
height: 400px
align: center
---
Lab 6 Scenario#1: Topology
```

## 2. CHANGE REQUEST - BGP Route Approval

- Activate the **Route Approval** feature for monitoring unauthorized advertisements that could be received by the DC.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **_ace-aws-eu-west-1-transit1_** GW **> Settings > Border Gateway Protocl (BGP)**, and enable the `Manual Learned CIDR Approval`
 toggle.

Then click on **Save**.
```

```{figure} images/lab6-routeapproval.png
---
height: 400px
align: center
---
Route Approval
```

Afterward, inform the trainer that you have activated the feature using the <span style='color:orange'>**“Raise Hand”**</span> tool on Zoom, as shown below, and then type your POD number in the Zoom chat!

```{figure} images/lab6-raise.png
---
align: center
---
Raise Hand tool on Zoom
```

```{figure} images/lab6-inform.png
---
align: center
---
Communicate your POD number
```

- Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the *ace-aws-eu-west-1-transit1* GW and then the **Route Approval** tab.

You’ll immediately notice three CIDRs already approved:

- 10.0.0.0/24  

- 10.0.111.0/24  

- 10.0.211.0/24

And one CIDR in a pending state:

- 169.254.74.128/30

Keep clicking Refresh—a new CIDR should appear shortly.

```{figure} images/lab6-pending000.png
---
height: 400px
align: center
---
Existing CIDRs
```

```{note}
Please wait approximately **one or two minutes** for BGP to send the Update. Then, click the <span style='color:orange'>**refresh button**</span> to see the default route being advertised from the Data Center.

This route will remain in a <ins>**pending state**</ins> and will not be advertised within the CNSF until it receives final approval from the *Aviatrix Administrator*.

**Important**: Do not approve the route yourself. If you accidentally approve it, you can click "Remove" to revert it back to the Pending status.
```

```{figure} images/lab6-pending.png
---
height: 400px
align: center
---
Refresh
```

```{caution}
**Do not approve the default route**. If you accidentally approve it, you can click "Discard" to revert it to Pending.
```

```{important}
You have successfully mitigated a potential threat in the Data Center that could have jeopardized the entire network across our multicloud environment.

<ins>Once again, please refrain from approving the default route</ins>.
```

## 3. SCENARIO#2

ACE’s On-Premises Partner needs to be connected to the Aviatrix CNSF in the GCP region. However, there is an IP overlap with BU1’s Analytics VPC.

You have been engaged to establish a <span style='color:#FF00FF'>**Site2Cloud**</span> connection between the GCP Spoke Gateway and the On-Premises Partner router, as well as to resolve the **IP conflict** using the **Mapped NAT** feature.

```{figure} images/lab6-topology2.png
---
height: 400px
align: center
---
Lab 6 Scenario#2: Topology
```

## 4. CHANGE REQUEST - Site-To-Cloud

- Create a new **S2C** connection.

```{tip}
Navigate to **CoPilot > Networking >  Connectivity > External Connection (S2C) >** then click on the  `"+ External Connection"` button, then select `"External Device"`.
```

```{figure} images/lab6-s2c.png
---
height: 200px
align: center
---
New S2C
```

```{figure} images/lab6-s2c100.png
---
height: 400px
align: center
---
External Device
```

Configure the new Site-to-Cloud (S2C) connection according to the schema below:

- **Name**: <span style='color:#479608'>S2C-PARTNER</span>

- **Type**: <span style='color:#479608'>Static Routing over IPsec</span>

- **Static Routing Type**: <span style='color:#479608'>Mapped</span>

- **Local Gateway**: <span style='color:#479608'>ace-gcp-us-east1-spoke1</span>

- **Real Local Subnet CIDR(s)**: <span style='color:#479608'>172.16.211.0/24</span>

- **Virtual Local Subnet CIDR(s)**: <span style='color:#479608'>192.168.1.0/24</span>

- **Remote Gateway Type**: <span style='color:#479608'>Generic</span>

- **Real Remote Subnet CIDR(s)**: <span style='color:#479608'>172.16.211.0/24</span>

- **Virtual Remote Subnet CIDR(s)**: <span style='color:#479608'>192.168.2.0/24</span>

- **Internet Key Exchange**: <span style='color:#479608'>IKEv2</span>

- **Authentication (Method)**: <span style='color:#479608'>Pre-Shared Key</span>

- **Remote Device 1 Tunnel Destination IP**:  <span style='color:tomato'>please refer to the Note below</span>

```{note}
Use the “**dig partner-csr-public.pod#.aviatrixlab.com +short**” command <ins>from your personal laptop terminal</ins> to resolve the symbolic public name of the OnPrem-Partner CSR router and retrieve the <ins>REMOTE GATEWAY PUBLIC IP address</ins>, as depicted in the example below.
```

```{figure} images/lab6-topologyas.png
---
height: 400px
align: center
---
Onprem Partner 
```

<ins>Replace the **#** symbol with your POD number!</ins>

The example is referring to POD **#2** (please issue the command based on your POD number!).

```{figure} images/lab6-podnumber.png
---
align: center
---
Retrieving the Public IP
```

```{tip}
For **Windows OS** you can use the command `"nslookup"`:

**nslookup partner-csr-public.pod#.aviatrixlab.com**

```{figure} images/lab6-nslookup.png
---
align: center
---
Nslookup 
```
  -  **Pre-Shared Key**: <span style='color:#479608'>Aviatrix123#</span>

```{important}
Do not forget to click on **Save**.
```

```{figure} images/lab6-finals2c.png
---
align: center
---
External Connection Configuration
```

Please wait a few seconds for the completion of the S2C. The new connection will appear as **"Down"**, indicated by a red arrow symbol.

```{figure} images/lab6-notdone.png
---
height: 250px
align: center
---
S2C is establishing the connection
```

Click on the <span style='color:orange'>**refresh**</span>
 button to see the status changing from red to green.

```{figure} images/lab6-s2cok.png
---
height: 250px
align: center
---
S2C is finally UP
```

### 4.1 Verifiy connectivity Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- If you need additional control, SSH to the **OnPrem partner** router (i.e., a Cisco router).

```{tip}
Access your personal POD Portal, locate the `"SSH widget for Cisco CSR"`, and use the provided credentials to connect to the CSR router as an On-Prem Partner.
```{figure} images/lab6-csrrouter.png
---
align: center
---
Cisco Router
```

After successfully logging into the On-Premises Partner router (also known as the _CSR router_), execute the following command to verify that the tunnel is active:

```bash
show ip int brief
```

```{figure} images/lab6-tunnelup.png
---
align: center
---
Tunnel1 up/up
```

Then from the **OnPrem Partner** router issue the following command:

```bash
ping 192.168.1.100 source gigabitethernet1
```

```{figure} images/lab6-pingok2.png
---
align: center
---
Ping is ok
```

- Execute the `Active Sessions` command on the **GCP Spoke Gateway**!

```{tip}
Navigate to **CoPilot > Diagnostics > Diagnostic Tools > Gateway Diagnostics**, select the **_ace-gcp-us-east1-spoke1_** GW and then select the **Active Sessions** tool.

Click on **Run** and almost simultaneously issue once again the ping command from the Partner CSR router.

Filter the results using the `"icmp"` keyword.
```

```{figure} images/lab6-final.png
---
height: 200px
align: center
---
Mapped NAT in action !
```
