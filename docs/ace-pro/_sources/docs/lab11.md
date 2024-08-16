# Lab 11 - IAC & NETWORK INSIGHTS API

## 1. Create VPCs, Transit GW, Spoke GW and Attachment through Terraform

All the elements you have created through the UI, can also be created through the API or the official **Aviatrix Terraform provider**. 

You can find more information on these here:
https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs

## 2. Validate

We have prepared some Terraform code for you, which you will explore and deploy.

Go to your personal POD Portal, identify the **_Lab11_** section and click on the `Open Workstation` button.

```{figure} images/lab11-edge.png
---
align: center
---
Lab 11 section on the POD Portal
```

Insert the corresponding credentials, available on the POD Portal, to log in to the remote "edge" Workstation.

```{figure} images/lab11-edge2.png
---
align: center
---
Edge Workstation credentials
```

* Open the **Visual Studio Code** located on the Desktop

```{figure} images/lab11-edge3.png
---
align: center
---
VS Studio
```

* Click on the **Home** icon and then select the folder `terraform-lab` and click **Open**
  * _When prompted to trust the authors of the files in this folder, select Yes_

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

* First thing we need to do is to initialize Terraform. This allows for the required providers and modules to be downloaded.  

`terraform init`

* Next we will execute a “plan”. This means that Terraform will compare the live environment with the desired state we declared in our Terraform files

```{figure} images/lab11-terraform.png
---
align: center
---
Visual Studio Code
```

`terraform plan`

* Investigate the proposed changes by Terraform. Now we will apply them to the live environment:

`terraform apply --auto-approve`

Once Terraform is finished, have a look at the newly created **terraform.tfstate** file. This contains information of all infrastructure created through Terraform. This is referred to as **“the state”**. Losing it can cause a lot of trouble, but that is for another (Terraform) lesson.

### Expected Results

By running the above commands, you should see how simple it can be to automate your infrastructure deployments using Terraform.  With a few lines of code and after about **6 minutes**, you should see the new transit and spoke in CoPilot Topology.  

```{figure} images/lab11-terraform-topology.png
---
align: center
---
Topology
```

## Lab 11.2 - Create Transit Peering

### Description

In the previous exercise, we deployed a new Transit VPC, Aviatrix Transit Gateway, a Spoke VPC, and an Aviatrix Spoke Gateway.  This new deployment is more or less an island, but let's see how we can use Infrastructure as Code to build a full mesh of the Transits.

### Validate

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

> Make sure that the Transit Gateway names match to your environment

* **SAVE** the file in Visual Studio Code
* Go back to the **LXTerminal** and run `terraform init` again to download the `mc-transit-peering` module
* Run the command `terraform plan` to assess the changes
* Run the command `terraform apply --auto-approve`

### Expected Results

After a few minutes, a full mesh between all Transit gateways should be created. You can go to CoPilot Topology and have a look at the full mesh Multicloud network that was created.

```{figure} images/lab6-topoloy-transit-peerings.png
---
align: center
---
Full Mesh
```

## Lab 11.2 Summary

* You deployed an Aviatrix Transit and Spoke using Terraform
* You added the new Transit to the Global Multicloud Transit Network with a few lines of code
* Infrastructure as Code and Terraform are a perfect complement to the Aviatrix solution
* In minutes, you can create the network, security and connectivity needed

## Lab 11.3 - Network Insights API

### Description

The `Aviatrix Network Insights API` simplifies the process of navigating network interface statistics and micro-gateway status data. By integrating this API with your visualization platforms (with vendors you already know and love!), you can easily make data-driven decisions.

### Validate

* Go to your personal POD Portal, identify the **_Lab11_** section and click on the `Open Grafana` button.

```{figure} images/lab11-edge10.png
---
align: center
---
Lab 11 section on the POD Portal
```

Enter the required credentials that are available on the POD Portal and then click on **Log in**.

```{figure} images/lab11-edge20.png
---
align: center
---
Lab 11 section on the POD Portal
```

You will immediately notice the Receive Rate and Transmit Rate stats!

```{figure} images/lab11-edge30.png
---
align: center
---
Receive Rate and Transmit Rate
```

```{caution}
Go to **CoPilot > Settings > Configuration** and then identify the `"Network Insights API"` widget.

You will find the API Key generated on the CoPilot and then used on Prometheus/Grafana!
```
