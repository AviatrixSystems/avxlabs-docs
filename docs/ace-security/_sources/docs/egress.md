# Aviatrix Cloud Firewall Lab

## 1. Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates exclusively within the AWS cloud environment. For internet egress traffic, they utilize AWS NAT Gateway. While the NAT Gateway effectively handles IP address translation, it is not intended to serve as a comprehensive security solution. This limited security capability has raised concerns among ABC Healthcare’s management regarding data privacy and HIPAA compliance.

## 2. Expensive and Limited CSP Native NAT Gateway

The current AWS NAT Gateway doesn’t provide the necessary visibility and logging capabilities, and it’s also very expensive due to 2 cents per GB egress data processing charges. ABC Healthcare’s average monthly egress traffic is around 500 TB.
The native solution also lacks visibility, is cost-prohibitive, and doesn’t support zero trust architecture—putting sensitive patient data and the healthcare provider’s reputation at risk.

### 2.1 Damaged Reputation and Employee Fired

To add to the challenges, ABC Healthcare recently experienced a data exfiltration attack, causing substantial disruptions, reputational harm, and a decline in its stock value. As a consequence, the company dismissed its previous cloud network architects.

### 2.2 You are the Newly Hired Cloud Networking Architect

As the newly appointed architect, your task is to secure this traffic using the Aviatrix Secure Egress solution. Your objective is to implement a solution that enhances visibility, offers comprehensive logging, and complies with regulatory requirements—all while being cost-effective and efficient.

## 3. LAB Objective

Your responsibility is to conduct a POC/POV in your lab environment and demonstrate how your company can leverage the Aviatrix Cloud Perimeter Solution to address this pain point. You should deploy the Aviatrix Secure Egress solution using the Aviatrix Spoke Gateway to protect internet-bound traffic more effectively than the AWS NAT Gateway.

<ins>The Zero Trust policy should allow only the specified domains and block all other FQDNs</ins>.

Please note that the lab provides only some of the steps needed to complete this exercise. If you encounter any difficulties, you are encouraged to consult the documentation at <a href="https://docs.aviatrix.com" target="_blank">docs.aviatrix.com</a>.

- `allowed-internet-http` domains
  - *.ubuntu.com
- `allowed-internet-https` domains
  - *.alibabacloud.com
  - azure.microsoft.com
  - aws.amazon.com
  - *.amazonaws.com
  - *.aviatrix.com
  - aviatrix.com
  - cloud.google.com
  - *.docker.com
  - *.docker.io
  - www.oracle.com

## 4. Aviatrix CoPilot

Now, let's access the Aviatrix Centralized Management Node, the **CoPilot**.

Navigate to your POD portal, click on the `"Copilot"` button, and enter your credentials.

Now let's access the Aviatrix Centralized Management Node, the CoPilot.

```{figure} images/lab-egress01.png
---
height: 300px
align: center
---
POD Portal
```

```{figure} images/lab-egress02.png
---
height: 400px
align: center
---
CoPilot UI - Welcome page
```

## 4. LAB Pre-Req

Before starting the lab, change the following timers to their lowest value

![Task Server](images/egress_timers.png)

```{warning}
DO NOT ATTEMPT TO CHANGE THESE TIMES IN PRE-PROD or PROD SETUP. THIS WOULD CAUSE SERIOUS ISSUES.
```

It should look like the following:

![Task Server](images/egress_timers_updated.png)

Do not change these times in the production setup

- Aviatrix Spoke GW must be deployed in Region us-east-1 in the “egress-vpc”
- GW Instance Size should be t3a.small

### 4.1 Successful Completion of LAB

After completing the lab, your screen should look more or less like the following. The IP addresses and UUIDs could be different.

![Topology](images/egress_topology.png)

![Gateway](images/egress_spoke.png)

![Monitor](images/egress_monitor.png)

![Filter](images/egress_monitor_filter.png)

![Routes](images/egress_vpc_routes.png)

## 5. Lab Hints

### 5.1 Create Secure Egress DCF Rules

- Create three rules
- The last DCF rule is a zero-trust rule
- Rule 100 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-https` web group
- Rule 0 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-http` web group

![DCF](images/egress_dcf_rules.png)

### 5.2 Create rfc1918 SmartGroup

![Group](images/egress_groups.png)

![rfc1918](images/egress_rfc1918.png)

### 5.3 Create WebGroup to Define FQDN Allowed to Access Internet

![WebGroup](images/egress_create_group.png)

![Edit Group](images/egress_edit_group.png)

![Polling](images/egress_polling.png)

### 5.4 Deploy Aviatrix Spoke GW

- The public IP address will be different (Public EIP automatically allocated by CSP)
- The Subnet CIDR could be different (automatically picked up by Aviatrix Controller)
- Region: us-east-1

![Spoke](images/egress_spoke_gw.png)

Check the Egress setting. The Egress traffic is going through the AWS NAT GW.

![Egress](images/egress_egress.png)

### 5.5 Enable spoke GW to become the Egress GW

1. Click +Local Egress on VPC/VNets.
2. In the Add Local Egress on VPC/VNets dialog, select the VPC/VNets on which to enable Local Egress.
3. Click Add.

[Read more at Aviatrix Documentation](https://docs.aviatrix.com/copilot/latest/network-security/index.html)

![Local](images/egress_add_local.png)

Add Local Egress on VPC/VNets
Adding Egress Control on VPC/VNet changes the default route on VPC/VNet to point to the Spoke Gateway and enables SNAT. Egress Control also requires additional resources on the Spoke Gateway.VPC/VNets

Now the diagram should look like the following:

![Vpc](images/egress_vpc.png)

## 6. Conclusion

Certainly! Here's a refined and polished version of your paragraph:


By implementing Aviatrix Secure Egress, our healthcare provider strengthened their security posture, reduced costs, and eliminated the visibility gaps associated with AWS NAT Gateway. Patient data remains protected, and the provider’s reputation is safeguarded.
Remember, Aviatrix Secure Egress is your trusted solution for secure and cost-effective management of internet-bound traffic. Need assistance? Our support team is here to help.
