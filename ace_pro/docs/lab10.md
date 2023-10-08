# Lab 10 - DISTRIBUTED CLOUD FIREWALL

## 1. Objective

This lab will demonstrate how the `Distributed Cloud Firewall` work.

 
## 2. Distributed Cloud Firewall Overview

The Distributed Cloud Firewall feature allows to create logical containers, called `Smart Groups`, that encompass instances that present similarities inside a VPC/VNet/VCN, and then it also allows to enforce rules (aka **_Distributed Firewalling_**) within a Smart Group (i.e. the `intra-rule`) or among Smart Groups (i.e. the `inter-rule`).

```{note}
At this point in the lab, there is a unique routing domain (i.e. a **_Flat Routing Domain_**), due to the connection policy applied in Lab 3, between the Green domain and the Blue domain.
```

All the Test instances have been deployed with the typical <ins>CSP tags</ins>. 

```{important}
The CSP tagging is the recommended method for defining the SmartGroups.
```

In this lab you are asked to achieve the following requirements among the instances deployed across the three CSPs:

- Create a Smart Group with the name `"bu1"` leveraging the tag `"environment"`
- Create a Smart Group with the name `"bu2"` leveraging the tag `"environment"`
- Create an `intra-rule` that allows ICMP traffic within bu1
- Create an `intra-rule` that allows ICMP traffic within bu2
- Create an `inter-rule` that allows ICMP traffic only from bu1 towards bu2