# Lab 5 - EGRESS

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

- As soon as the Egress Control is enabled, a `Default Route` is injected inside solely the Private RTBs (Public RTBs are not impacted, whereby, they will continue to have the defaulte route pointing towards the Native CSP IGW). Verify its presence in any Private RTBs inside the **_aws-us-east2-spoke1_** VPC.

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
curl www.football.com
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

- Apply the `"Any-Web"` **Predefined** Web Group and `"Logging"` to the Greenfield-Rule, in order to activate the Observabiliy for the Egress traffic, likewise the `"Monitor"` feature section within the Distributed Cloud Firewall

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the pencil icon for editing the **Greenfield-Rule**.
```

![lab6-pencil](images/lab6-pencil.png)
_Figure 150: Pencil icon_

Then go to the `"WebGroups"` field and select the **Any-Web** WebGroup. Moreover, enable the `"Logging"` turning on the corresponding knob. Do not forget to click on **Save in Drafts**.

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
curl www.football.com
```

```{warning}
Bear in mind that **ICMP** protocol will be not allowed with the current DCF rules configuration!
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and you will see a total of **4 logs**.

![lab6-overview](images/lab6-monitorpermit.png)
_Figure 153: Monitor: 4 logs_

Go to **CoPilot > Security > Egress > Overview (default)**

Now you have finally the egress observability with a full list of domains hit by the EC2 instance inside the private subnet.

![lab6-overview](images/lab6-overview.png)
_Figure 153: Overview_

Furthermore, go to **CoPilot > Security > Egress > Monitor** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east2-spoke1 VPC_**.

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

- **Name**: <span style='color:#33ECFF'>two-domains</span>
- **Type**: <span style='color:#33ECFF'>Domains</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.aviatrix.com</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.wikipedia.com</span>

Do not forget to click on **Save**.

![lab6-webgroup2](images/lab6-webgroup2.png)
_Figure 156: WebGroup creation_

```{important}
The purpose of this **WebGroup** is to authorize traffic only towards both the Domains *www.aviatix.com* and *www.wikipedia.com*, therefore the curl commands issued towards other Domains will be blocked.
```

## 5.2 New DCF Rule 
## 5.2.1 Create a new rule

Go to **CoPilot > Security Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

Create a new **_DCF rule_** with the following parameters:

- **Name**: <span style='color:#33ECFF'>allow-domains</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Public Internet</span>
- **WebGroups**: <span style='color:#33ECFF'>two-domains</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{warning}
Keep the other parameters with their default values!
```

![lab6-newrule](images/lab6-newrule.png)
_Figure 157: New Rule_

```{important}
- **Anywhere (0.0.0.0/0)** = 3x RFC1918's routes + Default Route
- **Publlic Internet** = Default Route
```

Create an `Explicit Deny Rule` that will allow to see the logs for the `"Denied"` actions.

- **Name**: <span style='color:#33ECFF'>Explicit-Deny-Rule</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Deny**</span>
- **Place Rule**: <span style='color:#33ECFF'>Below</span>
  - - **Existing Rule**: <span style='color:#33ECFF'>allow-domains</span>

Do not forget to click on **Save In Drafts**.

![lab6-newrule](images/lab6-explicitdeny.png)
_Figure 157: New Rule_

- Now you can proceed and click on the `"Commit"` button.

![lab6-finalcommit](images/lab6-newcommit.png)
_Figure 158: Commit_

Go to **CoPilot > Security > Egress > Monitor** and select the **_Live View_** from the `"Time Period"` field, then select the **_aws-us-east2-spoke1_** VPC from the `"VPC/VNets"` drop-down window.

### 5.2.1 Test the new rule

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
curl www.football.com
```

You will instanteously noticed that only **_www.aviatrix.com_** and **_www.wikipedia.com_** are allowed. Traffic towards **_www.espn.com_** and **_www.football.com_** will match the new `"Explicit Deny Rule"`, therefore it will be dropped.

![lab6-allowed](images/lab6-liveview.png)
_Figure 159: Denied_

## 5.3 IDS
### 5.3.1 Create a New Rule

Let's now test the **_IDS_** feature (i.e. Intrusion Detection System). Create a new DCF Rule with the following parameters:

- **Name**: <span style='color:#33ECFF'>Inspect-DNS</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Public Internet</span>
- **Protocol**: <span style='color:#33ECFF'>Any</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Intrustion Detection (IDS)**: <span style='color:#33ECFF'>On</span>

Do not forget to click on **Save In Drafts**.

![lab6-ids](images/lab6-ids.png)
_Figure 159: Inspect-DNS_

Proceed clicking on the **Commit** button.

![lab6-idscommit](images/lab6-idscommit.png)
_Figure 159: Commit_

### 5.3.2 Prepare the simulator

- SSH to the **_aws-us-east2-spoke1-test2_** instance and launch the following commands.

```bash
sudo su -
```
```bash
curl -sSL https://raw.githubusercontent.com/0xtf/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS
```

![lab6-sudo](images/lab6-sudo.png)
_Figure 159: Commands issued_

The last command will show up a **_simulator_** from whom you will be able to launch an attack for testing the `"Suricata IDS"`

![lab6-simulator](images/lab6-suricata.png)
_Figure 159: Simulator_

### 5.3.2 Test the New Rule and the IDS feature

- Before launching the attack, edit the new DCF rule, clicking on the pencil icon beside the **_Inspect-DNS_** rule.

![lab6-5](images/lab6-suricataedit.png)
_Figure 159: Edit existing rule_

Insert the following parameters and do not forget to click on **Save In Drafts**:

- **Protocol**: <span style='color:#33ECFF'>UDP</span>
- **Port**: <span style='color:#33ECFF'>53</span>

![lab6-commit3](images/lab6-dns.png)
_Figure 159: Modify the rule_

Now click on the **Commit** button.

![lab6-commit3](images/lab6-commit3.png)
_Figure 159: Commit_

From the EC2 instance **_aws-us-east2-spoke1-test2_**, type **5** for launching a malicious attack, specifically the attack will carry out an attempted connection towards a TOR server.

![lab6-5](images/lab6-5.png)
_Figure 159: Malicious known attack_

Now go to **CoPilot > Security > Distributed Cloud Firewall > Detected Intrusions**, click on the **refresh** button and you will be able to find indicators that detected that attempt to contact a TOR server, through a DNS request

![lab6-5](images/lab6-refresh.png)
_Figure 159: Detected Intrusions_

Click on any **Timestamp** to get additional insight on that specific attack.

![lab6-final](images/lab6-final.png)
_Figure 159: Additional insights_

```{note}
The indicator is showing clearly that we tried reaching out to Google DNS and then querying for a TOR domain!
```