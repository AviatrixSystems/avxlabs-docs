# Lab 2

## Build
Lab time: ~45 minutes

Let’s go build out the connectivity for our app!

![Lab Overview](images/lab1-topology.png)
_Figure: Lab Overview_

## Lab 2.1 - Client / Web Connectivity
### Description
Test Connectivity to Web App  
### Validate

```{hint}
 Use the Search on the left panel as shown in Lab 1, and search for Topology within CoPilot, or go to **_CoPilot_** > **_Topology_**, and then click the blue **azure-web-node_VNET_icon**.  The VNet should open up and you should see the Aviatrix Gateway.
* Click the Aviatrix Gateway icon, then on the right side, click **_Tools_** and then click on **_GAteway Diagnostics_**
```

![Topology](images/azure-vnet.png)  
_Figure: Gateway Diagnostics_

* This provides you with the Network Engineer's toolkit (ICMP-based tools, TCP/UDP connectivity, packet captures) directly from an Aviatrix Gateway
    * Try to ping:
        * client-int.pod**X**.aviatrixlab.com
        * app.pod**X**.aviatrixlab.com
        * db.pod**X**.aviatrixlab.com

```{note}
Replace X in the **URLs** with the pod number assigned to you. You can view the pod number by accessing the [ACE Associate Lab Portal](https://associate-portal.ace.aviatrixlab.com)
```
 
### Expected Results
We have not built any connectivity yet, so none of the connectivity tests should work yet.  

## Lab 2.2 - Attaching Spokes
### Description
Attach Spoke VPCs/VNets to their Transits
### Validate
* Open the CoPilot 
* Navigate to **_Cloud Fabric -> Gateways -> Spoke Gateways_** and edit the Spoke Gateway **_azure-web-node_**

![Attach Spokes](images/azure-web-node.png)
_Figure: Edit the Spoke Gateway_

Select the Transit Gateway **_azure-transit_** from the drop-down window from the "Attach To Transit Gateway" field, and then click on **_Save_**.
![Attach Azure Transit](images/azure-transit.png)
_Figure: Edit the Spoke Gateway_

* Perform the following actions:
    * Select _azure-web-node_ and connect it to the _azure-transit_
    * Select _azure-app-node_ and connect it to the _azure-transit_
    * Select _aws-db-node_ and connect it to the _aws-transit-gw_

```{note}
To speed things up, feel free to open multiple browser tabs, and run a Spoke Attachment step in each tab
```

```{hint}
You can verify if spokes were correctly attached by viewing notifications. On the top right corner, click on the **_task icon_**, you should see success notifications for the actions you performed above. See an example below:  
![Attachment Notifications](images/task-icon.png)
```

### Expected Results

Each attachment should take between 30-120 seconds. Check CoPilot Topology to see how the network looks after attaching the Gateways.  

![Toplogy](images/attach-topo.png)  
_Figure: Topology Spokes Attached_  

## Lab 2.3 - Test the Web App
### Description
Test the Web App
### Validate
* Open the Remote Access Server
* Open Firefox from the Desktop on the Remote Access Server
* Navigate to [http://web.pod**X**.aviatrixlab.com]()

```{note}
Replace **X** in the **URL** with the pod number assigned to you. You can view the pod number by accessing the [ACE Associate Lab Portal](https://associate-portal.ace.aviatrixlab.com)
```

### Expected Results
You should see something similar to this:  
![Web App](images/webapp-not-working.png)  
_Figure: Web App_

## Lab 2.4 - CoPilot Diagnostics
### Description
Using CoPilot Topology to Test Connectivity
### Validate
* In **_CoPilot_** use the search button to navigate to **_Topology_**, click the **_azure-web-node_vnet_** icon  
* Click on the Aviatrix Gateway icon and select **Tools > Gateway Diagnostics** on the right side of the screen
* Try to ping/traceroute:
    * client-int.pod**X**.aviatrixlab.com
    * app.pod**X**.aviatrixlab.com
    * db.pod**X**.aviatrixlab.com
* Check out the Active Sessions or try a Packet Capture

```{important}
**_There is a huge benefit to owning the data path in the cloud!  You have complete visibility and have the tools needed to troubleshoot and operate your network!_** 
```


### Expected Results
You should see something similar to this:
  
![CoPilot](images/copilot-ping.png)  
_Figure: CoPilot Ping_

```{note}
Since there is no connectivity to AWS, the DB node should not be pingable Find network issues quicker with Topology!
```

## Lab 2.5 - FlowIQ
### Description
Using CoPilot **FlowIQ** to debug flows
### Validate
1. Use the left hand search bar within CoPilot to locate **_FlowIQ_** 

![CoPilot](images/flowiq-screen.png)  
_Figure: CoPilot FlowIQ_

Select the **_Records_** tab.

1. Under **_Filters_**, click on the **+ symbol** and create the first condition: select **_Destination Port_**, **_equals_** and then port **8080** (traffic from Web to App).

![Add Filter](images/lab2-condition.png)
_Figure: First Condition_

1. Click **_Add Condition_** again, make sure to select **_OR_**, select **_Destination Port_** is equal to **443** (traffic from App to DB). 
   
![Add Filter](images/lab2-addcondition.png)
_Figure: Add Filter_
1. In the **_Time Period_** field, select **Last 60 Minutes**.
2. Add the **_Destination Port_** parameter among the existing columns.

![CoPilot](images/destination-port.png)  
_Figure: Destination Port_

6. Add also the **_TCP Flag Tags_** parameter among the existing columns.

![CoPilot](images/tcp-flag.png)  
_Figure: TCP Flag Tags_

7. Do not forget to click on **Apply**.

8. In the Records tab, you can view the raw flow logs. You should see the `App` (i.e. 10.x.`32`.x) trying to connect to the `DB` (i.e. 10.x.`64`.x) tier, but we only get a **SYN**.

### Expected Results
You should see something similar to this showing successful flows from Web to App, but unsuccessful from App to DB:  

![FlowIQ](images/syn.png)  
_Figure: FlowIQ_  

```{note}
No connectivity to the DB tier means that we only see SYNs. Use FlowIQ to get insights into all flows running over your secure cloud network.
```

## Lab 2.6 - CoPilot Topology
### Description
Using CoPilot Topology to visualize your secure cloud network.
### Validate
-	Log in to CoPilot
-	Search for **_Topology_**

### Expected Results
You should see something similar to this:
  
![Toplogy](images/attach-topo.png)  
_Figure: Topology_

```{note}
- Azure Spokes are connected to the Azure Transit
- AWS Spoke is connected to the AWS Transit. We do not have connectivity between AWS and Azure. Not yet at least!
```

**_Visualizing a network can be so helpful!_**

## Lab 2.7 - Multicloud Peering

### Description

By this point we should have verified that connectivity in Azure is good, but we are missing the connectivity between Azure and AWS. Aviatrix offers a simple but powerful method for interconnecting clouds.

### Validate

* Open the CoPilot
* Navigate **_Cloud Fabric -> Gateways -> Transit Gateways_** and edit the Transit Gateway **_aws-transit-gw_**, clicking on the pencil icon:
  
![Add Transit Peering](images/aws-transit-edit.png)
_Figure: Edit Transit Gateway_

* Select the Transit Gateway **_azure-transit_** from the drop-down window from the _"Peer To Transit Gateways"_ field, and then click on **_Save_**.

![Add Transit Peering](images/peering.png)
_Figure: Establish peering between Transit Gateways_
### Expected Results
* Verify that the peering has been established 
* Navigate **_Cloud Fabric -> Gateways -> Transit Gateways_** and click on the transit gateway **_aws-transit-gw_**

![Transit Peering](images/aws-transit-gw.png)  
_Figure: Select the Transit GW_

* Then select the tab **_Connections -> Transit-Transit peering_**, as depicted below.

![Transit Peering](images/peering-ok.png)  
_Figure: Transit Peering established_

```{warning}
You may need to refresh or wait **_60-120 seconds_** for the connection status to become green (UP).
- Check CoPilot Topology to verify the state of the links
```

**Congratulations!!! You have now built a Multicloud Network! Secure cloud networking has never been so easy...**

![CoPilot](images/peering-ok2.png)  
_Figure: CoPilot Transit Peering_  

After this lab, this is how the overall topology would look like:

![Final](images/lab2-final.png)  
_Figure: CoPilot Transit Peering_ 


## Lab 2.8 - Test the Web App
### Description
Check whether the Web App is up and running.

### Validate
* Using the RDP session, open Firefox and navigate to: 

[http://web.pod**X**.aviatrixlab.com]()
* If the site is already open, click the Refresh button

```{note}
Replace **X** in the **URL** with the pod number assigned to you. You can view the pod number by accessing the [ACE Associate Lab Portal](https://associate-portal.ace.aviatrixlab.com)
```

### Expected Results
You should see something similar to this, meaning the Database is still not accessible!
![Web App](images/webapp-db-down.png)  
_Figure: Web App_  

## Lab 2.9 - Debug the Egress Rules
### Description
The database is actually just a proxy to Amazon DynamoDB. Perhaps the proxy cannot reach DynamoDB.

### Validate
* After testing that the connection between Web and App works and seeing that the DB connection fails, login to the CoPilot
* Search within Copilot for **_Egress_**
* Navigate to **_Egress -> Monitor_**
* Select the VPC **_aws-db-node_**

![Web App](images/egress-monitor.png)  
_Figure: Egress_  


### Expected Results
* It appears that the Egress filter is not allowing access to:  **_dynamodb.us-west-2.amazonaws.com_**
* You should see somethiing like the following:

![Egress Search](images/egress-outcome.png)  
_Figure: Egress outcome_  
 
## Lab 2.10 - Modify the Egress Rules
### Description
Modify the Egress Rules.
### Validate
1. Open the Controller, navigate to **_Security_** -> **_Egress Control_**
2. Scroll down to **_Step #3 – Egress FQDN Filter_**
3. Click the **Edit** button next to **_Default-Egress-Policy_**
![Edit Egress Rule](images/edit-egress-rule.png)
_Figure: Edit Egress Rule_

4. Click **Add New** and enter:
    * **Domain Name:** `dynamodb.us-west-2.amazonaws.com`
    * **Protocol:** `tcp`
    * **Port:** `443`
    * **Action:** `Allow`
    * Click **Save** and then **Update**
![Add Egress Rule](images/add-egress-rule.png)
_Figure: Add Egress Rule_ 

### Expected Results
You should have seen **Allow** entries for **_*.ubuntu.com_** and **_github.com_**.  After adding an entry for **_dynamodb_**, your Web App should be working.  

## Lab 2.11 - Sign-in to the Web App
### Description
Now that we have built the connectivity, our Web App should be up and running.

### Validate
* Log in to the Remote Access Server
* Open Firefox on the Desktop and navigate to: [http://web.pod**X**.aviatrixlab.com]()

```{note}
Replace **X** in the **URL** with the pod number assigned to you. You can view the pod number by accessing the [ACE Associate Lab Portal](https://associate-portal.ace.aviatrixlab.com)
```

* Click **Sign In**, enter something in the **Comments** and click **Submit** to sign-in to the **WALL OF FAME**

### Expected Results
![DB Tier Working](images/db-working.png)  
_Figure: DB Tier Working_  

```{note}
You should see that all 3 App Tiers are now up and can talk to each other. You should also be able to register yourself in the form, and also be able to view the Wall of Fame!

**Nice work!** 
```