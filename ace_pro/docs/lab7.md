# Lab 7 - SITE2CLOUD

## 1. General Objectives

The objective of this lab is to resolve an IP address overlap between an on-premises partner and the cloud. You will be using the **`Site2Cloud Mapped NAT`** feature to achieve this. 

## 2. Site2Cloud Overview

Let's start with the deployment of the S2C.

Site2Cloud builds an encrypted connection between two sites over the Internet in an easy-to-use and template-driven manner. Its workflow is similar to any CSP’s virtual private network workflow.

On one end of the tunnel is an Aviatrix Gateway; on the other could be an on-premises router, firewall or another public cloud VPC/VNet that Aviatrix Controller does not manage.
 
## 3. Topology

In this lab, you will first establish Site-to-Cloud (S2C) connectivity to a `StrongSwan Router`, emulating an on-premises branch router.

Moreover, you will work with a typical scenario involving **overlapping IP addresses**, as depicted in the drawing below:

```{figure} images/lab8-topology.png
---
align: center
---
Lab 8 Initial Topology
```

## 4. Configuration

### 4.1. Site2Cloud Connection (Cloud to On-Prem)

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**. Here you will immediately notice the presence of an existing S2C connection. 

This is the connection established between an **Aviatrix Edge** device deployed in the remote DC in New York and a LAN Router.

```{figure} images/lab8-edge1.png
---
height: 150px
align: center
---
Existing S2C connection
```

```{figure} images/lab8-edge2.png
---
align: center
---
BGPoverLAN
```

The S2C connection with Edge will be configured on the subsequent task.

Now let's configure the **S2C** connection with the Partner site!

```{figure} images/lab8-partner.png
---
align: center
---
S2C between Partner and GCP
```

Click on the `"+ External Connection to"` button and let's create a new connection from scratch. 

Select the `"External Device"` option from the drop-down window!

```{figure} images/lab8-s2c.png
---
align: center
---
S2C creation
```

Create a connection from the Cloud (GCP) to an on-premises partner site using the following settings in the `"Add External Connection"` window:

- **Name**: <span style='color:#479608'>GCP-to-OnPremPartner</span>
- **Connecting Using**: <span style='color:#479608'>Static Route-Based</span>
- **Type**: <span style='color:#479608'>**Mapped**</span>
- **Local Gateway**: <span style='color:#479608'>gcp-us-central1-spoke1</span>
- **Real Local Subnet CIDR(s)**: <span style='color:#479608'>172.16.1.0/24</span>
- **Virtual Local Subnet CIDR(s)**: <span style='color:#479608'>192.168.200.0/24</span>
- **Remote Gateway Type**: <span style='color:#479608'>Generic</span>
- **Real Remote Subnet CIDR(s)**: <span style='color:#479608'>172.16.1.0/24</span>
- **Virtual Remote Subnet CIDR(s)**: <span style='color:#479608'>192.168.100.0/24</span>
- **Authetication Method**: <span style='color:#479608'>PSK</span>
- **Pre-Shared Key**: <span style='color:#479608'>[**Refer to your Pod assignment for the StrongSwan PSK - Lab8 section**]</span>
- **IKEv2**: <span style='color:#479608'>**ON**</span>
- **Remote Gateway IP**: <span style='color:#479608'>[**Refer to your Pod assignment for the **on-prem-partner1**'s Public IP - Lab7 section** - You will need to use the dig/host/nslookup command]</span>
- **Local Gateway Instance**: <span style='color:#479608'>gcp-us-central1-spoke1</span>

Then click on **Save**.

```{figure} images/lab8-newone.png
---
align: center
---
DNS Name of the StrongSwan router
```

```{note}
Use the command “**host strongswan.pod#.aviatrixlab.com**” <ins>from your personal laptop terminal</ins> to resolve the symbolic public name of the **on-prem-partner1 StrongSwan router** and retrieve the <ins>REMOTE GATEWAY PUBLIC IP address</ins>, as depicted in the example below.
```

<ins>Replace the **#** symbol with your POD number!</ins>

The example is referring to POD #100 (<ins>please issue the command based on your POD number</ins>).

```{figure} images/lab8-newdns.png
---
align: center
---
Dig command in action
```

```{tip}
For **Windows OS** you can use the command `"nslookup"`:

**nslookup strongswan.pod#.aviatrixlab.com**

```{figure} images/lab8-nslookup.png
---
align: center
---
Nslookup 
```

This is how your template would look like.

```{figure} images/lab8-s2ctemplate.png
---
align: center
---
S2C template
```

```{important}
The configuration template will be grayed out after clicking on **Save**.

Please be patient and wait for the **_Aviatrix Controller_** to complete the deployment. <ins>This will establish the first side of the connection on the GCP Spoke Gateway</ins>.

The StrongSwan router was preconfigured at the launch of each POD!
```

Since On-Prem-Partner1 uses the overlapping IP space, we will utilise the Aviatrix Mapped NAT feature and use two virtual subnets.

- **192.168.100.0/24** will be used for the cloud to reach on-prem resources, and 1:1 NAT will be used.

- **192.168.200.0/24** will be used from on-prem to reach cloud resources. 

For example, gcp-us-central1-test1 (172.16.1.100) will be reached at 192.168.200.100 due to 1:1 NAT.

### 4.2 StrongSwan's Configuration 
Now you have to complete the IPSec configuration of the **StrongSwan** router.

```{caution}
If you are using a corporate laptop that does not allow the use of any SSH client and also blocks port 22, please proceed with task 4.2.1. However, keep in mind that you may experience some slowness in the response of the commands sent to the Guacamole Server (jumphost), so please be patient during the configuration. 

If you have a laptop without these restrictions, please refer to task 4.2.2 and complete the configuration using your personal SSH client.
```

#### 4.2.1  <span style='color:#33ECFF'>Configuration through the Apache Guacamole Client (Jumphost)</span></summary>

Go to your personal POD portal, identify the section labeled `"Lab 7 and 8"`, then click on the `"Open Workstation"` button to log in to the **Workstation Edge** (a Guacamole clientless remote desktop gateway).

```{figure} images/lab8-podportal00.png
---
height: 400px
align: center
---
POD Portal - Sec. 7-8
```

```{figure} images/lab8-podportal01.png
---
height: 400px
align: center
---
POD Portal - Sec. 7-8
```

Open the **LX terminal** from the remote desktop.

```{figure} images/lab8-podportal02.png
---
height: 400px
align: center
---
LX terminal
```

```{note}
Please bear in mind that if you decide to use the Jumpbox, copy and paste do not work directly from the host machine. Therefore, activate the **Guacamole Menu**, which is a sidebar that is hidden until explicitly shown. On a laptop or other device with a hardware keyboard, you can display this menu by pressing **_Ctrl+Alt+Shift_** on a Windows machine (**_Control+Shift+Command_** on a Mac).
```

Type the SSH command to log in to the StrongSwan router (**on-prem-partner1**). Then copy the command and paste it insid ethe LX terminal.

```{tip}
You can retrieve its DNS name from your personal POD Portal.
```

```{figure} images/lab8-podportal03.png
---
height: 400px
align: center
---
Type the command
```

```{figure} images/lab8-podportal04.png
---
height: 400px
align: center
---
Copy the command
```

```{figure} images/lab8-podportal05.png
---
height: 400px
align: center
---
Paste the command
```

Now, before issuing the command, copy the password for logging into the StrongSwan router from the **Clipboard** window on the left-hand side, then close the Clipboard using the aforementioned commands (**_Ctrl+Alt+Shift_** on a Windows machine **_Control+Shift+Command_** on a Mac).

```{figure} images/lab8-podportal06.png
---
height: 400px
align: center
---
Copy the pwd and close the Clipboard window
```

```{figure} images/lab8-podportal07.png
---
height: 400px
align: center
---
Paste the pwd
```

Now you need to retrieve the **Public IP address** assigned to the GCP Spoke Gateway.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and then identify the GCP Spoke Gateway and **copy** its Public IP address.

```{figure} images/lab8-gcppublic.png
---
align: center
---
Public IP address
```

Now, go back to the **Guacamole Client LX Terminal** session and issue the following **_bash script_** to automatically update the content of the configuration file!

- Run  the following bash script:

```{tip}
Below replace the string **<REPLACE_WITH_SPOKE_GW_PUBLIC_IP>** with GCP Spoke gateway public IP of your POD.
```

```bash
sudo ./update_swanctl.sh <REPLACE_WITH_SPOKE_GW_PUBLIC_IP>
```

```{tip}
Use the Clipboard window for typing your commands and then copying them into the LX Terminal from your machine.
```

```{figure} images/lab8-podportal08.png
---
height: 500px
align: center
---
Bash cmd
```

```{figure} images/lab8-podportal09.png
---
height: 400px
align: center
---
Bash script issued successfully
```

#### 4.2.2  <span style='color:#33ECFF'>Configuration through the SSH Client</span></summary>

Alternatively, if you are NOT using a corporate laptop, you can SSH into the StrongSwan router and apply the configuration using your SSH client.

 You can either use the public IP address that you retrieved before using the host/nslookup command or you can use its dns name available on your personal POD portal.

```{figure} images/lab8-personalpod.png
---
align: center
---
StrongSwan's DNS name
```

```{figure} images/lab8-strong.png
---
align: center
---
SSH
```

- Now you need to **edit** the StrongSwan's router configuration file. 
Check the content of the following file: `/etc/swanctl/swanctl.conf`

Type the following command:

```bash
cat /etc/swanctl/swanctl.conf
```

You will need to replace the string **`REPLACE_WITH_SPOKE_GW_PUBLIC_IP`** with the public IP address of the GCP spoke gateway in 3 places within `/etc/swanctl/swanctl.conf`

```{figure} images/lab8-replace.png
---
align: center
---
cfg file
```

Let's retrieve the Public IP address assigned to the GCP Spoke Gateway.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and then identify the GCP Spoke Gateway and **copy** its Public IP address.

```{figure} images/lab8-gcppublic.png
---
align: center
---
Public IP address
```

Now go back on the **SSH** session established with the StrongSwan router and instead of editig the cfg file manually, you are going to use a **_script_** for automatically updating the content of the coonfiguration file!

- Run  the following bash script:

```{tip}
Below replace the string **<REPLACE_WITH_SPOKE_GW_PUBLIC_IP>** with GCP Spoke gateway public IP of your POD.
```

```bash
sudo ./update_swanctl.sh <REPLACE_WITH_SPOKE_GW_PUBLIC_IP>
```

```{figure} images/lab8-bash.png
---
align: center
---
sudo bash script
```

## 5. S2C - Verification

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**

```{tip}
Click on the **refresh** button to see the status icon changing from red to green.

The status will be reflected on the screen in about **1 minute**. 
```

```{figure} images/lab8-refresh.png
---
height: 250px
align: center
---
Connection is up
```

Go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)**

Filter out based on the **GCP** Cloud, expand all the VPCs and you will see the new S2C connection with the remote OnPrem-Partner site!

```{figure} images/lab8-onprem.png
---
height: 400px
align: center
---
OnPrem-Partner site
```

## 6. Connectivity Testing

Verify that ther VM in GCP can ping successfully the on-prem-partner1 router, pinging its `Virtual Remote IP`.

### 6.1 Connectivity Testing Using the Gatus App

First and foremost, navigate to your POD Portal, locate the `Gatus widget`, and select **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab8-podportal10.png
---
height: 400px
align: center
---
Gatus
```

Go back to your Guacamole Client (i.e. Workstation Edge) and issue the following command:

```bash
ping 192.168.200.100 
```

```{tip}
Do not stop the ping command
```

```{figure} images/lab8-podportal11.png
---
height: 400px
align: center
---
ping
```

### 6.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

Go back to your SSH terminal, and from the on-premises router’s console (i.e. StrongSwan), issue the following command to verify the connectivity with the **_gcp-us-central1-spoke1-test1_**:

```bash
ping 192.168.200.100 
```

```{tip}
Keep the ping running recursively!
```

```{figure} images/lab8-pingok.png
---
align: center
---
Ping ok
```

## 7. Gateway Diagnostics

Go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)** and click on the icon of the Spoke Gateway **_gcp-us-central1-spoke1_**, click on the `Tools` button and then click on `Gateway Diagnostics`.

```{figure} images/lab8-diag.png
---
height: 400px
align: center
---
Gateway Diagnostics
```

Choose the `“Active Sessions”` option, click on **Run**. After you receive the result, type `“icmp”` in the **Search** field.

You will notice the subnets involved (i.e. real and virtual subnets) in the Mapped NAT.

```{figure} images/lab8-active2.png
---
align: center
---
Active Sessions
```

After completing the S2C connection, this is what the overall lab topology will look like:

```{figure} images/lab8-finaltopology.png
---
height: 400px
align: center
---
The Topology with the new S2C connection
```
