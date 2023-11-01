# Pod Details

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

## URLs

[Controller](https://ctrl.pod149.aviatrixlab.com)

The Aviatrix Controller is a management and control plane or a single pane of glass that enables you to manage and support a single or multi-cloud network architecture. You can deploy an Aviatrix Controller through any of the four major CSP (Cloud Service Provider) marketplaces:

- AWS (Amazon Web Services)
- Microsoft Azure
- GCP (Google Cloud Platform)
- OCI (Oracle Cloud Infrastructure)

[CoPilot](https://cplt.pod149.aviatrixlab.com)

Aviatrix CoPilot provides a global operational view of your multicloud network, supporting business-critical cloud networking. CoPilot enables enterprises to implement intelligent cloud networking that embeds security into their networks with monitoring and diagnostic tools that help support application availability.

## Access credentials

User:

```bash
student
```

Password:

```bash
1021TBQXLP2!2vN
```
