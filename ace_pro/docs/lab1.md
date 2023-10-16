# Lab 1 - VPCs/VNets Creation

## 1. Introduction

In this lab, you will create **3** VPCs/VNets, one in each cloud, i.e., Azure, AWS and GCP. 

The purpose of creating these VPCs/VNets is to familiarise yourself with the user interface (UI).

Refer to your POD assignment for CoPilot login information, as depicted below. Click on the `URL` of the CoPilot and log in using the credentials assigned to your POD.

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
height: 400px
align: center
---
VNet creation
```

Insert the following values:

 - **Name**: <span style='color:#33ECFF'>azure-us-central-spoke1</span>
 - **Cloud**: <span style='color:#33ECFF'>Azure Global</span>
 - **Account**: <span style='color:#33ECFF'>azure-account [use the drop-down window]</span>
 - **Region**: <span style='color:#33ECFF'>Central US [use the drop-down window]</span>
 - **VPC CIDR**: <span style='color:#33ECFF'>192.168.12.0/24</span>
 - **VPC Function**: <span style='color:#33ECFF'>Default [use the drop-down window]</span>

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
height: 400px
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

### 2.2. Verify from Azure Portal

Log into the <a href="https://portal.azure.com/#home" target="_blank">Azure portal</a>. Refer to your pod info for login information.

Once you are logged in, navigate to `Virtual Networks`.

Be sure to filter your `Subscription` <ins>by your pod number</ins>, for example, for Pod 22 filter by `csp_azure_shared_ace_labs_22`.

```{figure} images/lab1-vnet6.png
---
align: center
---
Azure portal
```

Once in the VNet you just created, navigate to `Subnets`. Explore all the constructs that were created in Central US.

```{figure} images/lab1-vnet7.png
---
align: center
---
Azure subnets
```

From the Portal, navigate to **Home > Route tables**. Explore the route tables.

```{figure} images/lab1-vnet8.png
---
align: center
---
Route tables
```

## 3. AWS VPC

### 3.1. Create AWS VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets** and then click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-vpc1.png
---
height: 400px
align: center
---
VPC creation
```

Insert the following values:

 - **Name**: <span style='color:#33ECFF'>aws-us-west2-spoke1</span>
 - **Cloud**: <span style='color:#33ECFF'>AWS</span>
 - **Account**: <span style='color:#33ECFF'>aws-account [use the drop-down window]</span>
 - **Region**: <span style='color:#33ECFF'>us-west-2 (Oregon) [use the drop-down window]</span>
 - **VPC CIDR**: <span style='color:#33ECFF'>10.0.22.0/24</span>
 - **VPC Function**: <span style='color:#33ECFF'>Default [use the drop-down window]</span>

Then click on **Save**.

```{figure} images/lab1-vpc2.png
---
height: 400px
align: center
---
VPC configuration
```

Monitor the progress of the VPC creation through the hourglass icon.

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

Log in to the <a href="https://aws.amazon.com/console/" target="_blank">AWS console</a>. Refer to your pod info for login information (this screenshot is for **Pod 100**):

```{figure} images/lab1-vpc5.png
---
align: center
---
AWS console
```

Change the region to `Oregon (us-west-2)` in the top-right corner and invoke the VPC service. You can see the `VPC` created with the CIDR block.

From there, navigate to `Subnets`.

As you can see, the Controller has created 1 Public Subnet and 1 Private Subnet per Availability Zone. Since us-west-2 has 4 AZs, therefore eight subnets are created.

From there, navigate to `Route Tables`.

Here, also, you can see the eight route tables that the Controller created. They are mapped to each subnet. The Public Subnets have a 0/0 route pointing to the Internet Gateway, which the Controller also deployed.

## 4. GCP VPC

### 4.1. Create GCP VPC

Go to **CoPilot > Cloud Resources > Cloud Assets >** Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-gcp1.png
---
align: center
---
VPC creation
```

Insert the following values:

 - **Name**: <span style='color:#33ECFF'>gcp-us-west2-spoke1</span>
 - **Cloud**: <span style='color:#33ECFF'>GCP</span>
 - **Account**: <span style='color:#33ECFF'>gcp-account [use the drop-down window]</span>
 - **Name**: <span style='color:#33ECFF'>gcp-us-west2-spoke1-sub1</span>
 - **Region**: <span style='color:#33ECFF'>us-west2 [use the drop-down window]</span>
 - **CIDRn**: <span style='color:#33ECFF'>172.16.22.0/24</span>

Then click on **Save**.

```{note}
Make sure there are no white spaces at the beginning or end of the VPC name.
```

```{figure} images/lab1-gcp2.png
---
height: 400px
align: center
---
VPC template
```

Verify the VPC creation in the VPC list. Hit the Refresh icon if you do not see the CIDR immediately(alternatively, refresh the web page for triggering the change).

You can filter by CIDR `172.16.22.0/24`.

```{figure} images/lab1-gcp3.png
---
align: center
---
Verification
```

```{note}
The VPCs and VNet created in this lab will not be used in subsequent labs.
```