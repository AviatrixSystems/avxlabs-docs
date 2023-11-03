# Lab 5 Advanced NAT Scenarios

## 5.0 - What's in the lab?

If you recall in Lab#4 we had an issue where there was no proper connectivity from either the client in Customer-A or B to the shared service application (aws_log1). We will commence lab 5 by solving this issue.

In addition, we will be connecting our last and final customer (customer_c) to the cloud. Finally, we will look at how to solve an advanced scenario with overlapping addresses.

## 5.1 - Current Topology

The below diagram explains the problem. If you look at the CIDR ranges of Customer-A-DC and Customer-B-Branch you will notice an overlap as both of them use 172.16.0.0/16. This is the reason why one of them will be successful in connecting to the shared service (aws_log1) whereas the other will fail.

In order for us to solve the problem, we will configure a customized NAT on Customer A's spoke gateways as well as customer B.

![Current Setup Explaining the problem](images/lab5/nat_from_customer_a_&_customer_b_client_to_aws_log1.png)_Figure 1: Explaining the Problem & Solution_

## 5.2 - SNAT on Customer A

Figures (2 & 3) show how to get the private address of the Spoke GW of customer-a. The reason we need the IP Address is we will be doing SNAT to this IP Address to resolve the overlapping CIDR problem with Customer-B. Please makes sure to use the IP address from your Pod and not the address on the screenshot.

![Adjust Columns](images/lab5/private_ip_add_column.png)_Figure 2: Add the Private IP Column_

![Get the Private Address of the Spoke GW](images/lab5/gateway_customer_a.png)_Figure 3: Edit the Gateway Configuration_

Figures (4 _ 7) walk you through configuring the Customized SNAT Rule.

![Configure SNAT Part 1](images/lab5/customer_a_gw_1_nat_part_1.png)_Figure 4: Configuring SNAT Rule Part 1_

![Configure SNAT Part 2](images/lab5/customer_a_gw_1_nat_part_2.png)_Figure 5: Configuring SNAT Rule Part 2_

![Save the SNAT Rule](images/lab5/save_nat_rule.png)_Figure 6: Save the SNAT Rules_

![Update the SNAT Rule](images/lab5/update_nat_rules.png)_Figure 7: Update the SNAT Rules_

Follow the same procedure for Customer A hagw.

![Select customer-a-hagw](images/lab5/gateway_select_customer_ha.png)
Figure 8: Edit the HA Gateway Configuration_


![SNAT customer-a-hagw P1](images/lab5/customer_a_gw_2_nat_part_1.png)_Figure 9: Configuring SNAT Rule Part 1_

![SNAT customer-a-hagw P2](images/lab5/customer_a_gw_2_nat_part_2.png)_Figure 10: Configuring SNAT Rule Part 2_

Please don't forget to save and update.

Now let us do a Proxy test from customer-a-client towards AWS-Log1 should work. Notice the field Source Address, this should be one of the Spoke Gateway IP addresses.

![Edit the CSR Configuration](images/lab5/proxy_customer_a_client_to_aws_log1_snat.png)_Figure 11: Proxy Customer A Client to AWS-Log1_

## 5.3 - SNAT on Customer B

This will be very similar to how we solved the problem for Customer A. We will select the gateway (customer_b) edit its configuration by configuring the Customized SNAT rule. This is done in Figures 12-14

![Select the GW](images/lab5/gateway_select_customer_b.png)_Figure 12: Select Customer B Gateway_

![Customer B Gateway SNAT Part 1](images/lab5/customer_b_gw_nat_part_1.png)_Figure 13: Customer B Gateway SNAT Part 1_

![Customer B Gateway SNAT Part 2](images/lab5/customer_b_gw_nat_part_2.png)_Figure 14: Customer B Gateway SNAT Part 2_

Don't forget to save and update!

Now let us do a Proxy test from customer-b-client towards AWS-Log1 should work. Notice the field Source Address, this should be the private address of Spoke B (doesn't have to be 10.4.0.41, please check the address within your lab pod)

![Proxy Customer B Client to AWS-Log1](images/lab5/customer_b_proof_ip_natted_snat_proxy_client_to_log1.png)_Figure 15: Proxy Customer B Client to AWS-Log1_

## 5.4 - Advanced NAT

In sections 5.2/5.3 we have solved the problem of being able to access (aws_log1) from all clients in Customer A and B locations (customer_a_dc and customer b branch). What about the reverse direction i.e. initiating traffic from the AWS-Log1 VM towards Customer-A-Client and Customer-B-Client. You might want to try that before you proceed further.

Test leveraging the proxy from AWS-Log1 to customer-a-client and customer-b-client. One of these two tests will not work.

```bash
http://aws-log1.pod150.aviatrixlab.com/customer-a-client

http://aws-log1.pod150.aviatrixlab.com/customer-b-client
```

The problem in a nutshell is also a result of the overlapping addresses of Customers A and B. We will only solve the problem for Customer-A in order to avoid repetition.

The problem and the solution are explained in the below diagram. The instructor will also be explaining the solution before you proceed.

![Explaining Advanced NAT Scenario](images/lab5/advanced_nat_scenario.png)_Figure 16: Explaining Advanced NAT Scenario_

### 5.4.1 - Configure Advanced NAT on customer-a Gateway
![Edit customer-a GW configuration](images/lab5/gateway_customer_a.png)_Figure 17: Edit customer-a GW configuration_

![Configure DNAT Rule Part 1](images/lab5/dnat_customer_a_gw.png)
_Figure 18: Configure DNAT Rule Part 1_
![Configure DNAT Rule Part 2](images/lab5/dnat_customer_a_gw_p2.png)
_Figure 19: Configure DNAT Rule Part 2_
![Save DNAT Rule](images/lab5/dnat_customer_a_gw_save.png)_Figure 20: Save DNAT Rule_

![Update DNAT Configuration_](images/lab5/dnat_customer_a_gw_update.png)
_Figure 21: Update DNAT Configuration_
![Add new SNAT Rule P2](images/lab5/customer_a_gw_snat_new_rule.png)
_Figure 22: Add new SNAT Rule P1_
![Add new SNAT Rule P2](images/lab5/customer_a_gw_snat_new_rule_p2.png)
_Figure 23: Add new SNAT Rule P2_
![Add new SNAT Rule towards CSR2 Part 1](images/lab5/customer_a_gw_snat_3rd_rule_p1.png)
_Figure 24: Add new SNAT Rule towards CSR2 Part 1_
![Add new SNAT Rule towards CSR2 Part 2](images/lab5/customer_a_gw_snat_3rd_rule_p2.png)_Figure 25: Add new SNAT Rule towards CSR2 Part 2_

![All 3 SNAT Rules Part 1](images/lab5/3_snat_rules_together_p1.png)
_Figure 26: All 3 SNAT Rules Part 1_

![All 3 SNAT Rules Part 2](images/lab5/3_snat_rules_together_p2.png)_Figure 27: All 3 SNAT Rules Part 2_

### 5.4.2 - Configure Advanced NAT on customer-a ha Gateway

We will follow the same procedure we have done on the customer-a gateway.

We start by selecting the customer-a-hagw and configuring the DNAT Rule.

![DNAT Rule customer-a-hagw Part 1](images/lab5/dnat_customer_a_gw.png)_Figure 28: DNAT Rule customer-a-hagw Part 1_

![DNAT Rule customer-a-hagw Part 2](images/lab5/dnat_customer_a_gw_p2.png)_Figure 29: DNAT Rule customer-a-hagw Part 2_

![SNAT Rule customer-a-hagw Part 1](images/lab5/customer_a_gw_snat_new_rule.png)_Figure 30: SNAT Rule customer-a-hagw Part 1_

![SNAT Rule customer-a-hagw Part 2](images/lab5/customer_a_ha_gw_snat_new_rule_p2.png)_Figure 31: SNAT Rule customer-a-hagw Part 2_

![SNAT Rule customer-a-hagw towards CSR2 Part 1](images/lab5/customer_a_gw_snat_3rd_rule_p1.png)_Figure 32: SNAT Rule customer-a-hagw towards CSR2 Part 1_

![SNAT Rule customer-a-hagw towards CSR2 Part 2](images/lab5/customer_a_ha_gw_snat_3rd_rule_p2.png)_Figure 33: SNAT Rule customer-a-hagw towards CSR2 Part 2_

### 5.4.3 - Advertise Virtual IP Address

We have done everything right so far but how would the fabric know of the Virtual IP Address (192.168.100.100/32)? We need Customer-A gateways to actually advertise it into the Fabric as shown below.

![Customize Spoke Advertised VPC CIDR P1](images/lab5/customize_spoke_advertised_routes_192.168.100.100_lab_5_p1.png)_Figure 34: Customize Spoke Advertised VPC CIDR Part 1_

![Customize Spoke Advertised VPC CIDR P2](images/lab5/customize_spoke_advertised_routes_192.168.100.100_lab_5_p2.png)_Figure 35: Customize Spoke Advertised VPC CIDR Part 2_

### 5.4.4 - Verification

In this section, we will attempt to ping the virtual IP Addres (192.168.100.100) from AWS-Log1.

Start by SSHing to both instances & customer-a-client run the tcpdump command present in Figure 33. Once tcpdump is running on customer-a-client, initiate the ping towards the Virtual IP address that represents the client. You should see the packet being received by customer-a-client (dnat) while the source address is changed to one of the gateway addresses you configured (snat).

![Initiate the Ping from AWS-Log1 towards Virtual IP](images/lab5/ping_from_aws_log1_to_virtual_ip_address.png)_Figure 36: Initiate the Ping from AWS-Log1 towards Virtual IP_

![Run tcpdump on client to see the packetrs](images/lab5/tcpdump_on_customer_a_client.png)_Figure 37: Run tcpdump on customer-a-client_

## 5.5 - Mapped NAT Scenario

In this section, Customer C has asked us to connect its Colocation facility to its Landing Zone. It also needs to benefit from the applications within the shared services segment. One thing to note is that Customer C's told us that its on-premises router doesn't support BGP.

![Mapped NAT Scenario with Customer C Explained](images/lab5/mapped_nat_scenario_explained.png)_Figure 38: Mapped NAT Scenario with Customer C Explained_

### 5.5.1 - Configure S2C with Mapped NAT Towards Customer C

We will configure the Site2Cloud with Mapped NAT in Figures 34 & 35.

![Configure Mapped NAT Aviatrix Side Part 1](images/lab5/s2c_mapped_nat_to_customer_c_colocation_p1.png)_Figure 39: Configure Mapped NAT Aviatrix Side Part 1_

![Configure Mapped NAT Aviatrix Side Part 2](images/lab5/s2c_mapped_nat_to_customer_c_colocation_p2.png)_Figure 40: Configure Mapped NAT Aviatrix Side Part 2_

Now that we have configured the Site2Cloud connection from Aviatrix' side, we can get the templatized configuration for Customer C CSR.This is shown in Figures 34-36.

![Get the CSR Configuration Template P1](images/lab5/edit_customer_c_to_get_csr_config.png)_Figure 41: Get the CSR Configuration Template P1_



![Get the CSR Configuration Template P2](images/lab5/edit_customer_c_to_get_csr_config_p2.png)_Figure 42: Get the CSR Configuration Template P2_

Without Enable Traffic to Transit Gateway Mapped NAT will only work towards destinations within the same Spoke VPC. This doesn't apply to our case as the target VM we want to reach is in the Shared Services thus we need to enable traffic to transit gateway setting. This is shown in Figure 36.

![Enable Forward Traffic to Transit Gateway](images/lab5/forward_traffic_to_transit_gw.png)_Figure 43: Enable Forward Traffic to Transit Gateway_

![Get the CSR Configuration Template P3](images/lab5/download_customer_c_csr_config.png)_Figure 44: Get the CSR Configuration Template P3_

You need to make the changes highlighted in Figures 45 & 46.

![Edit Customer C CSR Config Part 1](images/lab5/csr_config_change_p1.png)_Figure 45: Edit Customer C CSR Config Part 1_

![Edit Customer C CSR Config Part 2](images/lab5/csr_config_change_p2.png)_Figure 46: Edit Customer C CSR Config Part 2_



### 5.5.2 - Verify Mapped S2C Configuration

We start by verifying that the tunnel is up on the CSR.

![CSR Tunnel Status is UP](images/lab5/check_tunnel_status_csr.png)_Figure 47: CSR Tunnel Status is UP_

We also head to Cloud Routes within Copilot to look at the status of the S2C connection. Please note that we are not running BGP on top of this connection.

![Head to Cloud Routes within Copilot](images/lab5/copilot_cloud_routes.png)_Figure 48: Head to Cloud Routes within Copilot_

![Edit the CSR Configuration](images/lab5/site2cloud_status_customer_c_copilot.png)_Figure 49: S2C Tunnel to Customer-C-Colo is UP_

Finally, we initiate traffic from customer-c-client towards AWS-Log1 VM.

![Ping the Mapped Address of AWS-Log1 VM](images/lab5/ping_client_c_172.18.0.44_(virtual_ip_works).png)
_Figure 50: Successful Ping from Customer-C-Client towards AWS-Log1 VM_

![tcpdump aws-log1](images/lab5/tcpdump_lab_5_aws_log1_.png)
_Figure 51: tcpdump output on AWS-Log1 VM_
