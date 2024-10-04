# Customized Spoke VPC Routing

```{important}
Estimated time to complete: `15 minutes`
```

In this lab, the goal is to demonstrate custom routing to a spoke for a cidr not configured in the vpc.

The instance, `aws-1` has a loopback interface defined on `172.16.0.1/32`.

A single route in the `aws-spoke` vpc has been pre-deployed, routing `172.16.0.1/32` to the `aws-1` instance. There is no other infrastructure to deploy as the rest of the configuration is within Aviatrix.

## Initial Connectivity

View the topology, now with the addition of the `172.16.0.1/32` cidr defined on the `aws-1` instance.

![Routing Topology](images/routing_topology.png)

The Azure gatus dashboard has a section called `Spoke routing - custom`. Unsurprisingly, there is no connectivity from `azure-1` to `aws-1` for that cidr. The `aws-spoke` Aviatrix gateway is only advertising the routes for the cidr defined by its vpc - the default behavior.

![Gatus](images/routing_gatus.png)

The solution is to implement a custom configuration on the `aws-spoke` gateway to advertise that cidr to the rest of the Aviatrix network.

## Configure and apply

Back to your `terraform.tfvars` file. Update the `apply_custom_spoke_routing` variable from `false` to `true`. Be sure to save the file after making the update.

Then from `LXTerminal` on the jumpbox, issue the following command.

```bash
terraform apply --auto-approve
```

It should only take a minute to apply the custom routing.

## Expected Results

The custom cidr is now advertised and the Aviatrix network now knows where to send traffic destined for `172.16.0.1/32`.

Gatus confirms.

![Gatus](images/routing_configured.png)

## Code

Let's take a look at the code behind the apply. Looking at the root `main.tf` you can see that the `apply_custom_spoke_routing` variable is passed to the `mcna` module.

```terraform
module "mcna" {
  count                      = var.deploy_mcna ? 1 : 0
  source                     = "./mcna"
  backbone                   = local.backbone
  apply_custom_spoke_routing = var.apply_custom_spoke_routing
}

```

Inside the local `mcna` module, you'll see the aws spoke affected by the that variables boolean state:

```terraform
module "spoke_aws" {
  source  = "terraform-aviatrix-modules/mc-spoke/aviatrix"
  version = "1.6.3"
  ...
  included_advertised_spoke_routes = var.apply_custom_spoke_routing ?"172.16.0.1/32, 10.1.2.0/24" : null
  ...
}
```

That completes the custom routing lab. Use the navigation below when you're ready to move onto the next section.
