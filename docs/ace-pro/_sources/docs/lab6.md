# Lab 6 - EGRESS

## 1. Objective

In this lab, we will demonstrate how to enable the `Egress Control` (one of the features that belongs to the *Distributed Cloud Firewall* functionality) on the VPC that we want to target. Of course, the selected VPC should have at least a subnet associated into a Private Routing table (i.e. without a default route pointing to the IGW). The Controller will reroute the traffic through the Aviatrix Spoke Gateway. The Egress Control can guarantee immediately better visibility and better control in order to replace the **CSP Native NAT Gateways**. <ins>The Egress Control allows to reduce the cloud costs and at the same time, improve the security without impacting the architecture</ins>.

## 2. Topology

Let's pinpoint the right candidate VPC where would be possible to enable the Egress Control.

![lab6-initialtopology](images/lab6-initialtopology.png)
_Figure 138: Lab 6 Initial Topology_

The VPC **aws-us-east2-spoke1** has a private subnet in its environment, whereby the Egress Control can be activated in this specific VPC.

- Explore the Private Routing Tables inside the VPC **aws-us-east2-spoke1**

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any of the Private RTBs fron the Route table field.
```

![lab6-spokegw](images/lab6-spokegw.png)
_Figure 139: Select the Spoke GW in US-EAST-2_

![lab6-vpc](images/lab6-vpc.png)
_Figure 140: Check the private RTB_

You will notice that any private RTBs has its own **CIDR** pointing to local and the three **RFC1918** routes. With this scenario, the EC2 instance can't reach the internet public zone, due to the absence of a default route.

## 3. SSH to the EC2 instance in the Private Subnet

- SSH to the **_aws-us-east2-spoke1-test1_** instance from your laptop. Refer to your POD portal or alternatively, you can retrieve the Public IP from the CoPilot's Topology.

![lab6-public](images/lab6-publicip.png)
_Figure 141: SSH to aws-us-east2-spoke1-test1_

- Then from the **_aws-us-east2-spoke1-test1_** instance SSH to the **_aws-us-east2-spoke1-test2_** instance.

```{note}
The **_aws-us-east2-spoke1-test2_** instance resides within a private subnet!
```

```{tip}
Retrieve the Private IP of the **_aws-us-east2-spoke1-test2_** from the Topology
```

![lab6-retrieve](images/lab6-retrieve.png)
_Figure 142: Retrieve the private IP_

## 4. Egress Control
### 4.1 Enable the Egress Control

Now let's enable the egress within the VPC that is hosting the **_aws-us-east2-spoke1-test2_** instance.

```{note}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and click on `"+ Local Egress on VPC/VNets"`, then select the **_aws-us-east2-spoke1_** VPC and click on **Add**.
```

![lab6-egress](images/lab6-egress.png)
_Figure 143: Enable Local Egress_

![lab6-vpc](images/lab6-vpcegress.png)
_Figure 144: Choose the correct VPC_

### 4.2 Inspect the Private RTB

- As soon as the Egress Control is enabled, a `Default Route` is injected inside the Private RTBs. Verify its presence in any Private RTBs inside the **_aws-us-east2-spoke1_** VPC.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any of the Private RTBs fron the Route table field.
```

![lab6-defaultroute](images/lab6-defaultroute.png)
_Figure 145: Default route has been injected_

```{important}
Now, thanks to the default route, the instance **_aws-us-east2-spoke1-test2_** will be able to generate traffic towards the internet public zone.
```

### 4.3 Generate Traffic

From the **_aws-us-east2-spoke1-test2_** instance, try to curl the following websites:

```bash
curl www.aviatrix.com
```
```bash
curl www.wikipedia.com
```
```bash
curl www.espn.com
```
```bash
curl www.facebook.com
```

![lab6-generatetraffic](images/lab6-generatetraffic.png)
_Figure 146: Generate traffic_

Let's now check whether the Spoke Gateway could gather **NetFlow** data after generating the aforementioned *curl* commands.

Go to **CoPilot > Security > Egress > Overview (default)**

![lab6-nodatafound](images/lab6-nodatafound.png)
_Figure 147: No Data Found_

You will notice the Message `"No Data Found"`. You have successfully activated your egress control without disrupting anything that is sitting on the private subnet, nevertheless, if you want to get the NetFlow information, you need to apply a `Distributed Cloud Firewall RULE`, such that you can start assess the behaviour of the Private Subnet and get a good understanding of what domains have been reached out from the private subnet.

### 4.4 Enable DCF

- Enable the Distributed Cloud Firewall.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on `"Begin Using Distributed Cloud Firewall"`
```

![lab6-activate](images/lab6-activate.png)
_Figure 148: Activate DCF_

After having enabled the DCF, the `Greendfield-Rule` gets generated automatically. This rule essentially allows all kind of traffic.

![lab6-greenfield](images/lab6-greenfield.png)
_Figure 149: Greenfield-Rule_

- Apply the `"Any-Web"` Web Group and `"Logging"` to the Greenfield-Rule, in order to activate the Observabiliy for the Egress traffic, likewise the `"Monitor"` feature section within the Distributed Cloud Firewall

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the pencil icon for editing the **Greenfield-Rule**.
```

![lab6-pencil](images/lab6-pencil.png)
_Figure 150: Pencil icon_

Then go to the `"WebGroups"` field and select the **Any-Web**. Moreover, enable the `"Logging"` turning on the corresponding knob. Do not forget to click on **Save in Drafts**.

![lab6-geditgreen](images/lab6-editgreen.png)
_Figure 151: Edit Greenfield-Rule_

The modified Greenfield-Rule will remain in *Draft state* until you do not push on the `"Commit"` button.

![lab6-commit](images/lab6-commit.png)
_Figure 152: Commit_

- Launch again the following curl commands from the instance **_aws-us-east2-spoke1-test2_**.

```bash
curl www.aviatrix.com
```
```bash
curl www.wikipedia.com
```
```bash
curl www.espn.com
```
```bash
curl www.facebook.com
```

Go to **CoPilot > Security > Egress > Overview (default)**

Now you have finally the egress observability with a full list of domains hit by the EC2 instance inside the private subnet.

![lab6-overview](images/lab6-overview.png)
_Figure 153: Overview_

Go to **CoPilot > Security > Egress > Monitor** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east2-spoke1 VPC_**.

![lab6-monitor](images/lab6-monitor.png)
_Figure 154: Monitor_

You will get a granular Layer 7 visibility that allows you to get a good understanding of how the egress traffic has been consumed and also allows you to help make decisions on how to potentially optimize that.

## 5. Zero Trust Policy
### 5.1 Create a New WebGroup

Let's move towards a posture where only the allowed egress domains are in place.

Go to **CoPilot > Security > Distributed Cloud Firewall > WebGroups** and click on `"+ WebGroups"` button*.

![lab6-webgroup](images/lab6-webgroup.png)
_Figure 155: + WebGroups_

Create a **_WebGroup_** with the following parameters:

- **Name**: <span style='color:#33ECFF'>allow-specific-domains</span>
- **Type**: <span style='color:#33ECFF'>Domains</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.aviatrix.com</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.wikipedia.com</span>

Do not forget to click on **Save**.

![lab6-webgroup2](images/lab6-webgroup2.png)
_Figure 156: WebGroup creation_

```{important}
The purpose of this **WebGroup** is to authorize traffic only towards both *www.aviatix.com* and *www.wikipedia.com*, therefore the curl commands issued towards *www.espn.com* and *www.facebook.com* will be blocked.
```

### 5.2 Create a New DCF Rule

Go to **CoPilot > Security Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

Create a new **_DCF rule_** with the following parameters:

- **Name**: <span style='color:#33ECFF'>allow-two-domains</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Public Internet</span>
- **WebGroups**: <span style='color:#33ECFF'>allow-specific-domains</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{warning}
Keep the other parameters with their default values!
```

![lab6-newrule](images/lab6-newrule.png)
_Figure 157: New Rule_

Furthermore, click on the **three dots** symbol on the left-hand side of the **_Greenfield-Rule_** and turn off the `Enforcement`.

```{important}
Unenforcing the Greenfield-Rule will restore the Implicit Invisible Deny Rule
```

![lab6-enforcementoff](images/lab6-turnoff.png)
_Figure 157: Enforcement off_

- Now you can proceed and click on the `"Commit"` button.

![lab6-finalcommit](images/lab6-finalcommit.png)
_Figure 158: Commit_

Go to **CoPilot > Security > Egress > Monitor** and select the **_Live View_** from the `"Time Period"` field, then select the **_aws-us-east2-spoke1_** VPC from the `"VPC/VNets"` drop-down window.

- Now launch again the following curl commands from the instance **_aws-us-east2-spoke1-test2_**.

```bash
curl www.aviatrix.com
```
```bash
curl www.wikipedia.com
```
```bash
curl www.espn.com
```
```bash
curl www.facebook.com
```

You will instanteously noticed that only **_www.aviatrix.com_** and **_www.wikipedia.com_** are allowed. Traffic towards **_www.espn.com_** and **_www.facebook.com_** will match the `"Invisible Implicit Deny"` therefore, it will be dropped.

![lab6-allowed](images/lab6-okfinal.png)
_Figure 159: Allowed_