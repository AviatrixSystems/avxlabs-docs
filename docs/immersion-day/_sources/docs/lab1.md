# Lab 1: AWS Networking Fundamentals

## Create VPCs, IGWs, and EC2 instances

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.

An internet gateway (IGW) is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet. It therefore imposes no availability risks or bandwidth constraints on your network traffic.
In this lab, we will create three VPC’s with Internet Gateways.

The lab modules build upon each other. Be sure to follow each step completely, build out in the specified region, and take note of IP addresses and CIDRs to ensure that future lab modules will work correctly.

![LAB1](images/image5-new.png)

## Navigate to CloudFormation

### 1.1.1

If you have not already signed into event engine, open the link you received in your email. Let the instructor know if you have any issues getting logged in. For instructions on using event engine, [click here](eventengine.md)

### 1.1.2

To get started, navigate to [VPC Dashboard Services](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks). Make sure that in the upper right-hand corner of the console it says `N. Virginia` (if not select N. Virginia from the drop down list)

![LAB1](images/region20211109.png)

### 1.1.3

Navigate to Cloudformation by searching for Cloudformation in Services search bar.

![LAB1](images/image6-20211109.png)

### 1.1.4

Click `CreateStack` to launch the CloudFormation stack for this lab. Accept the defaults on each page, and select `Create Stack`. Should you need to manually deploy the template [click here](https://s3.us-east-1.amazonaws.com/aws-immersion-day.aviatrixlab.com/AWS-ImmersionDay-lab-1-new.yaml)

### 1.1.5

Refresh every few minutes until the stack is in the status `CREATE_COMPLETE`

![LAB1](images/stackEvent20211109.png)

![LAB1](images/stackComplete20211109.png)

### 1.1.6

Using figure1 below as a reference, validate that the infrastructure has been created as desired

![LAB1](images/firgure1-20211109.png)

### 1.1.6.1

Navigate to the VPC service, validate that the VPCs and subnets have been created as follows:

![LAB1](images/vpcList20211109.png)

![LAB1](images/subnetList20211109.png)

**Note:** You may see a third subnet in each VPC, this is used later in the lab and can be ignored.

### 1.1.6.2

You should also have an IGW for the default VPC and three newly created IGWs available and attached to the VPCs:

![LAB1](images/IGWList20211109.png)

### 1.1.6.3

You should also have an IGW for the default VPC and three newly created IGWs available and attached to the VPCs:

![LAB1](images/IGWList20211109.png)

### 1.1.6.4

Validate that the route tables for the VPC have been created, and updated to direct Internet-bound traffic to the IGW for each VPC. The Route Tables are named VPC A Route Table, VPC B Route Table, and VPC C Route Table.

![LAB1](images/routeTable2011109.png)

### 1.1.6.5

Navigate to the EC2 service, validate that there are 2 EC2 instances in the “running” state.

![LAB1](images/InstanceList20211109-new.png)

### 1.1.6.6

For each EC2 instance, the Security Group rules under the Security tab should allow SSH (TCP port 22) and ICMP.

![LAB1](images/InstanceSelectA2011109-new.png)

![LAB1](images/SecurityGroup20211109.png)

![LAB1](images/SecurityGroupRules20211109.png)

This completes Lab 1.
