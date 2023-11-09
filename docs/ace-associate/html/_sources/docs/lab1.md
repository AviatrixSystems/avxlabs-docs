# Lab 1

## Lab Overview - What's in the lab??
Lab time: ~15 minutes

You have been tasked with building connectivity and security for an application in the cloud. It just so happens that the different tiers of the application are spread out across different VPCs and VNets in Azure and AWS.

The application components are already deployed in each of the VPCs/VNets, all you have to do is build out the connectivity. The app is only available via private IP address, which you can access using a web-based RDP server, which is available on-prem.

```{figure} images/lab1-topology.png
---
height: 400px
align: center
---
Lab Overview
```

## Lab 1.1 - Aviatrix CoPilot
The **Aviatrix CoPilot** is deployed in a *Shared Services VPC* in AWS, with the Aviatrix Controller. 
### Description
Log in to the Aviatrix CoPilot
### Validate
1. Open your browser and navigate to <a href="https://associate-portal.ace.aviatrixlab.com" target="_blank">ACE Associate Lab Portal</a>
2. Take note of the `username` and `password` for Aviatrix CoPilot.
3. Click on the **Open CoPilot** button

Alternatively, you can access the Aviatrix CoPilot using the information below:

**URL:** cplt.pod**X**.aviatrixlab.com  

```{note}
Replace X in the **URL** with the pod number assigned to you. 
>Username: ```student```   
>Password: ```refer to ACE Associate Lab Portal```
```

### Expected Results
You should be successfully logged into CoPilot and visualize the Dashboard section.

```{figure} images/lab1-dashboard.png
---
align: center
---
Dashboard
```

From the Navigation panel on the left-hand side, select **Cloud Fabric > Topology**. 

The initial topology should look similar to the following screenshot.  Throughout the labs, refresh the CoPilot Topology to show the connectivity, and latency of the links.  

```{figure} images/lab1-copilot-topology.png
---
align: center
---
CoPilot Topology
```

## Lab 1.2 - Remote Access Server
To access the cloud environment, you have been provided a **clientless** Remote Access Server, which is running in your “Data Center”. This conveniently offers a *web-based* HTML5 based RDP experience, so there should be no need to install any client.

### Description
Log in to the Remote Access Server.

### Validate
1. Open your browser and navigate to <a href="https://associate-portal.ace.aviatrixlab.com" target="_blank">ACE Associate Lab Portal</a>
2. Take note of the `username` and `password` for Remote Access Server.
3. Click on the **Open Server** button

Alternatively, you can access the Remote Access Server using the details below:

**URL:** client.pod**X**.aviatrixlab.com  

```{note}
Replace X in the **URL** with the pod number assigned to you. 
> **Username:** ```student```   
> **Password:** ```refer to ACE Associate Lab Portal```
```

```{figure} images/ras-login.png
---
align: center
---
Sample Remote Access Server Login
```

### Expected Results
Once you log in, you will see:
- An icon labeled `LX Terminal`. Use this for performing Pings from the Remote Access Server.
- An icon for `Firefox` web browser. Use this to perform other connectivity tests in Lab 2.

```{figure} images/ras-desktop.png
---
align: center
---
Sample Remote Access Server Desktop
```

```{note}
If you are experiencing sluggishness with the web-based RDP experience, you can use any RDP client. 

The username will be `podX`, where **X** is the pod number assigned to you.
```

## Lab 1.3 - Servers
The application tiers have been pre-deployed and are available under the following FQDNs and Ports:

| App Tier  | FQDN  | TCP Port  |
|:---------:|:-----:|:---------:|
| Web       | web.pod**X**.aviatrixlab.com  | 80  |
| App       | app.pod**X**.aviatrixlab.com  | 8080  |
| Database  | db.pod**X**.aviatrixlab.com | 443 |

### Description
Test connectivity between App tiers
### Validate
Throughout the labs, using the RDP Session, open Firefox and navigate to:

**URL:** web.pod**X**.aviatrixlab.com

You can also try to ping the different app tiers using the Terminal icon on the Desktop of the Remote Access Server:

```bash
ping web.podX.aviatrixlab.com
ping app.podX.aviatrixlab.com
ping db.podX.aviatrixlab.com
```

```{hint}
Replace **X** in the **URLs** with the pod number assigned to you. 

You can view the pod number by accessing the **ACE Associate Lab Portal**
```

### Expected Results
The connectivity is not yet built, so none of the tests should work!
