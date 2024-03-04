# Miva Merchant - ADPM (Add-Product Multiple) on PROD Page

## Table of Contents
- [Description](#description)
- [Goals](#goals)
- [Limitations](#limitations)
- [Implementation](#implementation)
    - [Step 1 -- Setting Up The Required Custom Fields](#step-1----setting-up-the-custom-fields-and-custom-groups)
    - [Step 2 -- Add-on Product Template](#step-2----add-on-product-template)
    - [Step 3 -- ADPR vs ADPM - Working Around The Attribute Machine](#step-3----adpr-vs-adpm---working-around-the-attribute-machine)
    - [Step 4 -- Handling Price changes](#step-4----handling-price-changes)
    - [Step 5 -- AJAX request](#step-5----ajax-request)
    - [Step 6 -- Error Handling](#step-6----error-handling)

## Description

This is a walk through on how to implement add-on products onto main product pages (PROD Page) in Miva Merchant eccomerce stores that use shadows readytheme. This works in tandem with the attribute machine and retains an "out-of-box" feel of shadows readytheme. 

[Live Example](https://www.weistec.com/m157_ecu_tune.html)


## Goals

- **Create An Easier, More Efficient And Enjoyable User Experience.** 

    The main goal of this project is to allow customers to add recommended or related products to their basket alongside a main product all in a single form submit. This makes the user shopping experience quicker, more efficient and more enjoyable. More items added to cart lead to increase sales average.

- **More Accurate Inventory Tracking And Discount/Tax Calculation**

    Prior to this project, we used product attributes to imnplement add-on products. This caused the add-on products to inherit certain characteristics from the main product like tax status, discounts, etc. By using the ADPM action each product becomes their own line item in the basket and thus doesn't inherit anything from the main product. Because of this inventory tracking and  discount calculations are more accurate.

- **Error Handling**

    We added error handling for this project. Errors display in minibasket.

- **Retain Attribute Machine Functionality**

    ADPM does not work with the attribute machine at the time of writing. This project works around this limitation to keep the same user experience as before.

## Limitations
- **Discounts on add-on products don't display on product page**

Discounts don't display on product page, but once added to cart discounts are applied and viewable in basket.

- **Attributes for add-on products aren't supported in this walkthrough**

ADPM does support attributes for multiple products so it is possible to implement it, but we thought that doing so would take up too much space on screen. Because of this we chose not to do so. 

## Implementation

### Step 1 -- Setting Up The Custom Fields And Custom Groups

Custom fields are a powerful tool to customize your eccomerce store. For more info read Miva's Documentation on [Custom Fields](https://docs.miva.com/template-language/custom-field-reference-documentation).

For this project we use three custom fields: add_on_product, add_on_product_image, add_on_product_short_name. For better Admin work flow we will seperate these custom fields into the custom field groups: Add-ons (Parent), Add-ons (Child)  If you don't want your add-on products to have images, only use add_on_product.

You can create a new custom field group in Home > Utility Settings > Custom Field Groups.

![](./Assets/Customfield-Group-Add-Group-Button.png)

This is the set up

![](./Assets/CustomField-Group-Set-up.png)

You can create a new custom field in Home > Utility Settings > Custom Fields.

- **add_on_product**
    
    This field is for main products and is part of the "Add-on (Parents)" custom field group. This custom field will hold a string of product codes of products that you would like to be add-on products. Each product code should be separated by a comma. The following is the set up.

![](./Assets/add_on_product-set-up.png)

- **add_on_product_image**

    This custom field is to be used on the add-on product itself and is part of the "Add-ons (Child). This field holds an image of the add-on product. In this project, the add-on image is 480 x 480 pixels and has a lower resolution for optimal performance. The following is  the set up.

    ![](./Assets/add_on_product_image-set-up.png)

- **add_on_product_short_name**

    This custom field is to be used on the add-on product itself. This is to customize how you want the add-on product name to appear. It is completely optioonal, but useful if your add-on product has a long name that would take up too much space on the screen.

    ![](./Assets/add_on_product_short_name-set-up.png)


### Step 2 -- Add-on Product template


### Step 3 -- ADPR vs ADPM - Working Around The Attribute Machine

**A little background info on MIVA Actions and Forms**

Miva Merchant uses hidden inputs named actions that let their engine know how to handle forms and each form has its own required inputs and format. For example, Login form uses the action LOGN and requires email and password. A full list of actions can be found [here](https://gist.github.com/steveosoule/29028906236ee5dbc561ac7ac858fc56). Here we are going to focus on the action ADPR and ADPM and their required inputs and form formats. 

By default the PROD page (Product Display page) uses a form that uses the action ADPR (Add Product) to add products to cart. The Attribute Machine recognizes the form format and all the expected functionality runs normally, but if we want to be able to add multiple products we will have to use the action ADPM (Add Product Multiple) and its form which the attribute machine doesn't recognize. The solution here is to load in the form in ADPR format.

### Step 4 -- Handling Price changes


### Step 5 -- AJAX request


### Step 6 -- Error Handling



## Admin Workflow

### Setting up an Add-on Product


### Setting up the Main Product