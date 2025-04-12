# Aviatrix Cloud Firewall Lab

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
|      32     |       <a href="https://cplt.pod32.aviatrixlab.com" target="_blank">POD32</a>     |
|      33     |       <a href="https://cplt.pod33.aviatrixlab.com" target="_blank">POD33</a>     |
|      34     |       <a href="https://cplt.pod34.aviatrixlab.com" target="_blank">POD34</a>     |
|      35     |       <a href="https://cplt.pod35.aviatrixlab.com" target="_blank">POD35</a>     |
|      36     |       <a href="https://cplt.pod36.aviatrixlab.com" target="_blank">POD36</a>     |
|      37     |       <a href="https://cplt.pod37.aviatrixlab.com" target="_blank">POD37</a>     |
|      38     |       <a href="https://cplt.pod38.aviatrixlab.com" target="_blank">POD38</a>     |
|      39     |       <a href="https://cplt.pod39.aviatrixlab.com" target="_blank">POD39</a>     |
|      40     |       <a href="https://cplt.pod40.aviatrixlab.com" target="_blank">POD40</a>     |
|      41     |       <a href="https://cplt.pod41.aviatrixlab.com" target="_blank">POD41</a>     |
|      42     |       <a href="https://cplt.pod42.aviatrixlab.com" target="_blank">POD42</a>     |
|      43     |       <a href="https://cplt.pod43.aviatrixlab.com" target="_blank">POD43</a>     |
|      44     |       <a href="https://cplt.pod44.aviatrixlab.com" target="_blank">POD44</a>     |
|      45     |       <a href="https://cplt.pod45.aviatrixlab.com" target="_blank">POD45</a>     |
|      46     |       <a href="https://cplt.pod46.aviatrixlab.com" target="_blank">POD46</a>     |
|      47     |       <a href="https://cplt.pod47.aviatrixlab.com" target="_blank">POD47</a>     |
|      48     |       <a href="https://cplt.pod48.aviatrixlab.com" target="_blank">POD48</a>     |


## Access credentials

Username:

```bash
student
```

Password:

```bash
127#rdpn8K4GBJ
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
