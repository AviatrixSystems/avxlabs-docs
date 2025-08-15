# Challenge 5 - Web to Database 

## Scenario

After resolving challenge #4, the Database VM administrator discovered that the workload was receiving requests from an unknown IP within the **11.64.0.0/16** subnet

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
By the end of this challenge, ensure that the WEB VM can reach the Database VM successfully.
```
