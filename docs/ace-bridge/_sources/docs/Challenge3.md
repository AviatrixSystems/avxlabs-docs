# Challenge 3 - Proxy VM to Web VM

## Scenario

This time, you have received another complaint that the connectivity issue between the Proxy VM and the Web VM is related to the routing!

Please investigate the connectivity from a **Network** point of view.

```{figure} images/proxy-web2.png
---
height: 400px
align: center
---
Proxy to Web Failure
```

<details>
  <summary>Click here for the <span style='color:#33ECFF'>Hints!</span></summary>

Use Copilot to figure out the IP address of the **Web VM**.

* Can you try to ping the Web VM from the *Proxy Spoke GW*?

```{hint}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_Proxy Spoke GW_** and launch a **traceroute** towards the private IP address of the Web VM.
```
</details>

```{attention}
By the end of this challenge you need to ensure that traffic is flowing from the Proxy VM to the Web VM, and viceversa.
```