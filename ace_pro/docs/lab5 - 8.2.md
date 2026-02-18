# Lab 5 - AVIATRIX CLOUD FIREWALL

## 1. Objective

In this lab, we will demonstrate how to enable the `Aviatrix Cloud Firewall`, a built-in service within the Aviatrix *Cloud Native Security Fabric (CNSF)*.

The Aviatrix Cloud Firewall enhances cloud security with enterprise-grade NAT, centralized management, and AI-driven threat detection. It offers comprehensive visibility and control, ensuring cost efficiency with a flat-rate billing model. Seamless integration and automated deployment make it ideal for scalable, resilient cloud environments.

In this lab, we will identify the _target VPC_. Of course, the selected VPC must have at least a subnet associated witho a **Private Routing table** (i.e., without a default route pointing to the IGW). The Controller will reroute the traffic through the Aviatrix Spoke Gateway. The Aviatrix Cloud Firewall provides enhanced visibility and control, effectively replacing the **CSP Native NAT Gateways**.

## 2. Topology

Let’s pinpoint the right candidate VPC where it would be possible to enable **Egress Control**.

```{figure} images/lab6-initialtopology.png
---
height: 400px
align: center
---
Lab 6 Initial Topology
```

The VPC **aws-us-east-2-spoke1** has a private subnet in its environment, whereby the Egress Control can be activated in this specific VPC.

At the moment, this private subnet is leveraging the **AWS NAT Gateway** to reach the Internet Public Zone.

```{figure} images/lab6-privateaws.png
---
align: center
---
Private Subnet and AWS NAT GW
```

- Explore the **Routing Table** to see the private subnet where the **test2** instance resides.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east-2-spoke1_** GW. Then, click on the **VPC/VNet Route Tables** tab and select the following Private RTB from the `Route Table` field: **aws-us-east-2-spoke1-Private-<span style='color:#33ECFF'>1</span></summary>-us-east-2a-rtb**.
```

```{figure} images/lab6-spokegw.png
---
align: center
---
Select the Spoke GW in US-EAST-2
```

```{figure} images/lab6-vpc.png
---
height: 400px
align: center
---
Check the private RTB
```

You will notice that this private RTB has its own **CIDR** pointing to local, the three **RFC1918** routes pointing to the Aviatrix Spoke Gateway (injected by the Aviatrix Controller) and a default route pointing to the AWS NAT Gateway. 

With this scenario, the EC2 instance can reach the **Internet Public Zone**; however, the AWS NAT Gateway is limited and provided by the logs and flows, which might not be adequate in an Enterprise scenario. Moreover, it doesn't provide any visibility.

## 3. Preliminary Connectivity Testing

Let's verify what traffic the instance in the private subnet is generating!

### 3.1 Preliminary Connectivity Testing Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select both **_aws-us-east-2-spoke1-test2_**. 

```{figure} images/lab6-gatus500.png
---
align: center
---
aws-us-east-2-spoke1-test2
```

The private workload is generating traffic to **four** domains:
1) aviatrix.ai
2) www.espn.com
3) www.football.com
4) www.wikipedia.org

```{figure} images/lab6-gatus501.png
---
height: 400px
align: center
---
Egress traffic through the AWS NAT GW
```

### 3.2 Preliminary Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

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
Once again, the **_aws-us-east-2-spoke1-test2_** instance resides within a private subnet; therefore, you first need to SSH into a public workload that will be used as a jumphost.
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

Now, launch the following curl commands:

```bash
curl -I https://aviatrix.ai
```
```bash
curl -I https://www.espn.com
```
```bash
curl -I https://www.football.com
```
```bash
curl -I https://www.wikipedia.org
```

```{figure} images/lab6-curl.png
---
height: 400px
align: center
---
curl command-01
```

```{figure} images/lab6-curl02.png
---
height: 400px
align: center
---
curl command-02
```

```{figure} images/lab6-curl03.png
---
height: 400px
align: center
---
curl command-03
```

```{figure} images/lab6-curl04.png
---
height: 400px
align: center
---
curl command-04
```

## 4. Aviatrix Cloud Firewall
### 4.1 Enable the Cloud Secure Egress

Now let's enable the cloud secure egress within the VPC that is hosting the **_aws-us-east-2-spoke1-test2_** instance.

```{note}
Navigate to **CoPilot > Security > Egress > Egress VPC/VNets** and click on `"Enable Local Egress on VPC/VNets"`, then select the **_aws-us-east-2-spoke1_** VPC and click on **Add**.
```

```{figure} images/lab6-egress79.png
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

```{figure} images/lab6-vpcegress97.png
---
align: center
---
Click Add
```

### 4.2 Inspect the Private RTB

Upon enabling Local Egress on the specified VPC, the Aviatrix Controller will immediately execute two actions on the specified VPC:

- Injecting a **Default Route**: this route is added exclusively to the Private Route Tables (RTBs). Public Route Tables (RTBs) remain unaffected and will continue to have their default route pointing towards the native Cloud Service Provider (CSP) Internet Gateway (**IGW**).

- Enabling **Single IP SNAT** on VPC Spoke Gateway: Source Network Address Translation (SNAT) will be enabled on the VPC spoke gateway using a single IP address.

At this point, the Aviatrix  performs the same functions as the **CSP NAT Gateway**.

- Verify its presence in any Private RTBs inside the **_aws-us-east-2-spoke1_** VPC.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways** and select the **_aws-us-east-2-spoke1 GW_**, then click on the **VPC/VNet Route Tables** tab and then select the following Private RTB from the `Route Table` field: **aws-us-east-2-spoke1-Private-<span style='color:#33ECFF'>1</span></summary>-us-east-2a-rtb**.
```

```{figure} images/lab6-defaultroute.png
---
align: center
---
Default route has been modified!
```

```{important}
The `Default Route` is now pointing to the Aviatrix Spoke Gateway.
```

### 4.3 Enable DCF

You have successfully activated `Cloud Secure Egress Control` without impacting any resources within the private subnet. The default route inserted into the Private Routing Tables now points to the Spoke gateway.

To collect NetFlow data and enforce firewall policies, please enable the `Aviatrix Cloud Firewall`. This will enable monitoring of the private subnet’s traffic and provide visibility into the domains accessed from it.

Action:
- Enable the **Distributed Cloud Firewall**.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and click on `"Begin using Distributed Cloud Firewall"` followed by `"Begin"`.
```

```{figure} images/lab6-activate8.2.png
---
align: center
---
Begin Using Distributed Cloud Firewall
```

```{figure} images/lab6-newjoe244.png
---
align: center
---
Begin
```

After enabling the Distributed Cloud Firewall feature, a **Default Action Rule** is created, within the **Post Rules Policy List (System)**. This is a system rule used to enforce zero trust principles by controlling how traffic is handled in the absence of explicit rules:

- `Default Action Rule` = ALLOW EVERYTHING

```{figure} images/lab6-greenfield568.2.png
---
height: 250px
align: center
---
Default Action Rule automatically injected by the Controller
```

Now, click **Ruleset** and select the `V1 Policy List`.

```{figure} images/lab6-greenfield56668.2.png
---
align: center
---
V1 Policy List
```

- Create a new rule clicking on the **"+ Rule"** button.

```{figure} images/lab6-greenfield566618.2.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>Greenfield-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab6-greenfield5666128.2.png
---
align: center
---
Greenfield-Rule
```

```{tip}
**Logging Options**:

- _Log at Start_: Captures session initiation details

- _Log at End_: Captures session termination details, including duration and traffic size

- _Log at Start & End_: Provides full lifecycle visibility

- _Off_: Disables logging for the rule
```

Click on **Commit**.

```{figure} images/lab6-greenfield56661238.2.png
---
align: center
---
Commit
```

## 5. Define a new SmartGroup 

At this stage, the **SmartGroup** feature enables the categorization of the `test2` instance within the aws-us-east-2-spoke1 VPC.

### 5.1 Identify the subnet where the private workload resides

First and foremost, you have to identify the **private subnet** where the **_aws-us-east-2-spoke1-test2_** instance resides.

```{figure} images/lab6-greenfieldneww.png
---
height: 400px
align: center
---
Private Subnet
```

Navigate to **CoPilot > Cloud Resources > Cloud Workloads > Virtual Machines** and search for the **_`aws-us-east-2-spoke1-test2`_** instance in the search field on the right-hand side.

From the results, you need to identify the `Availability Zone`.

```{figure} images/lab6-greenfieldneww2.png
---
height: 300px
align: center
---
AZ
```

Now that you know in which `Availability Zone` the private workload resides, you need to navigate to **CoPilot > Cloud Resources > Cloud Networks > VPC/VNets & Subnets** and filter based on the **_us-east-2a_** VPC.

Identify the spoke VPC, reveal its subnets, find the `private subnet`, and copy the `CIDR block` from the IP address CIDR column.

```{figure} images/lab6-greenfieldneww31.png
---
height: 300px
align: center
---
Private Subnet
```

### 5.2 Create an ad-hoc SmartGroup

Navigate to **CoPilot > Groups** and click on the `"+ SmartGroup"` button.

```{figure} images/lab6-newsg.png
---
height: 400px
align: center
---
SmartGroup
```

Now, click on the arrow icon  inside the `"+ Resource Type"` button and select `"IP / CIDRs"`.

```{figure} images/lab6-greenfieldneww4.png
---
height: 400px
align: center
---
Resource Selection
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>us-east-2-private-subnet</span>
- **IPs / CIDRs**: <span style='color:#479608'>10.0.1.0/27</span>

Before clicking on **SAVE**, delete the empty `"Virtual Machines"` additional condition.

```{figure} images/lab6-greenfieldneww45.png
---
align: center
---
New SG
```

## 6. Create a new DCF Rule

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies (default tab)** and create a new rule clicking on the `"+ Rule"` button.

```{figure} images/lab6-newrule108.2.png
---
align: center
---
New Rule
```

Enter the following parameters:

- **Name**: <span style='color:#479608'>Egress-Rule</span>
- **Source Smartgroups**: <span style='color:#479608'>us-east-2-private-subnet</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public Internet</span>
- **WebGroups**: <span style='color:#479608'>**All-Web**</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Log**: <span style='color:#479608'>**At Start & End**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab6-new8.2.png
---
align: center
---
Saving the new Rule
```

```{important}
`All-Web` _WebGroup_ = this is an "allow-all" WebGroup that you must select in a Distributed Cloud Firewall rule if you do not want to limit the Internet-bound traffic for that rule, but you still want to log the FQDNs that are being accessed.

Ultimately , this Webgroup will limit Internet traffic solely for **HTTP** and **HTTPS** protocols.
```

Click on the **Commit** button, and the rule previously created will be enforced in to the `Data Path`.

```{figure} images/lab6-newrule118.2.png
---
height: 250px
align: center
---
Egress-Rule
```

- Now, create an **Explicit Deny** rule and position it below the Egress Rule but above the Greenfield Rule to activate the `Zero Trust control`.

Click on the `"+ Rule"` button.

```{figure} images/lab6-newrule1008.2.png
---
align: center
---
New Rule
```

Insert the following parameters:

- **Name**: <span style='color:#479608'>ExplicitDenyAll</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere(0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>ON</span>
- **Action**: <span style='color:#479608'>**Deny**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>Egress-Rule</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab6-newrule10018.2.png
---
align: center
---
ExplicitDenyAll
```

Do not forget to click on **Commit**.

```{figure} images/lab6-newrule1001108.2.png
---
align: center
---
Commit
```

## 7. Connectivity Testing after enabling the Aviatrix Cloud Firewall

Let's verify how the TCP traffic generated by the Gatus got affected by the activation of the Aviatrix Cloud Firewall.

### 7.1 Connectivity Testing Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select both **_aws-us-east-2-spoke1-test2_**. 

```{figure} images/lab6-gatus500.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test2
```

The private workload is still generating traffic to those four domains.

```{tip}
If you want to expedite the results on the Gatus dashboard, feel free to adjust the timeout to **10 seconds** in the bottom left-hand corner.
```

```{figure} images/lab6-gatus5012.png
---
height: 400px
align: center
---
Egress traffic is now going through the Aviatrix Cloud Firewall embedded on the Spoke GW
```

On the other hand, the ICMP traffic tests will gradually start to fail, with the exception of the test directed towards **aws-us-east-2-spoke1-test1**. This exception is due to the intra-VPC traffic.

```{figure} images/lab6-gatuslab501.png
---
height: 400px
align: center
---
ICMP is ok only with aws-us-east-2-spoke1-test1
```

### 7.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

- From your SSH client connected to the **_aws-us-east-2-spoke1-test2_** instance, please re-run the following curl commands:

```bash
curl -I https://aviatrix.ai
```
```bash
curl -I https://www.espn.com
```
```bash
curl -I https://www.football.com
```
```bash
curl -I https://www.wikipedia.org
```

```{important}
The **_aws-us-east-2-spoke1-test2_** instance is located in a private subnet. Therefore, to connect to it, you must first SSH into **_aws-us-east-2-spoke1-test1_**, and from there, SSH into test2. Remember, the Distributed Cloud Firewall rules are enforced on the Spoke Gateway, so intra-VPC traffic is not affected by these rules unless `Security Group Orchestration` is enabled.
```

```{figure} images/lab6-curl.png
---
height: 400px
align: center
---
curl command-01
```

```{figure} images/lab6-curl02.png
---
height: 400px
align: center
---
curl command-02
```

```{figure} images/lab6-curl03.png
---
height: 400px
align: center
---
curl command-03
```

```{figure} images/lab6-curl04.png
---
height: 400px
align: center
---
curl command-04
```

### 7.3 Egress Analyze

Navigate to **CoPilot > Security > Egress > Analyze**, and you will see the visibility of the domain hits.

```{figure} images/lab6-newrul12.png
---
height: 500px
align: center
---
Analyze
```

You now have comprehensive **Cloud Secure Egress observability**, including a detailed list of all domains accessed by the EC2 instance within that private subnet.

Furthermore, navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east-2-spoke1 VPC_**.

```{figure} images/lab6-monitor.png
---
height: 400px
align: center
---
Monitor
```

You will get a granular Layer 7 visibility that allows you to get a good understanding of how the egress traffic has been consumed and also allows you to help make decisions on how to potentially optimize that.

## 8. ZTNA - Zero Trust Network Access

We have now enabled the `Zero Trust Network Access` control, which default-denies all access and only permits explicitly authorized connections.

### 8.1 Create a New WebGroup

Let's move towards a posture where only specific egress domains are in place.

Navigate to **CoPilot > Groups > WebGroups** and click on `"+ WebGroup"` button.

```{figure} images/lab6-webgroup.png
---
align: center
---
+WebGroup
```

Create a **_WebGroup_** with the following parameters:

- **Name**: <span style='color:#479608'>two-domains</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'>aviatrix.ai</span>
- **Domains/URLs**: <span style='color:#479608'>www.wikipedia.org</span>

Do not forget to click on **Save**.

```{figure} images/lab6-webgroup2.png
---
align: center
---
WebGroup creation
```

```{important}
The purpose of this **WebGroup** is to permit traffic exclusively to the domains *`aviatrix.ai`* and *`www.wikipedia.org`*.org. Consequently, any curl commands directed at other domains will be blocked.
```

### 8.2 Edit the Egress-Rule

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies**, click on the **pencil** button on the right-hand side of the `Egress-Rule`.

```{figure} images/lab6-webgroup23468.2.png
---
align: center
---
Pencil icon
```

- Now remove the WebGroup `"All-Web"` and then select the WebGroup `"two-domains"`.

Do not forget to click on **Save In Drafts** and then **Commit** your changes!

```{figure} images/lab6-webgroup2348.2.png
---
align: center
---
Editing the Egress-Rule
```

```{figure} images/lab6-webgroup23458.2.png
---
height: 250px
align: center
---
Commit the changes
```

```{important}
- **Anywhere (0.0.0.0/0)** = Represents all CIDR ranges or IP addresses. 
- **Publlic Internet** = Represents non-RFC 1918 IP ranges, or the public Internet
```

### 8.3 Connectivity Testing after enabling the Aviatrix Cloud Firewall

Let's verify whether the Egress rule is now effectively blocking traffic to both **_www.espn.com_** and **_www.football.com_**.

#### 8.3.1 Connectivity Testing Using the Gatus App

Navigate to your POD Portal, locate the `Gatus widget`, and select both **_aws-us-east-2-spoke1-test2_**. 

```{figure} images/lab6-gatus500.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test2
```

You will gradually notice that the TCP traffic to **_www.espn.com_** and **_www.football.com_** will start being blocked, which will cause it to turn red.

```{figure} images/lab6-gatus555.png
---
height: 400px
align: center
---
aws-us-east-2-spoke1-test2
```

#### 8.3.2 Connectivity Testing Using the SSH Client <span style='color:#33ECFF'>(BONUS)</span></summary>

- Re-launch the following curl commands from the SSH session opened with the **_aws-us-east-2-spoke1-test1_** instance:

```bash
curl -I https://aviatrix.ai
```
```bash
curl -I https://www.espn.com
```
```bash
curl -I https://www.football.com
```
```bash
curl -I https://www.wikipedia.org
```

```{figure} images/lab6-curl.png
---
height: 450px
align: center
---
curl command-01
```

```{figure} images/lab6-curl021.png
---
align: center
---
curl command-02
```

```{figure} images/lab6-curl031.png
---
align: center
---
curl command-03
```

```{figure} images/lab6-curl04.png
---
height: 400px
align: center
---
curl command-04
```

## 9. FQDN Monitor

Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and from the `"VPC/VNets"` drop-down window, select the **_aws-us-east-2-spoke1 VPC_**.

Enter the word **"denied"** in the `Search` field to identify the domains that were blocked after attaching the WebGroup `two-domains` to the rule named `Egress-Rule`.

```{figure} images/lab6-monitor989.png
---
height: 400px
align: center
---
Action Denied
```

## 10. Removal of the AWS NAT Gateway

With the successful deployment of the `Aviatrix Cloud Firewall` within the Cloud Native Security Fabric (CNSF), you can now eliminate the need for the AWS NAT Gateway.

### 10.1 AWS Console

Login to **AWS console**</a>. Refer to your pod info for login information (<ins>Please always refer to your personal POD Portal</ins>).

```{figure} images/lab5-deleteaws.png
---
align: center
---
AWS NAT GW
```

```{figure} images/lab5-newone.png
---
align: center
---
AWS URL and credentials
```

```{figure} images/lab5-awsconsole.png
---
align: center
---
AWS console
```

Change the region to **Ohio** and invoke **VPC** service.

```{figure} images/lab5-region22.png
---
height: 300px
align: center
---
Change the Region
```

```{figure} images/lab5-region01.png
---
height: 300px
align: center
---
Invoke the VPC service
```

Now click on the `NAT Gateways` section in the Navigation Panel on the right-hand side.

```{figure} images/lab5-region012.png
---
height: 450px
align: center
---
NAT gateways section
```

- Select the **_aws-region-2-spoke1-nat_** NAT Gateway, then click on thwe `"Actions"` button on the right-hand side and choose the option `"Delete NAT gateway"`. Afterwards, type `"delete"` to confirm the deletion of the AWS NAT Gateway and click on **Delete**.

```{figure} images/lab5-region99.png
---
height: 300px
align: center
---
Selection of the AWS NAT GW
```

```{figure} images/lab5-region9991.png
---
height: 300px
align: center
---
Actions
```

```{figure} images/lab5-region999.png
---
height: 300px
align: center
---
Deletion of the AWS NAT GW
```

After completing this lab, the overall topology will appear as follows:

```{figure} images/lab6-finaltopo.png
---
height: 400px
align: center
---
Final Topology for Lab 5
```

```{caution}
At the end of this lab <ins>the **East-West traffic** wll be blocked</ins>, due to the current configuration of the **Distributed Cloud Firewall**!
```