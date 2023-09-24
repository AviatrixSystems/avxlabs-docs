# Lab 2 Customer Isolation

## 2.0 - What's in the lab?

Having recently established our MSP, we have been getting initial demand from customers to host their applications within Cloud.

Customers **A** already has existing workloads that they are looking to migrate from On-prem to AWS, Customer **B** is a newly founded startup who decided to initiate their required applications within AWS, whereas Customer **C** has decided to take the Azure Route. It is also worth noting that the MSP started receiving requests for MultiCloud presence for different business requirements. 

The landing zones with the respective workload & spokes have already been created. We will start by connecting those customer landing zones to their respective transits based on their CSP preference as cited above. Next, we have a requirement to ensure total isolation between Tenants.

![Topology Verification](images/lab1/lab_2_initial_before_connecting_tenants.png)_Figure 1: Current State

![Topology Verification](images/lab1/topology_proof_aws_and_azure_hub_peering_transits.png)
_Figure 2: Copilot Topology Verification_


## 2.1 - Connect Customers A to AWS Hub

In this section, we will be connecting Customer A's landing zone to AWS Hub.

![Search Bar](images/lab1/search_bar_gateways.png)
_Figure 3: Search Bar to Reach Gateways_

![Customer A Edit](images/lab1/customer_a_spoke_edit.png)
_Figure 4: Customer A Spoke Edit_

![Customer-A AWS Attachment](images/lab1/customer_a_spoke_connect_to_transit.png)
_Figure 5: Customer-A AWS Attachment_

## 2.2 - Connect Customers B to AWS Hub

In this section, we will be connecting Customer B's landing zone to AWS Hub.

![Customer B Edit](images/lab1/customer_a_spoke_edit.png)
_Figure 6: Customer-B Spoke Edit_

![Customer-B AWS Attachment](images/lab1/customer_b_spoke_connect_to_aws_hub.png)
_Figure 7: Customer-B AWS Attachment_


## 2.4 - Connect Customers C to Azure Hub

In this section, we will be connecting Customer B's landing zone to Azure Hub.

![Customer C Edit](images/lab1/customer_c_spoke_edit.png)
_Figure 8: Customer-C Spoke Edit_

![Customer-C Azure Attachment](images/lab1/customer_c_spoke_connect_to_azure_hub.png)
_Figure 9: Customer-C Azure Attachment_

![Current State](images/lab1/lab_2_post_connecting_tenants.png)
_Figure 10: Current State_

![Copilot Topology](images/lab1/final_topology_lab_2_copilot.png)
_Figure 11: Copilot Topology_

## 2.5 - Pre-Network Segmentation

We look at the current state of tenant isolation before enabling any segmentation. The MSP wants to verify total seggregation and isolation between its Tenants.

Please login to Customer A's application(details are in your lab portal) and initiate a ping towards Customer B's application. The results are shown in Figure 8.

![Pre-Seg Cust A to B App ping](images/lab1/pre_segmentation_customer_a_to_customer_b_app_ping.png)
_Figure 12: Customer A app can ping Customer B app_

Figure13 leverages the NGINX Proxy that is built into the instances to check if customer A application can communicate with customer B.

![Pre-Seg Cust A to B App ping](images/lab1/pre_segmentation_policy_customer_a_to_customer_b_app_browser.png)
_Figure 13: Customer A app proxy to Customer B app_

The output shown above actually provies that as of now, Customer A and Customer B are not isolated and can communicate. 

You can also try pinging aws-log1 from customer A application.



## 2.6 - Create Network Domains

In order to achieve Tenant isolation, we will be creating multiple Network Domains (a.k.a vrfs) and associating those Network Domains to the respective Spoke Gateways.

One pre-requisite of Network Segmentation is to ensure that the Transits (aws_hub & azure_hub) have segmentation enabled. This is shown in Figure 12 below. 

![Network Segmentation Prereq](images/lab1/network_segmentation_enabled_on_transits.png)
_Figure 14: Enable Segmentation on Transits_

The below diagrams (figures 16 _ 19) show how the Network Domains are created and associated with the respective Spoke Gateways.

![Customer-A Domain Creation](images/lab1/customer_a_domain_creation.png)
_Figure 15: Create Customer-A Network Domain_

![Shared-svcs Domain Creation](images/lab1/shared_svcs_domain_creation.png)
_Figure 16: Create Shared Svcs Network Domain_



![Customer-B Domain Creation](images/lab1/customer_b_domain_creation.png)
_Figure 17: Create Customer-B Network Domain_

![Customer-C Domain Creation](images/lab1/customer_c_domain_creation.png)
_Figure 18: Create Customer-C Network Domain_



## 2.7 - Verify Customer Segmentation

Now, we will re-attempt some connectivity tests after we configured the Network Domains.

First, ping Customer B application from Customer A application.

![Customer A to Customer B Isolation Ping](images/lab1/post_segmentation_ping_customer_b_app.png)_Figure 19: Customer A to Customer B Isolation Ping_

Attempt the same test by making use of the NGINX Proxy.

![Customer A to Customer B Isolation Proxy](images/lab1/post_segmentation_customer_a_app_to_customer_b_app_use_nginx_proxy.png)
_Figure 20: Customer A to Customer B Isolation Proxy_

![Customer A to AWS Log1 Isolation Ping](images/lab1/post_segmentation_customer_a_aws_log1_ping.png)_Figure 21: Customer A to AWS log1 Isolation Ping_

In addition, we now notice that Customer A application can't access the resources in the shared services (e.g. aws_log1). This is an unintended behavior where the MSP wants to ensure all its Tenants can access the shared services workloads.

![Customer A to AWS Log1 Isolation Proxy](images/lab1/post_segmentation_customer_a_aws_log1_browser.png)_Figure 22: Customer A to AWS log1 Isolation Proxy_


## 2.8 - Access to Shared Services

The MSP wants to ensure that all its tenants can access the workloads in the Shared Services. A connection policy is required to allow all customer tenants (customers a,b and c) to access Shared Svcs Tenant. This is shown in Figure 21-23.

![Edit Shared Svcs Domain](images/lab1/edit_shared_svcs_domain.png)_Figure 23: Edit Shared Svcs Domain_

![Connection Policy from Shared Svcs to Customers A,B and C](images/lab1/connection_policy_shared_svcs___abc.png)_Figure 24: Connection Policy Shared Svcs to all Customers


![Connection Policy Verification](images/lab1/verify_connection_policy.png)_Figure 25: Connection Policy Configuration Verification

![Connection Policy Verification](images/lab1/post_connection_policy_customer_a_to_aws_log1_ping.png)_Figure 26: Connection Policy Configuration Verification

![Connection Policy Verification](images/lab1/post_connection_policy_customer_a_to_aws_log1_browser.png)_Figure 27: Connection Policy Configuration Verification

This is how the final topology actually looks like. 

![Final Topology Lab 2](images/lab1/final_topology_lab_2.png)_Figure 28: Current State of the Topology_

## 2.8 - Conclusion

In this lab we started by connecting all Customer Landing Zones to their respective Hubs. We also had to ensure that Customers are isolated from one another and that all Customers should be able to access Shared Services. This requirement on isolation and connectivity can be met by leveraging Network Domains as well as Connection Policies.
