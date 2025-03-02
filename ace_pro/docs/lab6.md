# Lab 6 - FIRENET

## 1. Objective

The objective of this lab is to learn how to deploy **Palo Alto** Networks (aka PAN) VM-series firewalls in the **Transit VNet** and inspect traffic between the two Spoke VNets using firewall policies.

## 2. FireNet Overview (Firewall Network)

Aviatrix Firewall Network (aka *`FireNet`*) is a turnkey solution to deploy and manage firewall instances in the cloud, as shown in the topology below. Firewall Network significantly simplifies virtual firewall deployment and allows the firewall to inspect VPC/VNet/VCN to VPC/VNet/VCN (East West) traffic, VPC/VNet/VCN to Internet (Egress) traffic, and VPC/VNet/VCN to on-prem including partners and branches (North South) traffic.

In addition, Firewall Network allows you to scale firewall deployment to multi AZ and multi instances/VMs in maximum throughput active/active state without the SNAT requirement.

## 3. Topology

Up until now, in Azure you have only worked on **_azure-west-us-spoke1_**.

This lab will introduce finally **_azure-west-us-spoke2_** GW, that is a gateway in the top-right corner of this topology that is already deployed (hence it is red), <ins>although its attachment has not been deployed yet</ins>!

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 6 Initial Topology
```

## 4. FireNet Configuration

### 4.1 Azure Transit to Spoke Peering

First, you will need to configure the grey Aviatrix Spoke-Transit connection in the topology between **_azure-west-us-spoke2_** and **_azure-west-us-transit_**.

```{figure} images/lab7-spoketopology.png
---
height: 350px
align: center
---
Topology in Azure
```

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **_azure-west-us-spoke2_**, clicking on the pencil icon:

```{figure} images/lab7-spoke.png
---
height: 250px
align: center
---
Edit Spoke GW
```

Attach **_azure-west-us-spoke2_** (pre-configured VNet) to **_azure-west-us-transit_** as shown below.
Do not forget to click on **Save**.

```{figure} images/lab7-attachment.png
---
align: center
---
Attachment
```

```{important}
The **_azure-west-us-transit_** is already enabled for **FireNet** functionality.
```

### 4.2 Network Segmentation Configuration

Before proceeding with the FireNet implementation, we need to associate **_azure-west-s-spoke2_** to the <span style='color:lightgreen'>Green</span> Network Domain.

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Click the *pencil icon* to edit the Green network domain.

```{figure} images/lab7-editnd.png
---
height: 150px
align: center
---
Edit Green
```

Select the gateway **_azure-west-us-spoke2_** from the drop-down window, selecting the `"Associations"` field.

Then click **Save**:

```{figure} images/lab7-nd2.png
---
align: center
---
Association
```

```{figure} images/lab6-finalsegmentation.png
---
height: 400px
align: center
---
Topology
```

### 4.3 Firenet Workflow

```{figure} images/lab7-workflow.png
---
height: 850px
align: center
---
Workflow
```

#### 4.3.1 PAN Firewall Deployment

In this step you will be deploying a PAN firewall from the Aviatrix CoPilot with a `Bootstrap package`. 

The bootstrap package will take care of the following pre-provisioning on the Firewall:

- Interfaces mapping
- Policies creation
- Logging

Specifically, you will be deploying a firewall image called **Palo Alto Networks VM-Series Next-Generation Firewall Bundle 1**.

Aviatrix has already taken care of the *`subscription`* from the Azure Marketplace.

Go to **CoPilot > Security > FireNet > Firewall**, then click on the `"+ Firewall"` button.

```{figure} images/lab7-firenetbutton.png
---
align: center
---
FireNet
```

Deploy a Firewall by entering these settings within the `Deploy Firewall` window:

- **FireNet Instance**: <span style='color:#479608'>azure-west-us-transit</span>
- **Name**: <span style='color:#479608'>azure-west-us-pan</span>
- **Firewall Image**: <span style='color:#479608'>Palo Alto Networks VM-Series Next-Generation Firewall Bundle 1</span>

```{warning}
When you click on the **Firewall Image** drop-down window, you will see a `"Loading"` message. The message means that the Controller is in the middle of contacting the Marketplace through the API calls, therefore, be patient and wait for about <span style='color:#479608'>**_3 minutes!_**</span>
```

```{figure} images/lab7-marketplace.png
---
align: center
---
Marketplace contact under loading
```

- **Firewall Image Version**: <span style='color:#479608'>9.1.0</span>
- **Firewall instance Size**: <span style='color:#479608'>Standard_D3_v2</span>
- **Management Interface Subnet**: <span style='color:#479608'>azure-west-us-transit-Public-gateway-and-firewall-mgmt-**1** [**Note**: Make sure you do not select the subnets that begin with *az-1, az-2, or az-3*]</span>

```{caution}
Pay close attention to select the right subnet: **_azure-west-us-transit-Public-gateway-and-firewall-mgmt-1_**
```{figure} images/lab7-firenetsubnet.png
---
align: center
---
Management Interface Subnet
```

- **Egress Interface Subnet**: <span style='color:#479608'>azure-west-us-transit-Public-FW-ingress-egress-**1** [**Note**: Make sure you do not select the subnets that begin with *az-1, az-2, or az-3*]</span>
- **Authentication**: <span style='color:#479608'>Password</span>
- **Username**: <span style='color:#479608'>avxadmin [**Note**: username *admin* is not permitted in Azure]</span>
- **Password**: <span style='color:#479608'>Aviatrix123#</span>
- **Bootstrap Configuration**: <span style='color:#479608'>turn **ON** the knob</span>
- **Storage**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab6 section)</ins></span>
- **Storage Access Key**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab6 section)</ins></span>
- **File-Shared Folder**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab6 section)</ins></span>

Then click on **Deploy**.

```{figure} images/lab7-newone.png
---
align: center
---
POD Portal: lab 6 section 
```

```{figure} images/lab7-firenetcfg.png
---
align: center
---
Firenet Deployment Template
```

```{warning}
Please be patient - firewall deployment can take a long time, **_up to 20 minutes_**, due to the slow responsiveness of Azure API calls to prepare the firewall. Even after the firewall is created and is assigned a Public IP address, it doesn't mean it can be accessed immediately. 
```

```{figure} images/lab7-inprogress.png
---
align: center
---
Deployment in progress
```

At this time, the interface mapping, and security policy configuration are all being handled. The **_Aviatrix Controller_** does a lot of magic in orchestrating and manipulating route tables.

You will know the Firewall is created when you see the corresponding entry like this (refresh the page after roughly **10-15** minutes):

```{figure} images/lab7-url.png
---
height: 150px
align: center
---
Deployment completed
```

Check the `Notifications` section on your CoPilot, during the deployment process.

- Go to **CoPilot > Monitor > Notifications > Tasks**

```{figure} images/lab6-firenettaskalert.png
---
height: 150px
align: center
---
Notification Panel
```

```{caution}
Before proceeding with the next task, please wait **20** minutes!
```

#### 4.3.2 Firewall Vendor Integration

The `Vendor Integration` allows inserting the Summary Routes into the the Firewall's Routing Table.

Go to **CoPilot > Security > FireNet > FireNet Gateways**, click on the `"three-dot"` symbol on the right-hand side of the **_azure-west-us-transit_** row, and then click on `Vendor Integration`.

```{figure} images/lab7-vendor.png
---
height: 150px
align: center
---
Vendor Integration
```

Insert the following parameters in the `"Vendor Integration"` pop-up window.

- **Management IP Address**: <span style='color:#479608'>**Auto populated**</span>
- **Vendor**: <span style='color:#479608'>Palo Alto Networks VM-Series</span>
- **Username**: <span style='color:#479608'>avxadmin</span>
- **Password**: <span style='color:#479608'>[the password you entered earlier]</span>

Then click on **Save**.

```{figure} images/lab7-vendor2.png
---

align: center
---
Vendor Integration template
```

```{note}
Wait for some seconds for the **Vendor Integration** to complete.

If you see an error message related to the *ethernet1/2*, wait some additional minutes before clicking again on **Save**.
```{figure} images/lab7-message.png
---
align: center
---
Possible error message
```

```{figure} images/lab7-vendor3.png
---
align: center
---
Vendor Integration accomplished successfully
```

Go to **CoPilot > Security > FireNet > Firewall** and click on the **_azure-west-us-pan_** firewall

```{figure} images/lab7-vendor4.png
---
height: 150px
align: center
---
Click on the Firewall
```

You will see the RFC1918 routes that the Controller automatically programmed on the Firewall, through the `"Vendor Integration"`. Notice how each RFC1918 route has a prefix of `"AVX-"` to show that it is programmed by Aviatrix.

```{figure} images/lab7-vendor5.png
---
height: 400px
align: center
---
Vendor Integration outcome
```

```{caution}
IP address **168.63.129.16** is a virtual public IP address that is used to facilitate a communication channel to Azure platform resources. Customers can define any address space for their private virtual network in Azure. Therefore, the Azure platform resources must be presented as a unique public IP address.
```

#### 4.3.3 Inspection Policy

Now it is time to select the VPC/VNet that will be involved in the _FireNet inspection_.

Go to **CoPilot > Security > FireNet > FireNet Gateways**, click on the **_azure-west-us-transit_** Transit FireNet GW and then choose the `"Policy"` tab.

```{figure} images/lab7-inspection2.png
---
align: center
---
Policy tab
```

Then select each Azure spoke gateway one by one, click on `"Actions"` and choose `"Add"` in order to add a specific VPC inside the **Inspection Policy**.

```{figure} images/lab7-inspection3.png
---
align: center
---
Inspection Policy assignment
```

```{figure} images/lab7-inspection4.png
---
align: center
---
Inspection Policy accomplished
```

```{figure} images/lab6-inspectionpolicy.png
---
height: 400px
align: center
---
Inspection Policy accomplished
```

## 5. Firewall Configuration Verification <span style='color:#33ECFF'>(BONUS)</span></summary>

Once you have deployed the firewall, applied both the Vendor Integration and the Inspection Policy, you can optionally decide to log in to the firewall.

```{important}
Please bear in mind that you will have to accept a **_self-signed certificate_**. <ins>If you are using a corporate laptop, please ignore this verification and skim through this section to understand the purpose of this task</ins>.
```

 Try to click on the *hyperlink* of the firewall. You should be able to see the page where entering the credentials (refer to you POD portal).

 ```{figure} images/lab6-mgmtfw.png
---
height: 350px
align: center
---
URL
```

 ```{figure} images/lab6-firefox.png
---
height: 400px
align: center
---
Firefox
```

 ```{figure} images/lab6-edge.png
---
height: 400px
align: center
---
Edge
```

Once you access the firewall in your web browser via **HTTPS**, you might get a warning about an invalid certification based on your browser settings. This is just because it has a **_self-signed certificate_**. Navigate past that to get to the login prompt. Sign in as `avxadmin` as the username and the password you entered earlier.

```{figure} images/lab7-paloalto.png
---
align: center
---
PaloAlto Welcome page
```

Dismiss the Welcome splash screen. This is an indication that the firewall is ready.

### 5.1 Verify Routes Installed on Firewall

Verify the same RFC1918 routes exist on the PAN Firewall.

Navigate to **Network tab > Virtual Routers >** click on **default**.

```{figure} images/lab7-palo1.png
---
align: center
---
PaloAlto dashboard
```

Click on `"Static Routes"` tab. You should be able to see the same **RFC1918** routes with `"AVX-"` prefixes that were programmed by the Aviatrix Controller.

```{figure} images/lab7-palo2.png
---
align: center
---
Static Routes (RFC1918 routes)
```

### 5.2 Verify Firewall Policy

Continue to explore the configuration of the PaloAlto FW. Navigate to `Policies` tab to find out that the firewall was pre-configured with a simple **Allow-All** policy.

```{figure} images/lab6-paloalto.png
---
height: 350px
align: center
---
Allow-All
```

## 6. DCF Rules

In Lab 5 (_Aviatrix Cloud Firewall_), the DCF functionality was enabled. The currently permitted rules are:

1) The `"Egress-Rule"` , that is only allowing http/https traffic towards two defined domains. 
2) The `"ExplicitDenyAll"` with _Logging_=on
3) The `"Greenfield-Rule"`
4) The `"DefaultDenyAll"`, at the very bottom.

```{figure} images/lab7-dcfrule.png
---
height: 250px
align: center
---
DCF rules
```

### 6.1 Move the Greenfield-Rule

- Navigate to **CoPilot > Security > Distributed Cloud Firewall > Rules**, then click on Move button on the Greenfield-Rule entry and, choose `Top`. Do not forget to click `"Save Draft"`.

```{figure} images/lab6-new101.png
---
height: 300px
align: center
---
Move
```

Now click on the **Commit** button.

```{figure} images/lab6-new102.png
---
height: 250px
align: center
---
Commit
```

You have successfully re-established the `Deny-List` model!

```{important}
Deny-List Model (formerly known as _blacklist_) = basic access control mechanism that allows through all elements except those explicitly mentioned on a “deny” list.
```

Moreover, continue editing the **Greenfield-Rule**: click on the three-dot icon beside the Greenfield-Rule entry and select the option `"Turn On Logging"`.

```{figure} images/lab6-new105.png
---
height: 500px
align: center
---
Logging ON
```

**Commit** your final change again.

```{figure} images/lab6-new106.png
---
height: 300px
align: center
---
Commit
```

## 7. Connectivity Testing

Verify if there is connectivity between the instances deployed in Azure and between those instances and the instance deployed in GCP.

### 7.1 Connectivity Test Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select both **_azure-west-us-spoke1-test1_**, **_azure-west-us-spoke1-test2_** and **_gcp-us-central1-spoke1-test1_**

```{figure} images/lab6-new888.png
---
height: 250px
align: center
---
Gatus
```

```{figure} images/lab6-gatus90.png
---
height: 400px
align: center
---
azure-west-us-spoke1-test1
```

```{figure} images/lab6-gatus91.png
---
height: 400px
align: center
---
azure-west-us-spoke2-test1
```

```{figure} images/lab6-gatus92.png
---
height: 400px
align: center
---
gcp-us-central1-spoke1-test1
```

All three instances can reach each other!

```{important}
AWS US-EAST is not reachable from either Azure or GCP due to the requirement for a **Full-Mesh** configuration.
```

### 7.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

If you wish, you can also check the ICMP test using your SSH client.

**SSH** into **_azure-west-us-spoke<span style='color:#479608'>1</span>-test1_** and from there, ping **_azure-west-us-spoke<span style='color:#bb05b9'>2</span>-test1_** on its private IP.

```{figure} images/lab7-ping.png
---
align: center
---
Ping is successful
```

```{note}
Pings are passing because the **`Allow-all`** rule on the **PAN Firewall** is permitting traffic from any zone to any zone to pass.
```

Back on the PAN firewall, click on the `Monitor` tab. Then paste this string in the filter bar and hit **Enter**, which will apply the filter:

```bash
(addr in 192.168.1.10)
```

```{figure} images/lab7-monitor2.png
---
height: 250px
align: center
---
Monitor on the PaloAlto
```

Traffic is passing through firewall because **_azure-west-us-spoke1_** and **_azure-west-us-spoke2_** both are in the **Inspection Policy**.

```{tip}
Click on the **refresh** button to see the logs almost in continuous stream.
```

```{figure} images/lab7-smallrefresh.png
---
align: center
---
Ping GCP
```

While on **_azure-west-us-spoke<span style='color:#33ECFF'>1</span>-test1_**, ping **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab7-pinggcp.png
---
align: center
---
Ping GCP
```

This still matches the **`Allow-all`** firewall rule. Moreover, it works because of the `Connection Policy` we had configured in the **Network Segmentation** Lab.

Back on the PAN firewall, click the **refresh** button in the top-right corner to see the log entries for pinging the GCP spoke test VM.

```{figure} images/lab7-finalmonitor.png
---
height: 200px
align: center
---
Monitoring traffic towards GCP
```

## 8. DCF Verification

Now, let's check the `DCF Monitor` section:

- Go to **CoPilot > Security > Distributed Cloud Firewall** and filter out based on **ICMP**.

```{figure} images/lab7-finalmonitor00.png
---
height: 300px
align: center
---
Filter
```

You will immediately notice the logs that successfully matched the Greenfield-Rule!

```{figure} images/lab7-finalmonitor01.png
---
height: 250px
align: center
---
Logs
```

## 9. FW Path Verification

Let's verify that the traffic generated from both VNets is indeed diverted to the firewall.

```{figure} images/lab6-verification.png
---
height: 400px
align: center
---
The inspected VNets
```

Let's retrieve the **Private IP address** of the VM in  the **_azure-west-us-spoke2_** VNet.

- Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for the **_azure-west-us-spoke2-test<span style='color:#33ECFF'>1</span></summary>_** instance. Identify its Private IP address and copy it.

```{figure} images/lab6-searchfor.png
---
height: 200px
align: center
---
Search for azure-west-us-spoke2-test1
```

Now navigate to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**, select the **_azure-west-us-spoke<span style='color:#33ECFF'>1</span></summary>_** Gateway, then select the **Traceroute** command. Paste the IP address previously copied from the **Cloud Assets** section into the `Destination (IP / Hostname)` field, and then click on **Run**.

```{figure} images/lab6-diag00.png
---
height: 400px
align: center
---
Diagnostics Tools
```

Let's analyze the outcome from the `Traceroute` command:

```{figure} images/lab6-diag01.png
---
height: 350px
align: center
---
Traceroute
```

```{important}
The second hop (i.e. the firewall) is not responding because the LAN interface of the Palo Alto firewall is not configured to accept inbound UDP packets.
```

1) The **first** hop represents the Transit Firenet Gateway.
2) The **second** hop is actually the FW, which did not respond to the UDP packet because ICMP traffic is not enabled inbound on the LAN interface of the FW.
3) The **third** hop is again the Transit FireNet Gateway.
4) The **forth** hop is the Spoke  GW azure-west-us-spoke2
5) The **fifth** hop is the final destination: **_azure-west-us-spoke2-test2_**

```{figure} images/lab6-diag018.png
---
height: 350px
align: center
---
Hop by Hop
```

```{note}
In this scenario, the `Health Check` mechanism involves the Aviatrix Controller pinging the **_management interface_** of the Palo Alto Firewall every **5** seconds.
```

## 10. Final Considerations

```{important}
Maybe it's time to finally embrace the full-blown **`"Distributed Cloud Firewall"`** solution and get rid of the expensive NGFW!
```

After completing this lab, the overall topology will appear as follows:

```{figure} images/lab7-finaltopology2.png
---
height: 400px
align: center
---
Final Topology for Lab 6
```