# 📄 Creating an SAP Fiori App Using SAP Business Application Studio and Deploy It to SAP Cloud ERP

This topic describes how to create an application based on custom SAP Fiori Elements list report in SAP Business Application Studio and deploy it to an SAP Cloud ERP system.

> **Note**  
> Prerequisite:  
> Ensure that a Dev Space of the SAP Fiori type is created in the Dev Space Manager of SAP Business Application Studio.
> Ensure the userid is added as an org user in the SAP BTP subaccount. 

> The object names/coding style used here, if any, is not a recommendation on coding guidelines/standards

---

## 🚀 Deployment Configuration

To deploy an app in SAP Cloud ERP, you need to use the deployment configuration. The target and destination are prefilled with the destination chosen earlier in the wizard. The package and the transport are self-explanatory.

The following objects are created after deployment:
- BSP application (WAPA type)
- SAP Fiori launchpad descriptor item (UIAD type)

---

## 🎛 SAP Fiori Launchpad Configuration

You need to configure the SAP Fiori launchpad so that you can display the app as a tile on the SAP Cloud ERP Fiori launchpad. The semantic object and action must be a unique combination. These are used for intent-based navigation on the launchpad.

The title and subtitle determine the texts displayed on the launchpad tile. Note that this information is encoded in the generated SAP Fiori launchpad descriptor item object (UIAD type).

Once you finish using the wizard and the project is generated, the **Application Info** view opens automatically. This view offers options to preview and deploy your app. You can also open the Application Info view by choosing:

View > Find Command > Fiori: Open Application Info


## 🛠 Configure the SAP Fiori Launchpad:

1️⃣ Log on to SAP Business Application Studio using your SAP BTP subaccount.

> For **Tutorial on Fiori Deployment** refer to [Create an SAP Fiori App and Deploy it to SAP BTP, ABAP Environment](https://developers.sap.com/tutorials/abap-environment-deploy-fiori-elements-ui.html#b244534e-c5a8-407b-9b14-00c9f15c1bc2).

2️⃣ Create a new folder called **`zxe4_giftcard`** in the projects folder.  

3️⃣ On the **Get Started** page, choose **Start from Template**.  
If the Get Started page is not visible, enter the following command in the search bar: 

```abap
> Help: Get Started
```

4️⃣ On the **Select Template and Target Location** screen, choose **SAP Fiori Application** and then choose **Start**.

5️⃣ On the **Template Selection** screen, choose **List Report Page**.

6️⃣ Choose **Next**.

7️⃣ On the **Data Source and Service Selection** screen choose **Connect to a System** from the **Data Source** menu.

   - From the System menu, choose the destination created for the SAP S/4HANA Cloud system.
   - Enter the service's user name and password.
   - From the Service menu, choose **ZXE4_UI_GIFTCARD_V4**.

```abap
Note
If you get an error message, enter the following command:
CF: Login to Cloud Foundry
```

8️⃣ Choose **Next**.

9️⃣ On the **Entity Selection** screen enter the **Main Entity** menu and choose **GiftCard**. 
Choose **Yes** to automatically add table columns to the list page and a section to the object page, if not available.

🔟 Choose **Next**.


## 📋 Project Attributes

On the **Project Attributes** screen, enter the following data:

- **Module Name**: `zxe4_giftcard`
- **Application Title**: `Gift Card Details`
- **Application Namespace**: `zxe4_giftcard`
- **Description**: `Application to maintain Gift Card details`
- **Project Folder Path**: `/home/user/projects`
- **Add Deployment Configuration**: `Yes`
- **Add FLP Configuration**: `Yes`
- **Configure Advanced Options**: `Yes`
- **UI5 Theme**: `Morning Horizon`
- **Add Eslint Configuration to the Project**: `No`
- **Use Virtual Endpoints for Local Preview**: `Yes`
- **Enable TypeScript**: `No`

Choose **Next**.

## 📄 Deployment Configuration

On the **Deployment Configuration** screen, enter the following data:

- **SAPUI5 ABAP Repository**: `ZXE4_GIFTCARD`
- **Deployment Description**: `Gift Card Details`
- **Transport Request**: `<Enter your transport request>`
- **Package**: `ZXE_S4`

Choose **Next**.

## 📄 Fiori Launchpad Configuration

On the **Fiori Launchpad Configuration** screen, enter the following data:

- **Semantic Object** : `GiftCard`
- **Action**: `maintain`
- **Title**: `Gift Card Details`
- **Subtitle**: `Maintain`

Choose **Finish**.

## Preview the Application

You can preview the application from the terminal using the following command:

```abap
npm run start
```


# 📦 Deployment

You can start the deployment from **Application Info** or from the terminal with the following command:

```abap
npm run deploy
```

Once started, you can review the deployment configuration and confirm your settings in the terminal by entering **Y**.
Once the terminal output stops and you see a success message, the process is complete.
You can use the **ABAP Developer Tools** with Eclipse to check whether the following objects were generated in the package `ZXE_S4`:

   - **ZXE4_GIFTCARD_UI5R** (the launchpad app descriptor item)
   - **ZXE4_GIFTCARD** (BSP application)

---
# 🛡 Creating an IAM App, a Business Catalog, and Assign the IAM Apps to the Business Catalog

## 🎯 Creating an IAM App

1️⃣ Right-click the package **`ZXE_S4`** and choose **New > Other ABAP Repository Object**

2️⃣ Search for **IAM App**, select it, and choose **Next**.

3️⃣ Enter the following data:
   - **Name:** `ZXE4_GIFTCARD_UI`
   - **Description:** `IAM App for Gift Card Scenario`
   - **Application Type:** `EXT = External App`

4️⃣ Choose **Next**.

5️⃣ Select a transport request and choose **Finish**.

6️⃣ In the **General** section of the **Overview** tab of the IAM app, enter the following data:
   - **Fiori Launchpad App Descr ID:** `ZXE4_GIFTCARD_UI5R`

7️⃣ Choose **Save** and then choose **Publish Locally**.

---

## 📚 Creating a Business Catalog and Assigning IAM Apps

1️⃣ Right-click the package **`ZXE_S4`** and choose **New > Other ABAP Repository Object**

2️⃣ Search for **Business Catalog**, select it, and choose **Next**.

3️⃣ Enter the following data:
   - **Name:** `ZBC_XE4_GIFTCARD`
   - **Description:** `Business Catalog for Gift Card Scenario`

4️⃣ Choose **Next**.

5️⃣ Select a transport request and choose **Finish**.

6️⃣ Choose the **Apps** tab.

7️⃣ In the **Apps** section of the business catalog, choose **Add…**.

8️⃣ Enter the following data:
   - **IAM App:** `ZXE4_GIFTCARD_UI_EXT`

9️⃣ Choose **Next**.

🔟 Select a transport request and choose **Finish**.

---

## 📤 Publish the Business Catalog

✅ In the `ZBC_XE4_GIFTCARD` business catalog, choose **Publish Locally**.

---

### 👥 Result

This business catalog can now be included in a **business role** by the administrator, which can then be assigned to a **business user**.  
Users with this business role can find your custom app when they use the **SAP Fiori launchpad**.


---

# 👥 Creating a Business Role and Assigning It to a Business User

## 🔐 Steps to Create and Assign the Business Role

1️⃣ Log on to the **SAP Fiori launchpad** with a user that has the **Administrator (SAP_BR_ADMINISTRATOR)** role assigned in the **customizing client (100)** of the development system.

2️⃣ Search for the **Maintain Business Roles** app and select it.

3️⃣ In the **Maintain Business Roles** app, choose **New** and enter the following data:
   - **Business Role ID:** `ZBR_XE4_GIFTCARD`
   - **Business Role Description:** `Gift Card Scenario`

4️⃣ Choose **Create**.

---

## 📚 Assign the Business Catalog

5️⃣ In the **Assigned Business Catalogs** section, choose **Add**.

6️⃣ Search for the **`ZBC_XE4_GIFTCARD`** business catalog, select it, and choose **OK**.

7️⃣ Close the **Add Business Catalog** dialog box.

---

## 🔗 Set Access Categories

8️⃣ In the **Access Categories** section of the **General Role Details** tab:
   - Change the value of the **Write, Read, Value Help** field to `Unrestricted`.
   - Change the value of the **Read, Value Help** field to `Unrestricted`.

---

## 👤 Assign the Role to a User

9️⃣ Go to the **Assigned Business Users** tab and choose **Add**. Search for the business user that needs to be enriched with the Gift Card Scenario business role.

🔟 Select the user, and choose **OK**. Choose **Save**.

---

## ✅ Result

The newly created **Gift Card Scenario (`ZBR_XE4_GIFTCARD`)** business role provides the user with access to the **gift card maintenance app**.


# 🎁 **Scenario Execution**

## 1️⃣ Enter the Gift Card Details

1. Log on to the customizing client of the development system with the user that has the **Gift Card Scenario (ZBR_XE4_GIFTCARD)** business role assigned to it.

2. On the SAP Fiori Launchpad, search for **Gift Card Details** and select it.

3. In the **Gift Card Details** app, choose **Create**.

4. Enter the following data:
   - **Gift Card Number**: `1001`
   - **Description**: `EUR 20 Gift Card`
   - **Gift Card Amount**: `20.00`
   - **Currency**: `EUR`

5. Choose **Create**.  
   - A gift card worth EUR 20.00 has been created.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 2️⃣ Create a Sales Order and Include a Gift Card

1. Log on to the customizing client of the development system with the user that has the role based on the **SAP_BR_INTERNAL_SALES_REP** business role template assigned to it.

2. On the SAP Fiori Launchpad, search for **Manage Sales Orders** and select it.

3. Choose **Create > Create Sales Order**.

4. Enter the following data:
   - **Sales Order Type**: `Standard Order (OR)`
   - **Sales Organization**: `Dom. Sales Org DE (1010)`
   - **Distribution Channel**: `DE distribution chan (10)`
   - **Division**: `Product Division 00 (00)`

5. Choose **Continue**.

6. On the **General Information** tab, in the **Basic Data** section, enter the following data:
   - **Sold-to Party**: `<Enter Customer ID>`

7. On the **Items** tab, enter the following data for the newly created line item:
   - **Product**: `Trad.Good 11, PD, Reg.Trading (TG11)`
   - **Requested Quantity**: `10 Pc`

8. Since the net value of the sales order is EUR 175.50 and it exceeds EUR 50, the **Use Gift Card** action on the **General Information** tab is active.

9. Choose **Use Gift Card** and enter the following data:
   - **Gift Card**: `EUR 20 Gift Card (1001)`

10. Choose **Use Gift Card**.

11. On the **Prices** tab of the sales order, go to **Price Elements**.
    - A price element of the **DRV1** condition type is created, with the gift card amount deducted from the net value of the sales order.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

➡️ [Build Loyalty Application with Steampunk](../../../03-REUSE/01-LOYALTY-MANAGEMENT-APPLICATION/01-CREATE-LOYALTY-MGMT-APPLICATION)
