# Challenge 3 - Network Isolation

## Scenario

You have received a call just now from the Web VM owner that reported that the remote Database route is not advertised.

```{figure} images/segmentation2.png
---
height: 400px
align: center
---
Network Isolation
```

* Search the Private IP of the Database VM and try to ping it from the *Web Spoke GW*. 
  Does the ping work?

```{hint}
Check the **RTBs** of all the gateways involved in the path between the Web VM and the Database VM!
```

```{attention}
By the end of this challenge you need to ensure that the Database route has been propagated throughout the whole cloud network (i.e. Transit and Spoke GWs).
```