# Lab 1 - VPCs/VNets CREATION

## 1. Introduction

In this lab, you will create **3** VPCs/VNets, one in each cloud, i.e., Azure, AWS and GCP. 

The purpose of creating these VPCs/VNets is to familiarise yourself with the user interface (UI).

Refer to your POD assignment for CoPilot login information, as depicted below. Click on the `URL` button of the CoPilot (i.e "Open Copilot") and log in using the credentials assigned to your POD.

```{important}
Always refer to your personal POD portal for both passwords and additional information. Please bear in mind that the screenshots refer to a different POD and they are shown just as examples!
```

```{figure} images/lab1-portal.png
---
height: 250px
align: center
---
POD portal
```

```{caution}
You will get only access to the **AWS** Console!

Access to both the _Azure_ console and the _GCP_ console are not granted.
```

## 2. Azure VNet
### 2.1. Create Azure VNet

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Verify whether the CIDR range `192.168.12.0/24` is overlapping with an existing in used cidr or not, as shown below.

```{figure} images/lab1-vnet1.png
---
height: 300px
align: center
---
Searching for a subnet conflict
```

Let’s create an **Application/Spoke** VNet. 

Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-vnet2.png
---
align: center
---
VNet creation
```

Insert the following values:

 - **Name**: <span style='color:#479608'>azure-us-central-spoke1</span>
 - **Cloud**: <span style='color:#479608'>Azure Global</span>
 - **Account**: <span style='color:#479608'>azure-account [use the drop-down window]</span>
 - **Region**: <span style='color:#479608'>Central US [use the drop-down window]</span>
 - **VPC CIDR**: <span style='color:#479608'>192.168.12.0/24</span>
 - **VPC Function**: <span style='color:#479608'>Default [use the drop-down window]</span>

Then click on **Save**.

```{figure} images/lab1-vnet3.png
---
align: center
---
VNet template
```

You will immediately notice a message on the top-right corner.

```{figure} images/lab1-vnet4.png
---
align: center
---
"In progress" message
```

From the CoPilot search bar, type `“task”` and then click the search result `“Notifications / Tasks”`. Observe the VNet creation Task and wait for it to complete.

```{figure} images/lab1-vnet5.png
---
height: 250px
align: center
---
Notification
```

```{note}
VPC Functions:

**1) Default** = Spoke VPC

**2) Transit + FireNet** 
```

It will take about **4-5** minutes for the creation of the VNet. You can periodically check the status of the task, clicking on the top right-hand side, on the *hourglass icon*. Click on the refresh button. Once the task gets colored in green, you can finally assume that the VNet was successfully created.

```{important}
**Clean up the SEARCH FIELD to restore the navigation panel view.**
```{figure} images/lab1-cleanup.png
---
align: center
---
Clean up the Search Field
```

```{figure} images/lab1-new.png
---
align: center
---
Hourglass icon
```

Go back to **Cloud Resources > Cloud Assets > VPC/VNets & Subnets** and type the name of the VNet previously created, **`azure-us-central-spoke1`**, inside the search field, then expand the VNet to explore the additional subnets created by the Aviatrix Controller.

```{tip}
**Click on the refresh button!**
```

```{figure} images/lab1-new2.png
---
height: 250px
align: center
---
Confirmation
```

```{note}
It will take <ins>**10** additional minutes</ins> to see all the four subnets (i.e 2x public subnets and 2x private subnets). 

Please, be patient! 

<ins>Continue with the subsequent tasks and afterwards verify that all the underlay constructs got deployed, successfully.</ins>
```

## 3. AWS VPC

### 3.1. Create AWS VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Verify once again whether the CIDR range `10.0.22.0/24` is overlapping with an existing CIDR or not, as shown below.

```{figure} images/lab1-newpic5.png
---
height: 400px
align: center
---
Searching for a subnet conflict
```

This time let’s create an **Application/Spoke** VPC. Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-vpc1.png
---
align: center
---
VPC creation
```

Insert the following values:

 - **Name**: <span style='color:#479608'>aws-us-west-2-spoke1</span>
 - **Cloud**: <span style='color:#479608'>AWS</span>
 - **Account**: <span style='color:#479608'>aws-account [use the drop-down window - **DO NOT** select the aws-admin]</span>
 - **Region**: <span style='color:#479608'>us-west-2 (Oregon) [use the drop-down window]</span>
 - **VPC CIDR**: <span style='color:#479608'>10.0.22.0/24</span>
 - **VPC Function**: <span style='color:#479608'>Default [use the drop-down window]</span>

Then click on **Save**.

```{figure} images/lab1-vpc2.png
---
align: center
---
VPC configuration
```

Monitor the progress of the VPC creation through the **hourglass** icon.

```{figure} images/lab1-vpc3.png
---
height: 250px
align: center
---
Hourglass icon
```

Verify the VPC creation in the VPC list. Hit the **Refresh icon** if you do not see the CIDR immediately (alternatively, <ins>refresh the web page for triggering the change</ins>). 

It will take a little bit for reflecting into the "VPC/VNets & Subnets" section (almost **2-3** minutes).

You can filter by CIDR `10.0.22.0/24`.

```{figure} images/lab1-vpc4.png
---
height: 250px
align: center
---
Verification
```

### 3.2. Verify from AWS Console

Log in to the **AWS console**. Refer to your pod info for login information (this screenshot is for **Pod 2**).

```{figure} images/lab1-newaws.png
---
align: center
---
AWS console
```

```{figure} images/lab1-vpc5.png
---
align: center
---
AWS console
```

Change the region to `Oregon (us-west-2)` in the top-right corner and invoke the **VPC** service.

```{figure} images/lab1-newpic6.png
---
align: center
---
Oregon region
```

```{figure} images/lab1-newpic61.png
---
align: center
---
Invoke VPC service
```

Now click on the VPCs link, as depicted below:

```{figure} images/lab1-newpic611.png
---
align: center
---
VPCs
```

You can see the `VPC` created with the assigned CIDR block.

```{figure} images/lab1-new3.png
---
height: 150px
align: center
---
VPC
```

From there, navigate to `Subnets` in the navigation panel, on the left-handf side.

As you can see, the Controller has created 1 Public Subnet and 1 Private Subnet per Availability Zone. Since us-west-2 has 4 AZs, therefore **eight** subnets are created.

```{figure} images/lab1-newpic7.png
---
height: 400px
align: center
---
Subnets created by the Aviatrix Controller
```

From there, navigate to `Route Tables`.

Here, also, you can see the **eight** route tables that the Aviatrix Controller created. They are mapped to each subnet. The Public Routing Tables have a 0/0 route pointing to the Internet Gateway, that the Controller also deployed.

```{figure} images/lab1-rt.png
---
align: center
---
Routing Tables created by the Aviatrix Controller
```

From there, navigate to `Internet gateways`.

Here, also, you can see the **IGW** created by the Aviatrix Controller.
You can notice that the IGW has the same name of the VPC that you defined, moreover it is also attached to this VPC.

```{figure} images/lab1-igw.png
---
align: center
---
IGW created by the Aviatrix Controller
```

## 4. GCP VPC

### 4.1. Create GCP VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Before starting the deployment of the VPC in GCP, verify once again whether the CIDR range `172.16.22.0/24` is overlapping with an existing cidr or not, as shown below.

```{figure} images/lab1-newpic8.png
---
height: 400px
align: center
---
Verification
```

This time let’s create an **Application/Spoke** VPC. Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-gcp1.png
---
align: center
---
VPC creation
```

Insert the following values:

 - **Name**: <span style='color:#479608'>gcp-us-west2-spoke1</span>
 - **Cloud**: <span style='color:#479608'>GCP</span>
 - **Account**: <span style='color:#479608'>gcp-account [use the drop-down window]</span>
 - **Name**: <span style='color:#479608'>gcp-us-west2-spoke1-sub1</span>
 - **Region**: <span style='color:#479608'>us-west2 [use the drop-down window]</span>
 - **CIDR**: <span style='color:#479608'>172.16.22.0/24</span>

Then click on **Save**.

```{note}
Make sure there are no white spaces at the beginning or end of the VPC name.
```

```{figure} images/lab1-gcp2.png
---
align: center
---
VPC template
```

Wait for some minutes and then verify the completion of the VPC creation in the VPC list, as soon as you see the confirmation pop-up message. Hit the Refresh icon if you do not see the CIDR immediately (alternatively, refresh the web page for triggering the change).

You can filter by CIDR `172.16.22.0/24`.

```{figure} images/lab1-gcp3.png
---
height: 250px
align: center
---
Verification
```

```{note}
Expand the **GCP VPC** if you want to see the subnet **"gcp-us-west2-spoke1-sub1"**
```

```{caution}
The VPCs and VNet created in this lab will **NOT** be used in the subsequent labs.
```