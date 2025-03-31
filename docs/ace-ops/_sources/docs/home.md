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

All applications in AWS and GCP have **Public** and **Private** IP’s, whereas instances in Azure <ins>only have Private IP’s</ins>.

These are the CIDR blocks per each CSP:

- AWS = 10.0.0.0/8
- GCP = 172.16.0.0/16
- Azure = 192.168.0.0/16

<ins>Databases do not have HTTP running</ins>. All other apps have HTTP running (i.e. curl command can be issued).

- 3rd octet of all Spoke 1’s is 211
- 3rd octet of all Spoke 2’s is 212
  
Management access from Internet for instances in AWS and GCP is enabled, however, none of the apps are currently exposed to the Internet.

