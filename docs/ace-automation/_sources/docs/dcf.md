# Distributed Cloud Firewall (DCF)

```{important}
Estimated time to complete: `30 minutes`
```

In this lab, the goal is to leverage Aviatrix's Distributed Cloud Firewall (DCF) to further secure cloud connectivity. We will do this by:

- Enabling DCF
- Applying a set of rules that act broadly on defined CIDR ranges
- Applying a set of rules that act specifically on compute with designated tags
- Enabling `Security Group (SG) Orchestration` to implement microsegmentation within the VPC/VNet

There is no infrastructure to deploy as this is all Aviatrix configuration. For SG orchestration, the controller will be orchestrating NSG and SGs within the respective cloud providers out-of-band from the terraform execution.

## Initial Connectivity

We're starting with the connected topology we ended with in the Segmentation lab.

![MCNA Topology](images/segmentation_topology.png)

As such open connectivity between clouds is established.

![Gatus](images/mcna_gatus_cloudx.png)

All throughout these labs, intra-VPC/Vnet connectivity has been uninhibited since that traffic never crosses an Aviatrix gateway.

![Gatus](images/dcf_intra_vpc.png)

## Configure and apply DCF rules

Back to your `terraform.tfvars` file. Update the `enable_dcf` variable from `false` to `true`. Be sure to save the file after making the update.

Then from `LXTerminal` on the jumpbox, issue the following command.

```bash
terraform apply --auto-approve
```

It should only take a minute to enable and apply the dcf rules.

## DCF Results

Let's take a look at the DCF rules that were applied. Open CoPilot and navigate to `Security`, then `Distributed Cloud Firewall` in the left hand nav.

![DCF](images/dcf_rules.png)

Notice there are 5 rules, only two of which are enforced (the green check (enforced) vs the grey slash (unenforced)).

Let's focus on the 2 enforced rules `allow-https` and `default-deny-all`.

The `allow-https` rule is allowing tcp port 443 between the `aws-spoke` and `azure-spoke` groups. Let's take a look at what that means.

In CoPilot, navigate to `Groups` in the left-hand nav and click on `aws-spoke`. Notice that the group is defined by a set of CIDRs. Now click on `azure-spoke` and confirm the same.

![DCF](images/dcf_groups.png)

We can conclude that all traffic throughout the Aviatrix network will be denied, except port 443 between the set of CIDRs defined by the `aws-spoke` and `azure-spoke` groups. Let's confirm with gatus.

![DCF](images/dcf_443.png)

## Configure and enable a tag-based rule

Back to your `terraform.tfvars` file. Update the `enable_dcf_tag_rule` variable from `false` to `true`. Be sure to save the file after making the update.

Then from `LXTerminal` on the jumpbox, issue the following command.

```bash
terraform apply --auto-approve
```

It should only take a minute to enable the rule.

## Expected Results

Back to CoPilot, navigate back to your DCF rules. Notice that rule 200, allowing port `1433` to connect between the `inter-cloud` group has now been enforced.

![DCF tag](images/dcf_tag_enable.png)

Let's take a look at what is defined by that group by navigating back to `Groups` and clicking on `inter-cloud`

![DCF tag](images/dcf_1433.png)

Here you'll see that this group applies to any instance with the `Allowxcloud` or `allowxcloud` tag set to `true`. Further you'll see that the `aws-1` and `azure-1` instances both meet that criteria. You can conclude that those two instances should now be able to communicated on port `1433`.

Let's confirm with gatus.

![Gatus](images/dcf_gatus_1433.png)

## Configure and apply microsegmentation

Back to your `terraform.tfvars` file. Update the `enable_sg_orchestration` variable from `false` to `true`. Be sure to save the file after making the update.

Then from `LXTerminal` on the jumpbox, issue the following command.

```bash
terraform apply --auto-approve
```

It should only take a minute to enable the sg orchestration feature.

## Microsegmentation Results

With SG Orchestration enabled, the dcf rules that were previously applied only to Aviatrix gateways are now implemented to control traffic inside the VPC/Vnets by, as the name implies, orchestrating NSGs, firewall rules, and security groups in the CSPs themselves.

In CoPilot, confirm that all dcf rules are now enabled.

![DCF](images/dcf_rules_applied.png)

Also, confirm that SG orchestration is enabled on the `AWS` and `Azure` VPC/VNet by navigating to `Security`, `Distributed Cloud Firewall` and then click `Settings` in the top nav, and then click `Manage` in the Security Group (SG) Orchestration section.

![DCF](images/dcf_sg_orch.png)

In the resulting view, you're looking to confirm the `Orchestration Status` for both `aws-spoke` and `azure-spoke` is `Completed`.

![DCF](images/dcf_sg_status.png)

If it's blank or In Progress, you can refresh with the icon in the bottom right after waiting a minute or so.

When completed, let's check the effect on gatus.

![Gatus](images/dcf_intra.png)

The rules that were previously applied to the Aviatrix gateways are now securing instances within the VPC/VNets also!

In the aws console, you can see the orchestrated security groups by filtering on `AVX-SEC`

![Security Groups](images/dcf_aws.png)

## Code

Let's take a look at the code behind the applies. Looking at the root `main.tf` you can see that the `enable_dcf` variable enables execution of the local `dcf` module. The `enable_dcf_tag_rule` and `enable_sg_orchestration` variables are passed to enable rules and orchestration.

```terraform
module "dcf" {
  count                   = var.enable_dcf ? 1 : 0
  source                  = "./dcf"
  azure_spoke_vpc_id      = module.mcna[0].spoke_vpc_azure.vpc_id
  aws_spoke_vpc_id        = module.mcna[0].spoke_vpc_aws.vpc_id
  aws_region              = local.backbone.aws.transit_region_name
  azure_region            = local.backbone.azure.transit_region_name
  aws_account             = local.backbone.aws.transit_account
  azure_account           = local.backbone.azure.transit_account
  enable_sg_orchestration = var.enable_sg_orchestration
  enable_dcf_tag_rule     = var.enable_dcf_tag_rule
}
```

Inside the local `dcf` module, you'll see the dcf rules definition, groups definition sg orchestration enablement, and rules (policy) enforcement.

That completes the dcf lab. Use the navigation below when you're ready to move onto the next section.
