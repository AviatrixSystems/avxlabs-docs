# Lab 2 - Connection Policy

## 1. SCENARIO

Now ACE Inc. has decided that BU1 and BU2 need to be able to communicate with each other. You are engaged for applying a **Connection Policy** in order to merge the two routing domains.
After the change has been applied, verify that both the network domains have been merged together, successfully.

```{figure} images/lab2-topology.png
---
height: 400px
align: center
---
Network Domains with the Connection Policy
```

## 2. CHANGE REQUEST

* Apply the Connection Policy

```{tip}
Go to **CoPilot > Networking > Network Segmentation > Network Domains** and then edit either network domains.
```

The Connection Policy works bidirectionally!

```{figure} images/lab2-editnd.png
---
align: center
---
Edit the Network Domain
```

```{figure} images/lab2-bu2nd.png
---
align: center
---
BU1 editing
```

- Check Network Segmentation on the CoPilot by searching segmentation and look at the **Logical View**.

```{tip}
Go to **CoPilot > Networking > Network Segmentation > Overview > Logical View**. 

This time you can notice the network relationships successfully established between the two Network Domains.
```

```{figure} images/lab2-logicalview.png
---
align: center
---
Logical View
```

- Verify the connectivity **between** BU1 and BU2 domains.

  - SSH to BU1 Frontend and carry out ping/ssh commands towards BU2 Mobile App.
  - Ping and SSH between the two BUs should finally work, thanks to the `Connection Policy` (aka **_VRF leaking_**).

```{figure} images/lab2-pingbu2.png
---
align: center
---
BU1 to BU2 is ok
```

- Check the different routing tables (VRFs) maintained by any of the Transit Gateways.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the gateway **ace-aws-eu-west-1-transit1** **> Gateway Routes** and filter out based on any Network Domains.
```

```{figure} images/lab2-bu1andbu2.png
---
align: center
---
View on a specific RTB
```

```{important}
The Spoke Gateways provide visibility only of the main routing table, the Global Routing Table (aka **GRT**), whereas the transit Gateways provide visibility of all routings tables (the main and the additional RTBs created through the Segmentation feature)
```

- Check the Route DB 

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the gateway **ace-aws-eu-west-1-transit1** **> Route DB** and filter out based on any Network Domains.
```

You will notice that the `"Segmentation"` is currently enabled and furthermore, you will see to what RTB each subnet belongs to.

```{figure} images/lab2-rib.png
---
align: center
---
Route DB
```
