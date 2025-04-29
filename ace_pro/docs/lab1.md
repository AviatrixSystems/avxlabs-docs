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
You will only have access to the **AWS** Console. Access to both the _Azure_ Console and the _GCP_ Console is not included.
```

## 2. Azure VNet
Let's begin our first task: creating a **VNet**.

### 2.1 Create Azure VNet

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Verify if the CIDR range `192.168.12.0/24` overlaps with any currently used **CIDR ranges**, as illustrated below.

```{figure} images/lab1-vnet1.png
---
height: 300px
align: center
---
Searching for a subnet conflict
```

- Great! Now that you have confirmed that the CIDR range 192.168.12.0/24 does not conflict with any existing CIDR ranges, please proceed to create an **Application/Spoke** Virtual Network (VNet) using the CoPilot

Click on the button `“+ VPC/VNET”`.

```{figure} images/lab1-vnet2.png
---
align: center
---
VNet creation
```

Please enter the following values:

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

You will see a message appear in the top-right corner right away!

```{figure} images/lab1-vnet4.png
---
align: center
---
"In progress" message
```

From the CoPilot search bar, type `“task”` and then click the search result `“Notifications / Tasks”`. Keep an eye on the VNet creation task and wait for it to complete.

```{figure} images/lab1-vnet5.png
---
height: 200px
align: center
---
Notification
```

```{note}
VPC Functions:

**1) Default** = Spoke VPC

**2) Transit + FireNet** 
```

The creation of the VNet will take approximately **4 to 5 minutes**. You can periodically check the status of the task by clicking the hourglass icon in the top right corner. Be sure to click the refresh button as needed. Once the task turns green, you can confidently conclude that the VNet has been successfully created.

```{important}
**Clear the search field to restore the navigation panel view.**
```{figure} images/lab1-cleanup.png
---
height: 350px
align: center
---
Clean up the Search Field
```

```{figure} images/lab1-new.png
---
height: 400px
align: center
---
Hourglass icon
```

Navigate to **Cloud Resources > Cloud Assets > VPC/VNets & Subnets**. In the search field, enter the name of the VNet you created earlier, **`azure-us-central-spoke1`**. Then, expand the VNet to view the additional subnets that were created by the **Aviatrix Controller**.

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
It will take an additional **10 minutes** for both the VNet and all the four subnets (two public and two private) to become visible.

Thank you for your patience!

<ins>Please continue with the next tasks, and once they are completed, confirm that all underlay constructs have been successfully deployed</ins>.
```

## 3. AWS VPC
It's now time to create a VPC in AWS.

### 3.1  Create AWS VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Please verify once more whether the CIDR range `10.0.22.0/24` overlaps with any existing CIDR ranges, as illustrated below.

```{figure} images/lab1-newpic5.png
---
height: 400px
align: center
---
Searching for a subnet conflict
```

This time, let's create an **Application/Spoke** VPC. Click the `“+ VPC/VNET”` button to get started.

```{figure} images/lab1-vpc1.png
---
align: center
---
VPC creation
```

Please enter the following values:

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

Keep an eye on the progress of the VPC creation by using the **hourglass** icon.

```{figure} images/lab1-vpc3.png
---
height: 200px
align: center
---
Hourglass icon
```

Check the VPC list to verify the creation of your VPC. If the CIDR does not appear right away, click the **Refresh** icon or refresh the web page to trigger the update.

Please allow a few minutes (approximately **2-3** minutes) for the changes to reflect in the "VPC/VNets & Subnets" section.

You can also filter by the CIDR `10.0.22.0/24` for easier access.

```{figure} images/lab1-vpc4.png
---
height: 250px
align: center
---
Verification
```

### 3.2 Verify from AWS Console

Log in to the **AWS console** using the credentials provided in your pod information. (**_Note_**: This screenshot is for **Pod 2**.)

```{figure} images/lab1-newaws.png
---
align: center
---
AWS console credentials
```

```{figure} images/lab1-vpc5.png
---
align: center
---
AWS console
```

Change the region to `Oregon (us-west-2)` in the top-right corner, then access the **VPC** service.

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

Now, click on the **VPCs** link shown below:

```{figure} images/lab1-newpic611.png
---
align: center
---
VPCs
```

You should see the `VPC` that was created, along with its assigned CIDR block.

```{figure} images/lab1-new3.png
---
height: 150px
align: center
---
VPC
```

In the VPC section, navigate to `Subnets` in the left-hand navigation panel.

As you can see, the Controller has created one Public Subnet and one Private Subnet for each Availability Zone. Since the us-west-2 region has four Availability Zones, a total of **eight** subnets have been created.

```{figure} images/lab1-newpic7.png
---
align: center
---
Subnets created by the Aviatrix Controller
```

Navigate to `Route Tables`.

Here, you will see the **eight** route tables created by the Aviatrix Controller, each mapped to a corresponding subnet. The Public Route Tables include a 0.0.0.0/0 route that points to the Internet Gateway, which has also been deployed by the Controller.

```{figure} images/lab1-rt.png
---
align: center
---
Routing Tables created by the Aviatrix Controller
```

Now, navigate to the `Internet gateways` section.

Here, you will find the Internet Gateway created by the Aviatrix Controller. You’ll notice that the **IGW** shares the same name as the VPC you defined and is attached to it as well.

```{figure} images/lab1-igw.png
---
align: center
---
IGW created by the Aviatrix Controller
```

## 4. GCP VPC
Next, we will complete this lab by creating a VPC in GCP.

### 4.1 Create GCP VPC

Go to **CoPilot > Cloud Resources > Cloud Assets > VPC/VNets & Subnets**.

Before proceeding with the deployment of the VPC in GCP, please verify once more whether the CIDR range `172.16.22.0/24` overlaps with any existing CIDR ranges, as illustrated below.

```{figure} images/lab1-newpic8.png
---
height: 200px
align: center
---
Verification
```

Create once again an **Application/Spoke** VPC. Click on the button `“+ VPC/VNET”`.

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
height: 200px
align: center
---
Verification
```

```{note}
Expand the **GCP VPC** if you want to see the subnet **"gcp-us-west2-spoke1-sub1"**
```

```{caution}
The VPCs and VNets created in this lab will **NOT** be utilized in the subsequent labs.
```