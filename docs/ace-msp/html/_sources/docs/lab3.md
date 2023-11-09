# Lab 3 FireNet

## 3.0 _ What's in the lab?

Having setup the MSP backbone and segmented our Network, and deployed our NGFWs 
Aviatrix Firewall Network (aka firenet) is a turnkey solution to deploy and manage firewall instances in the cloud. Firewall Network significantly simplifies virtual firewall deployment and allows the firewall to inspect East_West as well as North_South traffic.  In addition, Firewall Network allows you to scale firewall deployment to multi AZ and multi instances/VMs in maximum throughput active/active state without the SNAT requirement.

## 3.1 _ Current Topology

![Final Topology Lab 2](images/lab1/final_topology_lab_2.png)_Figure 1: Current State of the Topology_

You can see clearly that Firewalls within AWS Hub have already been provisioned. The objective behind this lab is to deploy the FireNet within Azure Hub.

## 3.2 _ Deploying Firewalls in Azure

FireNet is to be configured from the controller. The steps are detailed in Figure2. Please be extremely careful before to ensure all the parameters match exactly before launching the Firewall.

![Launch FW Part 1](images/lab3/launch_fw_instance_in_azure_part_1.png)
![Launch FW Part 2](images/lab3/launch_fw_instance_in_azure_part_2.png)_Figure 2: Launch FW in Azure_

## 3.3 _ Connect to the FW

Please ba patient, it might take up to 20 minutes for the FW deployment to finish.

You can find the Public IP address of the PAN VM_series mgmt interface here.

![Get the FW_IP](images/lab3/get_the_fw_ip_address.png)_Figure 3: Find the Firewall IP_

Once you have that, you can connect to the FW through your PC's browser.
Please make sure to login with the same credentials that you have assigned during the creationl.

![Connect to the Firewall UI](images/lab3/login_to_fw.png)_Figure 4: Login to the Firewall_

## 3.4 _ Vendor Integration

Aviatrix's Vendor Integration feature can install the relevant routes within the Azure Firewall to ensure it knows how to route the traffic. This is shown in Figures 5_7.

![Vendor Integration Azure FW](images/lab3/vendor_integ_part_1.png)_Figure 5: Vendor Integration Azure FW_

![Vendor Integration Azure FW Save](images/lab3/vendor_integ_part_2_save.png)_Figure 6: Vendor Integration Azure Save_

![Vendor Integration Azure FW Show](images/lab3/vendor_integr_part_3_show.png)
![Vendor Integration Azure FW Output](images/lab3/vendor_integ_part_4_show.png)_Figure 7: Vendor Integration Azure Sync_

## 3.5 Inspection Policy

The firewall is up, has the necessary policies as part of the bootsrapping and has the necessary routes as part of vendor Integration. Now we can selectively decide which traffic is to be inspected by the NGFW.

Let's add Azure Shared Services to Inspection Policy.

![Inspection Policy](images/lab3/firenet_inspection_policy.png)_Figure 8: Add Azure Shared Svcs to Inspected List_

![Inspection Policy](images/lab3/inspection_policy_azure.png)_Figure 9: Add Azure Shared Svcs to Inspected List_

## 3.6 _ Send Traffic

We will issue a ping from AWS_Log1 to Azure Log1 as shown below:

![Ping AWS_Log1 to Azure Log1](images/lab3/ping_aws_to_azure_log_1_.png)_Figure 10: Ping from AWS_Log1 to Azure Log1_

We will also issue a ping from Customer A application to Azure Log2 as shown below:

![Ping Customer_A App to to Azure Log2](images/lab3/ping_c1_app_to_azure_log_2.png)_Figure 11: Ping from Customer_A App to to Azure Log2_

## 3.7 _ Monitor Traffic on the Firewall

![Firewall Monitoring Ping from AWS_Log1 to Azure Log1](images/lab3/see_traffic_on_azure_fw_aws_to_azure_log.png)_Figure 12: Firewall Monitoring Ping from AWS_Log1 to Azure Log1_

![Firewall Monitoring Ping from Customer_A App to to Azure Log2](images/lab3/see_traffic_on_azure_fw_c1_app_to_azure_log_2.png)_Figure 13: Firewall Monitoring Ping from Customer_A App to to Azure Log2_

## 3.8 _ Conclusion

In this lab we learned to deploy Firewall Network within Azure by going through all necessary steps from creating the Firewall, Bootstrapping process, Vendor Integration, Inspection Policy and finally monitoring to ensure intended traffic was actually inspected by the PAN VM_series FWs.
