
# Lab-6: _(Optional)_ Future State: Aviatrix Cloud Firewall as the complete connectivity and security platform for your cloud 


Until this point in the labs, we have worked on the low friction use cases, replacing Nat gateways, SG/NSG orchestration, introducing egress firewall, and visibility capabilities, connecting on-prem resources to cloud, macro and micro segmentation. As you have seen already, all these easily deployed steps were very low friction and fast to deploy. 
Now we will look at the future and investigate what needs to be done to bring complete end-to-end Aviatrix Cloud Firewall capabilities to your network. 

For this lab, we will connect all the egress VPCs to your transit gateways and have a converged secure network. In a live environment, you can also remove the aws-tgw, and azure-rs etc. and deploy spoke gateways to these VPC/Vnets to have a single pane of glass connectivity and security platform. When you follow these step-by-step roadmaps, all the changes will become a low-friction deployment. We just need to approach this deployment with building blocks. 


## Lab 6.1 - Create a new Transit VPC
Our egress VPCs deployed in eu-west-2 region, so now we will create a new Transit VPC there so we can deploy a new Aviatrix Transit gateway for multi-region connectivity. 

First, we will go to Cloud Routes to find an IP Block. Search bar is very useful in this case. 


```{figure} images-lab6/1.png
---
align: center
---
Gateway Routes
```

`10.133.0.0/23` looks to be a free IP Block, so we can create the transit VPC using this block. Then we will go to `Cloud Assets->VPC/Vnets & Subnets` and click `+VPC/VNET` button.


| **Field**        | **Value**          |
| ---------------- | ------------------ |
| **Name**         | transit-aws-egress |
| **Region**       | eu-west-2          |
| **VPC CIDR**     | 10.133.0.0/23      |
| **VPC Function** | Transit+Firenet    |


```{figure} images-lab6/2.png
---
align: center
---
VPC/VNET Creation via Aviatrix Platform
```

Once it’s created it will be reflected in the VPC/Vnets section.

```{figure} images-lab6/3.png
---
align: center
---
Check the VPC/VNETs in the Aviatrix Platform
```
## Lab 6.2 - Create a new Aviatrix Transit Gateway and Connect to the others

Now we will create a new transit gateway in this newly created VPC. 

| **Field**                       | **Value**                  |
| ------------------------------- | -------------------------- |
| **Name**                        | transit-aws-egress         |
| **Account**                     | eu-west-2                  |
| **Region**                      | 10.133.0.0/23              |
| **VPC/Vnet**                    | transit-aws-egress         |
| **Instance Size**               | c5n.xlarge                 |
| **High Performance Encryption** | ON                         |
| **Attach to Transit Gateways**  | transit-aws, transit-azure |
| **Attach to Subnet**            | eu-west-2a, eu-west-2b     |

```{figure} images-lab6/4.png
---
align: center
---
Aviatrix Transit Gateway Creation
```

This configuration will create the new transit gateway and will connect it to the selected transit gateways, preparing the full mesh backbone deployment. Deploying Aviatrix Transit Gateway will take longer than the Aviatrix Spoke Gateways.

```{figure} images-lab6/5.png
---
align: center
---
Topology view
```

## Lab 6.3 - Connecting Aviatrix Spoke Gateways to Aviatrix Transit Gateways

Go to the `Cloud Fabric -> Gateways -> Transit Gateways` and choose the `transit-aws-egress`, then switch to Spoke Gateways connections section, and connect all 3 spoke gateways.

```{figure} images-lab6/6.png
---
align: center
---
Aviatrix Spoke Gateway to Aviatrix Transit Gateway Connectivity
```

Here in the routing tables, you can see the `Egress-VPC1-GW` can reach out to resources through Aviatrix Transit Gateway.

```{figure} images-lab6/7.png
---
align: center
---
Checking Gateway Routes 
```

## Lab 6.4 - IP Overlapping Issue

Now we have merged the 2 separate segments together, and ended up with overlapping IP scenario, now we need to apply destination NAT to resolve this problem. 

As seen from the list below, the IP blocks for Egress-VPC-1 and AWS-TGW, also Egress-VPC-2 and Azure-RS are overlapping, so that causes problems in the traffic. 

Now we need to apply destination NAT solution to get this traffic fixed. 

| **Egress-VPC-1**    |           |
| ------------------- | --------- |
| **aws-gatus-1**     | 10.1.2.53 |
| **aws-instance-1**  | 10.1.2.13 |
| **aws-instance-1b** | 10.1.2.21 |


| **AWS-TGW**      |           |
| ---------------- | --------- |
| **aws-instance** | 10.1.2.40 |

| **Egress-VPC-2**   |           |
| ------------------ | --------- |
| **aws-gatus-2**    | 10.2.2.56 |
| **aws-instance-2** | 10.2.2.4  |

| **Azure-RS**       |           |
| ------------------ | --------- |
| **azure-instance** | 10.2.2.40 |

| **Egress-VPC-3**   |           |
| ------------------ | --------- |
| **aws-gatus-3**    | 10.3.2.61 |
| **aws-instance-3** | 10.3.2.8  |

| **OnPrem**  |              |
| ----------- | ------------ |
| **Edge-VM** | 10.40.251.29 |


## _For traffic Aws-instance in AWS-TGW to Aws-instance-1 and Aws-instance-1b in Egress-VPC-1_

Go to `Cloud Fabric->Gateways->Spoke Gateways->Egress-VPC1-GW`, and select `Settings`. 

We need to set 2 settings in here, first enabling Destination NAT for Mapped-NAT feature to be activated, and second, we need to advertise the IP Block to the network while filtering the overlapping IP Block.

| **Egress-VPC1-GW**  | **Destination Nat**             |
| ----------- | ------------ |
| **Dst CIDR** | 10.111.2.0/24 |
|**Protocol**| all |
|**Connection**|None|
|**DNAT IPs**| 10.1.2.1-10.1.2.254|
|**Apply Route Entry**|ON|

| **Egress-VPC1-GW**  | **Routing**             |
| ----------- | ------------ |
| **Customize Spoke Advertised VPC/VNet CIDRs** | 10.111.2.0/24 |


```{figure} images-lab6/8.png
---
align: center
---
Destination NAT Configuration for IP Overlapping Scenario
```

When `AWS-instance` in `AWS-TGW` tries to reach to `AWS-Instance-1` and `1b` in `Egress-VPC-1`, it needs to use `10.111.2.0/24` IP block and then the spoke gateway will apply Mapped-NAT rules and translates it to `10.1.2.0/24` network and will keep the NAT translation session in place for response traffic. 

## _For traffic Aws-instance-1 and Aws-instance-1b in Egress-VPC-1 to Aws-instance in AWS-TGW_

Now we need to apply a Source-NAT solution in here to translate the source IP to the same IP Block, so the traffic can be translated on the Spoke Gateway and be delivered to the Aws-Instance in AWS-TGW.

| **Egress-VPC1-GW**  | **Source Nat**             |
| ----------- | ------------ |
| **Src CIDR** | 10.1.2.0/24 |
|**Protocol**| all |
|**Connection**|None|
|**SNAT IPs**| 10.111.2.1-10.111.2.254|
|**Apply Route Entry**|ON|


```{figure} images-lab6/9.png
---
align: center
---
Source NAT Configuration for IP Overlapping Scenario
```

## _For traffic Azure-instance in Azure-RS to Aws-instance-2 in Egress-VPC-2_

Go to `Cloud Fabric->Gateways->Spoke Gateways->Egress-VPC2-GW`, and select `Settings`.

We need to set 2 settings in here, first enabling Destination NAT for Mapped-NAT feature to be activated, and second, we need to advertise the IP Block to the network while filtering the overlapping IP Block.

| **Egress-VPC2-GW**  | **Destination Nat**             |
| ----------- | ------------ |
| **Dst CIDR** | 10.112.2.0/24 |
|**Protocol**| all |
|**Connection**|None|
|**DNAT IPs**| 10.2.2.1-10.2.2.254|
|**Apply Route Entry**|ON|

| **Egress-VPC1-GW**  | **Routing**             |
| ----------- | ------------ |
| **Customize Spoke Advertised VPC/VNet CIDRs** | 10.112.2.0/24 |


```{figure} images-lab6/10.png
---
align: center
---
Destination NAT Configuration for IP Overlapping Scenario
```

When `Azure-instance` in `Azure-RS` tries to reach to `AWS-Instance-2` in `Egress-VPC-2`, it needs to use `10.112.2.0/24` IP block and then the spoke gateway will apply Mapped-NAT rules and translates it to `10.2.2.0/24` network and will keep the NAT translation session in place for response traffic. 

## _For traffic Aws-instance-2 in Egress-VPC-2 to Azure-instance in Azure-RS_
Now we need to apply a Source-NAT solution in here to translate the source IP to the same IP Block, so the traffic can be translated on the Spoke Gateway and be delivered to the `Aws-Instance` in `AWS-TGW`.

| **Egress-VPC2-GW**  | **Source Nat**             |
| ----------- | ------------ |
| **Src CIDR** | 10.2.2.0/24 |
|**Protocol**| all |
|**Connection**|None|
|**SNAT IPs**| 10.112.2.1-10.112.2.254|
|**Apply Route Entry**|ON|

```{figure} images-lab6/11.png
---
align: center
---
Source NAT Configuration for IP Overlapping Scenario
```

# End of Lab 6

Ok, that's the end of Lab 6. Great job! 
