# On-stack Extensibility with SAP Cloud ERP

## Overview

**Extensibility** covers the ability for customers and partners to adapt standard SAP business software to meet unique business requirements.  
It includes adjustments that go beyond standard business configuration, such as:

- Modifying software behavior  
- Extending the data model  
- Exposing data  
- Customizing layouts in UIs, forms, or reports  
- Creating entirely new UIs or custom applications  

---

## Concept

The **extensibility concept of SAP Cloud ERP** ensures that all software products remain **lifecycle-stable**.  
This means SAP software updates are not dependent on customer or partner extensions.  

This is achieved through:

- Proper interface approache (for example, **released objects** and **public APIs**)  
- A clear separation between **core extensions** and **side-by-side extensions**  

As a result, extensions to SAP Cloud ERP are **upgrade-safe** — they do not affect system upgrades.  
Any extensions that can be decoupled from the core process are performed **side-by-side** on the **SAP Business Technology Platform (SAP BTP)**.

---

## Extensibility Options

SAP Cloud ERP provides three main extensibility options:

1. **Key User Extensibility**  
   Through built-in in-app capabilities available to business users.

2. **Developer Extensibility**  
   With **ABAP Cloud** (for professional developers).

3. **Side-by-Side Extensibility**  
   Via **SAP Business Technology Platform (SAP BTP)** for building and running decoupled extensions.

---

# Creating a Gift Card

## Overview

The **Gift Card** scenario demonstrates how extensibility in **SAP Cloud ERP** can be used to create custom business objects (for gift card) and processes that enhance standard functionality (applying the gift card in a sales order).  

---

## Business Context

Many organizations use gift cards as part of their customer engagement and loyalty strategies.  
Standard SAP Cloud ERP functionality may not fully cover all business-specific requirements.  

By using **Key User** and **On-stack Developer Extensibility**, you can build a custom **Gift Card** business object that integrates seamlessly with standard SAP business objects like **Sales Order**.

---

<!-----
➡️ [Create Custom Business Object to manage Gift Cards](./01-Custom-BO-for-GIFT-CARD)
----->
