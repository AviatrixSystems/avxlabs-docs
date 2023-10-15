# Lab 6 - EGRESS

## 1. Objective

In this lab, we will demonstrate how to enable the `Egress Control` (one of the features that belongs to the *Distributed Cloud Firewall* functionality) on the VPC that we want to target. Of course, the selected VPC should have at least a subnet associated into a Private Routing table (i.e. without a default route pointing to the IGW). The Controller will reroute the traffic through the Aviatrix Spoke Gateway. The Egress Control can guarantee immediately better visibility and better control in order to replace the **CSP Native NAT Gateways**. <ins>The Egress Control allows to reduce the cloud costs and at the same time, improve the security without impacting the architecture</ins>.

## 2. Topology

Let's pinpoint the right candidate VPC where would be possible to enable the Egress Control.

```{figure} images/lab6-initialtopology.png
---
height: 400px
align: center
---
Lab 6 Initial Topology
```

The VPC **aws-us-east2-spoke1** has a private subnet in its environment, whereby the Egress Control can be activated in this specific VPC.

- Explore the Private Routing Tables inside the VPC **aws-us-east2-spoke1**

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any of the Private RTBs fron the Route table field.
```

```{figure} images/lab6-spokegw.png
---
align: center
---
Select the Spoke GW in US-EAST-2
```

```{figure} images/lab6-vpc.png
---
align: center
---
Check the private RTB
```

You will notice that any private RTBs has its own **CIDR** pointing to local and the three **RFC1918** routes. With this scenario, the EC2 instance can't reach the internet public zone, due to the absence of a default route.

## 3. SSH to the EC2 instance in the Private Subnet

- SSH to the **_aws-us-east2-spoke1-test1_** instance from your laptop. Refer to your POD portal or alternatively, you can retrieve the Public IP from the CoPilot's Topology.

```{figure} images/lab6-publicip.png
---
align: center
---
SSH to aws-us-east2-spoke1-test1
```

- Then from the **_aws-us-east2-spoke1-test1_** instance SSH to the **_aws-us-east2-spoke1-test2_** instance.

```{note}
The **_aws-us-east2-spoke1-test2_** instance resides within a private subnet!
```

```{tip}
Retrieve the Private IP of the **_aws-us-east2-spoke1-test2_** from the Topology
```

```{figure} images/lab6-retrieve.png
---
align: center
---
Retrieve the private IP
```

## 4. Egress Control
### 4.1 Enable the Egress Control

Now let's enable the egress within the VPC that is hosting the **_aws-us-east2-spoke1-test2_** instance.

```{note}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and click on `"+ Local Egress on VPC/VNets"`, then select the **_aws-us-east2-spoke1_** VPC and click on **Add**.
```

```{figure} images/lab6-egress.png
---
align: center
---
Enable Local Egress
```

```{figure} images/lab6-vpcegress.png
---
height: 400px
align: center
---
Choose the correct VPC
```

### 4.2 Inspect the Private RTB

- As soon as the Egress Control is enabled, a `Default Route` is injected inside solely the Private RTBs (Public RTBs are not impacted, whereby, they will continue to have the defaulte route pointing towards the Native CSP IGW). Verify its presence in any Private RTBs inside the **_aws-us-east2-spoke1_** VPC.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any of the Private RTBs fron the Route table field.
```

```{figure} images/lab6-defaultroute.png
---
align: center
---
Default route has been injected
```

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

```{figure} images/lab6-generatetraffic.png
---
align: center
---
Generate traffic
```

Let's now check whether the Spoke Gateway could gather **NetFlow** data after generating the aforementioned *curl* commands.

Go to **CoPilot > Security > Egress > Overview (default)**

```{figure} images/lab6-nodatafound.png
---
align: center
---
No Data Found
```

You will notice the Message `"No Data Found"`. You have successfully activated your egress control without disrupting anything that is sitting on the private subnet, nevertheless, if you want to get the NetFlow information, you need to apply a `Distributed Cloud Firewall RULE`, such that you can start assess the behaviour of the Private Subnet and get a good understanding of what domains have been reached out from the private subnet.

### 4.4 Enable DCF

- Enable the Distributed Cloud Firewall.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on `"Begin Using Distributed Cloud Firewall"`
```

```{figure} images/lab6-activate.png
---
align: center
---
Activate DCF
```

After having enabled the DCF, the `Greendfield-Rule` gets generated automatically. This rule essentially allows all kind of traffic.

```{figure} images/lab6-greenfield.png
---
align: center
---
Greenfield-Rule
```

- Apply the `"Any-Web"` **Predefined** Web Group and `"Logging"` to the Greenfield-Rule, in order to activate the Observabiliy for the Egress traffic, likewise the `"Monitor"` feature section within the Distributed Cloud Firewall

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the pencil icon for editing the **Greenfield-Rule**.
```

```{figure} images/lab6-pencil.png
---
align: center
---
Pencil icon
```

Then go to the `"WebGroups"` field and select the **Any-Web** WebGroup. Moreover, enable the `"Logging"` turning on the corresponding knob. Do not forget to click on **Save in Drafts**.

```{figure} images/lab6-editgreen.png
---
align: center
---
Edit Greenfield-Rule
```

The modified Greenfield-Rule will remain in *Draft state* until you do not push on the `"Commit"` button.

```{figure} images/lab6-commit.png
---
align: center
---
Commit
```

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
Bear in mind that **ICMP** protocol will be not allowed with the current DCF rule configuration!
```

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and you will see a total of **4 logs**.

```{figure} images/lab6-monitorpermit.png
---
align: center
---
Monitor: 4 logs
```

Go to **CoPilot > Security > Egress > Overview (default)**

Now you have finally the egress observability with a full list of domains hit by the EC2 instance inside the private subnet.

```{figure} images/lab6-overview.png
---
align: center
---
Overview
```

Furthermore, go to **CoPilot > Security > Egress > Monitor** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east2-spoke1 VPC_**.

```{figure} images/lab6-monitor.png
---
align: center
---
Monitor
```

You will get a granular Layer 7 visibility that allows you to get a good understanding of how the egress traffic has been consumed and also allows you to help make decisions on how to potentially optimize that.

## 5. Zero Trust Network Approach
### 5.1 Create a New WebGroup

Let's move towards a posture where only the allowed egress domains are in place.

Go to **CoPilot > Security > Distributed Cloud Firewall > WebGroups** and click on `"+ WebGroups"` button*.

```{figure} images/lab6-webgroup.png
---
align: center
---
WebGroups
```

Create a **_WebGroup_** with the following parameters:

- **Name**: <span style='color:#33ECFF'>two-domains</span>
- **Type**: <span style='color:#33ECFF'>Domains</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.aviatrix.com</span>
- **Domains/URLs**: <span style='color:#33ECFF'>www.wikipedia.com</span>

Do not forget to click on **Save**.

```{figure} images/lab6-webgroup2.png
---
align: center
---
WebGroup creation
```

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

```{figure} images/lab6-newrule.png
---
align: center
---
New Rule
```

```{important}
- **Anywhere (0.0.0.0/0)** = 3x RFC1918's routes + Default Route (IGW)
- **Publlic Internet** = Default Route (IGW)
```

Before committing, click once again on the `"+ Rule"` button.

```{figure} images/lab6-beforecommitting.png
---
align: center
---
Before Commit
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

```{figure} images/lab6-explicitdeny.png
---
align: center
---
Explicit-Deny-Rule
```

- Now you can proceed and click on the `"Commit"` button.

```{figure} images/lab6-newcommit.png
---
align: center
---
Commit
```

Go to **CoPilot > Security > Egress > Monitor** and select the **_Live View_** from the `"Time Period"` field, then select the **_aws-us-east2-spoke1_** VPC from the `"VPC/VNets"` drop-down window.

### 5.2.2 Test the new rule

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

You will almost instanteously notice that only **_www.aviatrix.com_** and **_www.wikipedia.com_** are allowed. Traffic towards **_www.espn.com_** and **_www.football.com_** will match the new `"Explicit Deny Rule"`, therefore it will be denied and therefore dropped.

```{figure} images/lab6-liveview.png
---
align: center
---
Denied
```

## 5.3 IDS
### 5.3.1 Create a New Rule

Let's now test the **_IDS_** feature (i.e. Intrusion Detection System). 

Go to **CoPilot > Security Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

Create a new DCF Rule with the following parameters:

- **Name**: <span style='color:#33ECFF'>Inspect-DNS</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>Public Internet</span>
- **Protocol**: <span style='color:#33ECFF'>Any</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>
- **Intrustion Detection (IDS)**: <span style='color:#33ECFF'>On</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab6-ids.png
---
align: center
---
Inspect-DNS
```

Proceed clicking on the **Commit** button.

```{figure} images/lab6-idscommit.png
---
align: center
---
Commit
```

### 5.3.2 Prepare the simulator

- SSH to the **_aws-us-east2-spoke1-test2_** instance and launch the following commands.

```bash
sudo su -
```
```bash
curl -sSL https://raw.githubusercontent.com/0xtf/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS
```

```{figure} images/lab6-sudo.png
---
align: center
---
Commands issued
```

The last command will show up a **_simulator_** from whom you will be able to launch an attack for testing the `"Suricata IDS"`

```{figure} images/lab6-suricata.png
---
align: center
---
Simulator
```

### 5.3.2 Test the New Rule and the IDS feature

- Before launching the attack, edit the new DCF rule, clicking on the pencil icon beside the **_Inspect-DNS_** rule.

```{figure} images/lab6-suricataedit.png
---
align: center
---
Edit existing rule
```

Insert the following parameters and do not forget to click on **Save In Drafts**:

- **Protocol**: <span style='color:#33ECFF'>UDP</span>
- **Port**: <span style='color:#33ECFF'>53</span>

```{figure} images/lab6-dns.png
---
align: center
---
Modify the rule
```

Now click on the **Commit** button.

```{figure} images/lab6-commit3.png
---
align: center
---
Commit
```

From the EC2 instance **_aws-us-east2-spoke1-test2_**, type **5** for launching a malicious attack, specifically the attack will carry out an attempted connection towards a TOR server.

```{figure} images/lab6-5.png
---
align: center
---
Malicious known attack
```

Now go to **CoPilot > Security > Distributed Cloud Firewall > Detected Intrusions**, and you will be able to find indicators that detected that attempt to contact a TOR server, through a DNS request.

```{tip}
If you do not see the logs immediately, click on the **refresh** button
```

```{figure} images/lab6-refresh.png
---
align: center
---
Detected Intrusions
```

Click on any **Timestamp** to get additional insight on that specific attack.

```{figure} images/lab6-final.png
---
align: center
---
Additional insights
```

```{note}
The indicator is showing clearly that we tried reaching out to Google DNS and then querying for a TOR domain!
```