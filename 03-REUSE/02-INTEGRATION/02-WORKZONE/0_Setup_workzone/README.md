# [Set Up SAP Build Work Zone, Standard Edition](https://help.sap.com/docs/build-work-zone-standard-edition/sap-build-work-zone-standard-edition/initial-setup)
<!-- description --> To get started with building a site in SAP Build Work Zone, standard edition, you must perform the required onboarding steps

## You will learn
  - How to subscribe to SAP Build Work Zone, standard edition
  - How to assign yourself to the `Launchpad_Admin` role so that you can create and manage sites in SAP Build Work Zone, standard edition
  - How to access SAP Build Work Zone, standard edition

## Initial Setup of SAP Build Work Zone, Standard Edition

Before you can start working with **SAP Build Work Zone, standard edition**, you need to perform the steps described in this section.

### 1. Configure Entitlement

1. In the menu of the **SAP Business Technology Platform cockpit**, go to the **Entitlements** screen.
2. Click **Edit**, then click **Add Service Plans** to open the list of service plans. 
3. Select from the list of available plans (previously assigned to your global account) the plans that are relevant to the subaccount. 
4. Click **Add Service Plans**.  
5. For more information about which service plans are applicable to your use case, see **Service Plans and Metering**. 
    After assigning the appropriate entitlements, the selected plans will be added as tiles to the **Service Marketplace** screen.

---

### 2. Create Subscription (Application Type Standard Plan)

1. In the side menu of the **SAP Business Technology Platform cockpit**, go to **Services → Service Marketplace**.  
2. Search for **SAP Build Work Zone, standard edition**, and click the tile.  
3. In the top-right corner, click **Create** to create a subscription to the service. 
4. In the window that opens, keep all default settings and click **Create**.  
   For more information, see **Subscribe to Multitenant Applications Using the Cockpit**.

---

### 3. Create Service Instance and Service Key (Standard Plan)

1. In the side menu of the **SAP Business Technology Platform cockpit**, go to **Services → Instances and Subscriptions**.  
2. In the top-right corner, click **Create** and fill in the details for the new service instance.  

> **Note:**  
> - You must enable **Cloud Foundry** and create a space before creating a service instance  
> - For more details, see **Creating Service Instances in Cloud Foundry**  
> - Uploading configuration parameters when creating a service instance for SAP Build Work Zone, standard edition, is **not supported**

3. To create a **service key**, in the **Instances and Subscriptions** screen, click the **(Actions)** icon next to the service instance and create a service key.  
   For more information, see **Creating Service Keys in Cloud Foundry**.

---

### 4.  Assign Users to the Launchpad_Admin Role Collection

> **Note:**  
> The **Launchpad_Admin** role collection includes the following roles:
> - **Super_Admin:** Enables users to perform all administrative tasks in the Site Manager as well as manage role collections in the SAP Business Technology Platform cockpit  
> - **Theme_Admin**, **Editor**, and **Viewer:** Allow users to manage themes in the UI Theme Designer

1. In the side menu of the **SAP Business Technology Platform cockpit**, go to **Security → Role Collections**.  
2. Search for the role collection **Launchpad_Admin**.  
3. Open the role collection for editing by clicking the arrow on the far right, then click **Edit**.  
4. In the **Users** section, search for your email and click the **+** button to add it to the list of users.  
5. Click **Save**.

> **Note:**  
> It might take a few minutes for the admin role assignment to take effect. Until it does, you might get an **“Access Denied”** error when you click **Go to Application**

Alternatively, you can perform the same admin role assignment in **Security → Users**, by searching for your email and assigning it to the **Launchpad_Admin** role collection.

> New subscriptions now use the **Identity Authentication Service (IAS)** as the default authentication mechanism.  
> You must create a **trust using OIDC protocol** between your SAP Business Technology Platform subaccount and Identity Authentication.  
> For more information, see **Establish Trust and Federation Between SAP Authorization and Trust Management Service and SAP Cloud Identity Services**.

---

### 5. Access SAP Build Work Zone, Standard Edition

1. In the side menu of the **SAP Business Technology Platform cockpit**, go to **Services → Instances and Subscriptions**.  
2. Under the **Subscriptions** tab, find **SAP Build Work Zone, standard edition**.  
3. Click the **(Actions)** icon and select **Go to Application**.  
4. This opens the **Site Directory**, where you can create new sites.

---

<!-----
➡️ [Create a Site Using SAP Build Work Zone, standard edition](.././1_Create_a_Site_Using_SAP_Build_Work_Zone)
----->

