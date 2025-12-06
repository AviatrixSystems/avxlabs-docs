![ACE](../../docs/_logos/ace_operations_banner.new.png)
# Welcome to ACE Operations Lab

## 1. LAB TOPOLOGY
There are **8** labs in ACE Operations that are based on a fictitious company called **ACE Inc**.

ACE Inc. is a SaaS provider that has a presence in AWS, GCP, and Azure.

There is already one **Site2Cloud** connection:

- Onprem Data Center

Here is a view of the initial topology:

```{figure} images/topology-initial.png
---
height: 400px
align: center
---
Initial Topology
```

## 2. TECHNICAL DETAILS

All applications in AWS and GCP have both _public_ and _private_ IP addresses, whereas Azure VMs are configured with _private_ IP addresses only.

- **CIDR blocks by CSP**:

  - AWS: 10.0.0.0/8

  - GCP: 172.16.0.0/16

  - Azure: 192.168.0.0/16

- Databases do not expose HTTP. All other applications expose HTTP (i.e., curl can be used to test).

- **Network addressing**:

  - The 3rd octet of all Spoke 1 networks is 211

  - The 3rd octet of all Spoke 2 networks is 212

- **SSH access**:

  - Inbound SSH is enabled for instances in AWS and GCP.

  - Instances in Azure are not accessible via SSH because they reside in private subnets.
