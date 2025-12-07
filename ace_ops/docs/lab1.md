# Lab 1 - Network Domains

## 1. SCENARIO

The infrastructure has recently been divided into two network domains: **BU1** and **BU2**.

Please verify the segregation between these two domains, as implemented through the `Aviatrix Cloud Native Security Fabric (CNSF)`.

```{figure} images/lab1-segmentation.png
---
height: 400px
align: center
---
Initial Topology
```

## 2. VALIDATION REQUEST

### 2.1 Initial Configuration

* Navigate to **CoPilot > Settings > Resources > Task Server**.
  * Ensure that `Fetch GW Routes`, `Fetch VPC Routes` and `Fetch BGP` intervals are set to `1 second`. Then, click **Save** to apply the changes.

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

```{figure} images/lab1-fetchbgp.png
---
align: center
---
Fetch BGP
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

- Navigate to **CoPilot > Cloud Fabric > Topology** to explore your POD topology.

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
Topology Characteristics
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

```{figure} images/lab1-managedvpc321.png
---
height: 400px
align: center
---
Collapsed View
```

```{caution}
**Always** refer to your personal POD for the accurate IP addresses. The IP addresses displayed in the following screenshots are only <ins>examples</ins> sourced from a different POD and are intended solely for the creation of the lab guides.
```

### 2.2 SSH client and Jumpbox

There are **two** SSH access methods available to connect to any instance in the lab’s hybrid‑cloud environment.  

1. <span style='color:orange'>**SSH**</span>
from your laptop (recommended).

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

2. <span style='color:orange'>**Guacamole Jumpbox**</span>, from the POD Portal is especially useful for corporate networks that block outbound port 22.

```{figure} images/lab1-jumpbox.png
---
height: 200px
align: center
---
Guacamole Jumpbox
```

```{figure} images/lab1-guacamoleaccess.png
---
align: center
---
Guacamole Jumpbox Portal
```

Once you have logged into the Guacamole Jumpbox, you will land in a _Virtual Desktop_ from which you can open **LXTerminal**.

```{figure} images/lab1-guacamoleaccess098.png
---
align: center
---
LXTerminal
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

### 2.3 IP addresses

The CoPilot is your **_single pane of glass_**, offering a complete database of IP addresses for all instances in the hybrid-cloud environment it governs.

```{tip}
The IP addresses can be easily obtained using **three** different methods, according to your preference:

1) From the **Properties** section of each Virtual Machine in the Dynamic Topology.

2) From the **Virtual Machines** inventory within the Cloud Assets section.

3) From your **personal POD portal**, where you can also utilize the DNS symbolic names.
```

#### 2.3.1 Dynamic Topology - Properties
  
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

#### 2.3.2 Cloud Assets

Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**. Perform a search for "_frontend_" and extract the corresponding IP addresses from the resulting list.

```{figure} images/lab1-assets.png
---
height: 200px
align: center
---
Inventory
```

```{important}
Always refer to your personal CoPilot for the IP addresses. These images are only examples.
```

#### 2.3.3 POD Portal

Access your personal POD to obtain the symbolic names of any test instances.

```{caution}
- Use the symbolic name containing the word `"public"` to log into the instance from your laptop.

- Use the symbolic name containing the word `"private"` for East-West traffic verification after accessing any test instance from your laptop.
```

```{figure} images/lab1-podred.png
---
height: 300px
align: center
---
DNS Names
```

### 2.4 Enterprise-Grade Tools

Alternatively, you can skip using the SSH client or a Jumpbox altogether and rely entirely on the `enterprise-grade tools` available on all Aviatrix Gateways.

- Navigate to **Copilot > Diagnostics > Diagnostics Tools**. From here, select the Spoke gateway that reside in the Application VPC, where you’ll also find the instances and the VMs. The Spoke gateways act as the default gateway, enabling you to run **ping**, **traceroute**, **packet capture**, and more.

```{figure} images/lab1-diagnosticstools.png
---
height: 300px
align: center
---
Diagnostics Tools
```

### 2.5 Network Segmentation: verification

#### 2.5.1 Verify connectivity between clients **within** the same BU using the Diagnostic Tools

You can generate the **ping directly** from the Spoke1 Gateway deployed in the same VPC as the BU1 Frontend istance. First, retrieve the private IP addresses of **BU1 Analytics** and **BU1 DB** from the `Cloud Assets` section.

```{figure} images/lab1-diagnosticstools11.png
---
height: 300px
align: center
---
Default Gateway and the Destinations
```

- Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**, filter for the keywords **"analytics"** and **"bu1-db"**, and capture the private IP addresses of the resulting VMs.

```{tip}
Use multiple tabs on your browser!
```

```{figure} images/lab1-diagnosticstools990.png
---
height: 200px
align: center
---
bu1-analytics
```

```{figure} images/lab1-diagnosticstools991.png
---
height: 200px
align: center
---
bu1-db
```

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**, and generate **ICMP** traffic from the **_ace-aws-eu-west-1-spoke1_** gateway for **BU1-Analytics** and **BU1-DB**.

```{figure} images/lab1-diagnosticstools992.png
---
align: center
---
bu1-analytics is reachable
```

```{figure} images/lab1-diagnosticstools993.png
---
align: center
---
bu1-db is reachable
```

#### 2.5.2 Verify connectivity between clients **within** the same BU Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>
    
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

Let's use the **_Dynamic Topology_** for retrieving the Public IP address of the BU1 Frontend instance. Navigate to **CoPilot > Cloud Fabric > Topology > Overview (default tab)**, then click on **Filters** and search for `"frontend"`.

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

Open your **SSH client** and enter the SSH command to log in to the **BU1 Frontend** instance.

```{figure} images/lab1-assets22.png
---
height: 400px
align: center
---
SSH
```

 - From the **BU1 Frontend** instance, ping the <ins>private IP address</ins> of the **BU1 Analytics** instance in GCP.
  - Confirm that **SSH** works from the **BU1 Frontend** instance to the **BU1 Analytics** instance.

**Ping** and **SSH** will be successful **within** the same network domain!

```{tip}
Clear the filter field to restore the full topology view, then search for "_analytics_".
```{figure} images/lab1-filter.png
---
height: 400px
align: center
---
Remove the filter
```

```{figure} images/lab1-pingok.png
---
height: 400px
align: center
---
ICMP
```

Also attempt the **SSH** command to BU1 Analytics.

```{figure} images/lab1-pingok20.png
---
height: 400px
align: center
---
SSH
```

Additionally, execute the **CURL** command directed at BU1 Analytics.

```{figure} images/lab1-pingok29.png
---
height: 150px
align: center
---
CURL
```

```{tip}
Repeat the verification for **BU1-DB**. The curl command against BU1-DB will not work, since this VM does not run an Apache web server.
```

#### 2.5.3 Ensure workloads do not have **cross-BU** connectivity using the Diagnostic Tools

```{figure} images/lab1-diagnosticstools1100.png
---
height: 300px
align: center
---
Default Gateway and the Destinations
```

- Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**. Search for `mobile-app` and obtain its private IP address. Next, search for `bu2-db` and obtain its private IP address.

```{figure} images/lab1-mobileapp00.png
---
height: 200px
align: center
---
BU2-MobileApp
```

```{figure} images/lab1-mobileapp0078.png
---
height: 200px
align: center
---
BU2-DB
```

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**, select the **_ace-aws-eu-west-1-spoke1_** gateway, and ping the two VMs’ private IP addresses that you fetched earlier. This outcome occurs because of the isolation between the two segments.

```{figure} images/lab1-mobileapp001.png
---
height: 250px
align: center
---
ping to BU2-MobileApp
```

```{figure} images/lab1-mobileapp00147.png
---
height: 250px
align: center
---
ping to BU2-DB
```

#### 2.5.4 Ensure workloads do not have **cross-BU** connectivity Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- From **BU1 Frontend** try to ping the <ins>private IP address</ins> of the **BU2 Mobile App**.
- From **BU1 Frontend** try to SSH **BU2 Mobile App** (use again its Private IP address!).

**Ping** and **SSH** commands will not work this time due to the separation between the two segments; <ins>these are two different Routing Domains</ins>.

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

#### 2.5.5 Network Segmentation section

* Check the **Network Segmentation** section on the CoPilot, and then look at the **Logical View**.

```{tip}
Navigate to **CoPilot > Networking > Network Segmentation > Overview**
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
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways**, select the **_ace-aws-eu-west-1-transit_** Gateway, then open **Routes DB** and inspect the allocation of the routes to the specific BU routing tables (e.g., BU1 or BU2).

The Route DB tab provides visibility for all VRFs.
```

```{figure} images/lab1-bu1vrf.png
---
height: 400px
align: center
---
Network Domain (aka VRF)
```

```{important}
You will notice that the `Segmentation` feature is indeed enabled. 
```{figure} images/lab1-newjoe.png
---
height: 250px
align: center
---
RIB
```

- Select the `Gateway Routes` tab and filter by one of the two segments, e.g., **BU1**. You will see only the routes associated with the BU1 Network Domain.

```{figure} images/lab1-bu1vrf000.png
---
height: 400px
align: center
---
Gateway Routes
```

* Use the <span style='color:#FF0000'>**FlowIQ**</span> functionality from the CoPilot, <ins> for inspecting the NetFlow Data.

```{tip}
Navigate to **CoPilot > Monitor > FlowIQ**, click on the `"+"` icon and filter based, for instance, on the `"Destination IP Address"` **172.16.211.100** (i.e. BU1 Analytics).

Do not forget to click on **Apply**.
```

```{figure} images/lab1-plus.png
---
height: 250px
align: center
---
FlowIQ Filter
```

```{caution}
You probably won't see any results immediately after clicking Apply. It may take a few more minutes for CoPilot to display the NetFlow information collected through CNSF.
```{figure} images/lab1-plus78.png
---
height: 350px
align: center
---
FlowIQ Filter
```

Then check the `"Flow Exporters"` widget, then from the drop-down menu select the **`"Aviatrix Gateway"`** widget: you will see the list of all the Aviatrix Gateways involved along the path.

```{figure} images/lab1-flowiq.png
---
height: 400px
align: center
---
FlowIQ
```

```{note}
On the **Aviatrix Gateway** widget, the very first gateway from the list is the gateway with the highest traffic (in KibiBytes).
```

* Use **Cloud Routes**  to identify the _originator_ of the route **172.16.211.0/24**.


```{tip}
Navigate to **CoPilot > Diagnostics > Cloud Routes**, search for the subnet **172.16.211.0/24** on the *search field* and then filter based on the following condition: "Gateway *contains* spoke1".

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
The **Originator** has the egress interface that corresponds to the **eth0** interface (i.e. the _LAN_ interface), which in turn means, direct connected.
```

```{figure} images/lab1-cloudroutes.png
---
height: 300px
align: center
---
Originator = eth0
```

* Utilize **Cloud Routes** to pinpoint the source of the **10.0.0.0/24**.

```{tip}
Navigate to **CoPilot > Diagnostics > Cloud Routes** and search the subnet **10.0.0.0/24** on the *search field*. 

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
Cloud Routes - Recursive Routing Lookup
```