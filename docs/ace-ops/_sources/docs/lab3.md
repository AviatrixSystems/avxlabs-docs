# Lab 3 - FireNet - Interface

## 1. SCENARIO

BU1 and BU2 were able to communicate with each other as of Friday EOB (i.e end of LAB2).

However, the network team received a complaint from BU1 Frontend Team that the connectivity with BU2 Mobile App was no longer working.

```{figure} images/lab3-topology.png
---
height: 400px
align: center
---
Lab 3 Topology
```

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH to BU1 Frontend and launch ping/ssh towards BU2 Mobile App.

```{figure} images/lab3-pingfails.png
---
align: center
---
Ping fails
```

- Check whether the relevant Spoke Gateways have the required routes installed on their routing tables or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the gateway **ace-aws-eu-west-1-spoke1**  **> Gateway Routes** and search for the subnet **10.1.212.0/24**, where BU2 Mobile App resides.
```

```{figure} images/lab3-routecheck.png
---
align: center
---
Filtering out
```

From the outcome above, it is evident that Spoke1 in AWS has the destination route in its RTB.

- Use **Diagnostics Tools** from the *Spoke1 Gateway* in AWS and try to ping/traceroute the instance behind the other spoke (i.e. BU2 Mobile App).

```{tip}
Retrieve the private IP address of the **BU2 Mobile App** from the Cloud Assets inventory!

Go to **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** and search for `"mobile"`, then retrieve the private IP of the EC2 instance.
```

```{figure} images/lab3-personalpod.png
---
align: center
---
Private IP of BU2 Mobile App
```

```{tip}
Go to **CoPilot > Diagnostics > Diagnostics Tools** then select the GW **ace-aws-eu-west-1-spoke1** and ping the private IP of BU2 Mobile App
```

```{figure} images/lab3-pingfailss.png
---
align: center
---
Ping is unsuccessful
```

From the outcome above you can notice that from the Spoke1 GW in AWS, the BU2 Mobile App is not reachable.

- Now try to ping both workloads (i.e. BU1 Frontend and BU2 Mobile App) from the **Transit** GW in AWS.

```{figure} images/lab3-bu1ping.png
---
align: center
---
Ping is ok from the Transit GW towards BU1 Frontend
```

```{figure} images/lab3-bu2ping.png
---
align: center
---
Ping is ok from the Transit GW towards BU2 Mobile App
```

From the outcome above, you can notice that the Transit GW in AWS can ping both BU1 Frontend and BU2 Mobile App, successfully.

- Check the FireNet section!

```{tip}
Go to **CoPilot > Security > FireNet**
```

```{figure} images/lab3-unreachable.png
---
align: center
---
The FW is down
```

You will notice that the Firewall turns out being down.

Click on the **Firewall** tab!

- Try to ping the **LAN** interface of the FW from the **Transit FireNet Gateway**.

```{tip}
First you need to retrieve the LAN Interface IP address of the FW. 
Go to **Controller > FIREWALL NETWORK > List >** select the Transit Firenet GW, under the **Firenet TAB >** click on **Details > Diagnostics >** then click on **Run** and then on **Show**.
```

```{figure} images/lab3-details.png
---
align: center
---
Transit FireNet Gateway Details
```

Scroll through the whole configuration till the ACE-FW section: the IP is beside the string **“lan_private_ip”**.

```{figure} images/lab3-lanip.png
---
align: center
---
Lan Private IP of the FW
```

- **[WORKAROUND]** Try to find a way to exclude traffic from being sent to the FW.

```{tip}
Go to **Controller > FIREWALL NETWORK > List > Firenet** tab > select the Transit FireNet GW and click on **DETAILS**.
```

```{figure} images/lab3-details.png
---
align: center
---
Transit FireNet Gateway Details
```

Scroll down till the **Network List Excluded From East-West Inspection** section and add the source subnet **10.1.211.0/24** (e.g. this is the subnet where the BU! Frontend resides), as depicted below.

```{figure} images/lab3-excemption.png
---
align: center
---
Exemption
```

Try to ping once again the BU2 Mobile App from the B1 Frontend. This time the ping will be successful thanks to the workaround!

```{figure} images/lab3-successful.png
---
align: center
---
Ping is successful
```

- Remove the workaround previously applied and have a look at the dashboard on the main page of the Controller.

```{figure} images/lab3-alert.png
---
align: center
---
Dashboard: the Widget is showing an Alert!
```

You will notice a clear yellow alarm on the Trasit FireNet widget! Click on the widget, then select the Transit Firenet GW and click on **DETAILS**.

```{figure} images/lab3-down.png
---
align: center
---
FW is unreachable
```

This is a clear outcome that there is a problem on the FW. Ultimately, the firewall is not reachable.

- Log into the FW (refer to your POD portal for the credentials for logging in) and explore its configuration.

```{figure} images/lab3-down.png
---
align: center
---
FW credentials
```

```{figure} images/lab3-fw.png
---
align: center
---
FW access
```

```{tip}
Go to **FW > Network > Interfaces** and click the LAN interface (port2) and fix the issue!
```

The port was disabled to simulate the failure of the GW...

```{figure} images/lab3-fwint.png
---
align: center
---
FW interface
```

Relaunch the ping from **BU1 Frontend** towards **BU2 Mobile App**, after having <ins>re-enabled</ins> the LAN interface of the firewall.

```{figure} images/lab3-pingworks.png
---
align: center
---
Ping is ok!
```