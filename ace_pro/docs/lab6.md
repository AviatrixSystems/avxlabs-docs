# Lab 6 - FIRENET

## 1. Objective

The objective of this lab is to learn how to deploy **Palo Alto** Networks (aka PAN) VM-series firewalls in the Transit VNet and inspect traffic between the two Spoke VNets using firewall policies.
 
## 2. FireNet Overview (Firewall Nework)

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
Lab 7 Initial Topology
```

## 4. Configuration
### 4.1. Azure Transit to Spoke Peering

First, you will need to configure the grey Aviatrix Spoke-Transit connection in the topology between **_azure-west-us-spoke2_** and **_azure-west-us-transit_**.

Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and edit the Spoke Gateway **_azure-west-us-spoke2_**, clicking on the pencil icon:

```{figure} images/lab7-spoke.png
---
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

### 4.2. PAN Firewall Deployment

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
- **Egress Interface Subnet**: <span style='color:#479608'>azure-west-us-transit-Public-FW-ingress-egress-**1** [**Note**: Make sure you do not select the subnets that begin with *az-1, az-2, or az-3*]</span>
- **Authentication**: <span style='color:#479608'>Password</span>
- **Username**: <span style='color:#479608'>avxadmin [**Note**: username *admin* is not permitted in Azure]</span>
- **Password**: <span style='color:#479608'>[choose a **strong password** and remember it]</span>
- **Bootstrap Configuration**: <span style='color:#479608'>turn **ON** the knob</span>
- **Storage**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab7 section)</ins></span>
- **Storage Access Key**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab7 section)</ins></span>
- **File-Shared Folder**: <span style='color:#479608'>Retrieve this from your <ins>pod info (Lab7 section)</ins></span>

Then click on **Deploy**.

```{figure} images/lab7-newone.png
---
align: center
---
POD Portal: lab 7 section 
```

```{figure} images/lab7-firenetcfg.png
---
align: center
---
Firenet Deployment Template
```

```{warning}
Please be patient - firewall deployment can take a long time, **_up to 20 minutes_**, due to the slow responsiveness of Azure API calls to prepare the firewall. Even after the firewall is created and is assigned a Public IP address, it doesn't mean it can be accessed immediately. If you try accessing it too early, your experience may vary.
```

```{figure} images/lab7-inprogress.png
---
align: center
---
Deployment in progress
```

At this time, the interface mapping, security policy configuration, and RFC1918 static route creation are all being handled. The **_Aviatrix Controller_** does a lot of magic in orchestrating and manipulating route tables.

You will know the Firewall is created when you see the corresponding entry like this (refresh the page after roughly 10-15 minutes):

```{figure} images/lab7-url.png
---
align: center
---
Deployment completed
```

Even after that message, it doesn't mean you can access the firewall (i.e. **URL**). Within **5-10 minutes** after you receive confirmation about the firewall being created, you should be able to access it.

Now try to click on the *hyperlink* of the firewall. You should be able to see the page where entering the credentials (refer to you POD portal).

## 4.3. Firewall Configuration

Once you access the firewall in your web browser via **HTTPS**, you might get a warning about an invalid certification based on your browser settings. This is just because it has a **_self-signed certificate_**. Navigate past that to get to the login prompt. Sign in as `avxadmin` as the username and the password you entered earlier.

```{figure} images/lab7-paloalto.png
---
align: center
---
PaloAlto Welcome page
```

Dismiss the Welcome splash screen. This is an indication that the firewall is ready.

Do not end the firewall's HTTPS session yet. You will return to this web interface later.

## 4.4. Firewall Vendor Integration

Go to **CoPilot > Security > FireNet > FireNet Gateways**, click on the `"three dots"` symbol on the right-hand side of the **_azure-west-us-transit_** row, and then click on `Vendor Integration`.

```{figure} images/lab7-vendor.png
---
align: center
---
Vendor Integration
```

Insert the following paramenters in the `"Vendor Integration"` pop-up window.

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
Wait for some seconds for the Vendor Integration to complete.

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
align: center
---
Click on the Firewall
```

You will see the RFC 1918 routes that the Controller automatically programmed on the Firewall, through the `"Vendor Integration"`. Notice how each RFC1918 route has a prefix of `"AVX-"` to show that it is programmed by Aviatrix.

```{figure} images/lab7-vendor5.png
---
align: center
---
Vendor Integration outcome
```

## 4.5. Verify Routes Installed on Firewall

Verify the same RFC 1918 routes exist on the PAN Firewall.

Back on the **PAN Firewall**, navigate to **Network tab > Virtual Routers >** click on **default**.

```{figure} images/lab7-palo1.png
---
align: center
---
PaloAlto dashboard
```

Click on `"Static Routes"` tab. You should be able to see the same RFC 1918 routes with `"AVX-"` prefixes that were programmed by the Aviatrix Controller.

```{figure} images/lab7-palo2.png
---
align: center
---
Static Routes (RFC1918 routes)
```

```{caution}
Do not end the firewall's HTTPS session yet. You will return to this web interface later. 
```

## 4.6. FireNet Policy

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

## 5. Verification

Verify the traffic flows within Azure and from Azure to GCP as shown below, by following steps 5.1 - 5.2:

```{figure} images/lab7-topology2.png
---
height: 400px
align: center
---
Lab 7 Topology with FW deployed and the Inspection Policy applied!
```

## 5.1. Inside Azure

Before we can verify connectivity, we need to associate **_azure-west-us-spoke2_** to the <span style='color:lightgreen'>Green</span> Network Domain.

Go to **CoPilot > Networking > Network Segmentation > Network Domains**

Click the *pencil icon* to edit the Green network domain.

```{figure} images/lab7-editnd.png
---
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

After this step, this is how the topology should look like:

```{figure} images/lab7-finaltopology.png
---
height: 400px
align: center
---
Lab 7 Final Topology
```

```{warning}
On Lab 6 (Egress), the DCF functionality was enabled. The current permitted rules are the `"Inspect-DNS"`, that is only allowing traffic towards UDP port 53, and the `"allow-domains"` , that is only allowing http/https traffic towards two domains. Any other kind of traffic is denied because of the presence of the `"Explicit-Deny-Rule"`. 

Before launching any connectivity tests, <ins>you need to move the **_Greenfield-Rule_**  on the top of the list of the DCF rules!
```

```{figure} images/lab7-dcfrule.png
---
align: center
---
DCF rules
```

- Modify the **Greenfield-Rule's Priority**

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default)**, click on the the `"two arrows"` icon on the righ-hand side of the **Greenfield-Rule** and choose *`"Move Rule"`* at the very **Top**.
Then click on **Save in Draft**.
```

```{figure} images/lab7-top.png
---
align: center
---
Move at the Top
```

Do not forget to click on **Commit**.

```{figure} images/lab7-edit.png
---
align: center
---
Commit your changes
```

- Apply the "Logging" option to the Greenfield-Rule

```{tip}
Click on the pencil icon beside the Greenfield-Rule row, then turn on the toggle for Logging and click on **Save in Drafts**.

Once again do not forget to click on **Commit**.
```

```{figure} images/lab7-newone2.png
---
align: center
---
Edit the Greenfield-Rule
```

```{figure} images/lab7-newone3.png
---
align: center
---
Editing the Greenfield-Rule
```


```{figure} images/lab7-newone4.png
---
align: center
---
Commit
```

### 5.1.1 Launch connectivity test

**SSH** into **_azure-west-us-spoke<span style='color:#479608'>1</span>-test1_** and from there, ping **_azure-west-us-spoke<span style='color:#bb05b9'>2</span>-test1_** on its private IP.

```{figure} images/lab7-ping.png
---
align: center
---
Ping is successful
```

```{note}
Pings are passing because the **`Allow-all`** rule on the Firewall is permitting traffic from any zone to any zone to pass.
```

Back on the PAN firewall, click on the `Monitor` tab. Then paste this string in the filter bar and hit **Enter**, which will apply the filter:

```bash
(addr in 192.168.1.10)
```

```{figure} images/lab7-monitor2.png
---
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

## 5.2. Azure to GCP

While on **_azure-west-us-spoke<span style='color:#33ECFF'>1</span>-test1_**, ping **_gcp-us-central1-spoke1-test1_**.

```{figure} images/lab7-pinggcp.png
---
align: center
---
Ping GCP
```

This still matches the **`Allow-all`** firewall rule. Moreover, it works because of the Connection Policy we had configured in the **Network Segmentation** Lab.

Back on the PAN firewall, click the **refresh** button in the top-right corner to see the log entries for pinging the GCP spoke test VM.

```{figure} images/lab7-finalmonitor.png
---
align: center
---
Monitoring traffic towards GCP
```

After completing this Lab, the overall topology would look like this:

```{figure} images/lab7-finaltopology2.png
---
height: 400px
align: center
---
Final Topology for Lab 6
```

- Explore the logs on the Monitor section of the Distributed Cloud Firewall. Filter out based on protocol **ICMP**.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**
```

```{figure} images/lab7-newlog.png
---
align: center
---
Filter based on ICMP
```

```{figure} images/lab7-last.png
---
align: center
---
Logs on DCF Monitor section
```

```{important}
The same policy is present on both the PaloAlto firewall and on the Spoke Gateway...

Maybe it's time to finally embrace the full-blown **`"Distributed Cloud Firewall"`** solution, and getting rid of the NGFW!
```