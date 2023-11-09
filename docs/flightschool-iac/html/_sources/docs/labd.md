---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Terraform
  language: terraform
  name: terraform
---

# Lab D

## Create on-prem connectivity

## Exercise D.1 – Enable route approval

When we’re going to enable BGP over IPSEC in exercise D.2, we want to prevent to automatically accept and install routes on our cloud network. As such, we are going to enable a feature called route approval on the azure-transit.

Check the [aviatrix_transit_gateway resource documentation](https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs/resources/aviatrix_transit_gateway) for how to enable route approval and make sure it uses gateway mode. Configure that on the Azure transit gateway.

Spoiler code:

---

```{code-cell} terraform
:tags: [hide-cell]
resource "aviatrix_transit_gateway" "azure_transit" {
  ..
  enable_learned_cidrs_approval = true
}
```

---

## Exercise D.2 – Establish VPN tunnel

Now we will establish connectivity to the on-prem datacenter using s2c. Use the [external device connection resource](https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs/resources/aviatrix_transit_external_device_conn) to establish the tunnel using the settings in exercise 2.1 of the original lab guide. We need to gather some details from our Azure transit to do so. Check the [aviatrix_transit_gateway resource documentation](https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs/resources/aviatrix_transit_gateway) for the available reference attributes.

|                    |                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| Argument           | Value                                                                                           |
| Connection name    | azure-to-on-prem                                                                                |
| Local tunnel CIDR  | "169.254.100.2/30,169.254.101.2/30”                                                             |
| Remote tunnel CIDR | "169.254.100.1/30,169.254.101.1/30”                                                             |
| Remote gateway IP  | "<resolve onprem.pod`[pod#]`.aviatrixlab.com>, <resolve onprem-ha.pod`[pod#]`.aviatrixlab.com>” |

Spoiler code (make sure to adjust BGP AS and tunnel CIDRS):

---

```{code-cell} terraform
:tags: [hide-cell]
resource "aviatrix_transit_external_device_conn" "datacenter" {
  vpc_id             = aviatrix_transit_gateway.azure_transit.vpc_id
  connection_name    = "azure-to-on-prem"
  gw_name            = aviatrix_transit_gateway.azure_transit.gw_name
  connection_type    = "bgp"
  bgp_local_as_num   = 65[pod#] # _For Pods 1-9, double pad the pod# with an additional 0 (ie. 65004). For Pods 10-99 single pad (ie. 65010). For Pods > 100, no padding is needed (ie. 65100)_
  bgp_remote_as_num  = 65000
  remote_gateway_ip  = # insert resolved (comma-separated) onprem and onprem-ha domains
  pre_shared_key     = "mapleleafs"
  local_tunnel_cidr  = "169.254.100.2/30,169.254.101.2/30"
  remote_tunnel_cidr = "169.254.100.1/30,169.254.101.1/30"
}
```

---

Check your tunnel is up in the `Site2Cloud` -> `Setup` page. Also validate the BGP Peerings are up on the `Multicloud Transit` -> `BGP` page. This may take a couple of minutes to reflect the actual state.

## Exercise D.3 – Approving learned routes

When you go to `Multicloud Transit` -> `Approval` and select the Azure transit gateway you should see that approval mode is enabled and in gateway mode. You can see we learned a CIDR from our VPN connection and as the next step, we will want to approve that through Terraform.

Have a look at the [aviatrix_transit_gateway resource documentation](https://registry.terraform.io/providers/AviatrixSystems/aviatrix/latest/docs/resources/aviatrix_transit_gateway) for how to approve a list of CIDR’s and add this to the azure transit resource with the learned CIDR.

Spoiler code:

---

```{code-cell} terraform
:tags: [hide-cell]
resource "aviatrix_transit_gateway" "azure_transit" {
  ..
  approved_learned_cidrs        = ["172.16.0.0/16"]
}
```

---

Check again in the UI that the CIDR is now in the approved list.
