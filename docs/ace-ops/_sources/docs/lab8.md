# Lab 8 - Distributed Cloud Firewall

## 1. SCENARIO

ACE’s environment has been split up in two SmartGroups: **BU1** and **BU2**. Under the hood, there is a flat routing domain, due to the connection policy that merged the two network domains.

Furthermore, four DCF rules have been already applied so far.

```{figure} images/lab8-initialrule.png
---
height: 200px
align: center
---
Existing DCF rules
```

- The BU1 Frontend has raised a complaint that is not able to use the SSH protocol <span style='color:orange'>**within**</span> BU1.

- The BU1 Frontend has raised a complaint that is not able to use the ICMP protocol <span style='color:orange'>**within**</span> BU1.

- The BU1 Frontend has raised a complaint that is not able to use the ICMP protocol <span style='color:lightgreen'>**towards**</span> BU2.

- The BU2 Mobile App has raised a complaint that is not able to use the SSH protocol <span style='color:lightgreen'>**towards**</span> BU1.

You have been engaged to create the following **_four_** new additional rules:

- **Intra-rule**: allow SSH <span style='color:orange'>**within**</span> BU1
- **Intra-rule**: allow ICMP <span style='color:orange'>**within**</span> BU1
- **Inter-rule**: allow ICMP <span style='color:lightblue'>**from**</span> BU1 **to** BU2
- **Inter-rule**: allow SSH <span style='color:lightblue'>**from**</span> BU2 **to** BU1

```{figure} images/lab8-topology.png
---
height: 400px
align: center
---
Initial lab 8 Scenario Topology
```

## 2. CHANGE REQUEST

- SSH on the **BU1 Frontend** and try to SSH to any other instances in BU1.
  - SSH fails as expected.

```{figure} images/lab8-sshbu1.png
---
align: center
---
SSH fails within BU1
```

- Create an intra-rule that allows SSH **within** BU1 and then verify that SSH is permitted among BU1’s instances. 
  Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-rule01.png
---
height: 200px
align: center
---
Another intra-rule
```

- Retry to SSH to the BU1 Analytics from the BU1 Frontend; this time the operation will be accomplished!

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

- Check the Logs generated after the action carried out before.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall > Monitor**
```

```{figure} images/lab8-monitornew.png
---
height: 200px
align: center
---
Monitor: Logs
```

- Now terminate the SSH session with the BU1 Analytics, typing `"exit"`, and issue the **ping** command towards the BU1 Analytics from the BU1 Frontend. The ping will not work!

```{figure} images/lab8-pingbu1.png
---
align: center
---
Ping will fail
```

- Check again the **Logs** from the `Monitor` section. The ICMP attempt hit the **_Explicit-Deny-Rule_**, whereby the action was denied!

```{figure} images/lab8-denyicmp.png
---
height: 200px
align: center
---
Monitor: Denied ICMP
```

- Create another intra-rule that allows ICMP **within** BU1 and then verify that ICMP is permitted among BU1’s instances. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-icmprulebu1.png
---
height: 200px
align: center
---
New intra-rule
```

- The ping will work this time thanks to this new fresh intra-rule!

```{figure} images/lab8-icmprulebu1ok.png
---
align: center
---
Ping will be ok!
```

- Once again, inspect the Monitor section to see immediately the corresponding *logs* of the action carried out before.

```{figure} images/lab8-pingpermitted.png
---
height: 200px
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
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-icmp-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#479608'>BU1</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU2</span>
- **Protocol**: <span style='color:#479608'>ICMP</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-penultimaterule.png
---
height: 200px
align: center
---
DCF Rules List
```

- Retry to ping the BU2 Mobile App from the BU1 Frontend. The ping will be successful thanks to the inter-rule!

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

- Now, let's SSH to the Public IP of the **BU2 Mobile App** and then try to SSH to the Private IP of the **BU1 Frontend**. Of course, SSH will fail!

```{figure} images/lab8-bu2sshbu1.png
---
align: center
---
SSH from BU2 to BU1 will fail
```

- Create another inter-rule that allows SSH **from** BU2 **towards** BU1. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#479608'>inter-ssh-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#479608'>BU2</span>
- **Destination Smartgroups**: <span style='color:#479608'>BU1</span>
- **Protocol**: <span style='color:#479608'>TCP</span>
- **Port**: <span style='color:#479608'>22</span>
- **Logging**: <span style='color:#479608'>On</span>
- **Action**: <span style='color:#479608'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-lastrule.png
---
height: 200px
align: center
---
"inter-ssh-bu2-bu1" Rule
```

- Let's try to issue the SSH command from the BU2 Mobile App towards the BU1 Frontend. This time the SSH will work smoothly.

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

- Now before completing the lab, remove the `Inspection Policy` from the Transit GW in AWS. The Aviatrix Distributed Cloud Firewall has been enabled across the whole multicloud infrastructure, therefore we can get rid of the bolted Firewall in AWS.

```{tip}
Go to **CoPilot > Security > FireNet > FireNet Gateways** and click on the **_ace-aws-eu-west-1-transit1_** GW, then click on `Policy`, select the two Spoke VPCs in AWS and choose the `Remove` action.

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

Congratulations, you have completed all labs and created a nice set of DCF rules across your Multicloud infrastructure!
