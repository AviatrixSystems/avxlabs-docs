# Lab 1 - Network Domains

## 1. SCENARIO

Infrastructure <ins>was segmented</ins> recently into 2 network domains: **BU1** & **BU2**.

You are requested to ascertain the segregation between the two network domains.

```{figure} images/lab1-segmentation.png
---
height: 400px
align: center
---
Initial Topology
```

## 2. VALIDATION REQUEST

* Go to **CoPilot > Settings > Resources > Task Server**
  * Ensure that both **Fetch GW Routes** and **Fetch VPC Routes** intervals are set to <ins>“1 second”</ins> each and then click on **SAVE**.

```{figure} images/lab1-task.png
---
align: center
---
Task Server
```

```{figure} images/lab1-fetchgwroutes.png
---
align: center
---
Fetch GW Routes
```

```{figure} images/lab1-fetchvpcroutes.png
---
align: center
---
Fetch VPC Routes
```

Afterwards, click on **Commit**.

```{figure} images/lab1-commit.png
---
align: center
---
Commit
```

```{warning}
These are very aggressive settings. In a Production environment, you should NOT set these intervals that frequently!
```

Go to **CoPilot > Cloud Fabric > Topology**  to explore your POD topology. 

You will notice that the CoPilot is showing both the *Managed VPCs* and the *Unmanaged VPCs*.

```{figure} images/lab1-managedvpc.png
---
align: center
---
Dynamic Topology
```

Click on the `"Managed"` button on the right-hand side of the screen, for hiding the Unmanaged VPCs.

```{figure} images/lab1-managedvpc2.png
---
align: center
---
Properties of your Topology
```

```{note}
**Managed VPC** = Indicates an Aviatrix gateway is running in the VPC/VNet.

**Unmanaged VPC** = Indicates no Aviatrix gateways exist in the VPC/VNet.
```

After this action this is how your Topology should look like.

```{figure} images/lab1-managedvpc3.png
---
align: center
---
Managed VPCs only
```

```{caution}
Refer **always** to your personal POD for the IP addresses. The IP addresses visible on the subsequent screenshots are just examples taken from a different POD, used for creating the lab guides!
```

There are **two** methods for SSH to any instances inside the multicloud infrastructure of this lab:

1. Using an <span style='color:orange'>**SSH client**</span> from your laptop (<ins>recommended method!</ins>).

```{figure} images/lab1-publicip.png
---
align: center
---
Public IP
```

```{figure} images/lab1-publicname.png
---
align: center
---
DNS name
```

2. Using the <span style='color:orange'>**Apache Jumpbox**</span> from the POD Portal, <ins>for example if you are within your corporate network and tipically an outbound restriction rule is blocking the port **22**</ins>.

```{figure} images/lab1-jumpbox.png
---
align: center
---
Jumpbox
```

```{figure} images/lab1-guacamoleaccess.png
---
align: center
---
Apache Guacamole Portal
```

```{note}
Please bear in mind that if you decide to use the Jumpbox, *Copy and Paste* does not work directly from the host machine, therefore activate the **Guacamole Menu**, that is a sidebar which is hidden until explicitly shown. On a desktop or other device which has a hardware keyboard, you can show this menu by pressing **Ctrl+Alt+Shift** on Windows machine (**Control+Shift+Command** on Mac).
```

```{figure} images/lab1-guacamoleterminal.png
---
align: center
---
Guacamole Menu
```

```{tip}
The IP addresses can be easily retrieved using **3** different methods, as you like:
1) From the **Properties** section of each Virtual Machine on the Topology.
2) From the **Virtual Machines** Inventory.
3) From your personal POD portal, where you can also use the **DNS symbolic names**.
```

- **Dynamic Topology**:

Expand the **_ace-aws-eu-west-1-spoke1_** VPC and click on the test instance, then explore the `"Properties"` section on the right-hand side.

```{caution}
You can't connect to any Aviatrix Gateways using the **SSH** protocol. The port 22 is hardened!
```

```{figure} images/lab1-newpicture.png
---
align: center
---
EC2 Instance
```

```{figure} images/lab1-ec2.png
---
align: center
---
Instance Properties
```

- **Cloud Assets**:

Go to **CoPilot > Cloud resources > Cloud Assets > Virtual Machines** and from here you can search for any instances and retrieve their IP addresses!

```{figure} images/lab1-assets.png
---

align: center
---
Inventory
```

- **POD Portal**:

Use your personal POD in order to retrieve the symbolic names of any test instances.

```{caution}
- Use the symbolic name that contains the word `"public"` for logging to the instance from your laptop.
- Use the symbolic name that contains the word `"private"` for the East-West traffic verification, once you have landed onto any test instances from your laptop. 
```

```{figure} images/lab1-podred.png
---
align: center
---
DNS Names
```

- Verify connectivity between clients **within** the same BU:
    - SSH to the **BU1 Frontend** in AWS.
    - From BU1 Frontend ping the <ins>private IP address</ins> of the **BU1 Analytics** in GCP.

Ping and SSH will be successful **within** the same network domain!

```{figure} images/lab1-pingok.png
---
align: center
---
BU1 connectivity
```

* Verify the network segregation **between** the two BUs:
  * From **BU1 Frontend** try to ping the <ins>private IP address</ins> of the **BU2 Mobile App**.
  * From **BU1 Frontend** try to SSH **BU2 Mobile App** (use its Private IP address!).

Ping and SSH commands should not work this time, due to the separation between the two segments (i.e. <ins>these are two different Routing Domains</ins>).

```{important}
Once again refer always to your personal POD for the IP addresses. 

The screenshots are used as examples and might indicate different IP addresses from those shown on your personal POD portal!
```

```{figure} images/lab1-pingfails.png
---
align: center
---
BU1 to BU2 fails
```

* Check the **Network Segmentation** section on the CoPilot, and then look at the **Logical View**.

```{tip}
Go to **CoPilot > Networking > Network Segmentation > Overview**
```

```{figure} images/lab1-logicalview.png
---
align: center
---
Logical View
```

* Check the **Network Domains** Tab, too.

```{figure} images/lab1-noconnection.png
---
align: center
---
Network Domains
```
  
* Check the different routing tables (*VRFs*) maintained by any of the Transit Gateways.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **ace-aws-eu-west-1-transit** Gateway **> Gateway Routes** and filter out based on any Network Domains (i.e. either BU1 or BU2).
```

```{figure} images/lab1-bu1vrf.png
---
align: center
---
Network Domain (aka VRF)
```

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **ace-aws-eu-west-1-transit** Gateway and then select the tab **Route DB**. 

You will notice that the `Segmentation` feature is indeed enabled. Moreover, you will notice the different RTBs.
```{figure} images/lab1-newjoe.png
---
align: center
---
RIB
```

* Use the <span style='color:#FF0000'>**FlowIQ**</span> functionality from the CoPilot, <ins> for inspecting the NetFlow Data.

```{tip}
Go to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based, for instance, on the `"Destination IP Address"` **172.16.211.100** (i.e. BU1 Analytics).

Do not forget to click on **Apply**.
```

```{figure} images/lab1-plus.png
---
align: center
---
FlowIQ Filter
```

Then check the `"Flow Exporters"` widget, then from the drop-down menu select the **`"Aviatrix Gateway"`** widget: you will see the list of all the Aviatrix Gateways involved along the path.

```{figure} images/lab1-flowiq.png
---
align: center
---
FlowIQ
```

```{note}
On the **Aviatrix Gateway** widget, the very first gateway from the list is the gateway with the highest traffic (in KibiBytes).
```

* Use **Cloud Routes** for pinpointing the originator of the route **172.16.211.0/24**.

```{tip}
Go to **CoPilot > Diagnostics > Cloud Routes**, search for the subnet **172.16.211.0/24** on the *search field* and then filter based on the following condition: "Gateway *contains* spoke1".

The filter button is on the left-hand side of the screen and is shaped like a small funnel.
```

```{figure} images/lab1-filtersearch.png
---
align: center
---
Search field and Filter icon
```

```{tip}
The **Originator** has the egress interface that corresponds to the **eth0** interface (i.e. the LAN interface), which in turn means, direct connected.
```

```{figure} images/lab1-cloudroutes.png
---
align: center
---
Originator = eth0
```

* Use **Cloud Routes** for pinpointing the originator of the route **10.0.0.0/24**.

```{tip}
Go to **CoPilot > Diagnostics > Cloud Routes** and search the subnet **10.0.0.0/24** on the *search field*. 

<ins>Remove any previous filters</ins>!
```

This time you need to proceed with a <ins>recursive lookup</ins>: from any Spoke GWs check the **NEXT HOP GATEWAY** column and try to find the originator of 10.0.0.0/24.

```{figure} images/lab1-cloudroutes2.png
---
align: center
---
Cloud Routes - Recursive lookup
```