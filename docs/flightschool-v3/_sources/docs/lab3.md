
# Lab-3: Secure High-Performance Datacenter Edge


On-Prem to Cloud demands high throughput, and threats mandate securing the data whether on-disk or on-the-fly. Aviatrix Secure High-Performance Datacenter Edge provides you the line-rate encrypted connectivity helping customers overcome IPSec throughput limitation or CSP-native route count limitations and many more with its converged manner. With Aviatrix Edge, now cloud will come to customer’s on-premises environments. 


```{figure} images-lab3/1.png
---
align: center
---
Lab Overview
```


We will be focusing on the right side for this activity. This activity will include connecting brownfield environment to on-prem site with low friction. The Aviatrix Transit Gateway in AWS is connecting to AWS TGW, and Aviatrix Transit Gateway in Azure is connected to Azure Route Server. 

```{figure} images-lab3/2.png
---
align: center
---
Topology View
```

Servers on these 3 sites are trying to establish connectivity between each other. Once the connections between the gateways are established the servers should be able to reach out to each other.

```{figure} images-lab3/3.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/4.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/5.png
---
align: center
---
AWS Gatus Dashboard
```

## Lab 3.1 - Connecting Aviatrix Gateways on Azure and AWS 

In this activity we will establish connectivity between multi-cloud environments. 
Go to `Cloud Fabric -> Gateways -> Transit Gateways` and then choose either of the gateway to establish connectivity to the other. 

```{figure} images-lab3/6.png
---
align: center
---
Aviatrix Transit Gateway Section
```

Click `+Attachment` button.


```{figure} images-lab3/7.png
---
align: center
---
Aviatrix Transit Gateway Connection
```
Choose the transit gateway that needs to be connected, and click Save

```{figure} images-lab3/8.png
---
align: center
---
Aviatrix Transit Gateway Connection
```
Progress can be monitored in the notifications page.

```{figure} images-lab3/9.png
---
align: center
---
Aviatrix Transit Gateway Connection Progress
```

Topology will start building the connection and once it’s completed it will move from red to green in the map.


```{figure} images-lab3/10.png
---
align: center
---
Topology View while building the connection
```

```{figure} images-lab3/11.png
---
align: center
---
Topology View after the connection is built
```

After the connection is established and the routes are exchanged, the network will converge and there will be end-to-end reachability between the resources which reside  on Azure and AWS. 

```{figure} images-lab3/12.png
---
align: center
---
AWS Gatus Dashboard
```


```{figure} images-lab3/13.png
---
align: center
---
Azure Gatus Dashboard
```

Route tables on the transit gateways can also be checked in `Cloud Fabric->Gateways->Transit Gateways` and then choose your desired gateway and go to `Gateway Routes`, there the routes can be observed. 

```{figure} images-lab3/14.png
---
align: center
---
Transit Gateway Routing Tables
```


## Lab 3.2 - Connecting Aviatrix Edge Gateway to Azure and AWS

In this activity we will establish connectivity from on-prem to cloud. 

Go to `Cloud Fabric -> Hybrid Cloud -> Edge Gateways` and then choose the edge gateway to establish connectivity to the Aviatrix Transit gateways. 

```{figure} images-lab3/15.png
---
align: center
---
Edge Gateway Configuration
```

Click `+Attachment` button.

Choose the transit gateway that needs to be connected and click Save. Multiple connections can be configured at the same time.

```{figure} images-lab3/16.png
---
align: center
---
Edge to Cloud Connectivity
```

Progress can be monitored in the notifications page.

```{figure} images-lab3/17.png
---
align: center
---
Edge to Cloud Connectivity Progress
```

Topology will start building the connection and once it’s completed it will move from red to green in the map.

```{figure} images-lab3/18.png
---
align: center
---
Topology View
```

Route tables on the transit gateways can also be observed in `Cloud Fabric->Hybrid Cloud->Edge Gateways` and then choose the gateway and go to `Gateway Routes`, there the routes can be observed. 

```{figure} images-lab3/19.png
---
align: center
---
Edge Gateway Routes
```

After the connection is established and the routes are exchanged, the network will converge and there will be end-to-end reachability between the resources which reside  on On-Prem, Azure and AWS. 

```{figure} images-lab3/20.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/21.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/22.png
---
align: center
---
AWS Gatus Dashboard
```

## Lab 3.3 - Resiliency

Now we will dive into what-if scenarios a bit. In the real-world scenarios we can’t be certain about the connectivity we must be always prepared. 

## _What if connection between On-Prem to Azure goes down?_

This is a very likely event for any on-prem to cloud connectivity to go down. We will demonstrate this by removing the connection from On-Prem to Azure and validate the traffic flow from the On-Prem Server. 

Go to `Cloud Fabric -> Hybrid Cloud -> Edge Gateways` and choose the attachment and delete the azure connection from the corner and save.

```{figure} images-lab3/23.png
---
align: center
---
Edge Gateway Connections
```

Change can be observed from topology map, route table.

```{figure} images-lab3/24.png
---
align: center
---
Topology View
```

```{figure} images-lab3/25.png
---
align: center
---
Edge Gateway Routes
```

And as the result, there won’t be a problem with the traffic, because now the connection from Edge to Azure is sent towards AWS, so the traffic continues without an issue.


```{figure} images-lab3/26.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/27.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/28.png
---
align: center
---
AWS Gatus Dashboard
```

Now recover the connectivity back and wait until the connectivity is established correctly and reflected in the edge gateway’s route table accordingly.  It may take some time to reflect correctly.

```{figure} images-lab3/29.png
---
align: center
---
Recovery of Edge Gateway Connection
```

## _What if connection between On-Prem to AWS goes down?_

This is a very likely event for any on-prem to cloud connectivity to go down. We will demonstrate this with removing the connection from On-Prem to AWS and validate the traffic flow from the On-Prem Server. 

Go to `Cloud Fabric -> Hybrid Cloud -> Edge Gateways` and choose the attachment and delete the aws connection from the corner and save.


```{figure} images-lab3/30.png
---
align: center
---
Edge Gateway Connections
```

Change can be observed from topology map, route table.

```{figure} images-lab3/31.png
---
align: center
---
Topology View
```

```{figure} images-lab3/32.png
---
align: center
---
Edge Gateway Routes
```

And as the result, there won’t be a problem with the traffic, because now the connection from Edge to AWS is sent towards Azure, so the traffic continues without an issue.

```{figure} images-lab3/33.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/34.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/35.png
---
align: center
---
AWS Gatus Dashboard
```

Now recover the connectivity back and wait until the connectivity is established correctly and reflected in the edge gateway’s route table accordingly.  It may take some time to reflect correctly.


```{figure} images-lab3/36.png
---
align: center
---
Recovery of Edge Gateway Connection
```

## _What if connection between AWS to Azure goes down?_

This is a very unlikely event to occur, but we can’t be more prepared than this. We’d expect that will take the traffic towards on-prem to other cloud. We will demonstrate this with removing the connection from Azure to AWS and validate whether the traffic flow from the either Azure or AWS server.

Go to `Cloud Fabric -> Gateways-> Transit Gateways` and choose the attachment and delete the other transit gateway connection from the corner and save.

```{figure} images-lab3/37.png
---
align: center
---
Transit Gateway Connections
```

Change can be observed from topology map, route table.

```{figure} images-lab3/38.png
---
align: center
---
Topology View
```

```{figure} images-lab3/39.png
---
align: center
---
Transit Gateway Routes
```

And as the result, there will be a problem with the traffic, because as that will not be an efficient way to work and may end-up with very suboptimal traffic flows as well as costly results. The default behaviour is not to make the Edge Gateway capable of moving traffic between transit gateways. 

```{figure} images-lab3/40.png
---
align: center
---
AWS Gatus Dashboard
```


```{figure} images-lab3/41.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/42.png
---
align: center
---
Edge Gatus Dashboard
```

But that doesn’t mean Aviatrix is not coming up with a solution. In case of an urgent resolution requirement for this issue, edge gateway can provide connectivity to other cloud resources and relay the traffic with a small configuration change. 
Go to Cloud Fabric -> Hybrid Cloud-> Edge Gateways and select the gateway, then click to the settings and enable Transitive Routing.


```{figure} images-lab3/43.png
---
align: center
---
Enabling Transitive Routing on Edge Gateway
```

Once this option is enabled, the routing will re-converge and the traffic between Azure and AWS resources will continue using Edge Gateway as the transit. But very important to note that it should be assumed as a temporary solution.  

```{figure} images-lab3/44.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/45.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/46.png
---
align: center
---
AWS Gatus Dashboard
```

```{figure} images-lab3/47.png
---
align: center
---
Transit Gateway Routes
```

Now recover the connectivity back and wait until the connectivity is established correctly and reflected in the transit gateway’s route table accordingly.  It may take some time to reflect correctly.
Also remove the transitive routing configuration from the Edge Gateway. 


## Lab 3.4 - Route Filtering

In case there is a business need to not allow routes to exchange in between Aviatrix Transit Gateways. For example, you have an on-prem site, but you want the on-prem to stay in a single region due to data protection law such as GDPR. Then this rule can be excluded on the link between the transits so the route can’t converge, and the traffic will stop flowing. 

For this example, we will destroy the connection between on-prem and Azure. But the aim is not to allow traffic to flow from Azure to On-Prem. 

Go to `Cloud Fabric -> Hybrid Cloud -> Edge Gateways` and choose the attachment and delete the azure connection from the corner and save.


```{figure} images-lab3/48.png
---
align: center
---
Edge Gateway Connection
```

Change can be observed from topology map, route table.


```{figure} images-lab3/49.png
---
align: center
---
Topology View
```

```{figure} images-lab3/50.png
---
align: center
---
Edge Gateway Routes
```

And as the result, there won’t be a problem with the traffic, because now the connection from Edge to Azure is sent towards AWS, so the traffic continues without an issue.

```{figure} images-lab3/51.png
---
align: center
---
Edge Gatus Dashboard
```

But this doesn’t allow us to achieve the traffic to drop. So now we will filter the on-prem IP-block on the link between transit gateways, then the traffic from On-Prem to Azure should fail. 

IP block can be found from the transit-aws gateway’s Gateway routes section, after filtering with edge as keyword. For this example, it’s 10.40.251.0/29 and 10.40.251.16/28. 

```{figure} images-lab3/52.png
---
align: center
---
Aviatrix Transit Gateway routes
```
Now we will go ahead to the connection between the transit gateways and exclude these 2 IP Blocks. 

```{figure} images-lab3/53.png
---
align: center
---
IP Block Filtering
```
And as the result, Traffic between Edge and Azure will stop flowing while the traffic between AWS and Edge flows without any problems.

```{figure} images-lab3/54.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/55.png
---
align: center
---
Azure Gatus Dashboard
```


```{figure} images-lab3/56.png
---
align: center
---
AWS Gatus Dashboard
```

Now recover the connection and remove the excluded CIDRs for next lab activity.

## Lab 3.5 - Macro-Segmentation

Aviatrix gives flexibility to deploy macro-segmentation to keep resources isolated. Macro-Segmentation is a vrf-like solution which is defined in the system as Network domains. In case of need these domains can connect to each other to achieve vrf-leak type of scenario. 

Go to `Networking->Network Segmentation->Network Domains`, then select `Transit Gateways` to check if Network Segmentation is enabled, if not enable them. 


```{figure} images-lab3/57.png
---
align: center
---
Enabling Network Segmentation for Aviatrix Transit Gateways
```

For this deployment, we will create 3 network domains, and separate Azure, AWS, and Edge deployment from each other. 

| **Field**        | **Value**      |
| ---------------- | -------------- |
| **Name**         | Azure          |
| **Associations** | azure-rs-spoke |

| **Field**        | **Value** |
| ---------------- | --------- |
| **Name**         | AWS       |
| **Associations** | aws-tgw   |

| **Field**        | **Value**          |
| ---------------- | ------------------ |
| **Name**         | edge               |
| **Associations** | aviatrix-edge-site |

```{figure} images-lab3/58.png
---
align: center
---
AWS Network Domain Creation and Association
```

```{figure} images-lab3/59.png
---
align: center
---
Edge Network Domain Creation and Association
```

```{figure} images-lab3/60.png
---
align: center
---
Azure Network Domain Creation and Association
```

After this step, all the communication between Servers will fail, as now we separated the routing tables between each other, so became unreachable. 

```{figure} images-lab3/61.png
---
align: center
---
AWS Gatus Dashboard
```

```{figure} images-lab3/62.png
---
align: center
---
Azure Gatus Dashboard
```

```{figure} images-lab3/63.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/64.png
---
align: center
---
Network Domains Topology
```

The newly created routing tables can be seen from the Gateway routes for each of the domain. 

```{figure} images-lab3/65.png
---
align: center
---
Network Domain Routing Tables on Aviatrix Transit Gateways
```

Now we are going to create connections Edge and Azure, and Edge and AWS, which should allow Edge to communicate with Azure and AWS resources, but Azure and AWS won’t be able to communicate to each other. 


```{figure} images-lab3/66.png
---
align: center
---
Network Domain Connectivity
```

```{figure} images-lab3/67.png
---
align: center
---
Network Domain Connectivity Topology
```

Now the Edge Resources’ routes became visible and known to the Azure networking domain which enables the bi-directional communication between Edge and Azure Resources, the same will apply to the AWS network domain as well. 

```{figure} images-lab3/68.png
---
align: center
---
Gateway Routes in Network Domains on Aviatrix Transit Gateways
```
So, as you can see below, while Edge Resources can reach to both AWS and Azure resources, there are no communications are allowed between AWS and Azure resources.

```{figure} images-lab3/69.png
---
align: center
---
AWS Gatus Dashboard
```
```{figure} images-lab3/70.png
---
align: center
---
Azure Gatus Dashboard
```
```{figure} images-lab3/71.png
---
align: center
---
Edge Gatus Dashboard
```

To continue next lab activity, we can follow 2 ways.
-	Delete the network domains and everything will move back to default table
-	Add connection between AWS and Azure in the network domain which will keep all the network domains, but all will be converged. 

Once you apply your selection, confirm the servers’ dashboards for communication between all resources.

```{figure} images-lab3/72.png
---
align: center
---
Connectivity Recovered on Gatus Dashboards
```

## Lab 3.6 - Micro-Segmentation with Distributed Cloud Firewall 

Aviatrix also provides Zero-Trust in the resource level without agents. With help of SmartGroups, the resources can be identified, and rules can be applied.

We will define 3 SmartGroups for our test resources this time with another option; using CIDR.


| **Field**     | **Value**     |
| ------------- | ------------- |
| **Name**      | AWS-Resources |
| **IPs/CIDRs** | 10.1.2.40     |


| **Field**     | **Value**       |
| ------------- | --------------- |
| **Name**      | Azure-Resources |
| **IPs/CIDRs** | 10.2.2.40       |


| **Field**     | **Value**    |
| ------------- | ------------ |
| **Name**      | OnPrem       |
| **IPs/CIDRs** | 10.40.251.29 |


```{figure} images-lab3/73.png
---
align: center
---
SmartGroup Creation based on CIDRs for AWS-Resources
```

```{figure} images-lab3/74.png
---
align: center
---
SmartGroup Creation based on CIDRs for Azure-Resources
```

```{figure} images-lab3/75.png
---
align: center
---
SmartGroups are created for all 3 sites
```


We will now create the DCF rules to allow traffic flows shown below:
-	Edge->AWS port TCP:443
-	Edge->Azure port TCP:5000

```{figure} images-lab3/76.png
---
align: center
---
OnPrem to AWS Rules Creation
```

```{figure} images-lab3/77.png
---
align: center
---
OnPrem to Azure Rules Creation
```


Below the results can be seen, while Edge can reach out to Azure and AWS resources, the traffic initiated from either one will fail.

| **Field**              | **Value**         |
| :--------------------- | :---------------- |
| **Name**               | OnPrem-to-AWS-443 |
| **Source Groups**      | OnPrem            |
| **Destination Groups** | AWS-Resources     |
| **Protocol**           | TCP/443           |
| **Enforcement**        | ON                |
| **Logging**            | ON                |
| **SG Orchestration**   | ON                |
| **Action**             | Permit            |
| **Priority**           | 90                |

| **Field**              | **Value**            |
| :--------------------- | :------------------- |
| **Name**               | OnPrem-to-Azure-5000 |
| **Source Groups**      | OnPrem               |
| **Destination Groups** | Azure-Resources      |
| **Protocol**           | TCP/5000             |
| **Enforcement**        | ON                   |
| **Logging**            | ON                   |
| **SG Orchestration**   | ON                   |
| **Action**             | Permit               |
| **Priority**           | 100                  |

```{figure} images-lab3/78.png
---
align: center
---
DCF rules
```

```{figure} images-lab3/79.png
---
align: center
---
Edge Gatus Dashboard
```

```{figure} images-lab3/80.png
---
align: center
---
AWS Gatus Dashboard
```

```{figure} images-lab3/81.png
---
align: center
---
Azure Gatus Dashboard
```

# End of Lab 3

Ok, that's the end of Lab 3. Great job! 
