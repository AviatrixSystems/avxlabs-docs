# Lab 4 - RBAC

## 1. OBJECTIVE

Implement Role-Based Access Control using Local Authentication both on the **Controller** and on the **CoPilot**.

You will perform tasks to address the following scenario:

- For the **Controller**: create an Operator that has RW access to Dashboard, Useful Tools and Troubleshoot sections for all cloud accounts.
- For the **CoPilot**: create an Operator that has RW access to Dashboard, Topology, Monitor and Troubleshoot sections for all cloud accounts.

```{note}
Notice that the account you are provided in the labs is a local user called **student** that is a member of the Permission Group <ins>admin</ins>.
```

## 2. CONFIGURATION: RBAC FOR THE CONTROLLER

### 2.1. CREATE A PERMISSION GROUP

Navigate to **CoPilot > Administration > User Access > Permission Group**

Click `“+Permission Group”`.

![lab4-pgcopilot](images/lab4-pgcopilot.png)
_Figure 89: Permission Group creation on the CoPilot_

Name the group **controller-operators**, select all the **Access Accounts** available from the drop-down window and choose the **Controller Permissions**.

Then from the **API/Terraform Permissions** field, select **Dashboard**, **Useful Tools** and **Troubleshoot** privileges, as depicted below.

Click on **Save**.

![lab4-dropdown1](images/lab4-dropdown1.png)
_Figure 90: Permisionn Group configuration_

In the list of permission groups you should see the new created controller-operators.

![lab4-dropdown1](images/lab4-controlleroperators.png)
_Figure 91: "controller-operators" Permission group_

### 2.2. Create a User

Navigate to **CoPilot > Administration > User Access > Users**

Click `“+User”`.

![lab4-user](images/lab4-user.png)
_Figure 92: +User_

Ensure to enter the following values:

- **Username:** <span style='color:#33ECFF'>controller-op</span>
- **Email:** <span style='color:#33ECFF'>< Enter your own ></span>
- **Password:** <span style='color:#33ECFF'>< Pick a strong password and remember it ></span>
- **Confirm Password:** <span style='color:#33ECFF'>< Repeat the previous password ></span>
- **Permission Groups:** <span style='color:#33ECFF'>controller-operators</span>

Then click on **Save**.

![lab4-adduser](images/lab4-usercreation.png)
_Figure 93: Add User_

In the list of users, you should see the new created **_controller-op_**.

![lab4-adduser](images/lab4-controllerop.png)
_Figure 94: controller-op_

At this point, you should have received an email and a message in the **Notifications** tray such as this:

![lab4-notification](images/lab4-notification.png)
_Figure 95: Notification message_

## 3. Verification: RBAC on the Controller
 
### 3.1. Log out and log back in as an RBAC user

Go to **Controller** and then hover over the user icon in the top-right corner and click on **Sign out**.

![lab4-signbout](images/lab4-signout.png)
_Figure 96: Sign out_

Sign back in as a **controller-op** with the same password you picked earlier.

![lab4-signin](images/lab4-signin.png)
_Figure 97: Sign back with controller-op_

You should see in the top-right corner that you are logged in as the **controller-op** RBAC User, a member of the **controller-operators** RBAC Group.

![rbac](images/lab4-rbac.png)
_Figure 98: RBAC sign-in_

### 3.1.1. Positive Test

Navigate to **Controller > USEFUL TOOLS > Create a VPC** and click **+ Add New**

Create a new VPC as follows (make sure that **_Aviatrix Transit VPC_** is not selected).

- **Cloud Type:** <span style='color:#33ECFF'>AWS</span>
- **Account Name:** <span style='color:#33ECFF'>[your AWS account name will be auto-populated]</span>
- **VPC Name:** <span style='color:#33ECFF'>rbac-lab-test</span>
- **VPC Region:** <span style='color:#33ECFF'>us-east-1</span>
- **VPC CIDR:** <span style='color:#33ECFF'>10.0.150.0/24</span>
- **Advanced:** <span style='color:#33ECFF'>unchecked</span>
- **Aviatrix Transit VPC:** <span style='color:#33ECFF'>**unchecked**</span>
- **Aviatrix FireNet VPC:** <span style='color:#33ECFF'>unchecked</span>

![create vpc from controller](images/lab4-createvpc.png)
_Figure 99: Creating a VPC from the Controller_
