# Lab 1 - Network Domains

## 1. SCENARIO

The infrastructure has recently been divided into two network domains: **BU1** and **BU2**.

Please verify the segregation between these two domains, as implemented through the `Aviatrix Cloud Native Security Fabric`.

```{figure} images/lab1-segmentation.png
---
height: 400px
align: center
---
Initial Topology
```

## 2. VALIDATION REQUEST

* Navigate to **CoPilot > Settings > Resources > Task Server**. 
  * Ensure that both `Fetch GW Routes` and `Fetch VPC Routes` intervals are set to `1 second`. Then, click **Save** to apply the changes.


```{figure} images/lab1-task.png
---
height: 400px
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

After that, click on **Commit**.

```{figure} images/lab1-commit.png
---
height: 400px
align: center
---
Commit
```

```{warning}
These settings are quite aggressive. In a production environment, you should avoid setting these intervals to such frequent values!
```

Navigate to **CoPilot > Cloud Fabric > Topology** to explore your POD topology.

You'll see that CoPilot displays both *Managed* and *Unmanaged* VPCs.

```{figure} images/lab1-managedvpc.png
---
height: 400px
align: center
---
Dynamic Topology
```

Click the `"Managed"` button on the right side of the screen to hide the Unmanaged VPCs.

```{figure} images/lab1-managedvpc2.png
---
height: 400px
align: center
---
Properties of your Topology
```

```{note}
**Managed** VPC: Indicates that an Aviatrix gateway is operational within the VPC/VNet.

**Unmanaged** VPC: Indicates that no Aviatrix gateways are present in the VPC/VNet.
```

After completing this action, your Topology should appear as follows.

```{figure} images/lab1-managedvpc3.png
---
height: 400px
align: center
---
Managed VPCs only
```

```{caution}
**Always** refer to your personal POD for the accurate IP addresses. The IP addresses displayed in the following screenshots are only <ins>examples</ins> sourced from a different POD and are intended solely for the creation of the lab guides.
```

There are **two** methods to SSH into any instances within the multicloud infrastructure of this lab:

1. Use an <span style='color:orange'>**SSH client**</span> from your laptop (<ins>recommended method!</ins>).

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

2. Use the <span style='color:orange'>**Apache Jumpbox**</span> from the POD Portal, particularly if you are on your corporate network and typically face outbound restrictions blocking **port 22**.

```{figure} images/lab1-jumpbox.png
---
height: 200px
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
Please note that if you choose to use the Jumpbox, copy and paste functionality will not work directly from the host machine. To enable this feature, activate the **Guacamole Menu**, which is initially hidden. On a desktop or any device with a hardware keyboard, you can display this menu by pressing **_Ctrl+Alt+Shift_** on a Windows machine (**_Control+Shift+Command_** on a Mac).
```

```{figure} images/lab1-guacamoleterminal.png
---
height: 400px
align: center
---
Guacamole Menu
```

```{tip}
The IP addresses can be easily obtained using **three** different methods, according to your preference:

1) From the **Properties** section of each Virtual Machine in the Topology.

2) From the **Virtual Machines** Inventory*.

3) From your **personal POD portal**, where you can also utilize DNS symbolic names.
```

- **Dynamic Topology**:

Expand the **_ace-aws-eu-west-1-spoke1_** VPC and click on the test instance, then explore the `"Properties"` section on the right-hand side.

```{caution}
You **cannot** connect to any Aviatrix Gateways using the SSH protocol, <ins>as port 22 is secured</ins>!
```

```{figure} images/lab1-newpicture.png
---
height: 400px
align: center
---
EC2 Instance
```

```{figure} images/lab1-ec2.png
---
height: 400px
align: center
---
Instance Properties
```

```{tip}
To view the full hostnames of each node, select `"Show Labels"` from the bottom left sidebar.
```{figure} images/lab1-showlabels.png
---
height: 400px
align: center
---
Show Labels
```

- **Cloud Assets**:

Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, where you can search for any instances and retrieve their IP addresses!

```{figure} images/lab1-assets.png
---
height: 200px
align: center
---
Inventory
```

- **POD Portal**:

Access your personal POD to obtain the symbolic names of any test instances.

```{caution}
- Use the symbolic name containing the word `"public"` to log into the instance from your laptop."

- Use the symbolic name containing the word `"private"` for East-West traffic verification after accessing any test instance from your laptop.
```

```{figure} images/lab1-podred.png
---
height: 300px
align: center
---
DNS Names
```

- Verify connectivity between clients **within** the same BU:
    - SSH into the **BU1 Frontend**  instance in AWS.

```{important}
The credentials for accessing the EC2 instances and VMs are available in your personal POD Portal, within the dedicated widget labeled `"SSH for Test Instances"`.
```{figure} images/lab1-widget.png
---
height: 400px
align: center
---
SSH credentials
```

Let's use the Dynamic Topology for retrieving the Public IP address of the BU1 Frontend instance. Navigate to **CoPilot > Cloud Fabric > Topology > Overview (default tab)**, then click on **Filters** and search for `"frontend"`.

```{figure} images/lab1-assets20.png
---
height: 400px
align: center
---
BU1 frontend
```

Now, click on the EC2 instance icon in the topology and locate the Public IP address on the right side of the `Properties` window that appears.

```{figure} images/lab1-assets21.png
---
height: 400px
align: center
---
Properties
```

Open your SSH client and enter the SSH command to log in to the BU1 Frontend instance.

```{figure} images/lab1-assets22.png
---
height: 400px
align: center
---
SSH
```

 - From the BU1 Frontend instance, ping the <ins>private IP address</ins> of the **BU1 Analytics** instance in GCP.
  - Confirm that **SSH** works from the BU1 Frontend instance to the BU1 Analytics instance.

Ping and SSH will be successful **within** the same network domain!

```{tip}
Clear the keyword in the filter field to reestablish the complete topology view. 
```{figure} images/lab1-filter.png
---
height: 400px
align: center
---
Remove the filter
```

```{figure} images/lab1-pingok.png
---
height: 300px
align: center
---
ICMP
```

```{figure} images/lab1-pingok20.png
---
height: 400px
align: center
---
SSH
```

* Verify the network segregation **between** the two BUs:
  * From **BU1 Frontend** try to ping the <ins>private IP address</ins> of the **BU2 Mobile App**.
  * From **BU1 Frontend** try to SSH **BU2 Mobile App** (use its Private IP address!).

Ping and SSH commands should not work this time, due to the separation between the two segments (i.e. <ins>these are two different Routing Domains</ins>).

```{important}
Once again refer always to your personal POD for the IP addresses. 

The screenshots are used as examples and might indicate **different IP addresses** from those shown on your personal POD portal!
```

```{figure} images/lab1-pingfails.png
---
height: 400px
align: center
---
BU1 to BU2: ICMP test fails
```

```{figure} images/lab1-sshfails01.png
---
height: 400px
align: center
---
BU1 to BU2: SSH test fails
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
height: 200px
align: center
---
Network Domains
```
  
* Examine the various routing tables (i.e., *VRFs*) managed by the Transit Gateways.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the **_ace-aws-eu-west-1-transit_** Gateway, then go to **Gateway Routes** and filter by any *Network Domain* (such as BU1 or BU2).
```

```{figure} images/lab1-bu1vrf.png
---
height: 400px
align: center
---
Network Domain (aka VRF)
```

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the **ace-aws-eu-west-1-transit** Gateway and then select the tab **Route DB**. 

You will notice that the `Segmentation` feature is indeed enabled. Moreover, you will notice the different RTBs.
```{figure} images/lab1-newjoe.png
---
height: 400px
align: center
---
RIB
```

* Use the <span style='color:#FF0000'>**FlowIQ**</span> functionality from the CoPilot, <ins> for inspecting the NetFlow Data.

```{tip}
Navigate to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based, for instance, on the `"Destination IP Address"` **172.16.211.100** (i.e. BU1 Analytics).

Do not forget to click on **Apply**.
```

```{figure} images/lab1-plus.png
---
height: 150px
align: center
---
FlowIQ Filter
```

```{caution}
You probably won't see any results immediately after clicking Apply. It may take a few more minutes for CoPilot to display the NetFlow information collected through CNSF.
```{figure} images/lab1-plus78.png
---
height: 150px
align: center
---
FlowIQ Filter
```

Then check the `"Flow Exporters"` widget, then from the drop-down menu select the **`"Aviatrix Gateway"`** widget: you will see the list of all the Aviatrix Gateways involved along the path.

```{figure} images/lab1-flowiq.png
---
height: 500px
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
height: 150px
align: center
---
Search field and Filter icon
```

```{tip}
The **Originator** has the egress interface that corresponds to the **eth0** interface (i.e. the LAN interface), which in turn means, direct connected.
```

```{figure} images/lab1-cloudroutes.png
---
height: 400px
align: center
---
Originator = eth0
```

* Use **Cloud Routes** for pinpointing the originator of the route **10.0.0.0/24**.

```{tip}
Go to **CoPilot > Diagnostics > Cloud Routes** and search the subnet **10.0.0.0/24** on the *search field*. 

<ins>Remove any previous filters</ins>!
```{figure} images/lab1-removefilter.png
---
height: 300px
align: center
---
Remove filter
```

This time you need to proceed with a <ins>recursive lookup</ins>: from any Spoke GWs check the **NEXT HOP GATEWAY** column and try to find the originator of 10.0.0.0/24.

```{figure} images/lab1-cloudroutes2.png
---
height: 300px
align: center
---
Cloud Routes - Recursive lookup
```