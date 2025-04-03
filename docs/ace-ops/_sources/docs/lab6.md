# Lab 6 - Site2Cloud

## 1. SCENARIO#1

ACE’s OnPrem Data Center has recently hired a new network engineer.

You have been engaged for activating the <span style='color:orange'>**“Route Approval”**</span> feature in order to protect the MCNA from unauthorized advertisements.

```{figure} images/lab6-topology.png
---
height: 400px
align: center
---
Lab 6 Scenario#1: Topology
```

## 2. CHANGE REQUEST

- Activate the **Route Approval** feature for monitoring unauthorized advertisements that could be received by the DC.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **_ace-aws-eu-west-1-transit1_** GW **> Settings > Border Gateway Protocl (BGP) >** and turn on the **_Gateway Learned CIDR Approval_** knob.

Then click on **Save**.
```

```{figure} images/lab6-routeapproval.png
---
height: 400px
align: center
---
Route Approval
```

Afterwards, inform the trainer that you have activated the feature with the tool <span style='color:orange'>**“Raise Hand”**</span> on Zoom, as depicted below, and type the number of your POD in the Zoom chat!

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

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the *ace-aws-eu-west-1-transit1* GW **> Approval**

```{note}
Wait for approximately **one minute** for BGP in order to send the Update. Then click on the <span style='color:orange'>**refresh button**</span> to see a <ins>default route</ins> being advertised from the DC.

This route will remain in <ins>**pending state**</ins> and it will be not advertised within the MCNA untill it gets the final approval from the Aviatrix Administrator.

**Do not approve it**! If you accidentally approve it, you can click on **Remove** and store it back on the Pending status.
```

```{figure} images/lab6-pending.png
---
height: 400px
align: center
---
Refresh
```

```{important}
You have successfully prevented that somebody from the DC could jeopardize the whole network inside the multicloud infrastructure! 

<ins>Once again do not approve that default route</ins>!
```

## 3. SCENARIO#2

ACE’s OnPrem Partner needs to be connected to the MCNA in the GCP region, however, it has overlapping IP’s with BU1’s Analytics VPC.

You have been engaged for creating a <span style='color:#FF00FF'>**Site2Cloud**</span>
 connection between the GCP Spoke GW and the OnPrem Partner router and also for resolving the IP conflict through the **Mapped NAT** feature.

```{figure} images/lab6-topology2.png
---
height: 400px
align: center
---
Lab 6 Scenario#2: Topology
```

## 4. CHANGE REQUEST

- Create a new **S2C** connection.

```{tip}
Go to **CoPilot > Networking >  Connectivity > External Connection (S2C) >** then click on the  `"+External Connection"` button, then select `"External Device"`.
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
height: 200px
align: center
---
External Device
```

Configure the new S2C connection based on the schema below.

- **Name**: <span style='color:#479608'>S2C-PARTNER</span>

- **Connect Using:**: <span style='color:#479608'>Static-Route Based</span>
- **Type:**: <span style='color:#479608'>Mapped</span>

- **Local Gateway**: <span style='color:#479608'>ace-gcp-us-east1-spoke1</span>

- **Real Local Subnet CIDR(s)**: <span style='color:#479608'>172.16.211.0/24</span>

- **Virtual Local Subnet CIDR(s)**: <span style='color:#479608'>192.168.1.0/24</span>

- **Remote Gateway Type**: <span style='color:#479608'>Generic</span>

- **Real Remote Subnet CIDR(s)**: <span style='color:#479608'>172.16.211.0/24</span>

- **Virtual Remote Subnet CIDR(s)**: <span style='color:#479608'>192.168.2.0/24</span>

- **Authentication Method**: <span style='color:#479608'>PSK</span>

- **IKEv2**: <span style='color:#479608'>ON</span>

- **Remote Gateway IP**:  <span style='color:tomato'>follow the Note below</span>

```{note}
Use the “**dig partner-csr-public.pod#.aviatrixlab.com +short**” command <ins>from your personal laptop terminal</ins> to resolve the symbolic public name of the OnPrem-Partner CSR router and retrieve the <ins>REMOTE GATEWAY PUBLIC IP address</ins>, as depicted in the example below.
```

<ins>Replace the **#** symbol with your POD number!</ins>

The example is referring to POD #35 (please issue the command based on your POD number!).

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

  - **Local Gateway Instance**: <span style='color:#479608'>ace-gcp-us-east1-spoke1</span>
  -  **Local Tunnel IP**: <span style='color:#479608'>169.254.0.1/30</span>
  -  **Remote Tunnel IP**: <span style='color:#479608'>169.254.0.2/30</span>
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

Please wait a few seconds for the completion of the S2C. The new connection will appear as down (indicated by a red arrow symbol).

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

- Now it's time to **SSH** to the **OnPrem partner** router!

```{tip}
Access your personal POD Portal, locate the `"SSH widget for Cisco CSR"`, and use the provided credentials to connect to the CSR router as an On-Prem Partner.
```{figure} images/lab6-csrrouter.png
---
align: center
---
Cisco Router
```

After successfully logging into the On-Prem Partner router (also known as the _CSR router_), run the following command to verify that the tunnel is operational:

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

- Launch the `Active Sessions`.

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**, select the **_ace-gcp-us-east1-spoke1_** GW and then select the **Active Sessions** tool.

Click on **Run** and almost simultaneously issue once again the ping command from the CSR router.

Filter based on the `"icmp"` keyword.
```

```{figure} images/lab6-final.png
---
height: 300px
align: center
---
Mapped NAT in action !
```
