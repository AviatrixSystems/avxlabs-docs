# Lab 1 - VPCs/VNets Creation

## 1. Introduction

In this lab, you will create **3** VPCs/VNets, one in each cloud, i.e., Azure, AWS and GCP. 

The purpose of creating these VPCs/VNets is to familiarise yourself with the user interface (UI).

Refer to your POD assignment for CoPilot login information, as depicted below. Click on the `URL` of the CoPilot and log in using the credentials assigned to your POD.

```{important}
Always refer to your personal POD portal for both passwords and additional information. Please bear in mind that the screenshots refer to a different POD and they are shown just as examples!
```

```{figure} images/lab1-portal.png
---
align: center
---
POD portal
```

## 2. Azure VNet
### 2.1. Create Azure VNet

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Verify whether the CIDR range `192.168.12.0/24` is overlapping or not, as shown below.

```{figure} images/lab1-vnet1.png
---
align: center
---
Searching for a subnet conflict
```

Let’s create an Application/Spoke VNet. Click on the button `“+ VPC/VNET”`.

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
align: center
---
Notification
```

```{note}
VPC Functions:

**1) Default** = Spoke VPC

**2) Transit + FireNet** 
```

### 2.2. Verify from Azure Portal

Log into the <a href="https://portal.azure.com/#home" target="_blank">Azure portal</a>. Refer to your pod info for login information.

```{figure} images/lab1-newpic.png
---
align: center
---
Azure credentials
```

```{important}
If you are already signed in with another account, kindly sign out!
```

```{figure} images/lab1-signin.png
---
align: center
---
Signing in
```

Once you are signed in, navigate to `Virtual Networks`, from the navigation panel on the left-hand side.

```{figure} images/lab1-virtualnetwork.png
---
align: center
---
Azure Console
```

Click on the `"Subscription equals..."` field, then click on the `"All"` button and then click on **Apply**.

```{figure} images/lab1-subscriptionequal.png
---
align: center
---
Subscription equals...
```

```{figure} images/lab1-newpic2.png
---
align: center
---
All Subscriptions
```

Now use the Filter field beside the "Subscription equals" field and be sure to filter your `Subscription`, <ins>by your pod number</ins>, for example for Pod 140 filter by `csp_azure_shared_ace_labs_140`.

```{figure} images/lab1-virtualnetwork3.png
---
align: center
---
VNets
```

Once you get visibility of the Subscription associated to your POD number, you will be able to find the VNets you just created. 

Explore for instance, the **_azure-us-central-spoke1_** VNet and from there navigate to `Subnets`. Explore also all the constructs that were created in Central US.

```{figure} images/lab1-virtualnetwork4.png
---
align: center
---
Azure subnets
```

From the Portal, navigate to **Home > Route tables**. Be sure once again to filter your `Subscription`, <ins>by your pod number</ins>, for example for Pod 140 filter by `csp_azure_shared_ace_labs_140`.

```{figure} images/lab1-newpic3.png
---
align: center
---
Home
```

Explore the route tables.

```{figure} images/lab1-virtualnetwork5.png
---
align: center
---
Route tables
```

## 3. AWS VPC

### 3.1. Create AWS VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Verify once again whether the CIDR range `10.0.22.0/24` is overlapping or not, as shown below.

```{figure} images/lab1-newpic5.png
---
align: center
---
Searching for a subnet conflict
```

This time let’s create an Application/Spoke VPC. Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-vpc1.png
---
align: center
---
VPC creation
```

Insert the following values:

 - **Name**: <span style='color:#479608'>aws-us-west2-spoke1</span>
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
align: center
---
Hourglass icon
```

Verify the VPC creation in the VPC list. Hit the Refresh icon if you do not see the CIDR immediately (alternatively, refresh the web page for triggering the change).

You can filter by CIDR `10.0.22.0/24`.

```{figure} images/lab1-vpc4.png
---
align: center
---
Verification
```

### 3.2. Verify from AWS Console

Log in to the <a href="https://aws.amazon.com/console/" target="_blank">AWS console</a>. Refer to your pod info for login information (this screenshot is for **Pod 150**).

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

Change the region to `Oregon (us-west-2)` in the top-right corner and invoke the VPC service. You can see the `VPC` created with the CIDR block.

```{figure} images/lab1-newpic6.png
---
align: center
---
Oregon region
```

From there, navigate to `Subnets`.

As you can see, the Controller has created 1 Public Subnet and 1 Private Subnet per Availability Zone. Since us-west-2 has 4 AZs, therefore **eight** subnets are created.

```{figure} images/lab1-newpic7.png
---
align: center
---
Subnets created by the Aviatrix Controller
```

From there, navigate to `Route Tables`.

Here, also, you can see the eight route tables that the Controller created. They are mapped to each subnet. The Public Subnets have a 0/0 route pointing to the Internet Gateway, which the Controller also deployed.

## 4. GCP VPC

### 4.1. Create GCP VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Before starting the deployment of the VPC in GCP, verify once again whether the CIDR range `172.16.22.0/24` is overlapping or not, as shown below.

```{figure} images/lab1-newpic8.png
---
align: center
---
Verification
```

This time let’s create an Application/Spoke VPC. Click on the button `“+ VPC/VNET”`.

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

Verify the VPC creation in the VPC list. Hit the Refresh icon if you do not see the CIDR immediately (alternatively, refresh the web page for triggering the change).

You can filter by CIDR `172.16.22.0/24`.

```{figure} images/lab1-gcp3.png
---
align: center
---
Verification
```

```{tip}
Expand the GCP VPC if you want to see the subnet _"gcp-us-west2-spoke1-sub1"_
```

```{caution}
Kindly note that GCP console access is not provided!
```{figure} images/lab1-newpic9.png
---
align: center
---
Only AWS and Azure
```

```{note}
The VPCs and VNet created in this lab will not be used in subsequent labs.
```