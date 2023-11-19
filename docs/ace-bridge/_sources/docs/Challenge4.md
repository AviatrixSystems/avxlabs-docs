# Challenge 4 - Web VM to DB VM

## Scenario

After resolving the challenge#3, the Database VM administrator mentioned that the Database is receiving requests from an IP range in the **11.64.0.0/24** space.

```{figure} images/lab4-drawing.png
---
height: 400px
align: center
---
Network Isolation
```

<details>
  <summary>Click here for the Hints!</summary>
  
* Launch ping from the Web Spoke GW towards the Database VM.
* Simultaneously, launch packet capture on the *Database Spoke GW*.

```{hint}
Use the **packet capture** feature on a specific egress interface (both Tunnel and LAN interfaces) of the *Database Spoke GW*.
```
</details>

```{attention}
By the end of this challenge you need to ensure that the WEB VM is capable to reach the Database VM, successfully.
```
