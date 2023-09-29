# Lab 2 - Connection Policy

## 1. SCENARIO

Now ACE Inc. has decided that BU1 and BU2 need to be able to communicate with each other. You are engaged for applying a **Connection Policy** in order to merge the two routing domains.
After the change has been applied, verify that both the network domains have been merged together, successfully.

![Topology 2](images/lab2-topology.png)
_Figure 21: Network Domains with the Connection Policy_

## 2. CHANGE REQUEST

* Apply the Connection Policy

```{tip}
Go to **CoPilot > Networking > Network Segmentation > Network Domains** and then edit either network domains.
```

The Connection Policy works bidirectionally!

![edit](images/lab2-editnd.png)
_Figure 22: Edit the Network Domain_

![bu1 editing](images/lab2-bu2nd.png)
_Figure 23: BU1 editing_

- Check Network Segmentation on the CoPilot by searching segmentation and look at the **Logical View**.

```{tip}
Go to **CoPilot > Networking > Network Segmentation > Overview > Logical View**. This time you can notice the relationships established between the two Network Domains.
```

![Logical view](images/lab2-logicalview.png)
_Figure 24: Logical View_

- Verify the connectivity **between** BU1 and BU2 domains.

  - SSH to any instances inside any BUs and carry out ping/ssh commands between the two BUs.
  - Ping and SSH between the two BUs should finally work, thanks to the Connection Policy (aka **VRF leaking**).

![ok](images/lab2-pingbu2.png)
_Figure 25: BU1 to BU2 is ok_

- Check the different routing tables (VRFs) maintained by any of the Transit Gateways.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the relevant gateway **> Gateway Routes** and filter out based on any Network Domains.
```

![rtb](images/lab2-bu1andbu2.png)
_Figure 26: View on a specific RTB_
