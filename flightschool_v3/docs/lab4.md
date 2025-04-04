
# Lab-4: Observability and Troubleshooting

## Lab 4.1 - Dashboard

Investigate the dashboard for quick insight in your cloud environment, each tab in there will redirect you to their own pages for drill-down. 

## Lab 4.2 - Topology

Topology provides complete visibility to your cloud environment, including your VMs whether managed or unmanaged. And when you want to troubleshoot you can select the gateway and start taking advantage of embedded tools, such as packet capture, ping, traceroute, statistics and many more. 


```{figure} images-lab4/1.png
---
align: center
---
Embedded Troubleshooting Tools
```

## Lab 4.3 - Security	
So far, we have used this section extensively for troubleshooting as well as monitoring purposes. There are more to deep dive into ThreatIQ and Anomaly Detection, but they are out-of-scope of this lab. 

## Lab 4.4 - Cloud Resources	
This section provides a lot of insights about your cloud presence, including Virtual Machines, VPC/VNets & Subnets. Once enabled you can also gather a lot of details for K8s existence. 

## Lab 4.5 - FlowIQ
FlowIQ is the Aviatrix engine that processes the Netflow data and will enable you to drill-down all the events. 

```{figure} images-lab4/2.png
---
align: center
---
FlowIQ-Overview
```
```{figure} images-lab4/3.png
---
align: center
---
FlowIQ-Trends
```
```{figure} images-lab4/4.png
---
align: center
---
FlowIQ-Trends
```
```{figure} images-lab4/5.png
---
align: center
---
FlowIQ-Records
```
```{figure} images-lab4/6.png
---
align: center
---
FlowIQ-Flows
```

## Lab 4.6 - Performance
That section will help you to drill-down system performance data. 

```{figure} images-lab4/7.png
---
align: center
---
Performance Monitoring
```

## Lab 4.7 - Traffic & Latencies
This section will provide insights about traffic flows, top talkers from multiple perspective, and as well as provides historical data about latencies between sites. 
Fun fact: It will take just couple of seconds to find out which resource is generating up most traffic in your cloud deployment. 

```{figure} images-lab4/8.png
---
align: center
---
Traffic Monitoring
```

```{figure} images-lab4/9.png
---
align: center
---
Latency Monitoring
```

## Lab 4.8 - Notifications
Here you can gather the system notifications, create custom alerts, export the data to 3rd-party SIEM tools etc.

```{figure} images-lab4/10.png
---
align: center
---
System Alerts
```

## Lab 4.9 - AppIQ
Here, deep-dive views for the applications can be created as dashboard.

```{figure} images-lab4/11.png
---
align: center
---
AppIQ
```

## Lab 4.10 - FlightPath 
This is a very powerful troubleshooting tool, which will point-out exactly where the problem resides, for example you can try to check how TCP/555 will communicate between aws-instance-1 and aws-instance-1b. You will see that’s failing due to SG orchestration blocking such port, and it will be shown in the report, less than 45 seconds. So, no more lengthy troubleshooting sessions due limited visibility. 

```{figure} images-lab4/12.png
---
align: center
---
FlightPath
```

```{figure} images-lab4/13.png
---
align: center
---
Troubleshooting the traffic path with FlightPath for denied traffic
```

But if you try for the same source and destination but with port TCP/80 which we allowed in our lab, then you can see this traffic would work. To resolve the previous example, if you’ve gone to the DCF and add the port TCP/555 to that rule, that would also solve your issue in 1 minute!


```{figure} images-lab4/14.png
---
align: center
---
Troubleshooting the traffic path with FlightPath for allowed traffic
```

## Lab 4.11 - Diagnostic Tools
This section opens the doors for deep dive troubleshooting.

You can run networking diagnostic tools in every gateway, or you can dive into configuration details for the gateways, or run BGP commands on the gateways, check the controller’s state, UserVPN details, Firenet diagnostics. Here you will be exposed to all deep-dive troubleshooting tools, that you are missing in the cloud-native constructs.

```{figure} images-lab4/15.png
---
align: center
---
Diagnostic Tools
```

## Lab 4.12 - CloudRoutes
That section will allow you to check all routing tables, without having to go to the gateway’s individual sections. Provides very quick view into the routing domain. 

```{figure} images-lab4/16.png
---
align: center
---
Cloud Routes
```

## Lab 4.13 - Reports
You can create granular reports about your deployment and system. 


## Lab 4.14 - Upgrade
You can create upgrade plans, which system will apply automatically afterwards, without you needing to apply anything on the gateways individually.


```{figure} images-lab4/18.png
---
align: center
---
Software Upgrade
```

## Lab 4.15 - CostIQ 
That will help you to create views about the cloud resources’ usage and will enable you to chargeback to different divisions. With visibility, now you can see which division is the one owns the costs!


```{figure} images-lab4/19.png
---
align: center
---
CostIQ
```

## Lab 4.16 - Resources
This section will give detailed information and troubleshooting capabilities for the controller and copilot environments.

```{figure} images-lab4/20.png
---
align: center
---
System Health
```

## Lab 4.17 - Maintenance
You will be able to upgrade and apply patches to controller and manage backup in this section.


```{figure} images-lab4/21.png
---
align: center
---
Controller Upgrade
```

## Lab 4.18 - Support
You will be able to enable remote support, and upload log bundles directly from the system. 


```{figure} images-lab4/22.png
---
align: center
---
Aviatrix Support
```


# End of Lab 4

Ok, that's the end of Lab 4. Great job! 
