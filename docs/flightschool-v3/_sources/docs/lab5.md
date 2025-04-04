
# Lab-5: Infrastructure as Code


Aviatrix is fully capable integrating customer’s CI/CD pipeline with help of terraform. In this small example, we will deploy the Aviatrix gateway that we skipped in the second lab with help of terraform code. 

Go to the Jumpbox and run Visual Studio Code. Click Open folder and select terraform-lab and select open. Here you can analyse the code and see how the Aviatrix is utilizing the modules to deploy. 


```{figure} images-lab5/1.png
---
align: center
---
Terraform Code 
```

Then click on the top right button to open terminal inside the application.

```{figure} images-lab5/2.png
---
align: center
---
Opening Console in Visual Studio
```

Then to deploy the Aviatrix gateway go to the terminal that’s opened below, and init the commands shown below.
-	`terraform init`
-	`terraform plan`
-	`terraform apply`	
-   `yes`

This sequence will install the modules, simulates the deployment, and then will deploy the Aviatrix gateway.
To remove the Aviatrix gateway, issue the command below, and then the gateway will be removed.

-	`terraform destroy`
-	`yes`

This script only deploys the Aviatrix Spoke gateway to Egress-VPC-3, and doesn’t add any security, that’s why you can see the traffic is dropped to all egress domains. We can add the aws-instance-3 to the same SmartGroups as the others, and then the rules will automatically be applied. 

We will edit the existing SmartGroup `SG-Egress-VPC-1-2` with the information as shown below in the table.

| **Field** | **Value**           |
| :-------- | :------------------ |
| **Name**  | SG-Egress-VPC-1-2-3 |
| **tier**  | db                  |

```{figure} images-lab5/3.png
---
align: center
---
SmartGroup Creation for Egress-3 VPC
```


```{figure} images-lab5/4.png
---
align: center
---
Confirming Egress filtering on egress-3 dashboard
```


# End of Lab 5

Ok, that's the end of Lab 5. Great job! 
