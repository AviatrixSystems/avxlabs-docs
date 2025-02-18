# Lab  SECURE HIGH-PERFORMANCE DATACENTER EDGE
This lab will demonstrate how securely govern the `Egress traffic`.

## 1. General Objectives

You are  asked to interconnect the on-prem DC in New York to your MCNA. An **Aviatrix Edge** device has already been provisioned and it got already registered to the existing **Aviatrix Controller**.

Aviatrix Secure High-Performance Datacenter Edge solution gives power back to the network administrators to deliver cloud connectivity without compromise.  The solution delivers encrypted, line-rate performance with single region, multi region, or multi cloud redundancy options and full visibility and troubleshooting capabilities end to end.

## 2. Initial set-up

Here is a view of the initial topology:
```{figure} images/hybrid-initial.png
---
height: 400px
align: center
---
Initial Topology
```
All applications in AWS and GCP have only **Private** IP’s

These are the CIDR blocks per each CSP:

- AWS = 10.0.1.0/24
- GCP = 172.16.1.0/24

The AWS instance and GCP instance are running several services on the following ports: 1433, 1521, 443, 5000, and 50100.

During the initial setup, MCNA was deployed. Both AWS and GCP environments are connected using the Aviatrix backbone.

## 3. Gatus Dashboards

All of the pre-deployed instances are running [Gatus](https://gatus.io/) and attempting to connect to each other on various ports. There are two gatus dashboard(s), deployed to `aws-gatus` https://aws.pod#.aviatrixlab.com/ and `gcp-gatus`, https://gcp.pod#.aviatrixlab.com/ visualize this connectivity continuously and in real-time. `Green` means a tcp connection was successful and `red` means it was unsuccessful.

```{note}
Above on url replace the **pod#**, with your assigned pod# e.g for **pod97** `aws-gatus` will become https://aws.pod97.aviatrixlab.com/
```
With pre-deployment and initial setup below is the gatus initial sample look
![Gatus cloud](images/aws-gatus-initial.png)

Note that the AWS/GCP and Edge connectivity sections are all `red`. These networks are not connected.
While AWS and GCP connectivity section are all `Green`. As These networks are already connected.

## 4. Edge Connectivity

Edge gateway is already deployed as shown in below topology.
![Edge Gateway](images/init-edge-gateway.png)

### 4.1. Attachment between Edge and the AWS Transit

Let's establish a peering between the Aviatrix Edge device and the AWS Transit Gateway in **US-WEST-2**. 

In the topology shown below, there is a workstation named "Workstation Edge" connected to the LAN router. Once this connection is made, indicated by the grey links, initiate a ping from the workstation to verify connectivity. In the topology shown above, there is a workstation named "Workstation Edge" connected to the LAN router. Once this connection is made, indicated by the grey links, initiate a ping from the workstation to verify connectivity.

![Edge Init](images/edge-init-connect.png)

Now it's time to establish Edge Gateway to AWS Cloud attachment! 

Go to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Gateway Attachment"` button, on the right-hand side of the screen.
![Edge Attach](images/edge-manage-attach.png)


Click on the `"+Attachment"` button.

![Edge Attach1](images/edge-attach.png)

Fill in the attachment template using the following settings:

- **Transit Gateway**: <span style='color:#479608'>aws-us-west-transit</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>

Do not forget to click on **Save**.

![Edge aws](images/edge-aws-attach.png)

Wait a few seconds for the Aviatrix Controller to establish the attachment. You will then see a confirmation message like below, indicating that the operation has been successfully completed.

![Edge confirm](../../ace_pro/docs/images/lab8-edge14.png)

Let's verify the presence of the attachment previously created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

![Edge Attach aws](images/edge-attach-aws.png)

### 4.2. Attachment between Edge and the Local Lan Router

Now it's time to establish Edge Gateway to Local Lan Network 

Go to **CoPilot > Networking > Connectivity > External Connections(S2C)** and click on the `"+External Connection To"` and select `"External Device"`
![Edge Lan](images/edge-lan-ini.png)

Fill in the External Connectivity template using the following settings:

- **Name**: <span style='color:#479608'>Edge-Onprem-Lan</span>
- **Connect Using**: <span style='color:#479608'>BGP</span>
- **Type**: <span style='color:#479608'>LAN</span>
- **Local Gateway**: <span style='color:#479608'>avx-equinix-edge-1</span>
- **Remote ASN**: <span style='color:#479608'>64900</span>
- **Remote LAN IP**: <span style='color:#479608'>10.40.251.1</span>
- **Local LAN IP**: <span style='color:#479608'>10.40.251.2</span>

Do not forget to click on **Save**.
Wait a few seconds for the Aviatrix Controller to establish the attachment.
Let's verify the presence of the LAN attachment previously created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

![Edge Attach Lan](images/edge-lan-attach.png)

The On-Prem Edge device is now connected to both the local LAN and the cloud.
By default, the Edge device advertises all cloud routes to the LAN router and all LAN routes learned from the LAN router to the cloud.
This configuration establishes a path from on-premises to both AWS and GCP.
Connectivity is confirmed on both the GCP and AWS gatus dashboards

![Edge Attach Lan](images/aws-dash-full.png)

This is how the Topology would look like after the creation of the attachment.
![Edge Attach Lan](images/hybrid-aws-complete.png)

The **Edge** device allows to extend all the Aviatrix functionalities to the remote DC!

### 4.3. Attachment between Edge and the GCP Transit
The Aviatrix Edge device is capable to be connected to multiple Transit Gateway.

Let's establish a peering between the Aviatrix Edge device and the GCP Transit Gateway in **us-central-1**. 

![Edge Attach Lan](images/gcp-initial.png)

Go to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `"Manage Gateway Attachment"` button, on the right-hand side of the screen.
![Edge Attach](images/edge-manage-attach.png)

Click on the `"+Attachment"` button.

![Edge Attach1](images/gcp-attach.png)

Fill in the attachment template using the following settings:

- **Transit Gateway**: <span style='color:#479608'>gcp-central-us-transit</span>
- **Local Edge Gateway Interfaces**: <span style='color:#479608'>WAN(etho)</span>
- **Attach over**: <span style='color:#479608'>**Public Network**</span>

Do not forget to click on **Save**.

![Edge aws](images/gcp-attach-complete.png)

Wait a few seconds for the Aviatrix Controller to establish the attachment. You will then see a confirmation message like below, indicating that the operation has been successfully completed.

![Edge confirm](images/gcp-attach-success.png)

Let's verify the presence of the attachment created on the Topology. 

Go to **CoPilot > Cloud Fabric > Topology > Overview (default)**.

![Edge Attach aws](images/hybrid-final-topo.png)

## 5. Edge as DCF 
The rapid adoption of cloud services has transformed how organizations operate, offering scalability, flexibility, and reduced costs. However, it also introduces new security challenges. Traditional firewalls and security measures struggle to keep up with the dynamic nature of cloud environments. This is where Aviatrix Edge Gateway, functioning as a Distributed Cloud Firewall, makes a significant impact.

Edge Gateway as DCF provide
**Enhanced Security:** Comprehensive protection across multi-cloud and hybrid cloud environments.
**Simplified Management:** Centralized control with a unified security framework.
**Scalability:** Seamless scalability to meet the expanding needs of your cloud infrastructure.
**Operational Efficiency:** Reduced complexity leading to improved agility and reduced operational overhead.

### 5.1 Connectivity from on Prem Network Workstation to Cloud .
Previously, we successfully established connectivity between our On-Prem Network and both AWS and GCP Cloud environments. The topology is as follows:

![Edge Attach aws](images/hybrid-final.png)

Let’s launch a connectivity test, from the Workstation “Edge” inside the on-prem network.
You can access the  `workstation Edge` https://edge.pod#.aviatrixlab.com , simply replace the **#** with your assigned pod number.
and login with username as `student`.

You will land on the Desktop of the Workstation Edge and from here launch the `LX Terminal`.

```{figure} ../../ace_pro/docs/images/lab8-newjoe4.png
---
align: center
---
LX Terminal
```

Now execute the **curl** command towards the private IP address of the **aws-instance** instance (**i.e. 10.0.1.10**) and **gcp-instance** instance (**i.e. 172.16.1.10**) on following ports
1433, 1521, 5000, 50100
You will get output similar like below.

![Edge Attach aws1](images/hybrid-work-1.png)

Also execute **curl** for https like below

![Edge Attach Work2](images/hybrid-workstation-2.png)

### 5.2 Implementiong DCF rules

There is a new security requirement stating that on-premises network workstations must not be able to communicate with the cloud over HTTPS or port 1521.

Go to **CoPilot > Cloud Fabric > Security > Distributed Cloud Firewall** and click on the `+Rule` button.

![Hybrid DCF1](images/hybrid-dcf1.png)

Fill in the Create Rule template using the following settings:

- **Name**: <span style='color:#479608'>On-Prem-to-Cloud-block-1521-443</span>
- **Source Groups**: <span style='color:#479608'>On-Prem-network</span>
- **Destination Groups**: <span style='color:#479608'>rfc1918</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Ports**: <span style='color:#479608'>1521, 443</span>
- **Action**: <span style='color:#479608'>Deny</span>

Make sure to **Logging** toggle on
Do not forget to click on **Save in Drafts**.

![Hybrid DCF2](images/hybrid-dcf2.png)

Do not forget to click on **Commit**.
![Hybrid DCF3](images/hybrid-dcf3.png)

Execute the previously run curl commands which we confirmed earlier that the applications hosted on the cloud are accessible from the On-Prem network.

Next, execute the same curl commands on ports 1521 and 443, where we have deployed the deny rule in Aviatrix DCF.

![Hybrid DCF6](images/hybrid-dcf6.png)



Note that the AWS/GCP and Edge connectivity sections are all `red`. These networks are not connected.
While AWS and GCP connectivity section are all `Green`. As These networks are already connected.

Now, let's **SSH** on the EC2 _**aws-us-east-2-spoke1-test1**_  and then launch the command _traceroute_ towards the VM _**gcp-us-central1-spoke1-test1**_ in GCP

```{figure} images/lab8-edgedouble21.png
---
align: center
---
aws-us-east-2-spoke1-test1 
```

- Install the **`inetutils-traceroute`** package, typing the following command:

```bash
sudo apt install inetutils-traceroute
```

```{caution}
You will be asked to type the student's password!
```

```{figure} images/lab8-edgedouble22.png
---
align: center
---
inetutils-traceroute
```

When you see this pop-up message, just click on the **Enter** button on your keyboard!

```{figure} images/lab8-popup.png
---
align: center
---
confirm
```

Now type the traceroute command towards the test VM in GCP:

```bash
traceroute 172.16.1.100
```

```{figure} images/lab8-edgedouble23.png
---
align: center
---
Traceroute
```

The traceroute will reveal that the destination is exactly **5** hops away.

```{figure} images/lab8-edgedouble25.png
---
align: center
---
5 hops
```

Let's harness the **as-path prepend** feature for manipulating the traffic. 

```{important}
The routes exchanged between transit gateways are considered `BGP-like routes`! This is because the Aviatrix Controller orchestrating the SD routing, also has to use a mechanism for the routing decision, and therefore these routes seem BGP routes, indeed they have some attributes similar to the attributes used with BGP routes. For instance, each Transit has its own `AS PATH`, and this is used for the best path selection process. Nevertheless, bear in mind that the control plane within the MCNA is based on `SDN` (Software Defined Networking).
```

The objective of this task is to define a **Primary** path through the Edge device, whereas the path between the Transit gateways will be used as a **Backup** path.

```{figure} images/lab8-primary.png
---
align: center
---
Primary and Backup
```

Let's first check the `Route DB` of the **_aws-us-east-2-transit_** GW.

Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the **_aws-us-east-2-transit_** Gateway.

```{figure} images/lab8-primary01.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Route DB"` tab, then on right-hand side  type **172.16.1.0** on the Search field.

```{figure} images/lab8-primary03.png
---
height: 250px
align: center
---
Route DB
```

```{figure} images/lab8-primary02.png
---
align: center
---
1 AS Path length
```

From the **_aws-us-east-2-transit_** perspective, the destination route `172.16.1.0` is far just one single AS (i.e. 64514)

Now let's apply the route manipulation. Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW.

```{figure} images/lab8-edgedouble30.png
---
align: center
---
aws-us-east-2-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section, then under the `AS Path Prepend` widget,  select the `gcp-us-central1-transit-peering` connection and type **two times** the AS number 64513. 

Of course, then click on **Save**.

```{figure} images/lab8-edgedouble31.png
---
align: center
---
as-path prepend
```

Let's repeat the same kind of configuration on the **GCP** Transit GW.

Go to **CoPilot > Cloud fabric > Gateways > Transit Gateways** and click on the **_gcp-us-central1-transit_** GW.

```{figure} images/lab8-edgedouble32.png
---
align: center
---
gcp-us-central1-transit
```

Select the `"Settings"` tab and then expand the `"Border Gateway Protocol (BGP)"` section, then under the `AS Path Prepend` widget select the `aws-us-east-2-transit-peering` connection and type **two times** the AS number 64514. 

Click on **Save** to apply the change!

```{figure} images/lab8-edgedouble33.png
---
align: center
---
as-path prepend
```

Now go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW, then select the `"Route DB"` tab and then once again, on the right-hand side, type **172.16.1.0/24** inside the Search field. This time the AS Path Length will turn out being equal to 3, due to to the route manipulation that harnessed the `as-path prepend` feature.

```{figure} images/lab8-path.png
---
height: 300px
align: center
---
As path length = 3
```

Now, let's launch again the traceroute towards 172.16.1.100 from the **_aws-us-east-2-spoke1-test1_**.

```{figure} images/lab8-almostdone.png
---
align: center
---
traceroute
```

The traceroute is still showing the Transit peering between AWS and GCP as the preferred path, although the `as-path prepend` was correctly applied earlier. 

There is another option that needs to be enabled in order to complete this lab. 

Go to **CoPilot > Cloud Fabric > Hybrid Cloud > Edge Gateways** and click on the `Edge` device.

```{figure} images/lab8-almostdone02.png
---
align: center
---
Edge
```

Select the `"Settings"` Tab and then expand the `"Routing"` section, afterwards turn on the knob `Transitive Routing` and do not forget to click on **Save**.

```{figure} images/lab8-almostdone03.png
---
align: center
---
edge
```

## 7. Final verification

Let's relaunch the traceroute towards 172.16.1.100 from the **_aws-us-east-2-spoke1-test1_**.

```{figure} images/lab8-almostdone04.png
---
align: center
---
traceroute
```

```{figure} images/lab8-almostdone05.png
---
align: center
---
6 Hops
```

Now go back to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and click on the **_aws-us-east-2-transit_** GW, then select the `"Route DB"` tab and then once again, on the right-hand side, type `172.16.1.0` inside the Search field.

This time the AS Path Length will turn out being equal to **2**. 

<ins>The best path is via the Edge!</ins>

```{figure} images/lab8-2path.png
---
height: 300px
align: center
---
As path length = 2
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab8-edge25.png
---
height: 400px
align: center
---
Final Topology for Lab 8
```