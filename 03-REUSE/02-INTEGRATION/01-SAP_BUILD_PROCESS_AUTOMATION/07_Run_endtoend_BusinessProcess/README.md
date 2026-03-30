# Run End-to-End Business Process for Loyalty Application.

The end-to-end business process starts from creating a sales order, requesting tier upgrade, checking the inbox in SAP Build Process Automation for triggered workflow processes, actioning the approval form and also checking for tier status update during workflow trigger and call back.
   
1. Creating a sales order.
  
2. Go to Service Binding of your application. Choose **Entity > Preview**.

<img src="./IMAGES/E2EApplication/8-Preview.png" alt="Preview" width="90%">

3. Choose one entry from application.

<img src="./IMAGES/E2EApplication/1-ApplnOverview.png" alt="App In Overview" width="90%">

4. Click on **Request Tier Upgrade** button.

<img src="./IMAGES/E2EApplication/2-ReqTierUpgrade.png" alt="Request Tier Upgrade" width="90%">

5. Workflow raised for tier approval with tier status as waiting for approval.

<img src="./IMAGES/E2EApplication/3-Requested.png" alt="Requested Status" width="90%">

## Go to SAP Build Process Automation to Action the Workflow.

1. From the SAP Build Process Automation Lobby, choose **My Inbox** to access the Approval Form.

<img src="./IMAGES/E2EApplication/4-Myinbox.png" alt="My Inbox" width="90%">

Here you can see the workflow task created.

2. In the TierApprovalForm choose the task for customer that you created. This will display the Approval Form with the Loyalty Membership Details.

<img src="./IMAGES/E2EApplication/5-Inboxdetails.png" alt="Inbox Details" width="90%">

3. Click **Approve** to approve the Tierapproval form. A message **Task Processed Successfully** appears on successful approval of the form.

<img src="./IMAGES/E2EApplication/6-TaskProcessed.png" alt="Task Processed" width="90%">

## Go back to the Loyalty Application.

In this you can Tier status turned **Active** for Silver Tier and Previous Tier Status became **Inactive**.

<img src="./IMAGES/E2EApplication/7-InactiveSts.png" alt="Inactive Status" width="90%">

<!---
➡️ [Build a Unified Site with Workzone](../../../../03-REUSE/02-INTEGRATION/02-WORKZONE/0_Setup_workzone)
--->




