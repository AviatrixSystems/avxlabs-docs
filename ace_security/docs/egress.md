# Egress Lab

## Scenario

ABC Healthcare (a fictitious company), a leading healthcare provider, operates solely within the AWS cloud environment. For their internet egress traffic, they rely on AWS NAT Gateway. While AWS NAT Gateway serves well for IP address translation, it’s not designed to be a robust security solution. This lack of dedicated security has raised concerns about data privacy and HIPAA compliance among ABC Healthcare’s management.

## Expensive and Limited CSP Native NAT Gateway

The current AWS NAT Gateway doesn’t provide the necessary visibility and logging capabilities, and it’s also very expensive due to 2 cents per GB egress data processing charges. ABC Healthcare’s average monthly egress traffic is around 500 TB.
The native solution also lacks visibility, is cost-prohibitive, and doesn’t support zero trust architecture—putting sensitive patient data and the healthcare provider’s reputation at risk.

### Data Charges Reference

<a href="https://aws.amazon.com/blogs/networking-and-content-delivery/identify-and-optimize-public-ipv4-address-usage-on-aws" target="_blank">Networking and content delivery</a>

<a href="https://aws.amazon.com/blogs/apn/aws-data-transfer-charges-for-server-and-serverless-architectures" target="_blank">Data transfer</a>

<a href="https://www.cloudzero.com/blog/aws-egress-costs/#:~:text=Data%20transfers%20out%20of%20AWS,$0.0900%20and%20$0.0500%20per%20GB" target="_blank">Egress cost</a>

## Damaged Reputation and Employee Fired

To further complicate matters, ABC Healthcare recently suffered a data exfiltration attack, which led to significant disruptions, reputational damage, and a negative impact on its stock value. This incident resulted in the dismissal of the previous cloud network architects.

## You are the Newly Hired Cloud Networking Architect

You, the newly appointed architect, have been tasked with securing this traffic using the Aviatrix Secure Egress solution. Your mission is to implement a solution that enhances visibility, provides detailed logging, and complies with regulatory mandates while being cost-effective and efficient.

## LAB Objective

It is your job to do a POC/POV in your lab and demonstrate how your company can leverage Aviatrix Cloud Perimeter Solution to solve this pain point. You need to deploy the Aviatrix Secure Egress solution using Aviatrix Spoke Gateway to protect internet-bound traffic more effectively than the AWS NAT Gateway.
The Zero Trust policy should only allow the following domains and block all other FQDNs.
The lab intentionally only provides some of the steps for you to complete this lab. You should leverage <a href="https://docs.aviatrix.com" target="_blank">docs.aviatrix.com</a> if you are stuck.

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

## Listen to the following recording

Listen carefully. There will be quiz questions based on this 4 min video, also.
<iframe width="560" height="315" src="https://www.youtube.com/embed/cNx51ZJhxek?si=V83b5ledWF08f1lx" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


## LAB Access Details

| **POD#** | **Copilot** |
|:----------:|:---------------:|
|      1     |       <a href="https://cplt.pod1.aviatrixlab.com" target="_blank">POD1</a>     |
|      2     |       <a href="https://cplt.pod2.aviatrixlab.com" target="_blank">POD2</a>      |
|      3     |       <a href="https://cplt.pod3.aviatrixlab.com" target="_blank">POD3</a>      |
|      4     |       <a href="https://cplt.pod4.aviatrixlab.com" target="_blank">POD4</a>      |
|      5     |       <a href="https://cplt.pod5.aviatrixlab.com" target="_blank">POD5</a>     |
|      6     |       <a href="https://cplt.pod6.aviatrixlab.com" target="_blank">POD6</a>      |
|      7     |       <a href="https://cplt.pod7.aviatrixlab.com" target="_blank">POD7</a>      |
|      8     |       <a href="https://cplt.pod8.aviatrixlab.com" target="_blank">POD8</a>      |
|      9     |       <a href="https://cplt.pod9.aviatrixlab.com" target="_blank">POD9</a>     |
|      10     |       <a href="https://cplt.pod10.aviatrixlab.com" target="_blank">POD10</a>      |
|      11     |       <a href="https://cplt.pod11.aviatrixlab.com" target="_blank">POD11</a>      |
|      12     |       <a href="https://cplt.pod12.aviatrixlab.com" target="_blank">POD12</a>      |
|      13     |       <a href="https://cplt.pod13.aviatrixlab.com" target="_blank">POD13</a>     |
|      14     |       <a href="https://cplt.pod14.aviatrixlab.com" target="_blank">POD14</a>      |
|      15     |       <a href="https://cplt.pod15.aviatrixlab.com" target="_blank">POD15</a>      |
|      16     |       <a href="https://cplt.pod16.aviatrixlab.com" target="_blank">POD16</a>      |
|      17     |       <a href="https://cplt.pod17.aviatrixlab.com" target="_blank">POD17</a>     |
|      18     |       <a href="https://cplt.pod18.aviatrixlab.com" target="_blank">POD18</a>      |
|      19     |       <a href="https://cplt.pod19.aviatrixlab.com" target="_blank">POD19</a>      |
|      20     |       <a href="https://cplt.pod20.aviatrixlab.com" target="_blank">POD20</a>      |

## Access credentials

Username:

```bash
student
```

Password:

```bash
1012fw633#SYTY3
```

## LAB Pre-Req

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

## Successful Completion of LAB

After completing the lab, your screen should look more or less like the following. The IP addresses and UUIDs could be different.

![Topology](images/egress_topology.png)

![Gateway](images/egress_spoke.png)

![Monitor](images/egress_monitor.png)

![Filter](images/egress_monitor_filter.png)

![Routes](images/egress_vpc_routes.png)

## Lab Hints

### Create Secure Egress DCF Rules

- Create three rules
- The last DCF rule is a zero-trust rule
- Rule 100 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-https` web group
- Rule 0 is to allow traffic from the test instance on the private IP address to the public internet only to FQDNs specified in the `allowed-internet-http` web group

![DCF](images/egress_dcf_rules.png)

### Create rfc1918 SmartGroup

![Group](images/egress_groups.png)

![rfc1918](images/egress_rfc1918.png)

### Create WebGroup to Define FQDN Allowed to Access Internet

![WebGroup](images/egress_create_group.png)

![Edit Group](images/egress_edit_group.png)

![Polling](images/egress_polling.png)

### Deploy Aviatrix Spoke GW

- The public IP address will be different (Public EIP automatically allocated by CSP)
- The Subnet CIDR could be different (automatically picked up by Aviatrix Controller)
- Region: us-east-1

![Spoke](images/egress_spoke_gw.png)

Check the Egress setting. The Egress traffic is going through the AWS NAT GW.

![Egress](images/egress_egress.png)

### Enable spoke GW to become the Egress GW

1. Click +Local Egress on VPC/VNets.
2. In the Add Local Egress on VPC/VNets dialog, select the VPC/VNets on which to enable Local Egress.
3. Click Add.

[Read more at Aviatrix Documentation](https://docs.aviatrix.com/copilot/latest/network-security/index.html)

![Local](images/egress_add_local.png)

Add Local Egress on VPC/VNets
Adding Egress Control on VPC/VNet changes the default route on VPC/VNet to point to the Spoke Gateway and enables SNAT. Egress Control also requires additional resources on the Spoke Gateway.VPC/VNets

Now the diagram should look like the following:

![Vpc](images/egress_vpc.png)

## Conclusion

By bringing Aviatrix Secure Egress into play, our healthcare provider shored up their defenses, dropped the high costs, and eliminated the visibility black hole courtesy of the AWS NAT Gateway. Sensitive patient data is safe, and the provider’s reputation will be secured.
Remember, Aviatrix Secure Egress is your go-to for a secure, cost-effective solution for managing internet-bound traffic. Need more help? Our support team’s got your back.
