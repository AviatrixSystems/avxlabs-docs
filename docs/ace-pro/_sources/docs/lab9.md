# Lab 9 - THREATIQ & THREATGUARD

## 1. Objective

This lab will demonstrate how `ThreatIQ` and `ThreatGuard` work (both features of the **_Distributed Cloud Firewall_** functionality).
 
## 2. ThreatIQ Overview

Aviatrix gateways send NetFlow data to CoPilot. CoPilot uses this data in many ways. **FlowIQ** is one. **ThreatIQ** is another. ThreatIQ alerts you on Malicious IPs with bad reputations. These IPs are reported in the ThreatIQ database that CoPilot maintains.

## 3. Topology

In this lab, we will be using the same topology that was used in Lab 6 with the Egress functionality.

![lab9-topology](images/lab9-topology.png)
_Figure 205: Lab 9 Initial Topology_

## 4. Configure ThreaIQ

### 4.1. SSH to the EC2 instance in AWS US-EAST-2

- SSH to the **_aws-us-east2-spoke1-test2_** instance in AWS aws-us-east2-spoke1 VPC.

```{warning}
Bear in mind that the **_aws-us-east2-spoke1-test2_** instance is in a private subnet, therefore, you have first to SSH to an EC2/VM that resides in a public subnet!
```

![lab9-ssh](images/lab9-ssh.png)
_Figure 206: SSH to aws-us-east2-spoke1-test2_

- Try launching the following commands from the **_aws-us-east2-spoke1-test2_** instance. 

```bash
curl www.aviatrix.com
```
```bash
curl www.firefox.com
```
```bash
curl www.wikipedia.com
```

![lab9-curl](images/lab9-curl.png)
_Figure 207: Curl commands_

```{note}
All egress traffic should be allowed, thanks to the `Greenfield-Rule` still in place.
```

4.2. Enable ThreatIQ and ThreatGuard

Navigate to **CoPilot > Security > ThreatIQ > Configuration**

Click on Send Alert:

![lab9-ssh](images/lab9-sendalert.png)
_Figure 208: Enable ThreatIQ_

Then click on **Notification Settings**.

![lab9-notification](images/lab9-notification.png)
_Figure 209: Notification Settings_

Now click on the `"+ Email Address"` button.

![lab9-email](images/lab9-email.png)
_Figure 210: Email_

Choose an **alias**, insert your **personal email **and then click on **Save**:

![lab9-email2](images/lab9-email2.png)
_Figure 211: Alias and Personal Email_

Navigate back to **CoPilot > Security > ThreatIQ > Configuration**

Click again on **Send Alert**:

![lab9-sendalert2](images/lab9-sendalert2.png)
_Figure 212: Send Alert Settings_

Click on **Add Recipients**.

![lab9-addrecipient](images/lab9-addrecipient.png)
_Figure 213: Add Recipient(s)_

Select your email address from the pulldown menu and then click on **Confirm**.

![lab9-addjoe](images/lab9-addjoe.png)
_Figure 214: Add your email_

From this point onwards, if you enter a valid email address, you will receive email notifications about ThreatIQ alerts.

Before enabling the blocking, on the far right side, ensure that the **ThreatGuard firewall rules order** is set to `Prepend`.

![lab9-advanced](images/lab9-advanced.png)
_Figure 215: Prepend_

Enable Block Threats (i.e. `ThreatGuard`):

![lab9-advanced](images/lab9-threatguard.png)
_Figure 216: ThreatGuard_

By default, **all** VPCs are enabled for ThreatGuard. 

Click **Save** to continue.

![lab9-vpc](images/lab9-vpc.png)
_Figure 217: Select VPC_

Then, click **CONFIRM**.

![lab9-confirm](images/lab9-confirm.png)
_Figure 218: Confirm_

## 5. Verification

Wait for the instructor to provide a malicious IP. Let's call it `<malicious-IP>`. 

<ins>Note down this IP address!</ins>

SSH back to the EC2 instance **_aws-us-east2-spoke1-test2_**

- Now test `ThreatGuard` by first issuing this command (make sure to enter **HTTPS**):

```bash
curl https://<malicious-IP>
```

![lab9-malicious](images/lab9-malicious.png)
_Figure 219: Curl towards the malicious IP_

Navigate back to **CoPilot > Security > ThreatIQ > Overview**

```{note}
Wait for some minutes, before proceeding with the next action. Furthermore, set the **Time Period** to `"Custom"` and then set the end time a bit farther than your current time:
```