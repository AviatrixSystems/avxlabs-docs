# Lab 2 - Office Connectivity

Lab time: ~45 minutes  

**_Scenario_**:  Your development team needs direct access to the azure-build server in Azure. This requires a VPN connection from the office, into the cloud infrastructure.

During this lab we will build and verify the connectivity to your office.

```{figure} images/lab-2.png
---
height: 400px
align: center
---
Azure Deployment
```

## Lab 2.1 - Connection to the Office

### Description

Your on-prem Network team has already configured an IPSec tunnel on the office router, pointing to your Aviatrix Transit Gateway deployed in the Azure Transit VNET. In this lab you will need to configure the tunnel in the Aviatrix Transit Gateway for the tunnel to establish connectivity.

### Validate

In order to connect your Multicloud environment to the office, we will create an **_External Connection_**. This allows you to create a secure connectivity between the Cloud and the office with dynamic routing (BGP).

Now let’s add the office connection. In Copilot, navigate to **_Networking --> Connectivity --> External Connections (S2C)_**. Click the **`+ External Connection`** button and select `External Device` to add a new connection.

|                       |                                                                                                                                                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Name**              | azure-to-office                                                                                                                                                                                                    |
| **Connect Using**     | BGP                                                                                                                                                                                                                |
| **Type**              | IPSec                                                                                                                                                                                                              |
| **Local Gateway**     | azure-transit                                                                                                                                                                                                      |
| **Local ASN**         | 65pod[#] _For Pods 1-9, double pad the pod# with an additional 0 (ie. 65004). For Pods 10-99 single pad (ie. 65010). For Pods > 100, no padding is needed (ie. 65100)_                                             |
| **Remote Gateway IP** | <ip-address> _Please open a terminal session from your own pc and resolve the following FQDN to its IP address `onprem.pod[#].aviatrixlab.com`. Do NOT enter the FQDN on this field. Instead enter the IP address_ |
| **Remote ASN**        | 65000                                                                                                                                                                                                              |
| **Local Tunnel IP**   | 169.254.100.2/30                                                                                                                                                                                                   |
| **Remote Tunnel IP**  | 169.254.100.1/30                                                                                                                                                                                                   |
| **Pre-shared Key**    | mapleleafs                                                                                                                                                                                                         |

It should look something like the example below. Make sure to put in your own remote gateway IP and AS number.

```{figure} images/lab2-pod.png
---
align: center
---
Resolve the symbolic name with the "host" command
```

```{tip}
For Linux/Mac you can use the `"host"` command.

For Windows you can use the `"nslookup"` command.
```

```{figure} images/lab2-configure-ipsec.png
---
align: center
---
Configure BGPoverIPsec
```

* Hit **_Save_** to execute the changes.

After 1-2 minutes, you should see that the connection to the office is configured and up (green).

```{figure} images/lab2-tunnel-up.png
---
height: 200px
align: center
---
Tunnel up
```

In order to test connectivity between cloud and the office, a test VM is available in the office with the FQDN `client-int.pod[#].aviatrixlab.com`. The IPSec tunnel is up, but maybe there is another blocker that prevents connectivity. Try the following test:

* From CoPilot, navigate to **_Diagnostics-->Diagnostic Tools_**
* Click on `Gateway Instance` drop-down and select the **_azure-transit_** node.
* Select `Ping` in the vertical tabs and enter the hostname `client-int.pod[#].aviatrixlab.com` in the `Destination (IP / Host Name)` box. Then, click the **`Run`** button in the upper-right.

```{figure} images/lab2-onprem-ping.png
---
height: 400px
align: center
---
Diagnostic Tools
```

> Was the ping successful?

### Expected Results

The Site2Cloud connection should be green and the BGP session should be established.  Still, connectivity tests should fail due to the route filtering on the transit which we will fix in the next step.

Our lab environment now looks like this:  

```{figure} images/lab2-topology1.png
---
height: 400px
align: center
---
Topology with On-Prem Connectivity
```

## Lab 2.2 - Approve the Learned Routes

### Description

Aviatrix allows you to filter dynamically learned routes from external sites, and the azure-transit Gateway deployed in this lab has **Route Approval** enabled.  We want to approve the summarized route learned from on-prem.

### Validate

* Browse to the transit gateway settings in Copilot, under **_Cloud Fabric -> Gateways -> Transit Gateways_** and click on **_azure-transit_**.

```{figure} images/lab2-transitclick.png
---
height: 200px
align: center
---
azure-transit
```

* Navigate to the **Approval** tab and approve the pending CIDR.

If no CIDR's are showing up here, validate that the BGP peering has established under **_Troubleshoot -> Cloud Routes -> BGP Info_**. If the peering is down, you likely made a configuration error.

```{figure} images/lab2-route-approval-2.png
---
height: 200px
align: center
---
Approve route
```

```{figure} images/lab2-approve.png
---
height: 200px
align: center
---
Successful update
```

* Back to **_Diagnostics-->Diagnostic Tools_**
* Click on `Gateway Instance` drop-down and again select the **_azure-transit_** node.
* Select `Ping` in the vertical tabs and enter the hostname `client-int.pod[#].aviatrixlab.com` in the `Destination (IP / Host Name)` box. Again, click the **`Run`** button in the upper-right.

> Was the ping successful this time?

### Expected Results

After adding the connection to on-prem and approving the learned routes, the connectivity tests should be successful. The process of route approval allows you to be in charge of the learned routes!

## Lab 2.3 - Office Connectivity Tests

### Description

At this point, the office is connected to the Aviatrix Transit Gateway in Azure. This should provide the developers with connectivity to the azure-build server.

### Validate

* Open the Remote Access Server and open the **RDP - Client** (or navigate to `https://client.pod[#].aviatrixlab.com`). This is the on-prem Host.
* Open up the `Min` browser on the RDP desktop
* Navigate to `http://azure-build.pod[#].aviatrixlab.com` or just `http://azure-build`

### Expected Results

* You should see something similar to the following screenshot. Note you're connecting across the site2cloud connection to the dmz spoke via the transit gateway.
* Remember the **source port** for the next exercise

```{figure} images/lab2-priv-conn-test.png
---
height: 300px
align: center
---
http://azure-build
```

## Lab 2.4 - Verify the Flow Logs

### Description

CoPilot provides very rich visibility into all traffic going across the Aviatrix overlay network.  Once you generate some traffic, this should be visible in CoPilot.

### Validate

* Open **CoPilot** -> **Monitor** -> **FlowIQ** and click **Last 60 Minutes** to update the Date Range Filters
* Using the source port from the previous lab, let's create the following filter:
  * Click on the "+" button inside the **Filters** box.
* Under **Matches all conditions (AND)**, select the metric **Source Port**.
  * Select **equals**
  * Then, enter the source port from the website of the previous exercise and hit **_Apply_**. The Overview immediately filters to your criteria.

> New flow records might appear 1-2 minutes after they occurred, so just click on **Refresh Data** a few times to refresh the time period

```{figure} images/lab2-flowiq-filter.png
---
height: 200px
align: center
---
FlowIQ Filter
```

### Expected Results

* By filtering on the source port, you should be able to see all of the details behind the traffic generated from on-prem.

```{figure} images/lab2-flowiq-overview.png
---
height: 400px
align: center
---
FlowIQ Overview
```

* By clicking on the **Records** tab, you will see the raw flow records
* The most relevant fields are shown by default, but you can create custom views and show additional metadata relating to the traffic flows

```{figure} images/lab2-flowiq-records.png
---
height: 200px
align: center
---
FlowIQ Records
```

> **Note** - Gateway and Interface Names show up after a specific polling interval.  If the Gateway or Spoke Attachment is newly created, the fields will be displayed after the configured CoPilot polling interval (default is one hour)

* By looking at the flow records, you can trace the path that a flow took and also see over which gateways the flow crossed

## Lab 2.5 - Check office connectivity

### Description

While we have achieved our objective of providing the development team with access to the azure-build server, perhaps we have given them more access than desirable. Let's verify this.

### Validate

* Open the Remote Access Server and open the **RDP - Client** (or navigate to `https://client.pod[#].aviatrixlab.com`). This is the on-prem Host.
* Open up the `Min` browser on the RDP desktop
* Navigate to `http://localhost`

> Do we have access to more destinations than expected?

### Expected Results

* As you can see, you now have access to all servers in Azure.

```{figure} images/lab2-full-access.png
---
align: center
---
Full Access
```

## Lab 2.6 - Enable Network Segmentation

### Description

Now that we have established, that too much network access to the Azure environment has been granted to the development team, let's limit the exposure and reduce the security risks.

### Validate

* In the Copilot, go to **_Networking -> Network Segmentation -> Network Domains_**. and click the `Transit Gateways` button.
* In the transit gateway pane, enable segmentation for azure-transit and then click on **Save**.

```{figure} images/lab2-enable-segmentation.png
---
align: center
---
Transit Segmentation
```

```{figure} images/lab2-enable-segmentation-2.png
---
align: center
---
Enable Segmentation
```

### Expected Results

Now that segmentation is enabled on azure-transit, we can continue and build out Network Domains.

## Lab 2.7 - Create Network Domains

### Description

Let's create some network domains, which can be used for segmentation.

### Validate

In CoPilot, go to **_Networking -> Network Segmentation -> Network Domains_**. As you can see, we don't have any network domains set up currently. Use the **_+ Network Domains_** button to add the following domains and associations:

| Domain      | Association     |
| :---------- | :-------------- |
| Office      | azure-to-office |
| Azure-Build | azure-build     |
| Azure-Prod  | azure-prod      |
| Azure-DMZ   | azure-dmz       |

```{figure} images/lab2-add-network-domains.png
---
height: 400px
align: center
---
Add network domains
```

### Expected Results

After adding the network domains and associations, you should see the following in Copilot:

```{figure} images/lab2-network-domains-result.png
---
align: center
---
Network domains result
```

## Lab 2.8 - Check office connectivity

### Description

Now that we have implemented network segmentation, let's verify that we have limited the connectivity for developers in the office.

### Validate

* Go back to the Office connectivity dashboard and check the connectivity status.

> Were these connections successful?

### Expected Results

* None of the connections should succeed. We have successfully limited developer access to Azure, but now we have lost access to the build server as well!

```{figure} images/lab1-connectivity-from-office.png
---
align: center
---
Office connectivity
```

## Lab 2.9 - Create a connection policy

### Description

In order for our developers to be able to access the build server again, we need to create a connection policy, that allows traffic between the Office network domain and the Azure-Build network domain. In addition, the build server needs access to the azure-web and azure-app server. So we're going to need a connection policy for that as well!

### Validate

* Edit the **_Azure-Build_** network domain under **_Networking -> Network Segmentation -> Network Domains_** in Copilot.

```{figure} images/lab2-edit-network-domain.png
---
height: 200px
align: center
---
Edit network domain
```

* Now add the **_Office_** and **_Azure-Prod_** domain as a connected domain.

```{figure} images/lab2-add-connected-network-domain.png
---
align: center
---
Add network domain connection
```

### Expected Results

* The network domains should now be connected to each other, as shown on the screenshot below.
* Another great place to visualize the connectivity between network domains is **_Networking -> Network Segmentation-> Overview_**.

```{figure} images/lab2-add-connected-network-domain-result.png
---
align: center
---
Add Network domain connection
```

```{figure} images/lab2-network-domain-overview.png
---
height: 300px
align: center
---
Network domain overview
```

## Lab 2.10 - Check office connectivity

### Description

Now that we have implemented the connectivity policies, let's verify that we have limited the connectivity for developers in the office.

### Validate

* Go back to the Office connectivity dashboard and check the connectivity status.

> Were these connections successful?

### Expected Results

* The developers in the office now should only have access to the build server!

```{figure} images/lab2-access-to-build-only.png
---
align: center
---
Office connectivity
```

## Lab 2.11 - Fix Azure application

### Description

Now that we have enabled segmentation, we have not only limited connectivity from the office to Azure, but also between the network domains inside Azure. Since the application in Azure is dependent on connectivity between the DMZ and Prod VNET's, we need to fix this.

### Validate

First check that the connectivity is indeed broken.

* Log on to the Remote Access Server and connect to:
  * `http://azure-lb.pod[#].aviatrixlab.com/test`

As you can see, the Web web application is now broken:

```{figure} images/lab2-app-connectivity-broken.png
---
align: center
---
Office connectivity
```

* Check the connection policies on the **_Networking -> Network Segmentation-> Overview_** page. 

Do you see a line connecting Azure-DMZ and Azure-Prod?
* Create a connection policy between **Azure-DMZ** and **Azure-Prod**, similar to what you did in lab 2.9.

```{figure} images/lab2-add-connected-network-domain-2.png
---
align: center
---
Add connection policy
```

Check that the connectivity is restored.

```{figure} images/lab1-connectivity-3tier-app.png
---
align: center
---
Web app connectivity
```

### Expected Results

* After setting up the connectivity policy, you should be able to connect to all tiers of the Azure application again.

## Lab 2 Summary

* Congratulations - you successfully connected your office to Azure
* Not a single route table entry needed to be touched on the cloud provider side
* You have end to end visibility into all network traffic
* You have limited the scope of connectivity from the office, using network segmentation
* You have reestablished connectivity between the Azure-DMZ and Azure-Prod VNET's after implementing network segmentation.
