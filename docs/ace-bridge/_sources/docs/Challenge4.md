# Challenge 4 - Web VM to DB VM

## Scenario

After resolving the challenge#3, the Database VM administrator mentioned that the Database is receiving requests from an IP range in the **11.64.0.0/24** space.

* Launch ping from the Web Spoke GW towards the Database VM.
* Simultaneously, launch packet capture on the *Database Spoke GW*.

```{hint}
Use the **packet capture** feature on a specific egress interface (both Tunnel and LAN) of the *Database Spoke GW*.
```