
# Lab-2: Aviatrix Cloud Firewall

## Lab 2.1 - Overview

In this lab, we will focus on the Islands VPCs in the AWS deployment. Egress-VPC1 and Egress-VPC2. Egress-VPC3 will be excluded from this section, but we will re-visit in the last lab with infrastructure as a code coverage while discovering the terraform capabilities. 

The logical topology is shown as below, where we have 3 Island  VPCs with Gatus Servers which are reaching out to internet via NAT Gateway without any egress filtering with native capabilities.


```{figure} images-lab2/1.png
---
height: 400px
align: center
---
Lab Overview
```


We will be working on the VPCs shown below in this hands-on lab.

```{figure} images-lab2/2.png
---
height: 400px
align: center
---
VPC List
```

The topology below will be visible after filtering with “unmanaged” button is toggled, as there are no Aviatrix gateways deployed in these VPCs, they are marked as unmanaged in the topology, once we start deploying the gateways they will be moved  to “managed” section.

```{figure} images-lab2/3.png
---
height: 400px
align: center
---
Topology
```

The VMs are the ones generating internet traffic, and our Aviatrix Nat Gateways  will control their egress traffic.

```{figure} images-lab2/4.png
---
height: 400px
align: center
---
Testing Virtual Machines
```

## Lab 2.2 - Deploying Spoke Gateways

Now we will enable Aviatrix to replace CSP-native NAT  Gateways for egress and after that we will secure the Island VPC. 
Go to `Cloud Fabric -> Gateways` and select `Spoke Gateways`. As there are no Spoke Gateways deployed, that page will look empty. 

```{figure} images-lab2/5.png
---
height: 400px
align: center
---
Aviatrix Spoke Gateway Configuration Page
```

Select `+Spoke Gateway` to deploy a new Aviatrix gateway. The required information to deploy the gateway can be collected from Cloud Assets section, as shown before. That already includes the Cloud, Region, and Availability Zone. 

```{figure} images-lab2/6.png
---
height: 400px
align: center
---
Creating Aviatrix Spoke Gateway
```

| **Field**            | **Value**                      |
| :------------------- | :----------------------------- |
| **Name**             | Egress-VPC1-GW                 |
| **Cloud**            | AWS Standard                   |
| **Account**          | aws-account                    |
| **Region**           | eu-west-2 (London)             |
| **VPC/Vnet**         | egress-vpc-1                   |
| **Instance Size**    | t3.medium                      |
| **Attach to Subnet** | egress-vpc-1-public-eu-west-2a |

Aviatrix Spoke Gateway deployment progress can be monitored in the notifications section. 


```{figure} images-lab2/7.png
---
height: 400px
align: center
---
Aviatrix Gateway Creation Process
```

After clicking `Show All Active Operations` button, it will load the                  	    `Monitor->Notifications->Tasks->Active Gateway Operations` page. After clicking the `View Details` button page progress details can be seen. 

```{figure} images-lab2/8.png
---
height: 400px
align: center
---
Aviatrix Gateway Creation Process
```
Once the deployment is completed, it can be validated as shown below:

```{figure} images-lab2/9.png
---
height: 400px
align: center
---
Aviatrix Spoke Gateway
```
At this point we can also see the change occurred on Topology Map where our gateway in Egress-VPC-1 is visible, together with the Gatus-1 VM.  

```{figure} images-lab2/10.png
---
height: 400px
align: center
---
Topology View
```

Even though the Aviatrix Spoke Gateway is deployed, it won’t automatically replace the existing deployment, so this will be a hitless and controlled deployment. As you can see below, the VPC Route Table for private table on Availability zone 2b is still forwarding the traffic to Nat Gateway 


```{figure} images-lab2/11.png
---
height: 400px
align: center
---
Checking VPC Route Table with Aviatrix Spoke Gateway 
```

```{figure} images-lab2/12.png
---
height: 400px
align: center
---
Checking VPC Route Table with AWS Console
```

Now repeat the same task above for deploying the second Aviatrix Spoke Gateway.

| **Field**            | **Value**                      |
| :------------------- | :----------------------------- |
| **Name**             | Egress-VPC2-GW                 |
| **Cloud**            | AWS Standard                   |
| **Account**          | aws-account                    |
| **Region**           | eu-west-2 (London)             |
| **VPC/Vnet**         | egress-vpc-2                   |
| **Instance Size**    | t3.medium                      |
| **Attach to Subnet** | egress-vpc-2-public-eu-west-2a |

```{figure} images-lab2/12_1.png
---
height: 400px
align: center
---
Aviatrix Gateway Creation Process
```

Once the Aviatrix Gateway deployment finishes, you can check the VPC/VNET route table and see the private route table points to CSP NAT Gateway.

```{figure} images-lab2/12_2.png
---
height: 400px
align: center
---
Checking VPC Route Table with Aviatrix Spoke Gateway 
```


## Lab 2.3 - Enabling Aviatrix Nat Gateway

There are 3 ways to enable Aviatrix Nat Gateway.
-	Enable from the Gateway Settings
-	Enable from Security->Egress section
-	Enable via Terraform as part of CI/CD pipeline

## Lab 2.3.1 - Enable from the Gateway Settings

Go to the `Cloud Fabric -> Gateways -> Spoke Gateways` and choose the `Egress-VPC1-GW` from the list and click `Settings` of the `Spoke Gateway`. Then enable Network Address Translation `Source Nat` and `Save`.

```{figure} images-lab2/13.png
---
height: 400px
align: center
---
Aviatrix Spoke Gateway Settings
```

Route Table change can be validated directly from Aviatrix CoPilot as shown below. As shown below, the default route was showing the Nat gateway as next-hop, but now the next-hop is changed to Aviatrix Spoke Gateway.

```{figure} images-lab2/14.png
---
height: 400px
align: center
---
Checking VPC Route Table Change with Aviatrix Spoke Gateway 
```

And as per the screenshot below, you can see the operation is hitless. This action was only achieved the NAT Gateway replacement, now you can basically remove the Nat Gateway from AWS for this VPC and start saving from upkeeping costs of CSP Nat Gateway. 

```{figure} images-lab2/15.png
---
height: 400px
align: center
---
Egress-1 Test Dashboard
```

As soon as we enable the Nat Gateway, we can start seeing the traffic flow details in the system, and now as next step we can start deploying rules to only allow desired egress traffic.

## Lab 2.3.2 - Enable from Security->Egress section

When we go to Security->Egress->Egress VPC/VNET and click “Enable Local Egress on VPC/Vnets”, we can choose the VPC where we have deployed our gateway and save, Aviatrix copilot will enable the Source Nat on the relevant Aviatrix Gateway and will take over the traffic.

```{figure} images-lab2/15_1.png
---
height: 400px
align: center
---
Aviatrix Local Egress 
```

```{figure} images-lab2/15_2.png
---
height: 400px
align: center
---
Enabling Aviatrix Local Egress on the Aviatrix Spoke Gateway
```

```{figure} images-lab2/15_3.png
---
height: 400px
align: center
---
Confirm the Local Egress Enablement
```

Once the Local egress is enabled , you can check the VPC/VNET route table and see the private route table points out to Aviatrix Spoke Gateway.

```{figure} images-lab2/15_4.png
---
height: 400px
align: center
---
Checking VPC Route Table Change with Aviatrix Spoke Gateway 
```

## Lab 2.3.3 - Enable via Terraform as part of CI/CD pipeline

We will come back to this topic at the last lab with Terraform. 


## Lab 2.4 - Deploy Egress Security

Now we can continue embedding the egress security as the CSP Nat Gateways are replaced at this point. 

Go to `Security-> Distributed Cloud Firewall` and click begin button to start configuring firewall rules.

```{figure} images-lab2/16.png
---
height: 400px
align: center
---
Begin Using Distributed Cloud Firewall
```

```{figure} images-lab2/17.png
---
height: 400px
align: center
---
Monitoring traffic via Distributed Cloud Firewall
```

SmartGroups is a very flexible tool that will allow the systems to classify and group the resources for easy management. The Aviatrix platform can classify the resources based on multiple attributes  as shown below. For this lab, we will use CSP-tags. 

```{figure} images-lab2/18.png
---
height: 400px
align: center
---
SmartGroup
```

Choose the Virtual Machine option and then tier and app tag. And click save that will create a Smart Group to group for the DCF.

For the egress, aws-instance-1 and aws-instance-1b is sharing the same rules, so we can group them in a single SmartGroups.  


```{figure} images-lab2/19.png
---
height: 400px
align: center
---
Test Virtual Machines
```

| **Field** | **Value**       |
| :-------- | :-------------- |
| **Name**  | SG-Egress-VPC-1 |
| **Tier**  | app             |

```{figure} images-lab2/20.png
---
height: 400px
align: center
---
SmartGroup Creation
```

After clicking the Preview button, all the included resources will be listed.

```{figure} images-lab2/21.png
---
height: 400px
align: center
---
SmartGroup Preview Resources
```

```{figure} images-lab2/22.png
---
height: 400px
align: center
---
SmartGroup Preview Resources
```

Now we will go to WebGroups to define the domains that will be allowed to the resources. 

| **Field**        | **Value**       |
| :--------------- | :-------------- |
| **Name**         | WG-Egress-VPC-1 |
| **Type**         | Domains         |
| **Domains/URLS** | *.terraform.io  |
| **Domains/URLS** | github.com      |
| **Domains/URLS** | ubuntu.com      |

```{figure} images-lab2/23.png
---
height: 400px
align: center
---
WebGroup Creation
```

Here we will define the DCF rule to allow SG-Egress-VPC-1 resources to have specific access only to selected domains in the WG-Egress-VPC-1 smartgroup.

| **Field**              | **Value**            |
| :--------------------- | :------------------- |
| **Name**               | Allow-Egress-Domains |
| **Source Groups**      | SG-Egress-VPC-1      |
| **Destination Groups** | Public Internet      |
| **WebGroups**          | WG-Egress-VPC-1      |
| **Protocol**           | Any                  |
| **Enforcement**        | ON                   |
| **Logging**            | ON                   |
| **Action**             | Permit               |
| **Priority**           | 10                   |


```{figure} images-lab2/24.png
---
height: 400px
align: center
---
Creating Egress Filtering Rule
```

Then click Commit to push the configuration to the system. 

```{figure} images-lab2/25.png
---
height: 400px
align: center
---
Creating Egress Filtering Rule
```

You might think, the system is not correctly blocking, that’s true, because to keep the environment controlled and safe, there is a Greenfield rule that allows the traffic. We will just go ahead and Turn Off the Enforcement for the Greenfield Rule, and commit.

```{figure} images-lab2/26.png
---
height: 400px
align: center
---
Disable Greenfield Rule
```

Now the rules will apply correctly, and the egress traffic will be limited to the selected domains only. 

```{figure} images-lab2/27.png
---
height: 400px
align: center
---
Egress-1 Dashboard confirms egress filtering
```
To troubleshoot issues or get more insights, you can also add a deny rule with low priority to see which traffic is denied. 

```{figure} images-lab2/28.png
---
height: 400px
align: center
---
Adding an explicit Deny rule with logging enabled 
```

Below, you can see which traffic is allowed and which one is denied, that will match with the Gatus-1 dashboard reporting.

```{figure} images-lab2/29.png
---
height: 400px
align: center
---
Monitoring Explicit Deny rule in action
```

The traffic can also be monitored from the FQDN Monitoring based on the VPC/Vnet selection as shown below.

```{figure} images-lab2/30.png
---
height: 400px
align: center
---
Monitoring via FQDN Monitoring section
```

For the next example, if we  want to keep the same domains reachable, we only need to add the new resources to the SmartGroups, and then the system will automatically apply the same rules to the traffic flow.

`aws-instance-2` resource has tag of `dev` as `environment`. Now we can just add one more resource type `Virtual Machine`, and fill with required information, then in the preview, all included resources can be seen. 

```{figure} images-lab2/37.png
---
height: 400px
align: center
---
Creating a new SmartGroup for Egress-2 instance
```

```{figure} images-lab2/38.png
---
height: 400px
align: center
---
Egress-2 Dashboard confirms egress filtering
```

## Lab 2.5 - Security Group Orchestration

In case you want to manage VM/Instance communication inside the same VPC/VNET, Aviatrix can help managing the security groups, that doesn’t require Source Nat to be enabled. As soon as the Aviatrix Gateway is deployed CoPilot can start orchestrating the SGs in the deployed VPC/Vnet.

This feature first needs to be enabled in the required VPC/VNET. This feature requires planning, as all inbound traffic will be blocked, and Aviatrix DCF will be managing the rules in the SGs/NSGs.

```{figure} images-lab2/39.png
---
height: 400px
align: center
---
Enabling Security Group Orchestration for a selected VPC
```

In our deployment, there are 2 aws-instances (aws-instance-1 and aws-instance-1b) that are deployed on private subnet and 1 Gatus VM that is deployed in public subnet. So, the permissions should be set accordingly. 

Create the SmartGroups for all instances and create firewall rules to allow or block traffic. As aws-instance-1 and aws-instance-1b is deployed in 2 different availability zones the traffic between them will be blocked. Similarly, the Gatus server will also try to reach from different subnet, so that traffic will also be blocked. 

We need to create 2 new SmartGroups and 2 new DCF rules to demonstrate this lab.


| **Field** | **Value**   |
| :-------- | :---------- |
| **Name**  | Gatus-1     |
| **Name**  | aws-gatus-1 |

| **Field** | **Value**       |
| :-------- | :-------------- |
| **Name**  | SG-Egress-VPC-1 |
| **Name**  | aws-instance-1  |
| **Name**  | aws-instance-1b |


```{figure} images-lab2/40.png
---
height: 400px
align: center
---
SmartGroup Creation for Gatus-1 VM
```

```{figure} images-lab2/41.png
---
height: 400px
align: center
---
SmartGroup Creation for AWS-instances in Egress-1 VPC
```


**_DCF Rule-1_:**

| **Field**              | **Value**                          |
| :--------------------- | :--------------------------------- |
| **Name**               | Allow-Gatus-1-to-AWS-Instance-1-1b |
| **Source Groups**      | Gatus-1                            |
| **Destination Groups** | SG-VPC-1-Resources                 |
| **Protocol**           | Any                                |
| **Action**             | Permit                             |
| **SG Orchestration**   | ON                                 |
| **Enforcement**        | ON                                 |
| **Logging**            | ON                                 |
| **Priority**           | 0                                  |

**_DCF Rule-2_:**

| **Field**              | **Value**              |
| :--------------------- | :--------------------- |
| **Name**               | Allow-HTTP-Intra-VPC-1 |
| **Source Groups**      | SG-VPC-1-Resources     |
| **Destination Groups** | SG-VPC-1-Resources     |
| **Protocol**           | TCP/80                 |
| **Action**             | Permit                 |
| **SG Orchestration**   | ON                     |
| **Enforcement**        | ON                     |
| **Logging**            | ON                     |
| **Priority**           | 1                      |


```{figure} images-lab2/42.png
---
height: 400px
align: center
---
Creating the DCF rules
```

Once both rules are created, commit the change and then you can see all the traffic will stop between these 2 instances other than TCP/80 traffic. 

This operation will be slower than the regular DCF operation as Aviatrix needs to push the changes through CSP APIs to deploy these rules. 



```{figure} images-lab2/43.png
---
height: 400px
align: center
---
Egress-1 Dashboard confirms intra-vpc traffic filtering
```

# End of Lab 2 

Ok, that's the end of Lab 2. Great job! We will come back to this part at the last lab with some terraform magic! 
