# Lab 2 - Connection Policy

## 1. SCENARIO

Now ACE Inc. has decided that BU1 and BU2 need to be able to communicate with each other. You are engaged for applying a **Connection Policy** in order to merge the two Routing Domains.

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
Go to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the gateway **ace-aws-eu-west-1-transit1** **> Route DB**
```

You will notice that the `"Segmentation"` is currently enabled and furthermore, you will also see what RTB each subnet belongs to.

```{figure} images/lab2-rib.png
---
align: center
---
Route DB
```

## 3. TROUBLESHOOT REQUEST

* From the **BU1 Frontend**, execute the *`curl`* command towards the private IP of **BU2 Mobile App**.

```{figure} images/lab2-curl.png
---
align: center
---
curl fails...
```

You will notice that after issuing the curl command, it will hang and then, after some seconds, it will convey a message reporting that the attempt to connect to port **80** failed.

```{important}
Curl is not working, despite having both Ping and SSH working correctly.
```

```{figure} images/lab2-curl2.png
---
align: center
---
PING and SSH are successful
```

* Use `AppIQ` for checking the *native cloud constructs* that separate the two nodes involved on this test.

```{tip}
Go to **CoPilot > Diagnostics > AppIQ > FlightPath**

```{figure} images/lab2-curl3.png
---
align: center
---
FlightPath
```

* Select the following parameters and then click on **Run AppIQ**.
  - Source: <span style='color:#33ECFF'>ace-aws-eu-west-1-spoke1-bu1-frontend</span>
  - Destination: <span style='color:#33ECFF'>ace-aws-eu-west-1-spoke2-bu2-mobile-app</span>
  - Protocol: <span style='color:#33ECFF'>TCP</span>
  - Port: <span style='color:#33ECFF'>80</span>
  - Interface: <span style='color:#33ECFF'>Private *(default)*</span>

```{figure} images/lab2-curl4.png
---
align: center
---
AppIQ configuration
```

Wait for some seconds and then check the report!

The Security Group attached to BU2 Mobile APP is missing a rule that would allow inbound traffic on tcp/80.

```{figure} images/lab2-curl5.png
---
align: center
---
AppIQ report
```

```{note}
FlightPath is a troubleshooting tool. It retrieves and displays, in a side by side fashion, cloud provider’s network related information such as **Security Groups**, **NACLs** and **Route Tables**. 

This helps you to identify connectivity problems on the underlay environments of each CSP involved along the path of the communication between two nodes.
```