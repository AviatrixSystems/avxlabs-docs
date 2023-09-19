# Challenge 3 - Network Isolation

## Scenario

You have received a call just now from the Web VM owner that reported that the remote Database route is not advertised.

![Lab Overview](images/network-isolation.png)
_Figure 7: Network Isolation_

* Search the Private IP of the Database VM and try to ping it from the Web Spoke GW. Does the ping work?

**HINT**: Check the RTBs of all the gateways involved in the path between the Web VM and the Database VM!


By the end of this challenge you need to ensure that the Database route has been propagated throughout the whole cloud network (i.e. Transit and Spoke GWs).
