# Backbone Lab

## Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates solely within the **Azure** cloud environment. The company decided expaning their cloud infrastructure also within AWS, however, they are facing cloud skills gap, because they do not have experties in AWS yet. Moreover, the company is heavily using the native CSP transit solutions (e.g. Azure Route Server).

## Lab Objective

You, the newly appointed solutions architect, have been assigned the following task:

- Create a **_multicloud backbone architecture_** that works in every cloud.
- The solution should also provide **_hybrid connectivity_** towards the Data Center.

```{important}
**Azure Route Server** does not natively sypport connectivity to other Cloud Providers.

Aviatrix supports `Multicloud Transit`.
```

## Initial Set-up

1- You have already deployed both the `Aviatrix Controller` and the `Aviatrix CoPilot`, inside AWS.

2- You have also deployed a pair of `Transit Gateways` in Azure and attached them to the existing Azure Route Server.

3- You have also deployed an `Aviatrix Secure Edge` inside the on-prem Data Center.

```{figure} images/backbone-initial-topology.png
---
height: 400px
align: center
---
Initial Topology
```

## LAB Access Details

| **POD#** | **Copilot** |
|:----------:|:---------------:|
|      1     |       <a href="https://cplt.pod1.aviatrixlab.com" target="_blank">POD1</a>     |
|      2     |       <a href="https://cplt.pod2.aviatrixlab.com" target="_blank">POD2</a>      |
|      3     |       <a href="https://cplt.pod3.aviatrixlab.com" target="_blank">POD3</a>      |
|      4     |       <a href="https://cplt.pod4.aviatrixlab.com" target="_blank">POD4</a>      |
|      5     |       <a href="https://cplt.pod5.aviatrixlab.com" target="_blank">POD5</a>     |
|      6     |       <a href="https://cplt.pod6.aviatrixlab.com" target="_blank">POD6</a>      |
|      7     |       <a href="https://cplt.pod7.aviatrixlab.com" target="_blank">POD7</a>      |
|      8     |       <a href="https://cplt.pod8.aviatrixlab.com" target="_blank">POD8</a>      |
|      9     |       <a href="https://cplt.pod9.aviatrixlab.com" target="_blank">POD9</a>     |
|      10     |       <a href="https://cplt.pod10.aviatrixlab.com" target="_blank">POD10</a>      |

## Access credentials

Username:

```bash
student
```

Password:

```bash
1012fw633#SYTY3
```

## LAB Pre-Req

Before starting the lab, change the following timers to their lowest value

![Task Server](images/egress_timers.png)

```{warning}
DO NOT ATTEMPT TO CHANGE THESE TIMES IN PRE-PROD or PROD SETUP. THIS WOULD CAUSE SERIOUS ISSUES.
```

It should look like the following:

![Task Server](images/egress_timers_updated.png)

Do not change these times in the production setup

- Aviatrix Spoke GW must be deployed in Region us-east-1 in the “egress-vpc”
- GW Instance Size should be t3a.small

## Successful Completion of LAB

After completing the lab, your screen should look more or less like the following. The IP addresses and UUIDs could be different.

![Topology](images/egress_topology.png)

![Gateway](images/egress_spoke.png)

![Monitor](images/egress_monitor.png)

![Filter](images/egress_monitor_filter.png)

![Routes](images/egress_vpc_routes.png)

## Lab Hints

### Create Secure Egress DCF Rules

- Create three rules
- The last DCF rule is a zero-trust rule
- Rule 100 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-https` web group
- Rule 0 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-http` web group

![DCF](images/egress_dcf_rules.png)

### Create rfc1918 SmartGroup

![Group](images/egress_groups.png)

![rfc1918](images/egress_rfc1918.png)

### Create WebGroup to Define FQDN Allowed to Access Internet

![WebGroup](images/egress_create_group.png)

![Edit Group](images/egress_edit_group.png)

![Polling](images/egress_polling.png)

### Deploy Aviatrix Spoke GW

- The public IP address will be different (Public EIP automatically allocated by CSP)
- The Subnet CIDR could be different (automatically picked up by Aviatrix Controller)
- Region: us-east-1

![Spoke](images/egress_spoke_gw.png)

Check the Egress setting. The Egress traffic is going through the AWS NAT GW.

![Egress](images/egress_egress.png)

### Enable spoke GW to become the Egress GW

1. Click +Local Egress on VPC/VNets.
2. In the Add Local Egress on VPC/VNets dialog, select the VPC/VNets on which to enable Local Egress.
3. Click Add.

[Read more at Aviatrix Documentation](https://docs.aviatrix.com/copilot/latest/network-security/index.html)

![Local](images/egress_add_local.png)

Add Local Egress on VPC/VNets
Adding Egress Control on VPC/VNet changes the default route on VPC/VNet to point to the Spoke Gateway and enables SNAT. Egress Control also requires additional resources on the Spoke Gateway.VPC/VNets

Now the diagram should look like the following:

![Vpc](images/egress_vpc.png)

## Conclusion

By bringing Aviatrix Secure Egress into play, our healthcare provider shored up their defenses, dropped the high costs, and eliminated the visibility black hole courtesy of the AWS NAT Gateway. Sensitive patient data is safe, and the provider’s reputation will be secured.
Remember, Aviatrix Secure Egress is your go-to for a secure, cost-effective solution for managing internet-bound traffic. Need more help? Our support team’s got your back.
