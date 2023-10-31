# Lab 8 - Distributed Cloud Firewall

## 1. SCENARIO

ACE’s environment has been split up in two Smart Groups: **BU1** and **BU2**. Under the hood, there is a flat routing domain, due to the connection policy that merged the two network domains.

Furthermore, four DCF rules have been applied so far.

```{figure} images/lab8-initialrule.png
---
align: center
---
Existing DCF rules
```

- The BU1 Frontend has raised a complaint that is not able to SSH <span style='color:orange'>**within**</span> BU1.

- The BU2 Mobile App has raised a complaint that is not able to SSH <span style='color:lightgreen'>**towards**</span> BU1.

You have been engaged to create the following **two** additional rules:

- **Intra-rule**: allow SSH <span style='color:orange'>**within**</span> BU1
- **Inter-rule**: allow SSH <span style='color:lightblue'>**from**</span> BU2 **to** BU1


```{figure} images/lab8-topology.png
---
height: 400px
align: center
---
Lab 8 Scenario Topology
```

## 2. CHANGE REQUEST

- SSH on the **BU1 Frontend** and try to SSH to any other instance in BU1.
  - SSH fails as expected.

```{figure} images/lab8-topology.png
---
align: center
---
SSH fails within BU1
```

- Create an intra-rule that allows SSH **within** bu1 and then verify that SSH is permitted among BU1’s instances. Do not forget to enable **“Logging”**.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
Please bear in mind that once the rule has been created is not immediately applied on the Data Plane (i.e. it is just kept in *SAVED DRAFTS* state). You have to click on **COMMIT**.
```

```{figure} images/lab8-sshok.png
---
align: center
---
SSH is ok within BU1
```

Afterwards try also to check the **Monitor** section in order to explore the logs!

```{figure} images/lab8-monitor.png
---
align: center
---
Monitor
```

- SSH on the **BU2 Mobile App** and try to SSH to **BU1 Frontend**.
  - SSH fails as expected.
  - Refer to your POD portal or check the Topology for the FQDN/IP of BU1 Frontend. Moreover, refer to the private symbolic names or private IPs!

```{figure} images/lab8-bu2tobu1.png
---
align: center
---
BU2 to BU1 fails
```

- Create an inter-rule that allows SSH **from** BU2 **to** BU1 and then verify that SSH is permitted from Mobile App BU2 towards Frontend BU1. Do not forgert to enable the **“Logging”**.

```{tip}
Go to **CoPilot > Security > Distributed Cloud Firewall** and click on **+Rule**.
```

```{figure} images/lab8-bu2tobu1ok.png
---
align: center
---
BU2 to BU1 is ok
```

- Check again the **Monitor** section to find out the new logs for the inter-rule!

```{figure} images/lab8-monitor2.png
---
align: center
---
Monitor
```