# Lab 6 - Site2Cloud

## 1. SCENARIO#1

ACE’s OnPrem Data Center has recently hired a new network engineer.

You have been engaged for activating the <span style='color:#00FFFF'>**Route Approval**</span> feature in order to protect the MCNA from unauthorized advertisements.

![topology 6](images/lab6-topology.png)
_Figure 70: Lab 5 Scenario#1 Topology_

## 2. CHANGE REQUEST

- Activate the **Route Approval** feature for monitoring unauthorized advertisements that could be received by the DC.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the *ace-aws-eu-west-1-transit1* GW **> Settings > Border Gateway Protocl (BGP) >** and turn on the **_Gateway Learned CIDR Approval_** knob.
> Then click on **Save**.
```

![S2C](images/lab6-routeapproval.png)
_Figure 71: Route Approval_

Afterwards, inform the trainer that you have activated the feature with the tool <span style='color:#FFFF00'>**“Raise Hand”**</span> on Zoom, as depicted below.

![Zoom Raise Hand](images/lab6-raise.png)
_Figure 72: Raise Hand tool on Zoom_

<ins>Please wait for the trainer to inform you about the injection of the failure!</ins>

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the *ace-aws-eu-west-1-transit1* GW **> Approval**

```{note}
Wait for approximately **one minute** for BGP in order to send the Update. Then click on the <span style='color:orange'>**refresh button**</span> to see a default route being advertised from the DC.
This route will remain in <ins>pending state</ins> and it will be not advertised within the MCNA untill it gets the final approval from the Aviatrix administrator.
```

![Pending](images/lab6-pending.png)
_Figure 73: Refresh_

```{important}
You have successfully prevented that somebody from the DC could jeopardize the whole network inside the multicloud infrastructure! Of course do not approve that default route!
```

## 3. SCENARIO#2

ACE’s OnPrem Partner needs to be connected to the MCNA in the GCP region, however, it has overlapping IP’s with BU1’s Analytics VPC.

You have been engaged for creating a <span style='color:#FF00FF'>**Site2Cloud**</span>
 connection between the GCP Spoke GW and the OnPrem Partner router and also for resolving the IP conflict through the **Mapped NAT** feature.

![topology6-2](images/lab6-topology2.png)
_Figure 74: Lab 6 Scenario#2 Topology_

## 4. CHANGE REQUEST

- Create a new **S2C** connection.

```{tip}
Go to **CoPilot > Networking >  Connectivity > External Connection (S2C) >** then click on the  **+External Connection** button.
```

![New S2C](images/lab6-s2c.png)
_Figure 75: New S2C_

Configure the new S2C connection based on the schema below.

- **Name**: <span style='color:#33ECFF'>S2C-PARTNER</span>

- **Connect Public Cloud to:**
  -  <span style='color:#33ECFF'>External Device</span>
  -  <span style='color:#33ECFF'>Static Route-Based (Mapped)</span>

- **Local Gateway**: <span style='color:#33ECFF'>ace-gcp-us-east1-spoke1</span>

- **Real Local Subnet CIDR(s)**: <span style='color:#33ECFF'>172.16.211.0/24</span>

- **Virtual Local Subnet CIDR(s)**: <span style='color:#33ECFF'>192.168.1.0/24</span>

- **Remote Gateway Type**: <span style='color:#33ECFF'>Generic</span>

- **Real Remote Subnet CIDR(s)**: <span style='color:#33ECFF'>172.16.211.0/24</span>

- **Virtual Remote Subnet CIDR(s)**: <span style='color:#33ECFF'>192.168.2.0/24</span>

- **Advanced Settings:**
  -  **IkEv2**: <span style='color:#33ECFF'>On</span>

- **Connection:**
  -  **Remote Gateway IP**: <span style='color:tomato'>follow the Note below</span>

```{note}
Use the “**dig partner-csr-public.pod#.aviatrixlab.com +short**” command <ins>from your personal laptop terminal</ins> to resolve the symbolic public name of the OnPrem Partner CSR router and retrieve the <ins>REMOTE GATEWAY PUBLIC IP address</ins>, as depicted in the example below.
```

<ins>Replace the **#** symbol with your POD number!</ins>

The example is referring to POD #35 (please issue the command based on your POD number!).

![retrieving](images/lab6-podnumber.png)
_Figure 76: Retrieving the Public IP_
  - **Local Gateway Instance**: <span style='color:#33ECFF'>ace-gcp-us-east1-spoke1</span>
  -  **Local Tunnel IP**: <span style='color:#33ECFF'>169.254.0.1/30</span>
  -  **Remote Tunnel IP**: <span style='color:#33ECFF'>169.254.0.2/30</span>
  -  **Pre-Shared Key**: <span style='color:#33ECFF'>Aviatrix123#</span>

```{important}
Do not forget to click on **Save**.
```

![external](images/lab6-finals2c.png)
_Figure 77: External Connection Configuration_

Wait some seconds for the completion of the S2C. The new connection will show up with a red ball.

![established](images/lab6-notdone.png)
_Figure 78: S2C is establishing the connection_

Click on the <span style='color:orange'>**refresh**</span>
 button to see the status changing from red to green.

![establishing](images/lab6-s2cok.png)
_Figure 79: S2C is establishing the connection_

- SSH to the OnPrem partner router and issue the following command:

```bash
show ip int brief
```

```{note}
Note that the Cisco CSR is not acting as an actual branch router because it is being NAT'd by an AWS IGW. For that purpose, you need to specify that the **Remote Identifier** of the IKE tunnel is the private IP of the CSR, not the public IP.

To find out the Private IP of the CSR, SSH as admin to the on-premises router (same as the Remote Peer IP above) and issue the IOS command show ip interface brief (can be abbreviated to **_sh ip int br_**).
```
**Copy** the Private IP of the GigabitEthernet1 interface. It would be something in 172.16.1.0/24, such as 172.16.1.176, in the output beneath.

![Lab Overview](images/lab6-remoteidentifier.png)
_Figure 80: Remote Identifier_

Then go to **CoPilot > Networking > Connectivity >** click on the **_S2C_PARTNER_** connection, then **> Settings > General**, paste the IP on the **Remote Gateway Identifier** field and then click on ****Save**, as depicted below.

![Replace](images/lab6-remotegwidentifier.png)
_Figure 81: Remote Gateway Identifier_

Then from the OnPrem Partner router issue the following command:

```bash
ping 192.168.1.100 source gigabitethernet1
```

![ping is ok](images/lab6-pingok2.png)
_Figure 82: Ping is ok_
