# Lab 3 - FireNet - Interface

## 1. SCENARIO

BU1 and BU2 were able to communicate with each other as of Friday EOB (i.e end of LAB2).

However, the network team received a complaint from BU1 Frontend Team that the connectivity with BU2 Mobile App was no longer working.

![Lab Overview](images/lab3-topology.png)
_Figure 26: Lab 3 Topology_

## 2. TROUBLESHOOT REQUEST

- Verify that the connectivity between **BU1 Frontend** and **BU2 Mobile App** is actually broken.

  - SSH to BU1 Frontend and launch ping/ssh towards BU2 Mobile App.

![Lab Overview](images/lab3-pingfails.png)
_Figure 27: Ping fails_

- Check whether the concerned Spokes have the relevant routes or not.

```{tip}
Go to **CoPilot > Cloud Fabric > Gateways > Spoke Gateways >** select the relevant gateway **> Gateway Routes** and search for the subnet **10.1.212.0/24**, where BU2 Mobile App resides.
```

![Lab Overview](images/lab3-routecheck.png)
_Figure 28: Filtering out_

From the outcome above, it is evident that Spoke1 in AWS has the destination route in his RTB.

- Use **Diagnostics tools** by clicking on Spoke1 Gateway in AWS (identify the Aviatrix logo in the topology. Do not select a normal EC2 instance!) and try to ping/traceroute the instance behind the other spoke.

```{tip}
Go to **CoPilot > Coud Fabric > Topology** then select the relevant GW, click on **Tools** and then click on **Gateway Diagnostics**.
```

![Lab Overview](images/lab3-diagnostics.png)
_Figure 29: Diagnostics Tools_

![Lab Overview](images/lab3-pingfails2.png)
_Figure 30: Ping is unsuccessful_

Try to ping both workloads from the **Transit** GW in AWS.

![Lab Overview](images/lab3-pingok.png)
_Figure 31: Ping is ok from the Transit GW_

    	

- Check if the relevant Spoke VPCs are inspected by FireNet.

```{tip}
Go to **Controller > FIREWALL NETWORK > Policy**
```

![Lab Overview](images/lab3-policy.png)
_Figure 32: Inspection Policy Verification_

- Try to ping the **LAN** interface of the FW from the **Transit FireNet Gateway**.

```{tip}
First you need to retrieve the LAN Interface IP address of the FW. 
Go to **Controller > FIREWALL NETWORK > List >** select the Transit Firenet GW, under the **Firenet TAB >** click on **Details > Diagnostics >** then click on **Run** and then on **Show**.
```

![Lab Overview](images/lab3-details.png)
_Figure 33: Transit FireNet Gateway Details_

Scroll through the whole configuration till the ACE-FW section: the IP is beside the string **“lan_private_ip”**.

![Lab Overview](images/lab3-lanip.png)
_Figure 33: Lan Private IP of the FW_

    	

- **[WORKAROUND]** Try to find a way to exclude traffic from being sent to the FW.

```{tip}
Go to **Controller > FIREWALL NETWORK > List > Firenet** tab > select the Transit FireNet GW and click on **DETAILS**.
```

![Lab Overview](images/lab3-details.png)
_Figure 34: Transit FireNet Gateway Details_

Scroll down till the **Network List Excluded From East-West Inspection** section and add the source subnet **10.1.211.0/24** (e.g. this is the subnet where the BU! Frontend resides), as depicted below.

![Lab Overview](images/lab3-excemption.png)
_Figure 35: Excemption_

Try to ping once again the BU2 Mobile App from the B1 Frontend. This time the ping will be successful thanks to the workaround!

![Lab Overview](images/lab3-successful.png)
_Figure 36: Ping is successful_

- Remove the workaround previously applied and have a look at the dashboard on the main page of the Controller.

![Lab Overview](images/lab3-alert.png)
_Figure 37: Dashboard: the Widget is showing an Alert!_

You will notice a clear yellow alarm on the Trasit FireNet widget! Click on the widget, then select the Transit Firenet GW and click on **DETAILS**.

![Lab Overview](images/lab3-down.png)
_Figure 38: FW is unreachable_

This is a clear outcome that there is a problem on the FW. Ultimately, the firewall is not reachable.

- Log into the FW (refer to your POD portal for the credentials for logging in) and explore its configuration.

![Lab Overview](images/lab3-down.png)
_Figure 39: FW credentials_

![Lab Overview](images/lab3-fw.png)
_Figure 40: FW access_

```{tip}
Go to **FW > Network > Interfaces** and click the LAN interface (port2) and fix the issue!
```

The port was disabled to simulate the failure of the GW...

![Lab Overview](images/lab3-fwint.png)
_Figure 41: FW interface_

Relaunch the ping from **BU1 Frontend** towards **BU2 Mobile App**, after having <ins>re-enabled</ins> the LAN interface of the firewall.

![Lab Overview](images/lab3-pingworks.png)
_Figure 41: Ping is ok!_