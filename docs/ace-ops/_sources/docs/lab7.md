# Lab 7 - Distributed Cloud Firewall

## 1. SCENARIO

ACE’s environment has been split up in two SmartGroups: **BU1** and **BU2**. Under the hood, there is a flat routing domain, due to the connection policy that merged the two network domains.

```{figure} images/ops-newlab-000.png
---
align: center
---
Initial lab 7 Scenario Topology
```

Let’s enable the **Distributed Cloud Firewall** to secure and govern the traffic runtime within CNSF.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and then click on the `"Enable Distributed Cloud Firewall"` button, then to the subsequent `"Begin Using Distributed Cloud Firewall"` button, and last but not least to the `"Begin"` button
```

```{figure} images/lab7-enabledcf01.png
---
align: center
---
Enable DCF message
```

```{figure} images/lab7-enabledcf0222.png
---
align: center
---
Begin using DCF message
```

```{figure} images/lab7-enabledcf0322.png
---
align: center
---
Begin
```

The Aviatrix Controller has applied a `Greenfield-Rule` that allows all traffic; you should see this immediately.

```{figure} images/lab7-enabledcf0411.png
---
align: center
---
Greenfield-Rule
```

- Enable the `"Logging"` on the Greenfield-Rule.

```{tip}
Click on the Actions icon to the right of the **Greenfield-Rule** end select "Turn On Logging".
```

```{figure} images/lab7-editgreen.png
---
height: 150px
align: center
---
Edit the Greenfield-Rule
```

Remember to click on **Commit** to apply the changes to the Data Plane.

```{figure} images/lab7-lastone.png
---
height: 150px
align: center
---
Commit
```

Now enable the **Zero Trust Network Access** (ZTNA) inside the CNSF.

- Create an **Explicit Deny** rule:

Click on the `"+ Rule"` button.

```{figure} images/ops-newlab-01.png
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
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Deny**</span>

Do not forget to click on **Save In Drafts**.

```{figure} images/ops-newlab-02.png
---
align: center
---
ExplicitDenyAll
```

Do not forget to click on **Commit**.

```{figure} images/ops-newlab-03.png
---
align: center
---
Commit
```

- Log in to **BU1 Frontend**, then SSH to the **BU1 DB** private IP address.

```{figure} images/ops-newlab-04.png
---
align: center
---
SSH fails
```

The SSH attempt will fail because East-West traffic is blocked by the **_ExplicitDenyAll_** rule.

Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**. The SSH attempt initiated earlier was blocked by the `Explicit DenyAll` rule and the traffic was dropped.

```{figure} images/ops-newlab-05.png
---
align: center
---
Monitor
```

Define the following Distributed Cloud Firewall (DCF) rules to re-establish and govern East-West traffic:

  - **Intra-rule**: allow SSH <span style='color:orange'>**within**</span> BU1
  - **Intra-rule**: allow ICMP <span style='color:orange'>**within**</span> BU1
  - **Inter-rule**: allow ICMP <span style='color:lightblue'>**from**</span> BU1 **to** BU2
  - **Inter-rule**: allow SSH <span style='color:lightblue'>**from**</span> BU2 **to** BU1

## 2. CHANGE REQUEST

- Create an intra-rule that allows SSH **within** BU1 and then verify that SSH is permitted among BU1’s instances. 
  Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

```{figure} images/ops-newlab-06.png
---
align: center
---
intra-rule
```

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/ops-newlab-07.png
---
align: center
---
Commit
```

- Retry to SSH to the **BU1 DB** from the BU1 Frontend; this time the operation will be accomplished!

```{figure} images/lab8-sshok.png
---
align: center
---
SSH is ok within BU1
```

```{figure} images/lab8-intrassh-bu1.png
---
height: 400px
align: center
---
Intra-SSH BU1
```

- Check the **Logs** generated after the action carried out before.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall > Monitor**
```

```{figure} images/lab8-monitornew.png
---
align: center
---
Monitor: Logs
```

- Now terminate the SSH session with the **BU1 DB**, typing `"exit"`, and issue the **ping** command towards the BU1 DB from the BU1 Frontend. The ping will not work!

```{figure} images/lab8-pingbu1.png
---
align: center
---
Ping will fail
```

- Check the **Logs** in the `Monitor` section again. The ICMP attempt was blocked by the **_ExplicitDenyAll_** rule.

```{figure} images/lab8-denyicmp.png
---
align: center
---
Monitor: Denied ICMP
```

- Create another intra-rule that allows ICMP **within** BU1 and then verify that ICMP is permitted among BU1’s instances. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

```{figure} images/lab8-icmprulebu111.png
---
align: center
---
New intra-rule
```

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-icmprulebu1.png
---
align: center
---
Commit
```

- The ping will succeed this time thanks to the newly created intra-rule that permits it.

```{figure} images/lab8-icmprulebu1ok.png
---
align: center
---
Ping will be ok!
```

- Once again, inspect the **Monitor** section to find out immediately the corresponding *logs* of the action carried out before.

```{figure} images/lab8-pingpermitted.png
---
align: center
---
Monitor: successful logs
```

```{figure} images/lab8-intrasshicmp-bu1.png
---
height: 400px
align: center
---
Intra-SSH-ICMP BU1
```

- Let's try to ping the BU2 Mobile App from the BU1 Frontend. The ping will fail due to the absence of an **inter-rule**.

```{figure} images/lab8-pingbu1bu2.png
---
align: center
---
Ping will fail
```

- This time create an inter-rule that allows ICMP **from** BU1 **towards** BU2. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-icmp-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

```{figure} images/lab8-penultimaterule11.png
---
height: 200px
align: center
---
Inter-Rule
```

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-penultimaterule.png
---
align: center
---
DCF Rules List
```

- Retry to ping the **BU2 Mobile App** from the BU1 Frontend. The ping will be successful thanks to the inter-rule!

```{figure} images/lab8-penultimateping.png
---
align: center
---
Ping from BU1 to BU2
```

```{figure} images/lab8-intericmp-bu1bu2.png
---
height: 400px
align: center
---
Inter-ICMP BU1 to BU2
```

- SSH to the **BU2 Mobile App**’s public IP, then try SSH to the **BU1 Frontend**’s private IP. The second attempt will fail.

```{figure} images/lab8-bu2sshbu1.png
---
align: center
---
SSH from BU2 to BU1 will fail
```

- Create another inter-rule allowing SSH **from** BU2 **to** BU1, and enable **"Logging"** for auditing.

```{tip}
Navigate to **CoPilot > Security > Distributed Cloud Firewall** and click on **"+ Rule"**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU2</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

```{figure} images/lab8-lastrule111.png
---
height: 200px
align: center
---
"inter-ssh-bu2-bu1" Rule
```

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-lastrule.png
---
height: 200px
align: center
---
"inter-ssh-bu2-bu1" Rule
```

- Try issuing the SSH command from the **BU2 Mobile App** to the **BU1 Frontend**. This time it should work smoothly.

```{figure} images/lab8-lastssh.png
---
align: center
---
SSH works from BU2 to BU1
```

```{figure} images/lab8-interssh-bu2bu1.png
---
height: 400px
align: center
---
Inter-SSH BU2 to BU1
```

## 3. CHANGE REQUEST

- Now before completing the lab, remove the `Inspection Policy` from the Transit GW in AWS.

The **`Aviatrix Distributed Cloud Firewall`** is enabled across the CNSF. As a result, you can decommission the _ngfw_ in AWS.

```{tip}
Navigate to **CoPilot > Security > FireNet > FireNet Gateways** and click on the **_ace-aws-eu-west-1-transit1_** GW, then click on `Policy`, select the two Spoke VPCs in AWS and choose the `Remove` action.

Traffic will be not be sent towards the FW anymore. Th DCF will perform the firewalling very close to the source!
```{figure} images/lab8-manageatt.png
---
align: center
---
FireNet cfg
```

```{figure} images/lab8-lastone.png
---
align: center
---
Remove the inspection Policy
```

`Congratulations on completing all labs and establishing a comprehensive set of Distributed Cloud Firewall (DCF) rules across your hybrid cloud infrastructure`.
