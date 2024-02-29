# Lab 1 - Getting Started

## Lab Overview - What's in the lab?

Lab time: ~15 minutes

In this lab, we are going to explore our lab environment. The diagram below shows the current state of the lab environment. You will extend this environment during the exercises. The instructor will briefly explain the lab setup in the diagram. You can find a more complete and detailed diagram here.

![Lab Overview](images/lab-before2.png)
_Fig. Lab Overview_

## Lab 1.1 - CoPilot

### Description

Along with the Aviatrix Controller, CoPilot is also deployed. CoPilot will be your best buddy for configuring, visualizing and operating the Multicloud environment.

### Validate

Log in to CoPilot using the following details:

**URL:**  ```https://cplt.pod[#].aviatrixlab.com```  
**Username:**  ```student```  
**Password:** <span style="color:orange">***Please refer to your portal registration page for the password.***</span>  

* _Remember to replace <span style="color:orange">**[#]**</span> with your pod ID_

You can familiarize yourself with the different functions in CoPilot and after each lab, we will verify the results in CoPilot.
**Please note that it can take a minute for CoPilot to reflect changes on the network.**

The `Dashboard` tab provides a global overview of your Multicloud network, the status, how much traffic is flowing, locations deployed, etc.  

Take a look into `Cloud Fabric-->Topology` which visualizes the connectivity of the Multicloud environment. Topology will show you latency between links when you enable it on the top right hand corner in the latency pane. The topology view also allows you to initiate some troubleshooting commands directly from the map. Expand `aws-transit-west` in the middle of the screen, by clicking on it. Now you should see and be able to select the `aws-transit-west` gateway on the outside edge of the diagram. On the bottom right, you should see a diagnostics button. Click that and explore the diagnostics capabilities.

### Expected Results

You should be able to log in to CoPilot. You should be able to view Topology by clicking `Cloud Fabric` -> `Topology` and see the current Multicloud Network topology. Topology should look similar to this:  

![Topology1](images/lab1-copilot-topology.png)  
_Fig. Initial Topology_  

As you can see, we have the `aws-transit-west` gateway in the middle, with `aws-build`, `aws-dmz` and `aws-prod-west` spokes connected to it. The AWS VPC's are not yet connected to any transit.

After selecting the transit gateway and clicking the diagnostics tools button, you should see the diagnostics pane. Have a look around at the troubleshooting tools here.

![CoPilot Diagnostics](images/lab1-copilot-diagnostics.png)  
_Fig. CoPilot Diagnostics_  

![CoPilot Diagnostics Pane](images/lab1-copilot-diagnostics-pane.png)  
_Fig. CoPilot Diagnostics Pane_  

Close the diagnostics pane again.

## Lab 1.2 - Access Accounts

### Description

In order for the controller to be able to access the different CSP environments, we need to provide it with accounts with the correct privileges.

### Validate

Take a look at the access accounts already set up. You can see them in Copilot under **_Cloud Resources -> Cloud Account_**.

### Expected Results

An account in AWS has already been onboarded and you should see this account in the list. The account should also have an audit status **_pass_**, meaning the permissions in the accounts are correctly configured.

## Lab 1.3 - Connectivity Check

### Description

Each Spoke VPC contains one or multiple Linux VMs to test connectivity. There is a web server running on each of the instances which you can use to test the connectivity. The purpose of this exercise is to verify the connectivity between Linux VMs in the Spoke VPCs in the different regions as well as the Office network. To make life a little easier, we have created 2 dashboards to show the status of all connectivity.

### Validate

In order to view the connectivity dashboards, a Remote Access Server is placed in the office.  We need to log on to the Remote Access Server we have deployed throughout the lab which can be accessed here:

**URL:** ```https://client.pod[#].aviatrixlab.com```  
**User:** ```student```  
**Password:** <span style="color:orange">***Please refer to your portal registration page for the password.***</span>  

* _Remember to replace <span style="color:orange">**[#]**</span> with your pod ID_
* _If you want to use copy/paste in this Guacamole web console, use CTRL+ALT+SHIFT on a Windows device or CTRL+CMD+SHIFT on a MacOS device._

![Remote Access Server](images/lab1-remote-access.png)  
_Fig. Remote Access Server_  

```{note}
You can also access the Remote Access Server on port `3389` using the RDP client of your choice using the same credentials as above.
```

* Once logged in, open the web browser and browse to the office connectivity dashboard ```http://localhost```

Now that we have validated the office connectivity, lets check the public web application that is already deployed in AWS `us-west-2`.

* Open the web browser and browse to the Web App connectivity dashboard ```http://aws-lb.pod[#].aviatrixlab.com/test```
* _Remember to replace <span style="color:orange">**[#]**</span> with your pod ID_

### Expected Results

* None of the connections from the office will be successful. Look at the Diagram in the Lab 1 Overview and you will see that the office is not connected yet.

![Office connectivity](images/lab1-connectivity-from-office.png)  
_Fig. Office connectivity_  

* Connectivity to the appgw, web, app and db servers should be working, proving that the already deployed application in AWS `us-west-2` is up.

![Web app connectivity](images/lab1-connectivity-3tier-app.png)  
_Fig. Web app connectivity_  

## End of lab 1

Ok, that's the end of Lab 1. Great job!
