# Lab 11 - TERRAFORM & NETWORK INSIGHTS API

## 1. Objective

All the elements you have created through the UI, can also be created through the API or the official **Aviatrix Terraform provider**. 

You can find more information on these here:
https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs

## 2. Validate

We have prepared some Terraform code for you, which you will explore and deploy.

Go to your personal POD Portal, identify the Lab11 section and click on the `Open Workstation` button.

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

* Select the folder `terraform-lab` and click **Open**
  * _When prompted to trust the authors of the files in this folder, select Yes_
* Let’s explore the Terraform files in this directory:
  * Explore the file contents of **main.tf, variables.tf, providers.tf** and **terraform.tfvars**

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

`terraform plan`

* Investigate the proposed changes by Terraform. Now we will apply them to the live environment:

`terraform apply --auto-approve`

Once Terraform is finished, have a look at the newly created **terraform.tfstate** file. This contains information of all infrastructure created through Terraform. This is referred to as **“the state”**. Losing it can cause a lot of trouble, but that is for another (Terraform) lesson.

### Expected Results