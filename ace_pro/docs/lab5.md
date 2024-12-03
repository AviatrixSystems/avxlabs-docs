# Lab 5 - CLOUD PERIMETER SECURITY (Secure Cloud Egress)

## 1. Objective

In this lab, we will demonstrate how to enable the `Cloud Perimeter Security` (**Secure Cloud Egress**), one of the features that belongs to the *Distributed Cloud Firewall* functionality, on the VPC that we want to target. Of course, the selected VPC should have at least a subnet associated to a Private Routing table (i.e. without a default route pointing to the IGW). The Controller will reroute the traffic through the Aviatrix Spoke Gateway. The Egress Control can guarantee immediately better visibility and better control in order to replace the **CSP Native NAT Gateways**. <ins>Cloud Perimeter Security (Secure Cloud Egress) allows to reduce the cloud costs and at the same time, improve the security without impacting the architecture</ins>.

## 2. Topology

Let's pinpoint the right candidate VPC where would be possible to enable the Egress Control.

```{figure} images/lab6-initialtopology.png
---
height: 400px
align: center
---
Lab 6 Initial Topology
```

The VPC **aws-us-east-2-spoke1** has a private subnet in its environment, whereby the Egress Control can be activated in this specific VPC.

- Explore the Private Routing Tables inside the VPC **aws-us-east-2-spoke1**

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east-2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any of the Private RTBs from the `Route Table` field.
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

You will notice that any private RTBs has its own **CIDR** pointing to local and the three **RFC1918** routes pointing to the Aviatrix Spoke Gateway. 

With this scenario, the EC2 instance can't reach the *Internet Public Zone*, due to the absence of a <ins>default route</ins>.

## 3. SSH to the EC2 instance in the Private Subnet

- SSH to the **_aws-us-east-2-spoke1-test1_** instance from your laptop. Refer to your POD portal or alternatively, you can retrieve the Public IP from the CoPilot's Topology.

```{figure} images/lab6-publicip.png
---
height: 400px
align: center
---
SSH to aws-us-east-2-spoke1-test1
```

- Then from the **_aws-us-east-2-spoke1-test1_** instance SSH to the **_aws-us-east-2-spoke1-test2_** instance.

```{figure} images/lab6-ssh.png
---
align: center
---
From test1 to test2
```

```{note}
The **_aws-us-east-2-spoke1-test2_** instance resides within a private subnet! 
```

```{tip}
Retrieve the Private IP of the **_aws-us-east-2-spoke1-test2_** from the Topology
```

```{figure} images/lab6-retrieve.png
---
height: 400px
align: center
---
Retrieve the private IP
```

## 4. Egress Control
### 4.1 Enable the Egress Control

Now let's enable the egress within the VPC that is hosting the **_aws-us-east-2-spoke1-test2_** instance.

```{note}
Go to **CoPilot > Security > Egress > Egress VPC/VNets** and click on `"+ Local Egress on VPC/VNets"`, then select the **_aws-us-east-2-spoke1_** VPC and click on **Add**.
```

```{figure} images/lab6-egress.png
---
align: center
---
Enable Local Egress
```

```{figure} images/lab6-vpcegress.png
---
align: center
---
Choose the correct VPC
```

### 4.2 Inspect the Private RTB

Upon enabling Local Egress on the specified VPC, the Aviatrix Controller will immediately execute two actions on the specified VPC:

Injecting a **Default Route**: this route is added exclusively to the Private Route Tables (RTBs). Public Route Tables (RTBs) remain unaffected and will continue to have their default route pointing towards the native Cloud Service Provider (CSP) Internet Gateway (**IGW**).

Enabling Single IP SNAT on VPC Spoke Gateway: Source Network Address Translation (SNAT) will be enabled on the VPC spoke gateway using a single IP address.

At this point, the Aviatrix  performs the same functions as the CSP NAT Gateway.

- Verify its presence in any Private RTBs inside the **_aws-us-east-2-spoke1_** VPC.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east-2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab, then select any Private RTBs from the **Route Table** field.
```

```{figure} images/lab6-defaultroute.png
---
align: center
---
Default route has been injected
```

```{important}
Now, thanks to the default route, the instance **_aws-us-east-2-spoke1-test2_** will be able to generate traffic towards the internet public zone.
```

### 4.3 Generate Traffic

From the **_aws-us-east-2-spoke1-test2_** instance, try to curl the following websites:

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

Let's now check whether the Spoke Gateway could gather **NetFlow** data after generating the aforementioned *curl* commands, or not.

Go to **CoPilot > Security > Egress > Overview (default)**

```{figure} images/lab6-nodatafound.png
---
height: 400px
align: center
---
No Data Found
```

You will notice the Message `"No Data Found"`. You have successfully activated your egress control without disrupting anything that is sitting on the private subnet, nevertheless, if you want to get the NetFlow information, you need to apply a `Distributed Cloud Firewall RULE`, such that you can start evaluate the behaviour of the Private Subnet and get a good understanding of what domains have been reached out from the private subnet.

### 4.4 Enable DCF

- Enable the Distributed Cloud Firewall.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on `"Enable Distributed Cloud Firewall"`.
Afterwards click on `"Begin using Distributed Cloud Firewall"`, then click on `"Begin"`.
```

```{figure} images/lab6-activate.png
---
align: center
---
Enable Distributed Cloud Firewall
```

```{figure} images/lab6-newjoe.png
---
align: center
---
Begin using Distributed Cloud Firewall
```

```{figure} images/lab6-newjoe2.png
---
align: center
---
Begin
```

After having enabled the DCF, two Rules will get generated, automatically:
- `Greendfield-Rule`
- `DefaultDenyAll` = EXPLICIT DENY

The first rule essentially allows all kind of traffic.

```{figure} images/lab6-greenfield.png
---
height: 150px
align: center
---
Automatic rules injected by the Controller
```



#### 4.4.1 Identify the subnet where the private workload resides

First and foremost, you have to identify the **subnet** where the **_aws-us-east-2-spoke1-test2_** instance resides.

```{figure} images/lab6-greenfieldneww.png
---
height: 400px
align: center
---
Private Subnet
```

Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for the **_aws-us-east-2-spoke1-test2_** instance on the search field on the right-hand side.


From the outcome you have to pinpoint the `Availability Zone`.

```{figure} images/lab6-greenfieldneww2.png
---
height: 150px
align: center
---
Private Subnet
```

Now that you know in what `Availability Zone` the private workload resides, you need to select the `VPC/VNets & Subnets` TAB and filter out based on the **_aws-us-east-2-spoke1_** VPC.


Identify the `Private Subnet` that belongs to the `us-east-2a` AZ and copy the corresponding **_`IP Address CIDR`_** value!

```{figure} images/lab6-greenfieldneww3.png
---
height: 300px
align: center
---
Private Subnet
```

#### 4.4.2 Create an Ad-Hoc SmartGroup

Go to **CoPilot > Groups** and click on the `"+ SmartGroup"` button.

```{figure} images/lab6-newsg.png
---
height: 400px
align: center
---
SmartGroup
```

Afterwards, click on the arrow icon  inside the `"+ Resource Type"` button and select `"IP / CIDRs"`.

```{figure} images/lab6-greenfieldneww4.png
---
height: 400px
align: center
---
Private Subnet
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>us-east-2-private-subnet</span>
- **IPs / CIDRs**: <span style='color:#479608'>10.0.1.0/27</span>

Before clicking on **SAVE**, delete the empty `"Virtual Machines"` additional condition.

```{figure} images/lab6-greenfieldneww45.png
---
height: 400px
align: center
---
New SG
```

#### 4.4.3 Create a new Rule

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab6-newrule10.png
---
align: center
---
New Rule
```

Insert the following parameters

- **Name**: <span style='color:#479608'>Egress-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>us-east-2-private-subnet</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>**All-Web**</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Enforcement**: <span style='color:#479608'>**Off**</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab6-new.png
---
align: center
---
Committing the new Rule
```

Click on the **Commit** button and the rule previously created will work in **_watch/test_** mode due to the fact that the `enforcement` was turn off.

```{important}
If the **Enforcement** slider is **On** (the default), the rule is enforced in the data plane. If the Enforcement slider is **Off**, the packets are only watched. This allows you to observe if the traffic impacted by this rule causes any inadvertent issues (such as traffic being dropped).
```

```{figure} images/lab6-newrule11.png
---
height: 150px
align: center
---
Egress-Rule
```

Now delete the **Greenfield-Rule**: 

- click on the **three dots** icon on the right-hand side of the Greenfield-Rule entry and then choose the `"Delete Rule"` option.

Do not forget to click on **Commit**.

```{caution}
The deletion of the Greenfield-Rule will also cause the deletion of the DefaultDenyAll, because the Egress-Rule was not enforced on the data path, which in turn means that there will be an `Invisible Deny Rule` installed on the bottom.
```

```{figure} images/lab6-newruledelete.png
---
height: 150px
align: center
---
Deletion of the Greenfield-Rule
```

```{figure} images/lab6-newruledeleted.png
---
height: 150px
align: center
---
Egress-Rule solely
```


- Launch again the following curl commands from the instance **_aws-us-east-2-spoke1-test2_**.

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

Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and you will see the corresponding **logs**.

```{figure} images/lab6-monitorpermit.png
---
height: 200px
align: center
---
Monitor
```

```{important}
However, on the SSH client, yiou will not see any outputs, this is because the Rule was not enforced in the Data Path, therefore the traffic is dropped.
```

```{figure} images/lab6-monitorpermit22.png
---
height: 200px
align: center
---
SSH client output
```

Go to **CoPilot > Security > Egress > Overview (default)**

Now you have finally the  **egress observability** with a full list of domains hit by the EC2 instance inside that private subnet.

```{figure} images/lab6-newrul12.png
---
height: 400px
align: center
---
Overview
```

Furthermore, go to **CoPilot > Security > Egress > Monitor** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east-2-spoke1 VPC_**.

```{figure} images/lab6-monitor.png
---
height: 250px
align: center
---
Monitor
```

You will get a granular Layer 7 visibility that allows you to get a good understanding of how the egress traffic has been consumed and also allows you to help make decisions on how to potentially optimize that.

## 5. ZTNA - Zero Trust Network Architecture
### 5.1 Create a New WebGroup

Let's move towards a posture where only specific egress domains are in place.

Go to **CoPilot > Groups > WebGroups** and click on `"+ WebGroup"` button.

```{figure} images/lab6-webgroup.png
---
align: center
---
+WebGroup
```

Create a **_WebGroup_** with the following parameters:

- **Name**: <span style='color:#479608'>two-domains</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'>www.aviatrix.com</span>
- **Domains/URLs**: <span style='color:#479608'>www.wikipedia.com</span>

Do not forget to click on **Save**.

```{figure} images/lab6-webgroup2.png
---
align: center
---
WebGroup creation
```

```{important}
The purpose of this **WebGroup** is to authorize traffic only towards both the Domains *`www.aviatrix.com`* and *`www.wikipedia.com`*, therefore the curl commands issued towards other Domains will be blocked.
```

## 5.2 Edit the Egress-Rule 
### 5.2.1 Create a new rule

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules**, click on the **pencil** button on the right-hand side of the `Egress-Rule`.

- Now remove the WebGroup `"All-Web"` and then select the WebGroup `"two-domains"`.
- Turn on the `Enforcement` knob.

Do not forget to click on **Save In Drafts** and then **Commit** your changes!

```{figure} images/lab6-webgroup234.png
---
align: center
---
Editing the Egress-Rule
```

```{figure} images/lab6-webgroup2345.png
---
align: center
---
Commit the changes
```

```{important}
- **Anywhere (0.0.0.0/0)** = Default Route 
- **Publlic Internet** = NON-RFC1918 routes
```

Now you have effectively activated the **ZTNA** approach.

```{note}
After committing the changes, the Egress-Rule will be applied to the data path and moreover, the `DefaultDenyAll` rule will show up at the bottom.
```

```{figure} images/lab6-webgroup23456.png
---
align: center
---
ZTNA
```

### 5.2.2 Test the new rule

Go to **CoPilot > Security > Egress > Monitor** and select the **_Live View_** from the `"Time Period"` field, then select the **_aws-us-east-2-spoke1_** VPC from the `"VPC/VNets"` drop-down window.

```{figure} images/lab6-newview.png
---
height: 400px
align: center
---
Select the VPC
```



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

Only the first two curl commands will be successful!

```{figure} images/lab6-newpic.png
---
align: center
---
Curl commands
```

You will notice almost instanteously that only **_www.aviatrix.com_** and **_www.wikipedia.com_** are allowed. Traffic towards **_www.espn.com_** and **_www.football.com_** will match the new `"DefaultDenyAll"` rule, therefore towards traffic the last two domains will be immediately dropped.

```{figure} images/lab6-liveview.png
---
height: 250px
align: center
---
Permit
```

## 5.3 IDS
### 5.3.1 Create a New Rule

Let's now test the **_IDS_** feature (i.e. Intrusion Detection System). 

Go to **CoPilot > Security > Distributed Cloud Firewall > Rules** and click on the `"+ Rule"` button.

Create a new DCF Rule with the following parameters:

- **Name**: <span style='color:#479608'>Inspect-DNS</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
- **Intrusion Detection (IDS)**: <span style='color:#479608'>**On**</span>

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
height: 250px
align: center
---
Commit
```

### 5.3.2 Prepare the simulator

- From the **_aws-us-east-2-spoke1-test2_** instance, launch the following **two** commands.

```bash
sudo su -
```

```{note}
You will be asked to type again the student password!

```{figure} images/lab6-password.png
---
align: center
---
Root PWD
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

The last command will show up a **_simulator_** from whom you will be able to launch an attack for testing the `"Suricata IDS"`.

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
height: 250px
align: center
---
Edit existing rule
```

Insert the following parameters and do not forget to click on **Save In Drafts**:

- **Protocol**: <span style='color:#479608'>UDP</span>
- **Port**: <span style='color:#479608'>53</span>

```{figure} images/lab6-dns.png
---
align: center
---
Modify the rule
```

Now click on the **Commit** button.

```{figure} images/lab6-commit3.png
---
height: 250px
align: center
---
Commit
```

From the EC2 instance **_aws-us-east-2-spoke1-test2_**, type **5** and click enter for launching a malicious attack, specifically the attack will try to establish a connection towards a TOR server.

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
height: 150px
align: center
---
Detected Intrusions
```

Click on any **Timestamps** to get additional insight on that specific attack.

```{figure} images/lab6-final.png
---
height: 300px
align: center
---
Additional insights
```

```{note}
The indicator is showing clearly that you tried reaching out to Google DNS and then querying for a **TOR** domain!
```

After this lab, this is how the overall topology would look like:

```{figure} images/lab6-finaltopo.png
---
height: 400px
align: center
---
Final Topology for Lab 5
```