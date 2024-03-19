# POD Details

## Topology

The ACE Security course topology is deployed across 3 CSPs, comprising the following components:

- 1 Transit VPC/Vnet and gateway per CSP - AWS, Azure, and GCP.
- 2 Spoke VPCs in AWS and GCP with a single test instance deployed to each - 1 dev and 1 prod.
- 1 Spoke Vnet in Azure with both dev and prod VMs deployed.
- 1 Site-to-cloud connection to each of the AWS and Azure Transit gateways.

```{figure} images/ace_sec_topology.png
---
align: center
---
ACE Security pod topology
```

## URL

```{image} images/copilot.png
:alt: CoPilot
:class: bg-primary
:width: 25px
:align: left
```

<a href="https://cplt.pod1.aviatrixlab.com" target="_blank">CoPilot</a>

Aviatrix **CoPilot** provides a global operational view of your multicloud network, supporting business-critical cloud networking. CoPilot enables enterprises to implement intelligent cloud networking that embeds security into their networks with monitoring and diagnostic tools that help support application availability.

## Access credentials

User:

```bash
student
```

Password:

```bash
210Hf67x##wFkH
```
