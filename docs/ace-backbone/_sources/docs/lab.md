# Quiz - Hints

Below are hints for select quiz questions, particularly those that require access to the "Cloud Backbone" Pod.

<span style='color:#479608'>Q1.</span> Upon successfully completing your lab exercise, please identify the **Cloud Service Providers** associated with the Aviatrix Edge deployed on-premises in the data center.

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```

<span style='color:#479608'>Q2.</span> Upon completing your lab exercise, please determine whether the peering between transit-aws and transit-azure within the Aviatrix Cloud Backbone has been established using **Standard Encryption** or **High Performance Encryption**?

```{hint}
Check **CoPilot > Cloud Fabric > Topology > Legend**
```

<span style='color:#479608'>Q3.</span> Upon successful completion of your lab exercise, determine which routes AWS or Azure Transit Gateway is receiving from the Aviatrix Edge.

```{hint}
Check **CoPilot > Diagnostics > Cloud Routes > Gateway Routes** and look at the *transit-aws*'s RTB
```

<span style='color:#479608'>Q4.</span> For Data Plane Verification, how to find the public and private IP address of Instances deployed in CSP's?

```{hint}
Check **CoPilot > Cloud Resources > Cloud Assets > Virtual Machines** 

<span style='color:#479608'>Q5.</span> Upon completing your lab exercise, review the dynamic topology map in the Aviatrix Copilot User Interface (UI). Then, verify the number of available paths for communication between AWS and Azure.

```{hint}
Check **CoPilot > Cloud Fabric > Topology**
```
