# Lab 2 - Connection Policy

## 1. SCENARIO

ACE Inc. has decided that BU1 and BU2 must communicate. You are engaged to apply a `Connection Policy` to merge the two Network Domains. After the change is applied, verify that the two network domains have been merged successfully.

```{figure} images/lab2-topology.png
---
height: 400px
align: center
---
Network Domains with the Connection Policy
```

## 2. CHANGE REQUEST

* Apply the **Connection Policy**

```{tip}
Navigate to **CoPilot > Networking > Network Segmentation > Network Domains**, then edit one of the network domains.
```

The Connection Policy operates bidirectionally.

```{figure} images/lab2-editnd.png
---
height: 200px
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

- Check the **Logical View** inside the `Network Segmentation` section.

```{tip}
Navigate to **CoPilot > Networking > Network Segmentation > Overview > Logical View**. 

This time, you can observe the successful establishment of network relationships between the two Network Domains.
```

```{figure} images/lab2-logicalview.png
---
align: center
---
Logical View
```

### 2.1 Verify the connectivity between BU1 and BU2 domains Using thw Diagnostic tools

- Navigate to **CoPilot > Diagnostics > Diagnostic Tools**, select the **_ace-aws-eu-west-1-spoke1_** gateway, and run a ping to the **BU2-MobileApp** private IP address. This time ICMP traffic will be successful thanks to the Connection Policy.

```{figure} images/lab2-cp000.png
---
height: 400px
align: center
---
BU1 to BU2 is ok
```

### 2.1 Verify the connectivity between BU1 and BU2 domains Using the SSH client <span style='color:#33ECFF'>(BONUS)</span></summary>

  - SSH into the **BU1 Frontend** and run ping or SSH commands targeting the **BU2 Mobile App**.
  - Ping and SSH between the two BUs should now work, thanks to the `Connection Policy` (aka **_VRF leaking_**).

```{figure} images/lab2-pingbu2.png
---
height: 400px
align: center
---
BU1 to BU2 is ok
```

- Check the different routing tables (VRFs) maintained by any of the Transit Gateways.

```{tip}
Navigate to **CoPilot > Cloud Fabric > Gateways > Transit Gateways >** select the gateway **ace-aws-eu-west-1-transit1** **> Gateway Routes** and filter out based on any Network Domains.
```

```{figure} images/lab2-bu1andbu2.png
---
align: center
---
Network Domains
```

```{important}
**Spoke Gateways** only show the main routing table, known as the *Global Routing Table (GRT)*, while **Transit Gateways** provide visibility into all routing tables, including the main RTB and any additional RTBs created through segmentation.
```

## 3. TROUBLESHOOT REQUEST

* From the **BU1 Frontend**, execute the *`curl`* command towards the private IP of **BU2 Mobile App**.
  
```{important}
Refer always to your personal POD for the IP addresses. 
```

```{figure} images/lab2-curl.png
---
align: center
---
curl fails...
```

After running the curl command, it may hang for a moment, and then a message will appear indicating that the connection to port **80** has failed.

```{important}
Curl is not working, despite having both Ping and SSH working correctly.
```

```{figure} images/lab2-curl2.png
---
align: center
---
PING and SSH are successful
```

* Use `AppIQ` to verify the *CSP native cloud constructs* that isolate the two instances involved in this test.

```{tip}
Navigate to **CoPilot > Diagnostics > AppIQ > FlightPath**
```

```{figure} images/lab2-curl3.png
---
height: 300px
align: center
---
FlightPath
```

* Select the following parameters and then click on **Run AppIQ**.
  - Source: <span style='color:#479608'>ace-aws-eu-west-1-spoke1-bu1-frontend</span>
  - Destination: <span style='color:#479608'>ace-aws-eu-west-1-spoke2-bu2-mobile-app</span>
  - Protocol: <span style='color:#479608'>TCP</span>
  - Port: <span style='color:#479608'>80</span>
  - Interface: <span style='color:#479608'>Private</span>

```{figure} images/lab2-curl4.png
---
height: 250px
align: center
---
AppIQ configuration
```

Wait for some seconds and then check the report!

The Security Group attached to BU2 Mobile APP is missing a rule that should allow inbound traffic on tcp/80.

```{figure} images/lab2-curl5.png
---
height: 300px
align: center
---
AppIQ report
```

```{note}
`AppIQ` is a troubleshooting tool. It retrieves and displays, in a side by side fashion, cloud provider’s network related information such as **Security Groups**, **NACLs** and **Route Tables**. 

This helps you to identify connectivity problems on the underlay environments of each CSP involved along the communication path between two nodes.
```

### 3.1. Verify from AWS Console

Log in to the **AWS console**.

```{important}
Navigate to your personal POD portal and click on the **Console** button under the AWS Console section.

Sign in using the provided credentials (these screenshots refer to **Pod 143**).
```

```{figure} images/lab2-console.png
---
height: 300px
align: center
---
POD Portal - AWS Console button
```

```{figure} images/lab2-aws.png
---
align: center
---
AWS 
```

Change the region to `Ireland (eu-west-1)` in the top-right corner and invoke the **EC2** service, then click on the **Instances (running)** section.

```{figure} images/lab2-euwest.png
---
align: center
---
Regions selection 
```

```{figure} images/lab2-invoke.png
---
height: 200px
align: center
---
Invoke the EC2 service
```

```{figure} images/lab2-instance.png
---
height: 200px
align: center
---
Instances (running)
```

Search for **_ace-aws-eu-west-1-spoke2-bu2-mobile-app_**, select the instance and choose the tab `"Security"` below and then click on the security group identificator with a long string.

```{figure} images/lab2-sg.png
---
align: center
---
Select the Security Group
```

Explore the inbound rules and you will find out the absence of a rule that should permit the incoming traffic on port **tcp/80**.

```{figure} images/lab2-sg2.png
---
height: 200px
align: center
---
The Security Group attached to the Mobile App
```

Click on `"Edit inbound rules"`.

```{figure} images/lab2-sg3.png
---
height: 200px
align: center
---
Edit the SG
```

Click on `"Add rule"`.

```{figure} images/lab2-sg4.png
---
height: 300px
align: center
---
Add a new rule
```

Create the required inbound rule (i.e., _"allow **http** traffic from 10.0.0.0/8"_) as depicted below and then click on the `"Save rules"` button.

```{figure} images/lab2-sg5.png
---
height: 150px
align: center
---
New inbound rule: allow port 80
```

```{caution}
Please use the specific CIDR **10.0.0.0/8** instead of 0.0.0.0/0 to ensure precision in the Security Group rule!
```

Now re-run the **curl** command from the **_BU1 Frontend_** instance to the private IP address of the **_BU2 Mobile App_**.

Curl command will work this time.
```{figure} images/lab2-last.png
---
align: center
---
Curl is ok!
```
