# Lab 4 - FireNet - Routes

## 1. SCENARIO

BU1 and BU2 were able to communicate as end of Lab3.

Unfortunately, the network team received another complaint from BU1 Frontend Team that BU2 Mobile App was no longer reachable.

```{figure} images/lab4-topology.png
---
height: 400px
align: center
---
Lab 4 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH to BU1 Frontend and launch ping/ssh to BU2 Mobile App.

```{figure} images/lab4-pingunsucc.png
---
align: center
---
Ping fails
```

- Check whether both AWS Spoke1 and AWS Spoke2 have the required routes installed in their RTBs or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the **ace-aws-eu-west-1-spoke1** gateway for instance, and filter out based on the remote route.
```

```{figure} images/lab4-filter.png
---
align: center
---
Filter out
```

From the outcome above, it is evident that Spoke1 in AWS has the destination route in its RTB. 

Try to verify the inverse, checking the destination route from the AWS Spoke2 side.

 - Use **Diagnostics** tool by clicking on the Spoke1 Gateway in AWS and try to ping/traceroute the instance behind the other spoke. Repeat this action from the Spoke2 too.

```{tip}
Go to **CoPilot > Cloud Fabric > Topology** and select the **_ace-aws-eu-west-1-spoke1_** Gateway in AWS and click on **Tools** and then on **Gateway Diagnostics**.
```

```{figure} images/lab3-diagnostics.png
---
align: center
---
Enterprise-Grade Tools
```

```{figure} images/lab4-pingfails.png
---
align: center
---
Ping fails
```

- Try to ping both workloads from the Transit GW in AWS.

```{figure} images/lab4-pingok.png
---
align: center
---
Ping ok
```

From the outcome above, you can again notice that the Transit GW has full reachability towards the remote workloads.

- Check if the relevant Spoke VPCs are inspected by FireNet. This time control the Inspection Policy from the **CoPilot** dashboard.

```{tip}
Go to **CoPilot > FireNet > FireNet Gateways**, then click on the **_ace-aws-eu-west-1-transit1_** Gateway **> Policy**
```

The AWS Spoke VPCs are both correctly inspected.

```{figure} images/lab4-inspection.png
---
align: center
---
Inspection Policy Verification
```

- Have a look at the FireNet section on the Controller to figure out wheether there are any sort of alerts or not.

```{figure} images/lab4-dashboard.png
---
align: center
---
Dashboard
```

- Verify the Vendor Integration on the FireNet section on the CoPilot!

```{tip}
Go to **Controller > FIREWALL NETWORK > Vendor Integration**, select the **FW**, click on **EDIT** and then click on **SHOW**.
```

```{figure} images/lab4-edit.png
---
align: center
---
Vendor Integration
```

```{figure} images/lab4-missing.png
---
align: center
---
Missing route
```

You will notice that the **10.0.0.0/8** is not present inside the routing table of the FW.

- Fix the problem, clicking on the **SYNC** button, in order to inject again the 10.0.0.0/8 into the RTB.

```{figure} images/lab4-sync.png
---
align: center
---
SYNC
```

- Relaunch the ping from **BU1 Frontend** towards **BU2 Mobile App**.

```{figure} images/lab4-pingworks.png
---
height: 400px
align: center
---
Ping is ok
```