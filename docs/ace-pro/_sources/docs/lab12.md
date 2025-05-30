# Lab 12 - IAC & NETWORK INSIGHTS API

## 1. Create VPCs, Transit GW, Spoke GW and Attachment through Terraform

All the elements you have created through the UI, can also be created through the API or the official **Aviatrix Terraform provider**. 

You can find more information on these here:
https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs

## 2. Provision through Terraform

We have prepared some Terraform code for you, which you will explore and deploy.

Go to your personal POD Portal, identify the **_Lab12_** section and click on the `Open Workstation` button.

```{figure} images/lab11-edge.png
---
align: center
---
Lab 12 section on the POD Portal
```

Insert the corresponding credentials, available on the POD Portal, to log in to the remote "Workstation Edge".

```{figure} images/lab11-edge2.png
---
align: center
---
Edge Workstation credentials
```

* Open V**Visual Studio Code** located on the desktop, then click on the `Explorer` button.

```{figure} images/lab11-edge3.png
---
height: 400px
align: center
---
VS Studio
```

* Click on the **Open Folder** button, select the folder `terraform-lab` and click **Open**
  * _When prompted to trust the authors of the files in this folder, select **Yes, I trust the authors**_

```{figure} images/lab11-edge32.png
---
height: 400px
align: center
---
Open Folder
```

```{figure} images/lab11-edge4.png
---
align: center
---
terraform-lab folder
```

```{figure} images/lab11-newedge2.png
---
align: center
---
Click "Open"
```

```{figure} images/lab11-newedge.png
---
align: center
---
Yes, I trust the authors
```

* Let’s explore the Terraform files in this directory:
  * Explore the file contents of **main.tf, variables.tf, providers.tf** and **terraform.tfvars**

```{figure} images/lab11-terraform2.png
---
align: center
---
Manifest
```

> What do you expect will be created when we run this Terraform code?

In this lab, we are using Terraform modules, provided by Aviatrix. These allow you to quickly build out your environment, based on larger building blocks, rather than individual resources. You can find more available modules here:  

https://registry.terraform.io/namespaces/terraform-aviatrix-modules  

Let's run this code.

* Open the **LXTerminal** App on the Desktop
* Move over to the directory where the Terraform files are located:

`cd terraform-lab`

* First thing we need to do is to **initialize** Terraform. This allows for the required providers and modules to be downloaded.  

`terraform init`

```{figure} images/lab11-terraform.png
---
align: center
---
Visual Studio Code
```

* Next we will execute a “plan”. This means that Terraform will compare the live environment with the desired state we declared in our Terraform files.

`terraform plan`

* Investigate the proposed changes by Terraform. Now we will apply them to the live environment:

`terraform apply --auto-approve`

Once Terraform is finished, have a look at the newly created **terraform.tfstate** file. This contains information of all infrastructure created through Terraform. This is referred to as **“the state”**. Losing it can cause a lot of trouble, but that is for another (Terraform) lesson.

### 2.1 Expected Results

By running the above commands, you should see how simple it can be to automate your infrastructure deployments using Terraform.  With a few lines of code and after about **6 minutes**, you should see the new transit and spoke in CoPilot Topology.  

```{figure} images/lab11-terraform-topology.png
---
align: center
---
Topology
```

## 3. Create Transit Peering

### 3.1 Description

In the previous exercise, we deployed a new Transit VPC, Aviatrix Transit Gateway, a Spoke VPC, and an Aviatrix Spoke Gateway.  This new deployment is more or less an island, but let's see how we can use Infrastructure as Code to build a full mesh of the Transits.

### 3.2 Provision through Terraform

* Using the same **Visual Studio Code** session, let's create the `peering.tf` file.
* We will be using the following module:  `https://registry.terraform.io/modules/terraform-aviatrix-modules/mc-transit-peering/aviatrix/latest`
* Go back to the **Visual Studio Code** session and create a new file. Name it `peering.tf`.

```{figure} images/lab11-newfile.png
---
align: center
---
New File
```

```{figure} images/lab11-peering.png
---
align: center
---
peering.tf
```

* Now copy the following statements and paste them inside the file previously created:

```terraform
module "transit-peering" {
  source  = "terraform-aviatrix-modules/mc-transit-peering/aviatrix"
  version = "1.0.9"

  transit_gateways = [
    "aws-us-west-2-transit",
    "aws-us-east-2-transit"
  ]
} 
```

```{note}
*Copy and Paste* does not work directly from the host machine towards the Workstation "Edge", therefore activate the **Hidden Menu**, that is a sidebar that is maintained hidden until explicitly enabled. On a desktop or other device which has a hardware keyboard, you can show this menu by pressing **Ctrl+Alt+Shift** on Windows machine (**Control+Shift+Command** on Mac).
```

```{figure} images/lab11-clip1.png
---
align: center
---
Hidden Clipboard
```

```{figure} images/lab11-clip2.png
---
align: center
---
Copy the statemets from the Lab Guides and paste them in to the hidden clipboard
```

```{figure} images/lab11-clip3.png
---
align: center
---
Copy the statements from the hidden clipboard and close the clipboard
```

```{figure} images/lab11-clip33.png
---
align: center
---
Paste the commands
```

```{figure} images/lab11-clip4.png
---
align: center
---
Save the file!
```

* After creating the peering.tf manifest, be sure to **SAVE** the file in Visual Studio Code.
* Go back to the **LXTerminal** and run `terraform init` again to download the `mc-transit-peering` module
  
```{figure} images/lab11-clip5.png
---
height: 400px
align: center
---
Once again "terraform init"
```

* Run the command `terraform plan` to assess the changes

```{figure} images/lab11-clip6.png
---
height: 400px
align: center
---
Once again "terraform plan"
```

* Run the command `terraform apply --auto-approve`

```{figure} images/lab11-clip7.png
---
height: 400px
align: center
---
Once again "terraform apply"
```

### 3.3 Expected Results

After a few minutes, a new peering will be established between the **aws-us-east-2-transit** GW and the a**aws-us-west-2-transit** GW. You can then open CoPilot to visualize the updated topology.

```{figure} images/lab11-topoloy-transit-peerings.png
---
align: center
---
New Peering
```

`Congratulations! You have successfully deployed the complete Aviatrix solution.`


```{figure} images/lab11-lastdrawing.png
---
height: 400px
align: center
---
Full-Blown Aviatrix Solution
```

## 4. IAC Summary

* You deployed an Aviatrix Transit and Spoke using Terraform
* You added the new Transit to the Global Multicloud Transit Network with a few lines of code
* Infrastructure as Code and Terraform are a perfect complement to the Aviatrix solution
* In minutes, you can create the network, security and connectivity needed

## 5. Network Insights API

### 5.1 Description

The `Aviatrix Network Insights API` simplifies the process of navigating network interface statistics and micro-gateway status data. By integrating this API with your visualization platforms (with vendors you already know and love!), you can easily make data-driven decisions.

### 5.2 Validate

* Go to your personal POD Portal, identify the **_Lab12_** section and click on the `Open Grafana` button.

```{figure} images/lab11-edge10.png
---
align: center
---
Lab 12 section on the POD Portal
```

```{important}
The API Key was generated at the launch of all PODs and applied to the Prometheus/Grafana server!
```

Enter the required credentials that are available on the POD Portal and then click on **Log in**.

```{figure} images/lab11-edge20.png
---
align: center
---
Grafana Login Page
```

You will immediately notice the **Receive Rate** and **Transmit Rate** stats!

```{figure} images/lab11-edge30.png
---
align: center
---
Receive Rate and Transmit Rate
```

```{caution}
Navigate to **CoPilot > Settings > Configuration** and locate the "`"Network Insights API"` widget.

The **API Key** generated within CoPilot can be found there and should be used in Prometheus/Grafana.
```

```{figure} images/lab11-networkinsight1.png
---
height: 400px
align: center
---
Network Insight API section
```

```{tip}
Download the API Key and explore the content of the file!
```

```{figure} images/lab11-networkinsight2.png
---
align: center
---
API key 
```
