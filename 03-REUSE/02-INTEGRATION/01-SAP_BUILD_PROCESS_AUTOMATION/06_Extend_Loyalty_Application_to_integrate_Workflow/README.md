# Extend the RAP Loyalty Application to integrate workflow capability

Enhance the business object behavior to add workflow functionality.

> The object names/coding style used here, if any, is not a recommendation on coding guidelines/standards

## Prerequisites
- You have access to the latest version of ABAP Developer Tools (ADT) with [abapGit plugin](https://developers.sap.com/tutorials/abap-install-abapgit-plugin..html)
- You have created the loyalty app with workflow handler class as explained before.
  
In this, you will enhance the managed business object of the loyalty management solution by defining and implementing several determinations, actions, side effects, and UI refresh events.

## 1. Membership Entity

The LoyaltyMembership entity represents the core membership record in the loyalty program, handling creation, updates, tier upgrades, and UI refresh events while managing related transactions and tier information through its associations.

<details>
    <summary> 🔵 Click to Expand</summary>
   
### 1.1 Action: UpgradeTier

This is an instance-level action used to upgrade a member to the next higher tier.  

1. Define the action **UpgradeTier** in the behaviour definition of  ZLYMGT_R_MEMBERSHIP. For that, insert the following code as shown below.

```abap
 action ( features : instance ) UpgradeTier;
 ```

2. Activate the changes

3. Click on the **Quick Assist** besides the newly added line of code and then select "Add a method for action" to declare the required method in behavior implementation class.

4. As a result, you can see the method **UpgradeTier**  for action is added in the local handler class lhc_zlymgt_r_membership of Behaviour implementation class ZLYMGT_BP_R_MEMBERSHIP.

5. Add the following code:
   
```abap
METHOD UpgradeTier.
    "Read Loyalty membership and its related membership tiers
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
          ENTITY LoyaltyMembership
          FIELDS ( Membershipuuid )
          WITH CORRESPONDING #( keys )
          RESULT DATA(membership)
          BY \_MembershipTiers
          ALL FIELDS WITH VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid ) )
          RESULT DATA(tiers).

    "If no LoyaltyMembership record found, exit the method
    IF membership IS INITIAL.
      RETURN.
    ENDIF.

    "Keep only active tiers from the list, Keep only the currently active tier so we know what to upgrade from.
    DELETE tiers WHERE Tierstatus <> zlymgt_if_constants=>workflow_status_active.

    SORT tiers BY tierid.
    "Read the last (current) active tier based on number of active tiers
    READ TABLE tiers INTO DATA(currtier) INDEX lines( tiers ).

    IF sy-subrc <> 0.
      " No active tier found
      INSERT VALUE #( %msg = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-information
                                    text     = 'No active tier exists' ) )
             INTO TABLE reported-loyaltymembership ##NO_TEXT.
      RETURN.
    ENDIF.

    "Select the next tier from tier config table with higher TierID
    SELECT tierid, conversionfactor
      FROM zlymgt_tierconfg
      WHERE tierid > @currtier-tierid
      ORDER BY tierid
      INTO @DATA(newTier)
      UP TO 1 ROWS.
    ENDSELECT.
    IF sy-subrc <> 0.
      " No higher tier found
      INSERT VALUE #( %msg = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-information
                                    text     = 'Loyalty Member is already on the highest Tier' ) )
             INTO TABLE reported-loyaltymembership ##NO_TEXT.
      RETURN.
    ENDIF.
    "Create a new MembershipTier record for the member with new tier details
    MODIFY ENTITIES OF zlymgt_r_membership
           ENTITY LoyaltyMembership
           CREATE BY \_MembershipTiers SET FIELDS WITH
           VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid
                      %target    = VALUE #( ( %cid             = zlymgt_if_constants=>cid
                                              %is_draft        = zlymgt_if_constants=>draft
                                              Tierid           = newTier-tierid
                                              Tierstatus       = zlymgt_if_constants=>workflow_status_pending
                                              Conversionfactor = newTier-conversionfactor

                                            ) )
                      %is_draft  = zlymgt_if_constants=>draft ) )
           MAPPED   mapped
           FAILED   DATA(UpgradeTier_failed)
           REPORTED reported ##NO_LOCAL_MODE.
    "Add success message to reported table
    INSERT VALUE #( %msg = new_message_with_text( severity = if_abap_behv_message=>severity-information
                                                  text     = 'Workflow raised for Tier approval' ) )
           INTO TABLE reported-loyaltymembership ##NO_TEXT.
  ENDMETHOD.
```

6. Activate the changes.
   
The UpgradeTier method is responsible for upgrading a loyalty member to the next eligible tier. It performs the following steps:

- **Read the Membership and Its Active Tier**  
The method first retrieves the loyalty membership record and all its related membership tiers. It filters the tiers to keep only the currently active one.

- **Identify the Current Tier **
It reads the highest active tier (based on Tier ID) to determine the member’s current tier level.

- **Find the Next Higher Tier**
 Using the tier configuration table, the method selects the next available tier with a higher Tier ID. If no higher tier exists, a message is returned informing that the member is already at the highest tier.

- **Create a New Tier Entry Using EML**  
If a valid next tier is found, a new Membership Tier record is created with status "Waiting for approval", which will later trigger the workflow for tier approval.

- **Return Informational Messages**
Messages are added to the reported table to indicate whether:
   - No active tier exists,
   - The member is already at the highest tier.
 
### 1.2 Instance Features — Enable/Disable “Upgrade Tier”

1️. The **get_instance_features** method dynamically controls whether the "Upgrade Tier" action should be enabled or disabled for a loyalty membership record.

```abap
METHOD get_instance_features.
    " Disable the "Upgrade Tier" action if a Tierstatus is already pending for approval
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY LoyaltyMembership
           FIELDS ( Membershipuuid )
           WITH CORRESPONDING #( keys )
           RESULT DATA(lylmembership)
           BY \_MembershipTiers
           FIELDS ( Tierstatus )
           WITH VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid   ) )
           RESULT DATA(tierdata) .

*Evaluate the conditions, set the action Upgrade Tier as disabled if a tier change is already requested & Waiting for approval
    LOOP AT keys INTO DATA(key).
      APPEND VALUE #( %tky                = key-%tky
                      %action-UpgradeTier = COND #( WHEN ( tierdata IS NOT INITIAL
                                                          AND
                                              line_exists( tierdata[ Tierstatus = zlymgt_if_constants=>workflow_status_pending ] ) )
                                                   THEN if_abap_behv=>fc-o-disabled )
                    ) TO result.
    ENDLOOP.
  ENDMETHOD.
```

2. Activate the changes.

3. Read Membership Tier Data
The method retrieves all tier entries for the given membership, specifically reading the Tierstatus field.

4. Check for Pending Tier Changes
If any tier record is found with a status of Pending, it means a tier upgrade request is already in progress.

5. Disable the “Upgrade Tier” Action
When a Waiting for approval tier exists, the method marks the UpgradeTier action as disabled for that instance.
Otherwise, the action remains enabled.
   
###  1.3 Events and Side Effects

#### Event: UpdateUI

This event ensures that the UI refreshes automatically when membership tier data changes.

```abap
 event UpdateUI for side effects;
```
#### Side Effects Definitions

Side effects help update dependent data during UI refresh:
 side effects
  {
    action UpgradeTier affects entity _MembershipTiers, $self;
    event UpdateUI affects entity _MembershipTiers, $self, field _MembershipTiers.Tierstatus;
  }
  


🪄Activate the changes.

When UpgradeTier action runs, it refreshes:
- Main membership record
- All Membership Tiers under it

When UpdateUI event is raised, it updates:
- Membership data
- Tier records, specifically tier status

### 1.4 Saver Class: Triggering UI Refresh

1. The class **lsc_zlymgt_r_membership** is a local saver class that inherits from cl_abap_behavior_saver.

```abap
CLASS lsc_zlymgt_r_membership DEFINITION INHERITING FROM cl_abap_behavior_saver.

  PROTECTED SECTION.

    METHODS save_modified REDEFINITION.

ENDCLASS.

CLASS lsc_zlymgt_r_membership IMPLEMENTATION.

  METHOD save_modified.
    " Triggers the UpdateUI event after membership tier records are created or updated, so the Fiori UI refreshes the affected loyalty membership and tier details automatically.

    IF update-MembershipTiers IS NOT INITIAL.
      LOOP AT update-membershiptiers ASSIGNING FIELD-SYMBOL(<tier>).
        RAISE ENTITY EVENT zlymgt_r_membership~UpdateUI
              FROM VALUE #( ( Membershipuuid   = <tier>-Membershipuuid ) ).
      ENDLOOP.
    ENDIF.

    IF create-MembershipTiers IS NOT INITIAL.
      LOOP AT create-membershiptiers ASSIGNING FIELD-SYMBOL(<tiers>).

        RAISE ENTITY EVENT zlymgt_r_membership~UpdateUI
              FROM VALUE #( ( Membershipuuid   = <tiers>-Membershipuuid  ) ).
      ENDLOOP.
    ENDIF.
  ENDMETHOD.

ENDCLASS.
```


2. Activate the changes.

The save_modified method is responsible for raising the UpdateUI event whenever membership tier records are created or updated. This event is meant to refresh the UI so that Fiori applications immediately show the latest membership tier changes.

- Detect Updates to Membership Tiers: If any tier entries have been updated, the method loops through them and raises the UpdateUI event for each affected membership.

- Detect New Tier Creations: If new tier records are created, the method again raises the UpdateUI event for the corresponding memberships.

- Trigger UI Refresh: Raising the event ensures that the affected loyalty membership and its tier details are automatically refreshed in the UI through side effects defined in the behavior definition.

</details>

## 2. Transaction Entity Behavior 
The MembershipTransactions entity stores individual transaction records for each loyalty member and automatically recalculates loyalty points whenever key transaction fields (Transaction value/date/currency) are changed, ensuring accurate and real-time points management.

<details>
    <summary> 🔵 Click to Expand</summary>
  
### 2.1 Determination: CalculateLoyaltyPointsAccrued (on modify)

1. Define the determination **CalculateLoyaltyPointsAccrued** in the behaviour definition of ZLYMGT_R_TRANSACTION. For that, insert the following code:

```abap
determine CalculateLoyaltyPointsAccrued on modify 
    { create; field Transactiondate; field Transactionvalue; field Transactioncurrency; }
```

2. Activate the changes.

3. Click on the **Quick Assist** besides the newly added line of code and select "Add method for action" to declare the required method in behavior implementation class.

4. As a result, you can see the method **CalculateLoyaltyPointsAccrued**  for action is added in the local handler class lhc_membershiptransactions  of behaviour implementation class ZLYMGT_BP_R_MEMBERSHIP.
   This class contains the determination CalculateLoyaltyPointsAccrued, which is automatically triggered during a MODIFY operation on the MembershipTransactions entity.

```abap
METHOD CalculateLoyaltyPointsAccrued.
    " Calculates loyalty points for a transaction based on its value, currency, and the member's active tier, then updates the Loyaltypoints field in MembershipTransactions.

    DATA updateTransactions        TYPE TABLE FOR UPDATE zlymgt_r_transaction.
    " Read all relevant transaction instances.
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
         ENTITY MembershipTransactions
         FIELDS ( Transactionuuid Transactionvalue Loyaltypoints Membershipuuid )
         WITH CORRESPONDING #( keys )
      RESULT DATA(lylpointstransactions)
      FAILED DATA(failed).

    IF lylpointstransactions IS NOT INITIAL.
      READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
          ENTITY LoyaltyMembership
          FIELDS ( Membershipuuid )
          WITH VALUE #( (  Membershipuuid = lylpointstransactions[ 1 ]-Membershipuuid ) )
          RESULT DATA(lytymembership)
          BY \_MembershipTiers
          FIELDS (  Tierid Tierstatus  )
          WITH VALUE #( ( Membershipuuid = lylpointstransactions[ 1 ]-Membershipuuid  ) )
          RESULT    DATA(tiers) .
    ENDIF.
    IF lytymembership IS INITIAL.
      RETURN.
    ENDIF.

    IF line_exists( tiers[ Tierstatus = zlymgt_if_constants=>workflow_status_active ]   ).
      READ TABLE tiers REFERENCE INTO DATA(tier) WITH KEY Tierstatus = zlymgt_if_constants=>workflow_status_active.
      DATA(currtier) = tier->Tierid.
    ELSE.
      currtier = 1.
    ENDIF.
    SELECT SINGLE Conversionfactor FROM zlymgt_tierconfg WHERE tierid = @currtier INTO @DATA(factor).
    IF sy-subrc <> 0.
      factor = 0. " Default to 0 if no config found
    ENDIF.
    lylpointstransactions[ 1 ]-Loyaltypoints =   lylpointstransactions[ 1 ]-Transactionvalue * factor .
    updateTransactions  =  CORRESPONDING #( lylpointstransactions ).

    MODIFY ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY MembershipTransactions
           UPDATE FIELDS ( Loyaltypoints )
           WITH updateTransactions
         MAPPED   DATA(updated_transactions)
         FAILED   DATA(update_failed)
         REPORTED  DATA(update_reported).
    IF reported IS NOT INITIAL OR update_failed IS NOT INITIAL.
*Fill reported
      reported = CORRESPONDING #( DEEP update_reported ).
*Set failed keys
      APPEND VALUE #(  %tky = lytymembership[ 1 ]-%tky )
                    TO update_failed-loyaltymembership.
    ENDIF.
  ENDMETHOD.
```


5. Activate the changes.

This method calculates and updates the Loyalty Points for a transaction based on the member’s active tier and the transaction value.

- Read Transaction Details: The method begins by reading the transaction instances being modified, including value, currency, and membership reference.

- Retrieve Active Tier of the Member: It then reads the membership’s tier records to identify the active tier (Tierstatus = Active). If no active tier exists, it defaults to tier level 1.

- Get Conversion Factor_ Using the tier ID, it reads the conversion factor from a configuration table. This factor defines how many loyalty points are awarded per unit of transaction value.

- Calculate Loyalty Points: Loyalty points = Transaction Value × Conversion Factor. The calculated points are updated into the transaction data.

- Update the Transaction Using EML: The calculated loyalty points are written back to the MembershipTransactions entity using MODIFY ENTITIES.

- Error Handling & Reporting: Any failed updates or system errors are added to the reported or failed structures to ensure proper message handling in RAP.

### 2.2 Side Effects

Changing any transaction field affects Loyaltypoints field in Membership entity.

```abap
 side effects
  {
    field Transactiondate affects field Loyaltypoints, entity _LoyaltyMembership;
    field Transactionvalue affects field Loyaltypoints, entity _LoyaltyMembership;
    field Transactioncurrency affects field Loyaltypoints, entity _LoyaltyMembership;
  }
``` 


🪄Activate the changes.
</details>

## 3. Membership Tier Entity Behavior

The MembershipTiers entity manages the tier history of each loyalty member, automatically triggering tier-upgrade workflows and ensuring only one active tier is maintained at any time. If a valid next tier is found, a new Membership Tier record is created with status Waiting for approval, which will later trigger the workflow for tier approval.

<details>
    <summary> 🔵 Click to Expand</summary> 
  
### 3.1 Determination on save : Initiate_Tier_Upgrade_Workflow

This determination is triggered on save when Tier ID changes.

1. Define the determination **Initiate_Tier_Upgrade_Workflow** in the behaviour definition of  ZLYMGT_R_MEMBERSHIPTIER. For that, insert the following code as shown below.

```abap
determination Initiate_Tier_Upgrade_Workflow on save { field Tierid; }
```


2. Activate the changes.

3. Click on the **Quick Assist** besides the newly added line of code and then "Add method for action" to declare the required method in behavior implementation class.

4. As a result, you can see the method **Initiate_Tier_Upgrade_Workflow**  for action is added in the local handler class lhc_membershiptiers  of Behaviour implementation class ZLYMGT_BP_R_MEMBERSHIP.

```abap
METHOD Initiate_Tier_Upgrade_Workflow.
    " Reads membership tier and related membership details, retrieves approver information and initiates the tier upgrade approval workflow with current and new tier descriptions.

    READ ENTITIES OF zlymgt_r_membership  IN LOCAL MODE
           ENTITY MembershipTiers
           FIELDS ( Membershipuuid Tierid  )
           WITH CORRESPONDING #( keys )
           RESULT DATA(Tiers)
           FAILED DATA(membershiptier_failed).

    "If read operation failed, exit the method
    IF membershiptier_failed IS NOT INITIAL.
      RETURN.
    ENDIF.

    "If Tiers are found, read corresponding LoyaltyMembership record
    IF Tiers IS NOT INITIAL.
      READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY LoyaltyMembership
           FIELDS (  Membershipid Customername )
           WITH VALUE #( ( Membershipuuid = Tiers[ 1 ]-Membershipuuid ) )
           RESULT DATA(memberships).
    ENDIF.

    "If no LoyaltyMembership found, exit the method
    IF memberships IS INITIAL.
      RETURN.
    ENDIF.

    "Try to get current user's Business Partner ID from context information
    " its a workaround. In an actual business scenario, the initiator would not be an approver
    TRY.
        DATA(business_partner_id) = cl_abap_context_info=>get_user_business_partner_id( ).
      CATCH cx_abap_context_info_error INTO DATA(lx_abap_context_info_error).
        INSERT VALUE #( %msg = new_message_with_text( severity = if_abap_behv_message=>severity-error
                                                      text     = lx_abap_context_info_error->get_longtext( ) ) )
               INTO TABLE reported-loyaltymembership.
        " Set failed keys
        APPEND VALUE #( %tky = Tiers[ 1 ]-%tky )
               TO membershiptier_failed-membershiptiers.
    ENDTRY.
    DATA approveremail TYPE string.
    SELECT SINGLE DefaultEmailAddress
      FROM zlymgt_i_businessuservh
      WHERE BusinessPartner = @business_partner_id
      INTO @approveremail.

    "Read tier for descriptions and Sort tiers ascending by Tierid
    SELECT Tierid,
  Tierdescription
     FROM zlymgt_I_Tierconfig INTO TABLE @DATA(tiertxt).
    SORT tiertxt BY Tierid.
    SORT tiers  BY Tierid.

    NEW zlymgt_cl_membshiptierapproval( )->Initiate_Tier_Approval(
     lylpts_wf_type    = zlymgt_if_constants=>lylpts_wf_usertask
       memid             = CONV #( memberships[ 1 ]-Membershipid )
       name              = CONV #( memberships[ 1 ]-Customername )
       curtier           = SWITCH string(  Tiers[ 1 ]-TierId WHEN 1
                                                             THEN  tiertxt[ Tiers[ 1 ]-TierId ]-TierDescription
                                                             ELSE CONV #( tiertxt[ TierId = Tiers[ 1 ]-TierId - 1 ]-TierDescription ) )
       newtier           = CONV #( tiertxt[ Tiers[ 1 ]-TierId ]-TierDescription )
       validfrom         = cl_abap_context_info=>get_system_date( )
       approver_email_id = approveremail
       ).
  ENDMETHOD.
```


5. Activate the changes.

This method triggers the Tier Upgrade Workflow whenever a new tier with "Waiting for approval" status is created.
It prepares all necessary data and calls the workflow initiation class.

- Read the new tier record being created.
- Retrieve membership details (Membership ID & Customer Name).
- Determine the approver using the current user’s Business Partner (fallback logic).
- Read tier descriptions (current and new tier).
- Call the workflow class zlymgt_cl_membshiptierapproval to start the approval workflow.
- Workflow Data Passed: 
   - Member ID
   - Member Name
   - Current Tier Description
   - New Tier Description
   - Valid From Date
   - Approver Email
   - 
This determination automates the entire approval workflow process - no manual triggering is required.  
As soon as a new tier is saved, the approval workflow is launched.

### 3.2 Determination: SetPreviousTierToInactive (on save)

This determination is triggered on save when a new tier is activated.

1. Define the determination **SetPreviousTierToInactive** in the behaviour definition of ZLYMGT_R_MEMBERSHIPTIER. For that, insert the following code as shown below.

```abap
  determination SetPreviousTierToInactive on save { field Tieruuid, Tierstatus; }
```



2. Activate the changes

3. Click on the **Quick Assist** besides the newly added line of code and then "Add method for action" to declare the required method in behavior implementation class.

4. As a result, you can see the method **SetPreviousTierToInactive**  for action is added in the local handler class lhc_membershiptiers  of Behaviour implementation class ZLYMGT_BP_R_MEMBERSHIP.

```abap
METHOD SetPreviousTierToInactive.

    "Finds the current active tier for a membership and updates its status to inactive, when a new tier becomes active, ensuring only one active tier per membership.
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
                   ENTITY MembershipTiers
                   FIELDS ( Membershipuuid Tieruuid Tierstatus )
                   WITH CORRESPONDING #( keys )
                   RESULT DATA(currenttier) .
    IF currenttier IS NOT INITIAL AND currenttier[ 1 ]-Tierstatus <> zlymgt_if_constants=>workflow_status_active.
      RETURN.
    ENDIF.

    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
        ENTITY LoyaltyMembership
        FIELDS ( Membershipuuid )
        WITH CORRESPONDING #( keys )
        RESULT DATA(lyty_membership)
        BY \_MembershipTiers
        FIELDS ( Tieruuid Tierstatus )
        WITH VALUE #( ( Membershipuuid = currenttier[ 1 ]-Membershipuuid  ) )
        RESULT  DATA(tiers) .

    DELETE tiers WHERE %key-Tieruuid = currenttier[ 1 ]-Tieruuid .
    IF tiers IS NOT INITIAL .
      MODIFY ENTITIES OF  zlymgt_r_membership IN LOCAL MODE
      ENTITY MembershipTiers
      UPDATE FIELDS ( Tierstatus )
      WITH VALUE #( FOR Tier IN tiers INDEX INTO i WHERE ( Tierstatus = zlymgt_if_constants=>workflow_status_active ) (
                        %tky          = Tier-%tky
                        Tierstatus    = zlymgt_if_constants=>workflow_status_inactive ) )
      MAPPED   DATA(updated_inactive_tiers)
      FAILED   DATA(inactive_tier_update_failed)
      REPORTED DATA(tier_status_update_messages)  .
    ENDIF.

    IF tier_status_update_messages IS NOT INITIAL.
      reported = CORRESPONDING #( DEEP tier_status_update_messages ).
    ENDIF.

  ENDMETHOD.
```


5. Activate the changes.

This method ensures that only one active tier exists for each membership.
Whenever a new tier becomes active, the method automatically marks the previously active tier as inactive.

- Reads the tier record being saved.
- Identifies the currently active tier for the membership.
- Excludes the newly activated tier from the list.
- Updates any previously active tier to inactive using EML.
- Returns messages if any update issues occur.
- 
</details>

The entire code of behaviour definition should look like this:

<details>
    <summary> 🔵 Click to Expand</summary>
<br>

```abap
managed implementation in class ZLYMGT_BP_R_MEMBERSHIP unique;
strict ( 2 );
with draft;

define behavior for ZLYMGT_R_MEMBERSHIP alias LoyaltyMembership
persistent table zlymgt_membship
draft table zlymgt_memship_d
etag master Locallastchangedat
lock master total etag Lastchangedat
authorization master ( global )

{
  field ( readonly )
  Membershipid,
  Createdby,
  Createdat,
  Lastchangedby,
  Locallastchangedat,
  Lastchangedat;
  field ( readonly : update ) Customer, Customername, Sourcesystemid;
  field ( readonly, numbering : managed ) Membershipuuid;

  //Assign Membershipid from number range for new memberships on save
  determination SetMembershipId on save { create; }

  create;
  update;
  delete;

  draft action Activate optimized;
  draft action Discard;
  draft action Edit;
  draft action Resume;
  draft determine action Prepare;
  //Action to upgrade the loyalty member to the next higher tier
  action ( features : instance ) UpgradeTier;

  //Event to trigger UI refresh when membership tier data changes
  event UpdateUI for side effects;

  side effects
  {
    //when UpgradeTier action runs, refresh both the membership record ($self) and its related tiers
    action UpgradeTier affects entity _MembershipTiers, $self;

    //when UpdateUI event is raised, refresh membership record, related tiers, and specifically Tierstatus field
    event UpdateUI affects entity _MembershipTiers, $self, field _MembershipTiers.Tierstatus;
  }

  mapping for zlymgt_membship
    {
      Membershipuuid     = membershipuuid;
      Membershipid       = membershipid;
      Customer           = customer;
      Customername       = customername;
      Sourcesystemid     = sourcesystemid;
      Createdby          = createdby;
      Createdat          = createdat;
      Lastchangedby      = lastchangedby;
      Locallastchangedat = locallastchangedat;
      Lastchangedat      = lastchangedat;
    }

  association _MembershipTransactions { create; with draft; }
  association _MembershipTiers
  { with draft; create; }
}

define behavior for ZLYMGT_R_TRANSACTION alias MembershipTransactions
persistent table zlymgt_transctns
draft table zlymgt_transxn_d
lock dependent by _LoyaltyMembership
authorization dependent by _LoyaltyMembership

{
  field ( readonly, numbering : managed ) Transactionuuid;

  field ( readonly )
  Membershipuuid,
  Createdby,
  Createdat,
  Lastchangedby,
  Locallastchangedat,
  Lastchangedat;

  update;
  delete;

  //calculate loyalty points when a transaction is created or relevant fields are modified
  determination CalculateLoyaltyPointsAccrued on modify
  { create; field Transactiondate; field Transactionvalue;
    field Transactioncurrency; }

  //changing these fields in a transaction updates the Loyaltypoints field in the related LoyaltyMembership entity
  side effects
  {
    field Transactiondate affects field Loyaltypoints, entity _LoyaltyMembership;
    field Transactionvalue affects field Loyaltypoints, entity _LoyaltyMembership;
    field Transactioncurrency affects field Loyaltypoints, entity _LoyaltyMembership;
  }

  mapping for zlymgt_transctns
    {
      Membershipuuid      = membershipuuid;
      Transactionuuid     = transactionuuid;
      Transactionvalue    = transactionvalue;
      Transactioncurrency = transactioncurrency;
      Transactiondate     = transactiondate;
      Loyaltypoints       = loyaltypoints;
      Salesorderid        = salesorderid;
      Createdby           = createdby;
      Createdat           = createdat;
      Lastchangedby       = lastchangedby;
      Locallastchangedat  = locallastchangedat;
      Lastchangedat       = lastchangedat;
    }
  association _LoyaltyMembership { with draft; }

}

define behavior for ZLYMGT_R_MEMBERSHIPTIER alias MembershipTiers
persistent table zlymgt_memsptier
draft table zlymgt_tier_d
lock dependent by _LoyaltyMembership
authorization dependent by _LoyaltyMembership
with additional save with full data
{
  field ( readonly )
  Membershipuuid,
  Createdby,
  Createdat,
  Lastchangedby,
  Locallastchangedat,
  Lastchangedat;

  field ( readonly, numbering : managed )
  Tieruuid;

  //triggers the tier upgrade approval workflow when a tier change is saved.
  determination Initiate_Tier_Upgrade_Workflow on save { field Tierid; }

  //sets the previously active tier to inactive when a new tier is saved with active status.
  determination SetPreviousTierToInactive on save { field Tieruuid, Tierstatus; }

  update;
  delete;

  mapping for zlymgt_memsptier
    {
      Membershipuuid     = membershipuuid;
      Tieruuid           = tieruuid;
      Tierid             = tierid;
      Tierstatus         = tierstatus;
      Conversionfactor   = conversionfactor;
      Createdby          = createdby;
      Createdat          = createdat;
      Lastchangedby      = lastchangedby;
      Locallastchangedat = locallastchangedat;
      Lastchangedat      = lastchangedat;
    }
  association _LoyaltyMembership { with draft; }

}
```
</details>

The entire code for Behavior Implementation class should look like this:

<details>
    <summary> 🔵 Click to Expand</summary>
<br>

```abap
CLASS lsc_zlymgt_r_membership DEFINITION INHERITING FROM cl_abap_behavior_saver.

  PROTECTED SECTION.

    METHODS save_modified REDEFINITION.

ENDCLASS.

CLASS lsc_zlymgt_r_membership IMPLEMENTATION.

  METHOD save_modified.
    " Triggers the UpdateUI event after membership tier records are created or updated, so the Fiori UI refreshes the affected loyalty membership and tier details automatically.

    IF update-MembershipTiers IS NOT INITIAL.
      LOOP AT update-membershiptiers ASSIGNING FIELD-SYMBOL(<tier>).
        RAISE ENTITY EVENT zlymgt_r_membership~UpdateUI
              FROM VALUE #( ( Membershipuuid   = <tier>-Membershipuuid ) ).
      ENDLOOP.
    ENDIF.

    IF create-MembershipTiers IS NOT INITIAL.
      LOOP AT create-membershiptiers ASSIGNING FIELD-SYMBOL(<tiers>).

        RAISE ENTITY EVENT zlymgt_r_membership~UpdateUI
              FROM VALUE #( ( Membershipuuid   = <tiers>-Membershipuuid  ) ).
      ENDLOOP.
    ENDIF.
  ENDMETHOD.

ENDCLASS.

CLASS lhc_zlymgt_r_membership DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.
    METHODS:
      get_global_authorizations FOR GLOBAL AUTHORIZATION
        IMPORTING
        REQUEST requested_authorizations FOR LoyaltyMembership
        RESULT result,
      get_instance_features FOR INSTANCE FEATURES
        IMPORTING keys REQUEST requested_features FOR LoyaltyMembership RESULT result.

    METHODS SetMembershipId FOR DETERMINE ON SAVE
      IMPORTING keys FOR LoyaltyMembership~SetMembershipId.

    METHODS UpgradeTier FOR MODIFY
      IMPORTING keys FOR ACTION LoyaltyMembership~UpgradeTier.

ENDCLASS.

CLASS lhc_zlymgt_r_membership IMPLEMENTATION.
  METHOD get_global_authorizations.
  ENDMETHOD.

  METHOD SetMembershipId.

    DATA membershipid_max TYPE zlymgt_membershipid.
    "---CHECK IF A MEMBERSHIP ID IS ALREADY ASSIGNED---
    " Read memberships
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
         ENTITY LoyaltyMembership
         FIELDS ( Membershipuuid  Membershipid )
         WITH CORRESPONDING #( keys )
         RESULT DATA(memberships).

    "Remove memberships that already have an ID (idempotence)
    DELETE memberships WHERE Membershipid IS NOT INITIAL.
    "Exit if no memberships need ID assignment
    IF memberships IS INITIAL.
      RETURN.
    ENDIF.
    "---END OF CHECK FOR EXISTING MEMBERSHIP ID---
    " Request sequential numbers from Number Range Object
    TRY.
        cl_numberrange_runtime=>number_get( EXPORTING nr_range_nr       = zlymgt_if_constants=>nr_range_nr
                                                      object            = zlymgt_if_constants=>nr_obj_value
                                                      quantity          = CONV #( lines( memberships ) )
                                            IMPORTING number            = DATA(number_range_key) ).
      CATCH cx_number_ranges INTO DATA(nr_allocation_failed).
        INSERT VALUE #( %msg = new_message_with_text( severity = if_abap_behv_message=>severity-error
                                                        text     = nr_allocation_failed->get_text( ) ) )
               INTO TABLE reported-loyaltymembership.
        EXIT.
    ENDTRY.

    " Update memberships with sequential ID
    MODIFY ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY LoyaltyMembership
           UPDATE FIELDS ( Membershipid )
           WITH VALUE #(
                         ( %tky     = memberships[ 1 ]-%tky
                           Membershipid = number_range_key ) ).

  ENDMETHOD.



  METHOD get_instance_features.
    " Disable the "Upgrade Tier" action if a Tierstatus is already pending for approval
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY LoyaltyMembership
           FIELDS ( Membershipuuid )
           WITH CORRESPONDING #( keys )
           RESULT DATA(lylmembership)
           BY \_MembershipTiers
           FIELDS ( Tierstatus )
           WITH VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid   ) )
           RESULT DATA(tierdata) .

*Evaluate the conditions, set the action Upgrade Tier as disabled if a tier change is already requested & Waiting for approval
    LOOP AT keys INTO DATA(key).
      APPEND VALUE #( %tky                = key-%tky
                      %action-UpgradeTier = COND #( WHEN ( tierdata IS NOT INITIAL
                                                          AND
                                              line_exists( tierdata[ Tierstatus = zlymgt_if_constants=>workflow_status_pending ] ) )
                                                   THEN if_abap_behv=>fc-o-disabled )
                    ) TO result.
    ENDLOOP.
  ENDMETHOD.

  METHOD UpgradeTier.
    "Read Loyalty membership and its related membership tiers
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
          ENTITY LoyaltyMembership
          FIELDS ( Membershipuuid )
          WITH CORRESPONDING #( keys )
          RESULT DATA(membership)
          BY \_MembershipTiers
          ALL FIELDS WITH VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid ) )
          RESULT DATA(tiers).

    "If no LoyaltyMembership record found, exit the method
    IF membership IS INITIAL.
      RETURN.
    ENDIF.

    "Keep only active tiers from the list, Keep only the currently active tier so we know what to upgrade from.
    DELETE tiers WHERE Tierstatus <> zlymgt_if_constants=>workflow_status_active.

    SORT tiers BY tierid.
    "Read the last (current) active tier based on number of active tiers
    READ TABLE tiers INTO DATA(currtier) INDEX lines( tiers ).

    IF sy-subrc <> 0.
      " No active tier found
      INSERT VALUE #( %msg = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-information
                                    text     = 'No active tier exists' ) )
             INTO TABLE reported-loyaltymembership ##NO_TEXT.
      RETURN.
    ENDIF.

    "Select the next tier from tier config table with higher TierID
    SELECT tierid, conversionfactor
      FROM zlymgt_tierconfg
      WHERE tierid > @currtier-tierid
      ORDER BY tierid
      INTO @DATA(newTier)
      UP TO 1 ROWS.
    ENDSELECT.
    IF sy-subrc <> 0.
      " No higher tier found
      INSERT VALUE #( %msg = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-information
                                    text     = 'Loyalty Member is already on the highest Tier' ) )
             INTO TABLE reported-loyaltymembership ##NO_TEXT.
      RETURN.
    ENDIF.
    "Create a new MembershipTier record for the member with new tier details
    MODIFY ENTITIES OF zlymgt_r_membership
           ENTITY LoyaltyMembership
           CREATE BY \_MembershipTiers SET FIELDS WITH
           VALUE #( ( Membershipuuid = keys[ 1 ]-Membershipuuid
                      %target    = VALUE #( ( %cid             = zlymgt_if_constants=>cid
                                              %is_draft        = zlymgt_if_constants=>draft
                                              Tierid           = newTier-tierid
                                              Tierstatus       = zlymgt_if_constants=>workflow_status_pending
                                              Conversionfactor = newTier-conversionfactor

                                            ) )
                      %is_draft  = zlymgt_if_constants=>draft ) )
           MAPPED   mapped
           FAILED   DATA(UpgradeTier_failed)
           REPORTED reported ##NO_LOCAL_MODE.
    "Add success message to reported table
    INSERT VALUE #( %msg = new_message_with_text( severity = if_abap_behv_message=>severity-information
                                                  text     = 'Workflow raised for Tier approval' ) )
           INTO TABLE reported-loyaltymembership ##NO_TEXT.

  ENDMETHOD.


ENDCLASS.

CLASS lhc_membershiptransactions DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.

    METHODS CalculateLoyaltyPointsAccrued FOR DETERMINE ON MODIFY
      IMPORTING keys FOR MembershipTransactions~CalculateLoyaltyPointsAccrued.

ENDCLASS.

CLASS lhc_membershiptransactions IMPLEMENTATION.

  METHOD CalculateLoyaltyPointsAccrued.
    " Calculates loyalty points for a transaction based on its value, currency, and the member's active tier, then updates the Loyaltypoints field in MembershipTransactions.

    DATA updateTransactions        TYPE TABLE FOR UPDATE zlymgt_r_transaction.
    " Read all relevant transaction instances.
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
         ENTITY MembershipTransactions
         FIELDS ( Transactionuuid Transactionvalue Loyaltypoints Membershipuuid )
         WITH CORRESPONDING #( keys )
      RESULT DATA(lylpointstransactions)
      FAILED DATA(failed).

    IF lylpointstransactions IS NOT INITIAL.
      READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
          ENTITY LoyaltyMembership
          FIELDS ( Membershipuuid )
          WITH VALUE #( (  Membershipuuid = lylpointstransactions[ 1 ]-Membershipuuid ) )
          RESULT DATA(lytymembership)
          BY \_MembershipTiers
          FIELDS (  Tierid Tierstatus  )
          WITH VALUE #( ( Membershipuuid = lylpointstransactions[ 1 ]-Membershipuuid  ) )
          RESULT    DATA(tiers) .
    ENDIF.
    IF lytymembership IS INITIAL.
      RETURN.
    ENDIF.

    IF line_exists( tiers[ Tierstatus = zlymgt_if_constants=>workflow_status_active ]   ).
      READ TABLE tiers REFERENCE INTO DATA(tier) WITH KEY Tierstatus = zlymgt_if_constants=>workflow_status_active.
      DATA(currtier) = tier->Tierid.
    ELSE.
      currtier = 1.
    ENDIF.
    SELECT SINGLE Conversionfactor FROM zlymgt_tierconfg WHERE tierid = @currtier INTO @DATA(factor).
    IF sy-subrc <> 0.
      factor = 0. " Default to 0 if no config found
    ENDIF.
    lylpointstransactions[ 1 ]-Loyaltypoints =   lylpointstransactions[ 1 ]-Transactionvalue * factor .
    updateTransactions  =  CORRESPONDING #( lylpointstransactions ).

    MODIFY ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY MembershipTransactions
           UPDATE FIELDS ( Loyaltypoints )
           WITH updateTransactions
         MAPPED   DATA(updated_transactions)
         FAILED   DATA(update_failed)
         REPORTED  DATA(update_reported).
    IF reported IS NOT INITIAL OR update_failed IS NOT INITIAL.
*Fill reported
      reported = CORRESPONDING #( DEEP update_reported ).
*Set failed keys
      APPEND VALUE #(  %tky = lytymembership[ 1 ]-%tky )
                    TO update_failed-loyaltymembership.
    ENDIF.
  ENDMETHOD.
ENDCLASS.

CLASS lhc_membershiptiers DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.

    METHODS SetPreviousTierToInactive FOR DETERMINE ON SAVE
      IMPORTING keys FOR MembershipTiers~SetPreviousTierToInactive.

    METHODS Initiate_Tier_Upgrade_Workflow FOR DETERMINE ON SAVE
      IMPORTING keys FOR MembershipTiers~Initiate_Tier_Upgrade_Workflow.

ENDCLASS.

CLASS lhc_membershiptiers IMPLEMENTATION.

  METHOD SetPreviousTierToInactive.

    "Finds the current active tier for a membership and updates its status to inactive, when a new tier becomes active, ensuring only one active tier per membership.
    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
                   ENTITY MembershipTiers
                   FIELDS ( Membershipuuid Tieruuid Tierstatus )
                   WITH CORRESPONDING #( keys )
                   RESULT DATA(currenttier) .
    IF currenttier IS NOT INITIAL AND currenttier[ 1 ]-Tierstatus <> zlymgt_if_constants=>workflow_status_active.
      RETURN.
    ENDIF.

    READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
        ENTITY LoyaltyMembership
        FIELDS ( Membershipuuid )
        WITH CORRESPONDING #( keys )
        RESULT DATA(lyty_membership)
        BY \_MembershipTiers
        FIELDS ( Tieruuid Tierstatus )
        WITH VALUE #( ( Membershipuuid = currenttier[ 1 ]-Membershipuuid  ) )
        RESULT  DATA(tiers) .

    DELETE tiers WHERE %key-Tieruuid = currenttier[ 1 ]-Tieruuid .
    IF tiers IS NOT INITIAL .
      MODIFY ENTITIES OF  zlymgt_r_membership IN LOCAL MODE
      ENTITY MembershipTiers
      UPDATE FIELDS ( Tierstatus )
      WITH VALUE #( FOR Tier IN tiers INDEX INTO i WHERE ( Tierstatus = zlymgt_if_constants=>workflow_status_active ) (
                        %tky          = Tier-%tky
                        Tierstatus    = zlymgt_if_constants=>workflow_status_inactive ) )
      MAPPED   DATA(updated_inactive_tiers)
      FAILED   DATA(inactive_tier_update_failed)
      REPORTED DATA(tier_status_update_messages)  .
    ENDIF.

    IF tier_status_update_messages IS NOT INITIAL.
      reported = CORRESPONDING #( DEEP tier_status_update_messages ).
    ENDIF.

  ENDMETHOD.


  METHOD Initiate_Tier_Upgrade_Workflow.
    " Reads membership tier and related membership details, retrieves approver information and initiates the tier upgrade approval workflow with current and new tier descriptions.

    READ ENTITIES OF zlymgt_r_membership  IN LOCAL MODE
           ENTITY MembershipTiers
           FIELDS ( Membershipuuid Tierid  )
           WITH CORRESPONDING #( keys )
           RESULT DATA(Tiers)
           FAILED DATA(membershiptier_failed).

    "If read operation failed, exit the method
    IF membershiptier_failed IS NOT INITIAL.
      RETURN.
    ENDIF.

    "If Tiers are found, read corresponding LoyaltyMembership record
    IF Tiers IS NOT INITIAL.
      READ ENTITIES OF zlymgt_r_membership IN LOCAL MODE
           ENTITY LoyaltyMembership
           FIELDS (  Membershipid Customername )
           WITH VALUE #( ( Membershipuuid = Tiers[ 1 ]-Membershipuuid ) )
           RESULT DATA(memberships).
    ENDIF.

    "If no LoyaltyMembership found, exit the method
    IF memberships IS INITIAL.
      RETURN.
    ENDIF.

    "Try to get current user's Business Partner ID from context information
    " its a workaround. In an actual business scenario, the initiator would not be an approver
    TRY.
        DATA(business_partner_id) = cl_abap_context_info=>get_user_business_partner_id( ).
      CATCH cx_abap_context_info_error INTO DATA(lx_abap_context_info_error).
        INSERT VALUE #( %msg = new_message_with_text( severity = if_abap_behv_message=>severity-error
                                                      text     = lx_abap_context_info_error->get_longtext( ) ) )
               INTO TABLE reported-loyaltymembership.
        " Set failed keys
        APPEND VALUE #( %tky = Tiers[ 1 ]-%tky )
               TO membershiptier_failed-membershiptiers.
    ENDTRY.
    DATA approveremail TYPE string.
    SELECT SINGLE DefaultEmailAddress
      FROM zlymgt_i_businessuservh
      WHERE BusinessPartner = @business_partner_id
      INTO @approveremail.

    "Read tier for descriptions and Sort tiers ascending by Tierid
    SELECT Tierid,
  Tierdescription
     FROM zlymgt_I_Tierconfig INTO TABLE @DATA(tiertxt).
    SORT tiertxt BY Tierid.
    SORT tiers  BY Tierid.

    NEW zlymgt_cl_membshiptierapproval( )->Initiate_Tier_Approval(
     lylpts_wf_type    = zlymgt_if_constants=>lylpts_wf_usertask
       memid             = CONV #( memberships[ 1 ]-Membershipid )
       name              = CONV #( memberships[ 1 ]-Customername )
       curtier           = SWITCH string(  Tiers[ 1 ]-TierId WHEN 1
                                                             THEN  tiertxt[ Tiers[ 1 ]-TierId ]-TierDescription
                                                             ELSE CONV #( tiertxt[ TierId = Tiers[ 1 ]-TierId - 1 ]-TierDescription ) )
       newtier           = CONV #( tiertxt[ Tiers[ 1 ]-TierId ]-TierDescription )
       validfrom         = cl_abap_context_info=>get_system_date( )
       approver_email_id = approveremail
       ).
  ENDMETHOD.
ENDCLASS.
```
</details>

<!-----
➡️ [Run end to end business process for Loyalty Application](/03-REUSE/02-INTEGRATION/01-SAP_BUILD_PROCESS_AUTOMATION/07_Run_endtoend_BusinessProcess/)
----->
