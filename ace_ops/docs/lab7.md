# Lab 7 - Aviatrix Cloud Firewall

In this lab you will activate the `Aviatrix Cloud Firewall`, a cloud-native security solution that provides distributed firewalling, traffic control, and network segmentation. Unlike centralized traditional firewalls, it simplifies security management across multi-cloud environments, automates policy enforcement, and can operate in either learning or enforcement modes, offering flexibility for securing both egress and east–west traffic.

## 1. SCENARIO

The database VNet owner requires that prohibited egress traffic be dropped. Please review egress traffic for **BU1-DB** and **BU2-DB** and ensure any restricted destinations are blocked.

```{figure} images/lab7-topology.png
---
height: 400px
align: center
---
Lab 7 Scenario Topology
```

In your POD Portal, open the Gatus Dashboard for **BU1-DB** and **BU2-DB** and examine the `External` section to identify prohibited traffic.

```{figure} images/lab7-newgatus01.png
---
height: 400px
align: center
---
BU1-DB Gatus
```

```{figure} images/lab7-newgatus02.png
---
height: 400px
align: center
---
BU2-DB Gatus
```

```{important}
Even though the two virtual machines are deployed in private subnets, they can still access the public internet because the `Egress` feature was enabled when the POD was launched.

When `Local Egress` is enabled, **SNAT** is automatically turned on. As a result, all outbound traffic from the Spoke VPC/VNet is translated to use the gateway’s public IP address. In addition, the VPC/VNet’s default route (**0.0.0.0/0**) is updated to point to the Spoke gateway.
```

- Navigate to **CoPilot > Security > Egress > Egress VPC/VNets**. You’ll see that both **_ace-azure-east-us-spoke1_** and **_ace-azure-east-us-spoke2_** VNets are already configured with _Local Egress_ enabled.

```{figure} images/lab7-newgatus03.png
---
height: 250px
align: center
---
Cloud Secure Egress
```

```{important}
This action adds a **default route** to each private route table in the Azure spoke VNets. The routes direct traffic to the Aviatrix Spoke Gateways.

To verify, go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways**, select a spoke gateway (for example, ace-azure-east-us-spoke2), then open the `VPC/VNet Route Tables` tab. From the Route Table drop-down, select any private route table.
```{figure} images/lab7-defaultroute.png
---
align: center
---
Default Route injected by the AVX Controller
```

## 2. CHANGE REQUEST

### 2.1 Distributed Cloud Firewall

To control and enforce egress traffic, you must enable the `Distributed Cloud Firewall`.

#### 2.1.1 DCF - Activation

- Enable the **Distributed Cloud Firewall** feature.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall**, click `Begin Using Distributed Cloud Firewall`, and then click `Begin` on the next screen.
```

```{figure} images/lab7-enabledcf01.png
---
align: center
---
Begin Using Distributed Cloud Firewall
```

```{figure} images/lab7-enabledcf0222.png
---
align: center
---
Begin
```

The Aviatrix Controller has pushed a `Default Action Rule` permitting all traffic; you should observe the impact right away.

```{figure} images/lab7-enabledcf0411.png
---
align: center
---
Default Action Rule
```

Now, click **Ruleset** and select the `V1 Policy List`.

```{figure} images/lab7-greenfield5666.png
---
align: center
---
V1 Policy List
```

#### 2.1.2 Ad-hoc Greenfield-Rule

- Create a new rule clicking on the **"+ Rule"** button.

```{figure} images/lab7-greenfield56661.png
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
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>Permit</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-greenfield566612.png
---
align: center
---
Greenfield-Rule
```

Click on **Commit**.

```{figure} images/lab7-greenfield5666123.png
---
align: center
---
Commit
```

### 2.2 Smart Groups

Create three Smart Groups to identify:
1) **BU1 Frontend**
2) **BU1 DB**
3) **BU2 DB**

- Navigate to **CoPilot > Groups > SmartGroups**, then click the `“+ SmartGroup”` button.

```{figure} images/lab7-SGnew00.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-aws-eu-west-1-spoke1-bu1-frontend</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-aws-eu-west-1-spoke1-bu1-frontend
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart31.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart311.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-aws-eu-west-1-spoke1-bu1-frontend_**

Create another Smart Group by clicking the `“+ SmartGroup”` button again.

```{figure} images/lab7-SGnew10.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU1-DB</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-azure-east-us-spoke1-bu1-db</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-azure-east-us-spoke1-bu1-db
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart41.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart411.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-azure-east-us-spoke1-bu1-db_**

Add another Smart Group by tapping the `“+ SmartGroup”` one more time.

```{figure} images/lab7-SGnew20.png
---
align: center
---
SmartGroup
```

Ensure these parameters are entered in the pop-up window `"Create SmartGroup"`:

- **Name**: <span style='color:#479608'>BU2-DB</span>
- **Matches all conditions (AND)/Name**: <span style='color:#479608'>ace-azure-east-us-spoke2-bu2-db</span>

```{figure} images/lab7-smart3.png
---
align: center
---
Name = ace-azure-east-us-spoke2-bu2-db
```

Before clicking **Save**, enable `"Preview"` to identify which instances match the condition.

```{figure} images/lab7-smart51.png
---
align: center
---
Resource Selection
```

```{figure} images/lab7-smart511.png
---
align: center
---
Preview
```

The CoPilot shows that there is one instance that perfectly matches the condition:

- **_ace-azure-east-us-spoke2-bu2-db_**

#### 2.2.1 Updating the Greenfield Rule

Navigate to **CoPilot > Security > Distributed Cloud Firewall**, and click the `pencil icon` beside the **Greenfield Rule** to update its configuration.

```{figure} images/lab7-green00.png
---
height: 200px
align: center
---
Edit the Greenfiled-Rule
```

Select the `"All-Web"` WebGroup from the **_WebGroups_** field, then click **"Save In Drafts"**.

```{figure} images/lab7-green01.png
---
align: center
---
All-Web
```

```{important}
Applying the `All-Web`  WebGroup to the Greenfield rule enables logging of the FQDNs accessed, specifically in the Egress section.
```

Proceed by enforcing the policy in the data plane and click **Commit**. From this point onward, East–West traffic will be blocked. However, you will still be able to intercept traffic generated by both databases, thanks to the WebGroup applied to the Greenfield-Rule.

```{figure} images/lab7-green02.png
---
align: center
---
Commit
```

#### 2.2.2 Egress - Analyze section

Now that the Distributed Cloud Firewall is enabled, we can gather useful insights from the Egress section. Navigate to **Security > Egress > Analyze**, and in `Top Domains`, identify the domains that are being blocked.

```{figure} images/lab7-green02aa.png
---
height: 450px
align: center
---
Analyze
```

Block the following domains and allow all others:

- www.ransomware.org

- www.botnet.com

- www.malware.com

### 2.3 DCF Rules

Let’s add two more rules to permit SSH traffic from the **BU1 Frontend** to the DB VMs.

Navigate to **Security > Distributed Cloud Firewall**, then click `"+ Rule"`.

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu1frontend-bu1db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **PORT**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-interssh001.png
---
align: center
---
inter-ssh
```

Before clicking **Commit**, continue adding a new DCF Rule by clicking `"+ Rule"`.

```{figure} images/lab7-green03.png
---
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu1frontend-bu2db</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-FRONTEND</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2-DB</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **PORT**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-interssh002.png
---
align: center
---
inter-ssh
```

You can now proceed and click **Commit** to enforce the policies in the data plane.

```{figure} images/lab7-interssh0023.png
---
align: center
---
Commit
```

```{caution}
Once again, the last two policies enable secure SSH access from the BU1 Frontend to the Azure database VMs hosted in private subnets. Because these VMs have no public IP addresses, direct SSH access from your laptop is not possible.
```

### 2.4 ZTNA

Now let’s enable the `Zero Trust` control by creating an explicit deny rule, which must be placed above the greenfield rule.

```{important}
**Zero trust** in networking is a security approach that “trusts no one by default,” whether inside or outside the network perimeter. Instead of assuming that users or devices inside the network are trustworthy, it requires strict verification for every access attempt and continuous evaluation of risk.
```

- Clicking the `"+ Rule"` button.

```{figure} images/lab7-green53.png
---
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>ExplicitDenyAll</span>
- **Source Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Destination Smartgroups**: <span style='color:#479608'>Anywhere (0.0.0.0/0)</span>
- **Protocol**: <span style='color:#479608'>Any</span>
- **Logging**: <span style='color:#479608'>**On**</span>
- **Action**: <span style='color:#479608'>**Deny**</span>
- **Place Rule**: <span style='color:#479608'>Below</span>
  - **Existing Rule**: <span style='color:#479608'>inter-ssh-bu1frontend-bu1db</span>
  
Do not forget to click on **Save In Drafts**.

```{figure} images/lab7-exdeall.png
---
align: center
---
ExplicitDenyAll
```

Click **Commit** to enforce the policies in the data plane.

```{figure} images/lab7-xdeall00.png
---
height: 300px
align: center
---
Commit
```

### 2.5 ACCESS FROM BU1 FRONTEND TO DATABASE VMs <span style='color:#33ECFF'>(BONUS)</span></summary>

Let’s verify the East–West traffic communication between the **BU1 Frontend** and the database VMs in Azure.

- SSH into the **BU1 Frontend** (retrieve its public IP address from either the `Topology` view or the `Cloud Workloads` section). Then SSH to the private IP address of **BU1 DB** first, and then to **BU2 DB**.

```{figure} images/lab7-sshbu2db.png
---
height: 400px
align: center
---
BU1 DB 
```

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor** and search for `inter-ssh-bu1frontend-bu1db`. You’ll see that the policy was triggered by the SSH attempt you initiated from **BU1 Frontend** to **BU1 DB**.

```{figure} images/lab7-test-logs02.png
---
height: 450px
align: center
---
inter-ssh-bu1frontend-bu1db
```

```{important}
Repeat the same check: try to SSH from **BU1 Frontend** to **BU2 DB**, then verify that the corresponding logs appear in the Monitor section of the **Distributed Cloud Firewall**.
```

### 2.6 WebGroup

- Create two **WebGroup** that match the _allowed domains_ identified previously.

```{tip}
Navigate to **CoPilot > Groups > WebGroups** and then click on `"+ WebGroup"`.
```{figure} images/lab7-webgroupp.png
---
align: center
---
WebGroup
```

Ensure these parameters are entered in the pop-up window `"Create WebGroup"`:

- **Name**: <span style='color:#479608'>bu1-allowed-domains</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'> www.digitalocean.com</span>
- **Domains/URLs**: <span style='color:#479608'> www.microsoft.com</span>
- **Domains/URLs**: <span style='color:#479608'> www.terraform.com</span>

then click on **Save**.

```{caution}
These are the domains identified before using the `Discovery Mode` (**= Greenfield-Rule + All-Web**).


```{figure} images/lab7-webgroup.png
---
height: 400px
align: center
---
bu1-allowed-domains
```

Let’s create another WebGroup—click the `"+ WebGroup"` button.

```{figure} images/lab7-webgrouppnew.png
---
align: center
---
A new WebGroup
```

Ensure these parameters are entered in the pop-up window `"Create WebGroup"`:

- **Name**: <span style='color:#479608'>bu2-allowed-domains</span>
- **Type**: <span style='color:#479608'>Domains</span>
- **Domains/URLs**: <span style='color:#479608'> cloud.google.com</span>
- **Domains/URLs**: <span style='color:#479608'> *.amazon.com</span>

then click on **Save**.

```{figure} images/lab7-webgroup11.png
---
height: 400px
align: center
---
bu2-allowed-domains
```

```{important}
The FQDN _www.aws.amazon.com_ can be matched using the wildcard `*.aws.amazon.com`, which covers any single-label subdomain under _aws.amazon.com_ (e.g., _foo.aws.amazon.com_), but not the apex _amazon.com_.
```

### 2.7 NEW DCF RULES

- Now create an `inter-rule` that allows **BU1 DB** to access the **Internet** only to reach the domains defined in the `bu1-allowed-domains `WebGroup.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Policies** and click on the `"+ Rule"` button.
```

```{figure} images/lab7-webgroup001.png
---
height: 400px
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:.

- **Name**: <span style='color:#479608'>inter-bu1db-internet</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public internet</span>
- **WebGroups**: <span style='color:#479608'>bu1-allowed-domains</span>
- **Protocol**: <span style='color:#479608'>Any</span>

- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule once again!

```{figure} images/lab7-lastrule.png
---
align: center
---
inter-bu1db-internet
```

```{figure} images/lab7-lastcommit.png
---
align: center
---
DCF rules list
```

- Create another `inter-rule` that allows **BU2 DB** to access the **Internet** only for the domains defined in the `bu2-allowed-domains` WebGroup.

```{figure} images/lab7-webgroup0012.png
---
height: 400px
align: center
---
+Rule
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-bu2db-internet</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1-DB</span>
- **Destination Smartgroups**: <span style='color:#479608'>Public internet</span>
- **WebGroups**: <span style='color:#479608'>bu2-allowed-domains</span>
- **Protocol**: <span style='color:#479608'>Any</span>

- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit**.

```{figure} images/lab7-lastrule22.png
---
align: center
---
inter-bu2db-internet
```

```{figure} images/lab7-lastcommit.png
---
align: center
---
DCF rules list
```

### 2.8 Verification Using Gatus

Review the Gatus dashboards for both **BU1 DB** and **BU2 DB**.

**BU1-DB** will be able to reach _www.digitalocean.com_, _www.microsoft.com_, and _www.terraform.com_. The domain _www.ransomware.org_ will be blocked.

```{figure} images/lab7-newgatus02new012.png
---
height: 400px
align: center
---
BU1-DB Gatus
```

**BU2-DB** will be able to access the following sites: _www.google.com_, _www.aws.amazon.com_, and _www.terraform.com_. The domains _www.botnet.com_ and _www.malware.net_ will be blocked.

```{figure} images/lab7-newgatus02new022.png
---
height: 400px
align: center
---
BU2-DB Gatus
```

### 2.9 Verification Using SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

With the BU1 SSH session active, please execute the curl commands toward the following domains:

```bash
www.digitalocean.com

www.microsoft.com

www.terraform.com

www.ransomware.org
```

With the BU2 SSH session active, please execute the curl commands toward the following domains:

```bash
www.aws.amazon.com

www.terraform.com

wwww.botnet.com 

www.malware.net
```

- Now check the logs within the **Egress** section!

```{tip}
Navigate to **CoPilot > Security > Egress > FQDN Monitor (Legacy)** and select the **ace-azure-east-us-spoke2** VNet, then filter by `"Allowed"`.
```

```{figure} images/lab7-last.png
---
height: 250px
align: center
---
ace-azure-east-us-spoke2
```

```{figure} images/lab7-lastlog.png
---
height: 450px
align: center
---
Allowed domains!
```

```{caution}
In the logs, you’ll probably see an allowed entry for `"api.snapcraft.io"`. This domain was allowed <ins>before </ins>applying the ExplicitDenyAll rule and the specific inter-rule that uses the `ubuntu-update` WebGroup.
```

You have successfully applied Secure Egress Control, leveraging both the `Egress` feature and the `Distributed Cloud Firewall` policy.
