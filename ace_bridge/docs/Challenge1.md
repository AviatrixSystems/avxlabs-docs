# Challenge 1 - Ingress to Proxy

## Scenario

Let’s start the investigation to bring up back our application. The communication between the ALB inside the Ingress VPC and the Proxy VM is broken.

```{figure} images/ingress-proxy.png
---
height: 400px
align: center
---
Ingress to Proxy Failure
```

<details>
  <summary>Click here for the <span style='color:#33ECFF'>Hints!</span></summary>
  
* Search for the Private IP address of the Proxy VM

```{hint}
Go to **CoPilot > Cloud Fabric > Topology** and find the Proxy VM and retrieve its IP from the **Properties** section.
```

* Try to ping the Proxy VM from the *Ingress Spoke GW*.

```{hint}
Go to **CoPilot > Diagnostics > Diagnostics Tools > Gateway Diagnostics**.

Select the **_Ingress Spoke GW_** and launch a **traceroute**/**ping** towards the private IP address of the Proxy VM.
```
</details>


```{attention}
By the end of this challenge you need to ensure that the flow goes beyond the Ingress Spoke to the Transit and to the Proxy Spoke.
```
