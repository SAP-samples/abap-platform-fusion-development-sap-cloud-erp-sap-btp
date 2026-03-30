# SAP Cloud ERP – Fusion Development

This mission demonstrates how to extend SAP Cloud ERP by integrating with SAP Business Technology Platform (SAP BTP).  
The scenario showcases **Extending a Standard Sales Order**, **Loyalty Points Management**, and **End-to-End Integration** using SAP Build and SAP Event Mesh 

---

## SAP Cloud ERP

### 1. On-stack Developer Extensibility – Extend Sales Order with Gift Card
In this use case, we extend the standard **Sales Order** business object in **Cloud ERP** using **Key User and On-stack Developer Extensibility**.

The extension allows addition of a **gift card** to a sales order.  
Key steps include:
- Creating a custom business object for gift card.  
- Extending the sales order CDS view and behavior definition.  
- Implementing logic to validate and update gift card data.  
- Exposing OData APIs for integration.

🔗 **Reference:** [Custom BO for Gift Card Implementation](./01-SAP-Cloud-ERP/01-GIFT-CARD-IMPLEMENTATION)

---

### 2. Side by Side Extensibility - Build Loyalty Points Management Application on SAP BTP ABAP Environment
A **Loyalty Points Management** solution is developed using **SAP BTP ABAP Environment**.  
It enables customers to manage loyalty points based on their sales orders and gift card usage.

The application is built using:
- RAP (ABAP RESTful Application Programming Model)  
- Fiori Elements   
- Integration with SAP Cloud ERP via Event Mesh  

🔗 **Reference:** [Build Loyalty Application with BTP ABAP Environment using ABAP Cloud](./03-REUSE/01-LOYALTY-MANAGEMENT-APPLICATION/01-CREATE-LOYALTY-MGMT-APPLICATION)

---

### 3. Integration – Event Mesh
Event-driven communication is established between **SAP Cloud ERP** and **SAP BTP ABAP Environment** using **SAP Event Mesh**.

- SAP Cloud ERP publishes events.  
- Event Mesh routes events to the Loyalty Points Management solution in SAP BTP ABAP Environment for transaction processing.  
- Loyalty Points Management solution updates loyalty points and invokes tier upgrade workflows.  

🔗 **Reference:** [Integration via Event Mesh](./01-SAP-Cloud-ERP/02-INTEGRATION/01-EVENT-MESH)

---

### 4. Automate Tier Upgrade Workflow with SAP Build Process Automation
Tier upgrade requests by loyalty program customers are automated using **Build Process Automation**.

Workflow steps:
- Tier upgrade button in the Loyalty Points Application is invoked to instantiate a process. 
- Approval flow is routed to the approver's inbox.  
- Once approved, the tier level is updated in the Loyalty Points Management application.  

🔗 **Reference:** [SAP Build Process Automation Workflow](./03-REUSE/02-INTEGRATION/01-SAP_BUILD_PROCESS_AUTOMATION)

---

### 5. Unified Access via SAP Build Work Zone
All applications — **Create Sales Order**, **Manage Gift Card**, **Loyalty Points Management**, and **My Inbox from SAP Build Process Automation** — are accessible through **SAP Build Work Zone**.

Work Zone provides a unified, secure, and role-based access point for end users.

🔗 **Reference:** [Work Zone Setup](./03-REUSE/02-INTEGRATION/02-WORKZONE)

---




