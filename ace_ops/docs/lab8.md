# Lab 8 - Distributed Cloud Firewall

## 1. SCENARIO

ACE’s environment has been split up in two SmartGroups: **BU1** and **BU2**. Under the hood, there is a flat routing domain, due to the connection policy that merged the two network domains.

Furthermore, four DCF rules have been already applied so far.

```{figure} images/lab8-initialrule.png
---
align: center
---
Existing DCF rules
```

- The BU1 Frontend has raised a complaint that is not able to use the SSH protocol <span style='color:orange'>**within**</span> BU1.

- The BU1 Frontend has raised a complaint that is not able to use the ICMP protocol <span style='color:orange'>**within**</span> BU1.

- The BU1 Frontend has raised a complaint that is not able to use the ICMP protocol <span style='color:lightgreen'>**towards**</span> BU2.

- The BU2 Mobile App has raised a complaint that is not able to use the SSH protocol <span style='color:lightgreen'>**towards**</span> BU1.

You have been engaged to create the following **four** new additional rules:

- **Intra-rule**: allow SSH <span style='color:orange'>**within**</span> BU1
- **Intra-rule**: allow ICMP <span style='color:orange'>**within**</span> BU1
- **Inter-rule**: allow ICMP <span style='color:lightblue'>**from**</span> BU1 **to** BU2
- **Inter-rule**: allow SSH <span style='color:lightblue'>**from**</span> BU2 **to** BU1


```{figure} images/lab8-topology.png
---
height: 400px
align: center
---
Lab 8 Scenario Topology
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

- **Name**: <span style='color:#33ECFF'>intra-ssh-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Protocol**: <span style='color:#33ECFF'>TCP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-rule01.png
---
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

- Now terminate the SSH session with the BU1 Analytics, typing `"exit"`, and issue the ping command towards the BU1 Anlytics from the BU1 Frontend. The ping will not work!

```{figure} images/lab8-pingbu1.png
---
align: center
---
Ping will fail
```

- Create another intra-rule that allows ICMP **within** BU1 and then verify that ICMP is permitted among BU1’s instances. Do not forget to enable **“Logging”**, for auditing purposes.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

Ensure these parameters are entered in the pop-up window `"Create Rule"`:

- **Name**: <span style='color:#33ECFF'>intra-icmp-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-icmprulebu1.png
---
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

- **Name**: <span style='color:#33ECFF'>inter-icmp-bu1-bu2</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>BU2</span>
- **Protocol**: <span style='color:#33ECFF'>ICMP</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-penultimaterule.png
---
align: center
---
Ping will fail
```

- Retry to ping the BU2 Mobile App from the BU1 Frontend. The ping will be successful thanks to the inter-rule!

```{figure} images/lab8-penultimateping.png
---
align: center
---
Ping from BU1 to BU2
```

- Now, let's SSH to the BU2 Mobile App and then let's try to SSH to the BU1 Frontend. Of course, SSH will fail!

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

- **Name**: <span style='color:#33ECFF'>inter-ssh-bu2-bu1</span>
- **Source Smartgroups**: <span style='color:#33ECFF'>BU2</span>
- **Destination Smartgroups**: <span style='color:#33ECFF'>BU1</span>
- **Protocol**: <span style='color:#33ECFF'>TCP</span>
- **Port**: <span style='color:#33ECFF'>22</span>
- **Logging**: <span style='color:#33ECFF'>On</span>
- **Action**: <span style='color:#33ECFF'>**Permit**</span>

Do not forget to click on **Save In Drafts**, and then **Commit** your rule.

```{figure} images/lab8-lastrule.png
---
align: center
---
The last inter-rule!
```

- Let's try to issue the SSH command from the BU2 Mobile App towards the BU1 Frontend. This time the SSH will work smoothly.

```{figure} images/lab8-lastssh.png
---
align: center
---
The last inter-rule!
```

Congratulations, you have completed all labs and created a nice set of DCF rules across your Multicloud infrastructure!










