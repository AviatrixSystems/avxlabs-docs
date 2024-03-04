# Lab 4 - RBAC

## 1. Objective

Implement **Role-Based Access Control** using Local Authentication both on the **Controller** and on the **CoPilot**.

You will perform tasks to address the following scenario:

- For the **Controller**: create an Operator that has RW access to **Dashboard**, **Useful Tools** and **Troubleshoot** sections for all cloud accounts.
- For the **CoPilot**: create an Operator that has RW access to **Dashboard**, **Topology**, **Monitor** and **Troubleshoot** sections for all cloud accounts.

```{note}
Notice that the account you are provided in the labs is a local user called **student** that is a member of the Permission Group <ins>"admin"</ins>.
```

## 2. Configuration: RBAC for the Controller

### 2.1. Create a Permission Group

Navigate to **CoPilot > Administration > User Access > Permission Group**

Click on `“+Permission Group”`.

```{figure} images/lab4-pgcopilot.png
---
align: center
---
Permission Group creation on the CoPilot
```

Name the group **"controller-operators"**, select all the **Access Accounts** available from the drop-down window and choose the **Controller Permissions**.

Then from the **API/Terraform Permissions** field, select **Dashboard**, **Useful Tools** and **Troubleshoot** privileges, as depicted below.

Click on **Save**.

```{figure} images/lab4-dropdown1.png
---
align: center
---
Permisionn Group configuration
```

In the list of permission groups you should see the new created **controller-operators**.

```{figure} images/lab4-controlleroperators.png
---
align: center
---
"controller-operators" Permission group
```

### 2.2. Create a new User

Navigate to **CoPilot > Administration > User Access > Users**

Click on `“+User”`.

```{figure} images/lab4-user.png
---
align: center
---
+User
```

Ensure to enter the following values:

- **Username:** <span style='color:#479608'>controller-op</span>
- **Email:** <span style='color:#479608'>[Enter your own]</span>
- **Password:** <span style='color:#479608'>[Pick a strong password and remember it]</span>
- **Confirm Password:** <span style='color:#479608'>[Repeat the previous password]</span>
- **Permission Groups:** <span style='color:#479608'>controller-operators</span>

Then click on **Save**.

```{figure} images/lab4-usercreation.png
---
align: center
---
Add User
```

In the list of users, you should see the new created **_controller-op_**.

```{figure} images/lab4-controllerop.png
---
align: center
---
controller-op
```

At this point, you should have received an email and a message in the **Notifications** tray, similar to the message below:

```{figure} images/lab4-notification.png
---
align: center
---
Notification message
```

## 3. Verification: RBAC on the Controller
 
### 3.1. Log out and log back in as new RBAC User (controller-op)

- Go to your personal POD portal and click on the URL of the **Controller**.

```{figure} images/lab4-credentials.png
---
align: center
---
POD Portal
```

Sign in as a **controller-op** with the same password you picked earlier.

```{figure} images/lab4-signin.png
---
align: center
---
Sign back with controller-op
```

You should see in the top-right corner that you are logged in as the **controller-op** RBAC User, a member of the **controller-operators** RBAC Group.

```{figure} images/lab4-rbac.png
---
align: center
---
RBAC sign-in
```

### 3.1.1. Positive Test

Navigate to **Controller > USEFUL TOOLS > Create a VPC** and click `"+ Add New"`

Create a new VPC as follows (make sure that **_Aviatrix Transit VPC_** is not selected).

- **Cloud Type:** <span style='color:#479608'>AWS</span>
- **Account Name:** <span style='color:#479608'>[your AWS account name will be auto-populated]</span>
- **VPC Name:** <span style='color:#479608'>rbac-lab-test</span>
- **VPC Region:** <span style='color:#479608'>us-east-1</span>
- **Advanced:** <span style='color:#479608'>unchecked</span>
- **Aviatrix Transit VPC:** <span style='color:#479608'>**unchecked**</span>
- **Aviatrix FireNet VPC:** <span style='color:#479608'>unchecked</span>

```{figure} images/lab4-createvpc.png
---
align: center
---
Creating a VPC from the Controller
```

You will receive confirmation that your VPC was created successfully!

```{figure} images/lab4-success.png
---
align: center
---
Successfully created
```

### 3.1.2. Negative Test

Navigate to **Controller > MULTI-CLOUD TRANSIT > Segmentation** and scroll down to **Step 3, Add / Modify Connection Policy**

Attempt to undo your work in the Network Segmentation Lab, by disconnecting the <span style='color:lightgreen'>Green</span> security domain from the <span style='color:lightblue'>Blue</span> one.

Select the Green domain first and then click on the **DEL** button.

```{figure} images/lab4-changesegmentation.png
---
align: center
---
Modify Segmentation
```

You should immediately receive an error in the Notifications tray such as this:

```{figure} images/lab4-error.png
---
align: center
---
Error
```

```{attention}
Before moving to the subsequent section, be sure to sign out as a **controller-op** and sign back in as a **student**.
```

## 4. Configuration: RBAC for the CoPilot
 
### 4.1. Create a new Permission Group

Navigate to **CoPilot > Administration > User Acces > Permission Group**

Click on `“+ Permission Group”`.

```{figure} images/lab4-permission2.png
---
align: center
---
New Permission Group
```

Ensure to insert the following values inside the pop-up window `“Create Permission Group”`:

- **Name**: <span style='color:lightgreen'>copilot-operators</span>
- **Users**: <span style='color:lightgreen'>[empty]</span>
- **Access Accounts**: <span style='color:lightgreen'>choose all the accounts</span>

Click on **“Clear All Views”**, and then select the following sections (and subsections):

- **Cloud Fabric** (and only the subsection **Topology**)
- **Monitor**
- **Diagnostics**

```{figure} images/lab4-pg2.png
---
align: center
---
Create Permission Group
```

```{figure} images/lab4-pg3.png
---
align: center
---
Create Permission Group (cont.ed)
```

You should notice a **Notification** message popping up.

```{figure} images/lab4-message.png
---
align: center
---
Successfully created
```

### 4.2. Create a new User

Navigate to **CoPilot > Administration > User Acces > Users** 

Click on `“+ User”`.

```{figure} images/lab4-newuser.png
---
align: center
---
+User
```

Ensure to enter the following values:

- **Username**: <span style='color:lightgreen'>copilot-op</span>
- **Email**: <span style='color:lightgreen'>[Enter your own]</span>
- **Password**: <span style='color:lightgreen'>[Pick a strong password and remember it]</span>
- **Confirm Password**: <span style='color:lightgreen'>[Repeat the password previously inserted]</span>
- **Permission Groups**: <span style='color:lightgreen'>copilot-operators</span>

Then click on **Save**.

This is the list of all users that you should be able to see at this point.

```{figure} images/lab4-list.png
---
align: center
---
List of users
```

## 5. Verification: RBAC on the CoPilot
 
### 5.1. Log out and log back in as new RBAC User (copilot-op)

Hover over the user icon in the top-right corner and click **Logout**.

```{figure} images/lab4-logout.png
---
align: center
---
Log out
```

Sign back in as a **copilot-op** with the same password you picked earlier.

```{figure} images/lab4-logincopilot.png
---
align: center
---
Log in CoPilot
```

```{attention}
You will immediately notice that the RBAC feature on the CoPilot is capable to limit to each persona the visibility of the platform, showing only those sections that have been permitted.

The user copilot-op can only interact with the Cloud Fabric section (but only with the sub-section **Topology**, whereby, this user is not authorized to deploy gateways), the **Monitor** section and the **Diagnostics** section.

The other sections are not more visible.
```

```{figure} images/lab4-newview.png
---
align: center
---
Restricted Dashboard view
```

```{note}
Be sure to sign out as a **copilot-op** and sign back in as a **student**, in order to complete this lab!
```
