# 🚀Deploying Loyalty Management Application using Quick Fiori Deploy 

This topic describes how to deploy the Loyalty Management Application using Quick Fiori Deploy in ABAP Development Tools.

---



1️⃣ Create a separate package for UI artefacts

1. Create a new package "ZLOYALTY_MGMT_UI" for the storing the UI artefacts under the existing package "ZLOYALTY_MGMT". This is to achieve better separation of concerns. It should also be possible to deploy within the same package instead of creating a subpackage.
   
   - Fill in the super package as "ZLOYALTY_MGMT".
   - Click on next after filling in the package name and description.

    <img src="./IMAGES/Creation_of_UI_artefacts_package.png" alt="Package" width="50%">

2. Choose an existing transport request or create a new transport request, then click on next.

    <img src="./IMAGES/Choose_transport_request.png" alt="Package" width="50%">

   
3. Your final package structure should look like this.
   
    <img src="./IMAGES/Package_structure.png" alt="Package" width="50%">




---



2️⃣  Perform deployment using the Quick Fiori Application Generator

1. After activating and publishing your service binding, you will be able to view the option of 'Create Fiori App'. Select the entity 'ZLYMGT_C_MEMBERSHIP' and click on the button 'Create Fiori App'. 

    <img src= "./IMAGES/Create_fiori_app_button.png" alt="Package" width="50%">

2. In the dialog that appears, choose the radio button "Create SAP Fiori app with Quick Fiori Application generator in ADT".

    <img src= "./IMAGES/Choose_quick_deploy_with_adt.png" alt="Package" width="50%">

3.  After clicking on create, in the following wizard, update the package name to 'ZLOYALTY_MGMT_UI'.

    <img src= "./IMAGES/Update_package_name.png" alt="Package" width="50%">

4. Fill in the appropriate parameter values ensuring the repository name is unique and within 15 characters, then click next.

    <img src= "./IMAGES/Configure_generator.png" alt="Package" width="50%">

5. The  artefacts generated list is shown, then click next.

    <img src= "./IMAGES/Artefacts_after_generation.png" alt="Package" width="50%">

6. Select your transport request, then click next.

   <img src= "./IMAGES/Choose_transport_request_ui.png" alt="Package" width="50%">

7. Click on open after the successful creation of artefacts.

   <img src= "./IMAGES/Open_artefacts_button.png" alt="Package" width="50%">

8. Refresh using F5 on the 'ZLOYALTY_MGMT_UI' package, you should be able to see the following artefacts.

   <img src= "./IMAGES/Final_ui_artefacts.png" alt="Package" width="50%">

On clicking on the 'Fiori App URL' inside the service binding, we see the following error, this is because access rights are not provided yet. 

   <img src= "./IMAGES/403_forbidden_error.png" alt="Package" width="50%">



In the next steps, we will be creating IAM app and Business catalog for this purpose.


---



3️⃣ Creation of IAM App and Business Catalog

1. Right click on the UI artefacts package 'ZLOYALTY_MGMT_UI', select 'New' > Other ABAP Repository Object > IAM App, as shown.

   <img src= "./IMAGES/Create_iam_app_1.png" alt="Package" width="50%">
   
2. Select 'IAM App' under 'Identity and Access Management' and click on next.

   <img src= "./IMAGES/Create_iam_app_2.png" alt="Package" width="50%">

3. Enter the package, name and description for the IAM App and click on next.  

   <img src= "./IMAGES/Create_iam_app_3.png" alt="Package" width="50%">

4. Select your transport request and click on finish.
   
   <img src= "./IMAGES/Create_iam_app_4.png" alt="Package" width="50%">
   
5. After clicking on finish, you should be able to see the following IAM app created.

   <img src= "./IMAGES/Create_iam_app_5.png" alt="Package" width="50%">

6. In your project explorer, you can see the 'Launchpad App Descriptor Items' under 'Fiori User Interface' folder in your UI artefacts package. Make note of the name of this Launchpad App Descriptor Item as this is used in your IAM app.

    <img src= "./IMAGES/Create_iam_app_6.png" alt="Package" width="50%">

7. Enter the Launchpad App Descriptor Item in your IAM App. On doing this, the OData service will get automatically populated under the 'Services' tab in the IAM App.

    <img src= "./IMAGES/Create_iam_app_7.png" alt="Package" width="50%">
    <img src= "./IMAGES/Create_iam_app_8.png" alt="Package" width="50%">

8. Ensure your IAM App is published, if not, click on 'Publish locally' to publish it.
   
9. Click on 'Create a new Business Catalog and assign the App to it' within your IAM App as shown below. 

   <img src= "./IMAGES/Create_business_catalog_1.png" alt="Package" width="50%">

10. Enter the name, description for the Business Catalog and click on next.

    <img src= "./IMAGES/Create_business_catalog_2.png" alt="Package" width="50%">

11. Select your transport request, then click on finish.

    <img src= "./IMAGES/Create_business_catalog_3.png" alt="Package" width="50%">

12. The dialog to create a new Business Catalog IAM App Assignment opens up. Enter the UI artefacts package name, review the details, then click next.

    <img src= "./IMAGES/Create_business_catalog_4.png" alt="Package" width="50%">

13.  Select your transport request, then click on finish.                                                                                                                                                                                                                                                                                                                                                                                                                                   
    <img src= "./IMAGES/Create_business_catalog_5.png" alt="Package" width="50%">

14. The Business Catalog IAM App Assignment is created as shown below.

    <img src= "./IMAGES/Create_business_catalog_6.png" alt="Package" width="50%">

15. The Business Catalog is also created, but in an unpublished state as shown below. Go to the 'Apps' tab underneath, and make sure the IAM App is added.

    <img src= "./IMAGES/Create_business_catalog_7.png" alt="Package" width="50%">
    <img src= "./IMAGES/Create_business_catalog_8.png" alt="Package" width="50%">

16. Click on 'Publish Locally'.

    <img src= "./IMAGES/Create_business_catalog_9.png" alt="Package" width="50%">
    
    Your folder structure should now look like the following :
    
    <img src= "./IMAGES/Create_business_catalog_10.png" alt="Package" width="50%">


---

4️⃣ Assigning the Business Catalog to the Business User

> **Note**  
> The following steps help assign the created business catalog along with the deployed app to relevant business users.


1. Open the Fiori Launchpad of the system, login as the administrator and find "Maintain Business Roles" App. Here, we have to create a new business role by clicking on 'New'.

    <img src= "./IMAGES/Assign_business_catalog_1.png" alt="Package" width="50%">

    <img src= "./IMAGES/Assign_business_catalog_2.png" alt="Package" width="50%">
3. Enter the name, description and click on create.

    <img src= "./IMAGES/Assign_business_catalog_3.png" alt="Package" width="50%">

4. Go to the Business Catalogs Tab and add the business catalog created earlier

    <img src= "./IMAGES/Assign_business_catalog_4.png" alt="Package" width="50%">

    <img src= "./IMAGES/Assign_business_catalog_5.png" alt="Package" width="50%">

5. You can also add the relevant business users in the Business Users Tab. This can also be done via the "Maintain Business Users" app. 

    <img src= "./IMAGES/Assign_business_catalog_6.png" alt="Package" width="50%">

6. Click on Maintain Restrictions and make the "Write , Read , Value Help" access to unrestricted, then save the role.

    <img src= "./IMAGES/Assign_business_catalog_7.png" alt="Package" width="50%">

---

5️⃣ Create Launchpad Space and Page

> **Note**  
> The following steps show how to create Launchpad Space and Page for the Loyalty Management Application, and also how to link the business role we created earlier with the Launchpad Space.

1. Login as administrator and navigate to Administration > Launchpad as shown.
   
    <img src= "./IMAGES/Launchpad_creation_1.png" alt="Package" width="50%">

2. Access the 'Manage Launchpad Spaces' application, as shown below.

    <img src= "./IMAGES/Launchpad_creation_2.png" alt="Package" width="50%">

3. After opening the 'Manage Launchpad Spaces' application, click on create, and fill the details of the Space, and click on 'Also create page' as shown below. Click on create and save the space.

    <img src= "./IMAGES/Launchpad_creation_3.png" alt="Package" width="50%">

4. Access the 'Maintain Business Roles' application, and navigate to the role we created in the earlier steps.

    <img src= "./IMAGES/Launchpad_creation_4.png" alt="Package" width="50%">

5. Inside the role, navigate to the 'Launchpad Spaces'and click on add. Then select 'Use Space' and enter the name of the space created earlier, as shown below.
   Then click on 'Assign Space'.

    <img src= "./IMAGES/Launchpad_creation_5.png" alt="Package" width="50%">

6. Navigate back to the 'Manage Launchpad Spaces' application and the space created earlier. Navigate to the page created in the space, and click on edit.

  <img src= "./IMAGES/Launchpad_creation_6.png" alt="Package" width="50%">

7. Under 'Derived From Roles', you should be able to see the option to add the page content corresponding to our business role. Click on add.

    <img src= "./IMAGES/Launchpad_creation_7.png" alt="Package" width="50%">
    
8. Click on save, then navigate to your home page on the FLP, you will be able to access the Loyalty Points Management as a Launchpad Space.

   <img src= "./IMAGES/Launchpad_creation_8.png" alt="Package" width="50%">

