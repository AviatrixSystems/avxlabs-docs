# Quiz - Hints

## Initial Topology

This is the current topology of a multicloud enterprise that has two Data Centers: the first one is connected to the transit GW in AWS, whereas the second one is connected to the transit GW in Azure. The connectivity across the three CSPs occurs through the Aviatrix Transit Core Backbone.

```{figure} images/security-topology.png
---
height: 400px
align: center
---
Initial Topology
```

## Hints

<span style='color:#479608'>Q1.</span> Which Aviatrix CoPilot feature can block IP traffic to/from IP addresses associated with particular geographic regions.

```{hint}
Check the **CoPilot > Security**
```

<span style='color:#479608'>Q2.</span> What encryption scheme is being used by the site-to-cloud connection with the Azure transit gateway?

```{hint}
Check the **CoPilot > Networking > Connectivity**
```

<span style='color:#479608'>Q3.</span> A major intrusion has occurred. From which instance was this detected? Mention Name

```{hint}
Check the **CoPilot > Security > ThreatIQ**
```

<span style='color:#479608'>Q4.</span> List if any region(s) are being blocked by GeoBlocking.

```{hint}
Check the **CoPilot > Security**
```

<span style='color:#479608'>Q5.</span> Spoke-aws-dev is isolated from 2 of the 6 spokes or onprem network domains. 
Which 2 spokes can it not communicate with due to the configured network segmentation?

```{hint}
Check the **CoPilot > Networking > Network Segmentation**
```

<span style='color:#479608'>Q6.</span> Traffic from the gcp-dev-vm instance (in spoke-gcp-dev, network domain: gcp-dev) cannot connect to the azure-prod-vm instance (in spoke-azure-all, 
network domain: azure-all) on port 443 despite the fact that the two network domains are connected. Why is this?

```{hint}
Check the **CoPilot > Security > Distributed Cloud Firewall**
```

<span style='color:#479608'>Q7.</span> How many https egress domains are being explicitly allowed?

```{hint}
Check the **CoPilot > Security > Distributed Cloud Firewall**
```

<span style='color:#479608'>Q8.</span> The azure-dev-vm cannot communicate with the azure-prod-vm despite being deployed to the same vnet. Why is this?

```{hint}
Check the **CoPilot > Security > Distributed Cloud Firewall**
```

<span style='color:#479608'>Q9.</span> There are 3 VMs that have been automatically added to the SmartGroup named dev. Why were they added to this group?

```{hint}
Check the **CoPilot > SmartGroups**
```

<span style='color:#479608'>Q10.</span> Name the Aviatrix gateway where ThreatIQ has detected malicious inbound attacks.

```{hint}
Check the **CoPilot > Security > ThreatIQ**
```

<span style='color:#479608'>Q11.</span> Name 3 domains where egress connectivity is being attempted, but denied.

```{hint}
Check the **CoPilot > Security > Egress > Monitor**
```
<span style='color:#479608'>Q12.</span> As a member of the security team, you've been alerted to a compromised subnet, 10.1.2.0/24, posing a security risk through lateral movement. Your task is to swiftly identify the associated VPC. With limited time and unavailable IPAM and networking teams on a late Friday evening, and lacking knowledge of the cloud platform hosting the subnet, what approach do you employ to swiftly identify the cloud and VPC housing the subnet for immediate resolution?.

```{hint}
Check the **CoPilot > Cloud Resources**
```