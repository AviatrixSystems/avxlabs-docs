# FireNet

```{important}
Estimated time to complete: `45 minutes`
```

In this lab, the goal is to insert a Palo Alto Firewall into the east-west data path at the AWS transit using Aviatrix FireNet. A Palo boostrap configuration has been deployed to S3, but otherwise, this will be done solely with the Aviatrix terraform provider.

The following infrastructure will be deployed terraform:

- A Palo Alto Networks VM-Series Next-Generation Firewall (BYOL)

In addition to the explicit infrastructure deployed via Terraform, the Aviatrix platform will also handle programming routes into the respective CSP route tables such that traffic passing through the AWS transit will be forwarded to the FW before returning and continuing across the Aviatrix network (firewall rules allowing).

The Palo bootstrap configuration is set to allow and log RFC1918, so no traffic will be dropped.

## Initial Connectivity

We're starting with the connected topology we ended with in the MCNA lab.

![MCNA Topology](images/mcna_topology.png)

As such open connectivity between clouds is established.

![MCNA Topology](images/mcna_gatus_cloudx.png)

## Configure and apply

Back to your `terraform.tfvars` file. Update the `enable_firenet` variable from `false` to `true`. Be sure to save the file after making the update.

Then from `LXTerminal` on the jumpbox, issue the following command.

```bash
terraform apply --auto-approve
```

It takes roughly 23 minutes for the firewall to be deployed, initialized, and inserted into the data path.

```{note}
If you're watching your gatus dashboards through the entirety of the process, you'll notice connectivity never stops. The Aviatrix transit waits until the fw is fully initialized and configured before inserting it into the data path.
```

## Expected Results

Once the apply is complete, our topology is now:

![FireNet Topology](images/firenet_topology.png)

And, inter-cloud connectivity is still noted.

![Gatus](images/mcna_gatus_cloudx.png)

Let's take a look the Palo. By default, the management UI is closed to the internet, so we'll have to modify the security group to allow it.

From the pod registration portal, use the `Console` link and credentials to log into AWS.

![Portal](images/firenet_aws.png)

Navigate to `EC2` and select `Security Groups` under `Network & Security` in the left-hand nav.

Search for `management`. Only 1 security group will match.

![Security Group](images/firenet_sg.png)

Click on its `Security Group ID` then `Edit inbound rules` in the lower-right and add a rule that allows type `HTTPS` with a source of `My IP`. If you're using the jumpbox to access the PAN UI, you'll either need to find and use its public IP explicitly, or use  `0.0.0.0/0`.

Then go to CoPilot. In the left-hand nav, select `Security`, then `FireNet`, then the `Firewall` tab at the top. Then, click the `Management UI` to open Palo.

![FireNet](images/firenet_firenet.png)

If you're blocked from accessing the Palo management UI due to corporate security policy, you can use Firefox on the jumpbox for access.

Log in with the credentials provided in the portal registration page. After clicking through several pop-ups, click on the `Monitor` tab in the top nav.

![Palo](images/firenet_palo.png)

Note the traffic is now flowing across the Palo firewall as it traverses from `AWS` to `Azure`.

## Code

Let's take a look at the code behind the apply. Looking at the root `main.tf` you can see that the firenet module was executed because `enable_firenet` was set to `true`.

```terraform
module "firenet" {
  count                             = var.enable_firenet ? 1 : 0
  source                            = "./firenet"
  firenet_transit_gateway_name      = module.mcna[0].transit_aws.transit_gateway.gw_name
  firenet_transit_peer_gateway_name = module.mcna[0].transit_azure.transit_gateway.gw_name
  firenet_spoke_gateway_name        = module.mcna[0].spoke_gw_aws.spoke_gateway.gw_name
  firewall_vpc_id                   = module.mcna[0].firewall.vpc_id
  firewall_instance_id              = module.mcna[0].firewall.instance_id
  firewall_public_ip                = module.mcna[0].firewall.public_ip
  firewall_name                     = module.mcna[0].firewall.firewall_name
  firewall_username                 = local.firewall_username
  firewall_password                 = var.firewall_password
}
```

The `enable_firenet` variable also modified the `mcna` module, setting `firenet = true`. This is were the bulk of the work was done. The code in the local firenet module itself simply:

- Configured vendor integration, specific to Palo, with Aviatrix.
- Marked the two connections to the AWS transit, the spoke attachment and transit peering, for inspection (forward to the firewall).

Use the navigation below when you're ready to move onto the next section.
