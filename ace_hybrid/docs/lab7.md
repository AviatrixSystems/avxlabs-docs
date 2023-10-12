# Lab 8 - SITE2CLOUD

## 1. Objective

The objective of this lab is to resolve an IP address overlap between an on-premises partner and the cloud. You will be using the **Site2Cloud Mapped NAT** feature to achieve this.
 
## 2. Site2Cloud Overview

Site2Cloud builds an encrypted connection between two sites over the Internet in an easy-to-use and template-driven manner. Its workflow is similar to any CSP’s virtual private network workflow.

On one end of the tunnel is an Aviatrix Gateway; on the other could be an on-premises router, firewall or another public cloud VPC/VNet that Aviatrix Controller does not manage.
 
## 3. Topology

In this lab, you will achieve Site2Cloud connectivity to a Cisco Cloud Services Router 1000v (CSR) emulating an on-premises branch router.

In this lab, you will work with the overlapping IP addresses in this topology:

![lab8-topology](images/lab8-topology.png)
_Figure 191: Lab 8 Initial Topology_

## 4. Configuration

### 4.1. Site2Cloud Connection (Cloud to On-Prem)

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)** and click on the `"+ External Connection"` button.

![lab8-s2c](images/lab8-s2c.png)
_Figure 192: S2C creation_

Create a connection from Cloud (GCP) to an on-prem Partner site.

Use the following settings on the `"Add External Connection"` window:

- **Name**: <span style='color:#33ECFF'>GCP-to-OnPremPartner</span>
- **External Device**: <span style='color:#33ECFF'>Static Route-Based (Mapped)</span>
- **Local Gateway**: <span style='color:#33ECFF'>gcp-us-central1-spoke1</span>
- **Real Local Subnet CIDR(s)**: <span style='color:#33ECFF'>172.16.1.0/24</span>
- **Virtual Local Subnet CIDR(s)**: <span style='color:#33ECFF'>192.168.200.0/24</span>
- **Remote Gateway Type**: <span style='color:#33ECFF'>Generic</span>
- **Real Remote Subnet CIDR(s)**: <span style='color:#33ECFF'>172.16.1.0/24</span>
- **Virtual Remote Subnet CIDR(s)**: <span style='color:#33ECFF'>192.168.100.0/24</span>
- **Authetication Method**: <span style='color:#33ECFF'>PSK</span>
- **Pre-Shared Key**: <span style='color:#33ECFF'>Aviatrix123#</span>
- **Remote Gateway IP**: <span style='color:#33ECFF'>[**Refer to your Pod assignment for the CSR Public IP**]</span>
- **Local Gateway Instance**: <span style='color:#33ECFF'>gcp-us-central1-spoke1</span>

Then click on **Save**.

![lab8-s2ctemplate](images/lab8-s2ctemplate.png)
_Figure 193: S2C template_

```{caution}
The configurationt template will grey out after clicking on Save. Be patient and wait for the Aviatrix Controller to complete the deployment. <ins>This will create the first leg of the connection, on the GCP Spoke GW</ins>. This will stay **Down** until the other end is configured.
```

Since On-Prem-Partner1 uses the overlapping IP space, we will utilise the Aviatrix Mapped NAT feature and use two virtual subnets.

- **192.168.100.0/24** will be used for the cloud to reach on-prem resources, and 1:1 NAT will be used.

- **192.168.200.0/24** will be used from on-prem to reach cloud resources. 

For example, gcp-us-central1-test1 (172.16.1.100) will be reached at 192.168.200.100 due to 1:1 NAT.

### 4.2. Site2Cloud Connection (On-prem to Cloud)

The CoPilot provides a **_template_** that can be used to configure the remote router/firewall.

Select the Site-to-Cloud connection row you created and click on the `"Three dots"` icon and choose `"Download Configuration"` to generate and download the configuration template.

![lab8-template](images/lab8-template.png)
_Figure 193: CFG file_

Insert the following value inside the `"Download Configuration"` window:

- **Vendor**: <span style='color:#33ECFF'>Cisco</span>
- **Platform**: <span style='color:#33ECFF'>ISR,ASR, or CSR</span>
- **Software**: <span style='color:#33ECFF'>IOS(XE)</span>

and then click on **Download**.

![lab8-template2](images/lab8-template2.png)
_Figure 194: Download CFG file_

```{note}
The Cisco CSR is not acting as an actual branch router because it is being NAT'd by an AWS IGW. For that purpose, you need to specify that the **`Remote Identifier`** of the IKE tunnel is the private IP of the CSR, not the public IP.
```

To find out the **Private IP** of the CSR, SSH as `admin` to the on-premises router (same as the Remote Peer IP above) and issue the IOS command `show ip interface brief` (can be abbreviated to **_sh ip int br_**). 

```{tip}
Refer to your **POD** portal for retrieving the Public IP of the CSR.
```

The Private IP that you need to copy is that one assigned to the **_GigabitEthernet1_** interface.

![lab8-template2](images/lab8-giga.png)
_Figure 195: GigabitEthernet1 IP address_

Use the Private IP of the GigabitEthernet1 interface. It would be something in 172.16.1.0/24, such as 172.16.1.176, in the above output.

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)** and click on the `GCP-to-OnPremPartner` connection.

![lab8-s2craw](images/lab8-s2craw.png)
_Figure 195: S2C node_

Then click on the `"Settings"` tab, expand the `"General"` section and paste the Private IP on the `"Remote Gateway Identifier"` field, as depicted below. 

Do not forget to click on **Save**.

![lab8-settings](images/lab8-settings.png)
_Figure 196: Remote Gateway Identifier_

### 4.2.1 Modify the txt.file 

Make the following four changes to the downloaded Site2Cloud text file:

`<crypto_policy_number>` : <span style='color:#33ECFF'>1</span>

`<tunnel_number1>`: <span style='color:#33ECFF'>1</span>

`<ios_wan_interface1>`: <span style='color:#33ECFF'>gi1</span>

ip route 192.168.200.0 255.255.255.0 Tunnel`<tunnel_number>`: <span style='color:#33ECFF'>1</span>

![lab8-txt1](images/lab8-txt1.png)
_Figure 197: txt.file modification_

![lab8-txt2](images/lab8-txt2.png)
_Figure 198: txt.file modification_

Copy the entire configuration to the clipboard.

```{important}
If your SSH session to the Cisco CSR hasn't timed out, SSH back in (please refer to your **POD** portal file for its public IP). 
```

Type `configure terminal` (can be abbreviated to **_conf t_**).

![lab8-txt2](images/lab8-ssh.png)
_Figure 199: txt.file modification_

- **Paste** the configuration to the terminal.

After doing so, type `end` to exit configuration mode, followed by `sh ip int br` to verify that the tunnel interface is up on the CSR.

![lab8-up](images/lab8-up.png)
_Figure 200: Connection is up/up_

## 5. Verification

Go to **CoPilot > Networking > Connectivity > External Connection (S2C)**

```{tip}
Click on the **refresh** button to see the colored ball changing from red to green.
```

![lab8-refresh](images/lab8-refresh.png)
_Figure 201: Connection is up/up also on the CoPilot_

- Verify the newly created tunnel is up (might take a few seconds once configuration is applied on the CSR (alternatively you can just click on the `Refresh` button).

Go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)**

Click to expand the **_gcp-us-central1-spoke1_** VPC to view the on-premises connection.

![lab8-onprem](images/lab8-onprem.png)
_Figure 202: OnPrem partner site_

Go to **OnPrem Router > SSH Console**.

From the on-premises router’s console, enter a ping sourced from the **_GigabitEthernet1_** interface to the test instance in GCP (**_gcp-us-central1-spoke1-test1_**) as follows: 

```bash
ping 192.168.200.100 source gi1
```

![lab8-pingok](images/lab8-pingok.png)
_Figure 202: Ping ok_

Then go to **CoPilot > Cloud Fabric > Topology > Overview (default TAB)** and click on the icon of the Spoke Gateway **_gcp-us-central1-spoke1_**, click on `Tools` and then click on `Gateway Diagnostics`.

![lab8-diag](images/lab8-diag.png)
_Figure 203: Gateway Diagnostics_

Choose the `“Active Sessions”` option and in the Search field write `“icmp”` and then click on **Run**. You will notice the subnets involved (i.e. real and virtual subnets) in the Mapped NAT.

![lab8-active](images/lab8-active2.png)
_Figure 204: Active Sessions_

```{important}
You will have to relaunch the ping command once again from the CSR router and click on **Run** on the CoPilot for capturing the Active Sessions!
```

After completing this lab, this is what the overall lab topology would look like:

![lab8-final](images/lab8-finaltopology.png)
_Figure 204: Lab 8 Final Topology_