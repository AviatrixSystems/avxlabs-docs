# Lab 1 - Network Domains

## 1. SCENARIO

The `Aviatrix Cloud Native Security Fabric (CNSF)` was deployed within the hybrid-cloud environment to establish initial network connectivity through IPSec tunnels.

```{figure} images/lab1-segmentation90.png
---
height: 400px
align: center
---
Initial Topology
```

The administrator divided the environment into two distinct _network domains_, **BU1** and **BU2**, for enhanced isolation.

```{figure} images/lab1-segmentation.png
---
height: 400px
align: center
---
Topology after the Network Segmentation
```

## 2. INITIAL CONFIGURATION

Prior to commencing the evaluation of the first request, configure CoPilot such that any modifications to CNSF are immediately reflected for operator assessment.

Under normal circumstances, the `Task Server's frequency` does not need to be adjusted. However, in a lab environment it is convenient to lower these values so that changes are immediately visible on our CoPilot—for example, a new route added to the routing table or a new connection appearing in the Dynamic Topology.

### 2.1 Task Servers

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

### 2.2 Dynamic Topology

After adjusting the Task Servers’ timers, the Dynamic Topology section of CoPilot enables inspection of the current Aviatrix Gateway connectivity, verification of instances within the Spoke VPCs, retrieval of IP addresses, and access to the Diagnostic Tools.

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

## 3. WHERE TO FIND THE IP ADDRESSES

The CoPilot is your **_single pane of glass_**, offering a complete database of IP addresses for all instances in the hybrid-cloud environment it governs.

The IP addresses can be easily obtained using **two** different methods, according to your preference.

### 3.1 Dynamic Topology - Properties

You can retrieve IP addresses from the **Properties** section of each instance in the Dynamic Topology.

- Navigate to **CoPilot > Cloud Fabric > Topology** and
expand the **_ace-aws-eu-west-1-spoke1_** VPC icon, click the test instance "bu1-frontend", then explore the `"Properties"` pane on the right-hand side.

```{tip}
To view the full hostnames of each node, select `"Show Labels"` from the bottom left sidebar.
```{figure} images/lab1-showlabels.png
---
height: 400px
align: center
---
Show Labels
```

```{figure} images/lab1-vpcicon.png
---
height: 400px
align: center
---
VPC icon
```

```{figure} images/lab1-ec2.png
---
height: 400px
align: center
---
Instance Properties Pane
```

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

```{important}
Important: choose the correct instance. The Aviatrix gateway uses the standard router icon (four arrows). Always select the User Instance, because the gateway itself cannot be accessed via SSH.
```

### 3.2 Cloud Workloads

Another important and useful method for obtaining IP addresses in CoPilot is through the **Virtual Machines** section.

- Navigate to **CoPilot > Cloud Resources > Cloud Workloads > Virtual Machines**. Perform a search for "_frontend_". The results will display the information you need about the instance, including its IP addresses.

```{figure} images/lab1-assets.png
---
height: 200px
align: center
---
Virtual Machines Inventory
```

```{important}
Always refer to your personal CoPilot for the IP addresses. 

<ins>These images are only examples!</ins>
```

## 4 TRAFFIC-GENERATING INSTANCES

The instances deployed within the multi-cloud environment automatically generate traffic using an application named `Gatus`.

```{figure} images/lab1-gatus909.png
---
height: 400px
align: center
---
All instances are currently generating traffic
```

```{important}
Gatus is an open-source, self-hosted health-check dashboard that periodically generates traffic to endpoints and presents their status, response times, and outages in a user-friendly web UI.
```

### 4.1 Gatus Dashboard

You can access the Gatus dashboard directly from your POD Portal.

```{figure} images/lab1-gatus000.png
---
height: 450px
align: center
---
Gatus from the POD Portal
```

You can select any instance within the three CSP environments to identify the traffic type it generates toward the other instances.

```{tip}
The credentials can be retrieved from your personal POD Portal. Once you have entered the credentials, click on **"Sign In"**.
```

- Select for example the EC2 instance "**_BU1 Frontend_**" and then enter the credentials to access the Gatus App!

```{figure} images/lab1-gatus001.png
---
height: 250px
align: center
---
BU1 Frontend
```

```{figure} images/lab1-gatus002.png
---
height: 350px
align: center
---
BU1 Frontend on the Dynamic Topology
```

```{figure} images/lab2-gatus03.png
---
height: 400px
align: center
---
Current situation on BU1 Frontend
```

After signing into the Gatus App, you'll observe that two protocols are currently in use: `ICMP` and `TCP`.

TCP is monitored on ports 22 and 80.

```{figure} images/lab2-gatus04.png
---
height: 400px
align: center
---
BU1-Frontend Gatus
```

```{note}
Reduce the polling timer from 5 minutes to **10 seconds** to expedite the visibility of the results.
```{figure} images/lab2-gatus05.png
---
height: 400px
align: center
---
Timer
```

## 5. ENTERPRISE-GRADE TOOLS ON THE AVIATRIX GATEWAYS

All Aviatrix gateways include built-in `enterprise-grade tools`. If you prefer, you can skip the Gateways Dashboard and connect directly to the instances via SSH/Guacamole (see the next section). The Spoke Gateway serves as the VPC’s default gateway, so you can run ping, traceroute, or packet captures right from the Aviatrix Gateway.

- Navigate to **Copilot > Diagnostics > Diagnostic Tools**. From here, select the Spoke gateway that reside in the Application VPC, where you’ll also find the instances and the VMs. The Spoke gateways act as the default gateway, enabling you to run **ping**, **traceroute**, **packet capture**, and more.

```{figure} images/lab1-diagnosticstools.png
---
height: 400px
align: center
---
Diagnostic Tools
```

Diagnostic Tools can also be launched directly from the Topology. Please select an Aviatrix Gateway (<ins>identified by the gateway icon with four arrows</ins>). After selecting the gateway, click the **wrench icon** on the right-hand side to initiate the Diagnostic Tools.

```{figure} images/lab1-diagnosticstools90.png
---
height: 400px
align: center
---
Diagnostic Tools from the Topology
```

```{figure} images/lab1-diagnosticstools90123.png
---
height: 400px
align: center
---
Diagnostic Tools
```

## 6. SSH client and Jump Box

Want direct access to the instances to generate traffic? Use your own SSH client, or the Guacamole  client (i.e., the Jumpbox) available in the POD Portal.

```{caution}
In the subsequent tasks, certain items will be labeled <span style='color:#33ECFF'>"BONUS"</span></summary>. These tasks pertain to generating traffic directly from the instances and may be skipped.

For corporate laptops, port 22 is typically blocked; therefore, <ins>please skip the Bonus tasks</ins>. If you are on a non-corporate device without port 22 restrictions, you may complete the Bonus tasks using an SSH client or Jump Box to access the instances.
```

### 6.1 Personal SSH client

- Open your <span style='color:orange'>**SSH**</span> client and connect to the public IPs of the instances you need. You’ll find the credentials in the POD Portal.

```{figure} images/lab1-publicip.png
---
align: center
---
Public IP
```

```{figure} images/lab1-credportal.png
---
align: center
---
Username and Password
```

```{important}
Please be aware that the AWS and GCP instances are reachable from your laptop, as they reside in **public** subnets; in contrast, the two Azure instances are not accessible from your laptop, since they reside in **private** subnets.
```

### 6.2 Jump Box

The Jumpbox button on the POD Portal lets you access the Guacamole client, a lightweight virtual desktop from which you can open an SSH terminal.

- The <span style='color:orange'>**Jumpbox**</span> provided via the POD Portal is especially useful in networks that block outbound port 22.

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

## 7. VALIDATION REQUEST

Having explored IP address retrieval and the available traffic-generation tools, you are now ready to begin the lab with its first task.

The initial request is to verify the separation between the two network domains. Note that the verification consists of checking the CoPilot configuration and, of course, testing with the test instances and the traffic they generate to each other. This section shows how to use Gatus, the enterprise-grade tools (<ins>the recommended method</ins>), the SSH client, and the Jumpbox.

### 7.1 Verify connectivity between clients **within** the same network domain Using Gatus

### 7.2 Verify connectivity between clients **within** the same network domain Using the Diagnostic Tools

You can generate the **ping directly** from the Spoke1 Gateway deployed in the same VPC as the BU1 Frontend istance. First, retrieve the private IP addresses of **BU1 Analytics** and **BU1 DB** from the `Cloud Workloads` section.

```{figure} images/lab1-diagnosticstools11.png
---
height: 400px
align: center
---
Default Gateway and the Destinations
```

- Navigate to **CoPilot > Cloud Resources > Cloud Workloads > Virtual Machines**, filter for the keywords **"analytics"** and **"bu1-db"**, and capture the private IP addresses of the resulting VMs.

```{tip}
Use multiple tabs on your browser!
```

```{figure} images/lab1-diagnosticstools990.png
---
height: 250px
align: center
---
bu1-analytics
```

```{figure} images/lab1-diagnosticstools991.png
---
height: 250px
align: center
---
bu1-db
```

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**, and generate **ICMP** traffic from the **_ace-aws-eu-west-1-spoke1_** gateway for **BU1 Analytics** and **BU1 DB**.

```{figure} images/lab1-diagnosticstools992.png
---
height: 200px
align: center
---
bu1-analytics is reachable
```

```{figure} images/lab1-diagnosticstools993.png
---
height: 200px
align: center
---
bu1-db is reachable
```

```{tip}
Diagnostics tools can also be launched from Topology.
```

BU1 Analytics and BU1 BD are reachable because they share the same network domain as BU1 Frontend.

### 7.3 Verify connectivity between clients **within** the same network domain Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

- SSH into the **BU1 Frontend**  instance in AWS.

```{important}
The credentials for accessing the EC2 instances and VMs are available in your personal POD Portal, nder the section labeled `"SSH for Test Instances"`.
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
height: 450px
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

### 7.4 Ensure workloads remain isolated across network domains Using Gatus.

### 7.5 Ensure workloads remain isolated across network domains Using the Diagnostic Tools.

We should ensure that, from the BU1 Frontend perspective, connectivity to the BU2 Mobile App and the BU2 DB is blocked.

```{figure} images/lab1-diagnosticstools1100.png
---
height: 400px
align: center
---
Default Gateway and the Destinations
```

- Navigate to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines**. Search for `mobile-app` and obtain its private IP address. Next, search for `bu2-db` and obtain its private IP address.

```{tip}
Feel free to open and maintain multiple tabs to keep the information you need readily available.
```

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

### 7.6 Ensure workloads remain isolated across network domains Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

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

### 7.7 Network Segmentation section

Now that we have verified connectivity within the Network Domain and across the two different network Domains, let's explore the Network Segmentation section and other useful areas of CoPilot.

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

### 7.8 FlowIQ

On the FlowIQ overview page, CoPilot shows all traffic that has traversed your Aviatrix transit network in the last hour, day, week, month, or a custom timeframe.

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

### 7.9 Cloud Routes

Cloud Routes provides a unified view of routing information for all resources in your Aviatrix transit network, including multicloud and on-premises (external/Site-to-Cloud) connections. This central view lets cloud engineers monitor across clouds without signing into each cloud provider console.

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
height: 200px
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