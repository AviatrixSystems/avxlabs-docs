# Lab 5 - EGRESS

## 1. OBJECTIVE

In this lab, we will demonstrate how to enable the `Egress Control` (that is one of the features that belong to the Distributed Cloud Firewall service) on the VPC that we want to target. Of course, the selected VPC should have subnets associated into Private Routing tables (i.e. without a default route pointing to the IGW). The Controller will reroute the traffic through the Aviatrix Spoke Gateway. The Egress Conbtrol can guarantee immediately better visibility and better control in order to replace the **CSP Native NAT Gateways**. <ins>The Egress Control allows to reduce the cloud costs and at the same, improve the security without impacting the architecture</ins>.

## 2. TOPOLOGY

Let's pinpoint the right candidate VPC, where would be possible to enable the Egress Control.

![lab6-initialtopology](images/lab6-initialtopology.png)
_Figure 138: Lab 6 Initial Topology_

The VPC **aws-us-east2-spoke1** has a private subnet, whereby the Egress Control can be activated in this specific VPC.

- Explore the Private Routing Tables inside the VPV **aws-us-east2-spoke1**

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways** and select the **_aws-us-east2-spoke1 GW_**, then click on the VPC/VNet Route Tables tab, then select any of the Private RTBs fron the Route table field.
```

![lab6-spokegw](images/lab6-spokegw.png)
_Figure 139: Select the Spoke GW in US-EAST-2_

![lab6-vpc](images/lab6-vpc.png)
_Figure 140: Check the private RTB_

You will notice that any private RTBs has the local cidr pointing to local and the three RFC19178 routes. The EC2 instance can't reach the internet public zone, because of the absence of a default route.

## 3. SSH TO THE EC2 INSTANCE IN THE PRIVATE SUBNET

