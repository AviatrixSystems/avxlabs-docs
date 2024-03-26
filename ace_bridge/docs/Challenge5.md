# Challenge 5 - Web to Database 

## Scenario

After resolving the challenge #3, the Database VM administrator found out that the workload is receiving requests from an unknown IP that belongs to the **11.64.0.0/16** space... 

```{figure} images/lab4-drawing.png
---
height: 400px
align: center
---
Unknown CIDR
```

<details>
  <summary>Click here for the <span style='color:#33ECFF'>Hints!</span></summary>
  
* Launch ping from the Web Spoke GW towards the Database VM.
* Simultaneously, launch packet capture on the *Database Spoke GW*.

```{hint}
Use the **packet capture** feature on a specific egress interface (both Tunnel and LAN interfaces) of the *Database Spoke GW*.
```
</details>

```{attention}
By the end of this challenge you need to ensure that the WEB VM is capable to reach the Database VM, successfully.
```
