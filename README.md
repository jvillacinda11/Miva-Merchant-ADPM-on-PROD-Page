# Miva Merchant - ADPM (Add-Product Multiple) on PROD Page

## Table of Contents
- [Description](#description)
    - [Demo](#demo)
- [Goals](#goals)
- [Limitations](#limitations)
- [Implementation](#implementation)
    - [Step 1 -- Setting Up The Required Custom Fields](#step-1----setting-up-the-custom-fields-and-custom-groups)
    - [Step 2 -- Add-on Product Template](#step-2----add-on-product-template)
    - [Step 3 -- ADPR vs ADPM - Working Around The Attribute Machine](#step-3----adpr-vs-adpm---working-around-the-attribute-machine)
    - [Step 4 -- Handling Price Changes](#step-4----handling-price-changes)
    - [Step 5 -- AJAX request](#step-5----ajax-request)
    - [Step 6 -- Error Handling](#step-6----error-handling)
- [Admin Workflow](#admin-workflow)
    - [Setting Up and Add-on Product](#setting-up-an-add-on-product)
    - [Setting Up the Main Product](#setting-up-the-main-product)

## Description

This is a walk through on how to implement add-on products onto main product pages (PROD Page) in Miva Merchant eccomerce stores that use shadows readytheme. This works in tandem with the attribute machine and retains an "out-of-box" feel of shadows readytheme. This is an project is very involved and should only be done by a developer. 

### Demo

![](./Assets/Demo.gif)

[Live Example](https://www.weistec.com/m157_ecu_tune.html).  

**NOTE:** This version may have additional functionality and styling that is not covered here. 


## Goals

- **Create An Easier, More Efficient And Enjoyable User Experience.** 

    The main goal of this project is to allow customers to add recommended or related products to their basket alongside a main product all in a single form submit. This makes the user shopping experience quicker, more efficient and more enjoyable. More items added to cart can lead to increase sales average.

- **More Accurate Inventory Tracking And Discount/Tax Calculation**

    Prior to this project, we used product attributes to imnplement add-on products. This caused the add-on products to inherit certain characteristics from the main product like tax status, discounts, etc. By using the ADPM action each product becomes their own line item in the basket and thus doesn't inherit anything from the main product. Because of this inventory tracking and  discount calculations are more accurate.

- **Error Handling**

    We added error handling for this project. Errors display in minibasket.

- **Retain Attribute Machine Functionality**

    ADPM does not work with the attribute machine at the time of writing. This project works around this limitation to keep the same user experience as the default.

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

This is the set up.

![](./Assets/CustomField-Group-Set-up.png)

You can create a new custom field in Home > Utility Settings > Custom Fields.

- **add_on_product**
    
    This field is for main products and is part of the "Add-on (Parent)" custom field group. This custom field will hold a string of product codes of the products that you would like to be add-ons. Each product code should be separated by a comma. The following is the set up.

    ![](./Assets/add_on_product-set-up.png)

- **add_on_product_image**

    This custom field is to be used on the add-on product itself and is part of the "Add-ons (Child). This field holds an image of the add-on product. In this project, the add-on image is 480 x 480 pixels and has a lower resolution for optimal performance. The following is  the set up.

    ![](./Assets/add_on_product_image-set-up.png)

- **add_on_product_short_name**

    This custom field is to be used on the add-on product itself and is part of the "Add-ons (Child). This is to customize how you want the add-on product name to appear. It is completely optioonal, but useful if your add-on product has a long name that would take up too much space on the screen.

    ![](./Assets/add_on_product_short_name-set-up.png)


### Step 2 -- Add-on Product template

This template is based on a [MIVA code sample](https://docs.miva.com/code-samples/product-multi-add-as-attributes). We are not going to use the javascript from this code sample. We will handle quantities in step 3. This template should be put under product attributes in the PROD page template.

```xml
<div class="product-add-ons">
<mvt:comment><!--Call in Add-On Products--></mvt:comment>
<mvt:item name="customfields" param="Read_Product_Code(l.settings:product:code, 'add_on_product',g.add_on_codes)" />

<mvt:if expr="NOT ISNULL g.add_on_codes">
    <mvt:comment><!-- Make arrray of add-on product codes --></mvt:comment>
	<mvt:assign name="g.array_count" value="miva_splitstring( g.add_on_codes, ',', g.codes_array, '' )" />

	<mvt:if expr="miva_array_elements(g.codes_array) GT 0">
	<mvt:assign name="g.count" value="2" />

	<h3>Optional Add-Ons</h3>
		<div class="add_on_product">
		<mvt:foreach iterator="add_on_code" array="global:codes_array">
            <mvt:comment> <!--Load add-on product --></mvt:comment>
			<mvt:do file="g.Module_Library_DB" name="l.success" value="Product_Load_Code( l.settings:add_on_code, l.settings:add_on:product )" />
            <mvt:comment> <!-- Get add-on image and shortened names --></mvt:comment>
            <mvt:item name="customfields" param="Read_Product_Code(l.settings:add_on_code, 'add_on_product_image' ,l.settings:add_on_image)" />
            <mvt:item name="customfields" param="Read_Product_Code(l.settings:add_on_code, 'add_on_product_short_name', l.settings:add_on:short_name)" />
            <mvt:comment> <!--Formats price--> </mvt:comment>
			<mvt:do name="l.settings:add_on:product:formatted_price" file="g.Module_Root $ g.Store:currncy_mod:module" value="CurrencyModule_AddFormatting( g.Store:currncy_mod, l.settings:add_on:product:price )" />

			

			<label title="&mvt:add_on:product:name; + &mvt:add_on:product:formatted_price;">
            <mvt:comment><!-- data-add_on_price will be used to update price --></mvt:comment>
            <input type="checkbox" name="Products[&mvt:global:count;]:code" id="add_on_&mvt:global:count;" value="&mvt:add_on:product:code;" data-add_on_price="&mvt:add_on:product:price;"> 
                <mvt:if expr="NOT ISNULL l.settings:add_on_image">
                    <img loading="lazy" src="&mvt:add_on_image;" alt="&mvt:add_on:product:name; + &mvt:add_on:product:formatted_price;"> 
                </mvt:if>
                <mvt:if expr="NOT ISNULL l.settings:add_on:short_name">
                    <span class="u-text-center">&mvt:add_on:short_name;</span> + &mvt:add_on:product:formatted_price;
                <mvt:else>
                    <span class="u-text-center">&mvt:add_on:product:name;</span> + &mvt:add_on:product:formatted_price;
                </mvt:if>  
            </label><br>

			
			<mvt:assign name="g.count" value="g.count + 1" />
		</mvt:foreach>
		</div>

	</mvt:if>
    <mvt:comment><!-- Outputs add-on product quantities --></mvt:comment>
	<div id="add_on_quantity"></div>

</mvt:if>

</div>


```


### Step 3 -- ADPR vs ADPM - Working Around The Attribute Machine

**A little background info on MIVA Actions and Forms**

Miva Merchant uses hidden inputs named actions that let their engine know how to handle forms and each form has its own required inputs and format. For example, Login form uses the action LOGN and requires email and password. A full list of actions can be found [here](https://gist.github.com/steveosoule/29028906236ee5dbc561ac7ac858fc56). Here we are going to focus on the action ADPR and ADPM and their required inputs and form formats. 


**ADPR Structure**

```html
<!-- Compatible with Attribute machine -->
<input type="hidden" name="Old_Screen"                              value="PROD">
<input type="hidden" name="Old_Search"                              value="">
<!-- Action -->
<input type="hidden" name="Action"                                  value="ADPR">
<input type="hidden" name="Category_Code"                           value="rivian">
<input type="hidden" name="Offset"                                  value="">
<input type="hidden" name="AllOffset"                               value="">
<input type="hidden" name="CatListingOffset"                        value="">
<input type="hidden" name="RelatedOffset"                           value="">
<input type="hidden" name="SearchOffset"                            value="">

<!-- Single Product only -->
<input type="hidden" name="Product_Code"                            value="main-product">
<input type="hidden" name="Product_Attributes[1]:code"              value="size">
<input type="hidden" name="PProduct_Attributes[1]:template_code"    value="size-template">
<input type="hidden" name="Product_Attributes[1]:value"             value="small">
```

**ADPM Structure**
```html
<!-- Not Compatible with Attribute Machine -->
<input type="hidden" name="Old_Screen" value="PROD">
<input type="hidden" name="Old_Search" value="">
<!-- Action -->
<input type="hidden" name="Action" value="ADPM">
<input type="hidden" name="Category_Code" value="mb">
<input type="hidden" name="Offset" value="">
<input type="hidden" name="AllOffset" value="">
<input type="hidden" name="CatListingOffset" value="">
<input type="hidden" name="RelatedOffset" value="">
<input type="hidden" name="SearchOffset" value="">

<!-- Main Product -->
<input type="hidden" name="Products[1]:code"                        value="main-product">
<input type="hidden" name="Products[1]:quantity"                    value="1">
<input type="hidden" name="Products[1]:attributes[1]:code"          value="size">
<input type="hidden" name="Products[1]:attributes[1]:template_code" value="size-template">
<input type="hidden" name="Products[1]:attributes[1]:value"         value="small">
<input type="hidden" name="Products[1]:subterm_id"                  value="test-subscription">

<!-- Additional Product -->
<input type="hidden" name="Products[2]:code"                        value="add-on-product">
<input type="hidden" name="Products[2]:quantity"                    value="1">
<input type="hidden" name="Products[2]:attributes[1]:code"          value="size">
<input type="hidden" name="Products[2]:attributes[1]:template_code" value="size-template">
<input type="hidden" name="Products[2]:attributes[1]:value"         value="small">

```

By default the PROD page (Product Display page) uses a form that uses the action ADPR (Add Product) to add products to cart. The Attribute Machine recognizes the form format and all the expected functionality runs normally, but if we want to be able to add multiple products we will have to use the action ADPM (Add Product Multiple) and its form which the attribute machine doesn't recognize. The solution here is to load in the form in ADPR format and convert it ADPM format when the user submits the form by clicking the add to cart button. This script is a modified version of the javascript that handles quantities from the MIVA code sample from before.  [rguisewite](https://www.miva.com/forums/forum/online-merchants/miva-merchant-9/692445-error-unable-to-locate-form-for-inventory-attributes-when-setting-up-multiadd?p=692534#post692534) made the original modified script that made this possible. I slightly changed it to fit our needs but it is the same script.

**NOTE:** For this script to work you must you must give your add-to-cart form the id "AddToCartForm" and the Quantity input the id "Quantity". 

```html
<form id="AddToCartForm" name="add" method="post" action="&mvte:urls:BASK:auto;">

    <!-- make sure the Product_Code hidden input is here -->
    ...

    Quantity: <input type="tel" id="prod_quantity"
```

Here is the script

```js
function write_quantity_inputs()
    {
        var i, i_len, element_input, element_quantity, elementlist_inputs, element_add_on_quantity, element_form;

        element_form                = document.getElementById( 'AddToCartForm' );
        element_quantity            = document.getElementById( 'Quantity' );
        element_add_on_quantity        = document.getElementById( 'add_on_quantity' );
        elementlist_inputs            = document.querySelectorAll( '.add_on_product input' );

        if ( !element_form || !element_quantity || !element_add_on_quantity )
        {
            return;
        }

        element_add_on_quantity.innerHTML = '';

        for ( i = 0, i_len = elementlist_inputs.length; i < i_len; i++ )
        {
            if ( elementlist_inputs[ i ].type.toLowerCase() === 'checkbox' && elementlist_inputs[ i ].checked )
            {
                element_input            = document.createElement( 'input' );
                element_input.type        = 'hidden';
                element_input.name        = 'Products[' + ( i + 2 ) + ']:quantity';
                element_input.value        = element_quantity.value;

                element_add_on_quantity.appendChild( element_input );
            }
        }

        /* Convert Product_Code, Quantity, and Product_Attributes[ over to the Multi-Add-To-Basket format */

        for ( i = 0, i_len = element_form.elements.length; i < i_len; i++ )
        {
            if ( !element_form.elements[ i ].hasAttribute( 'name' ) || element_form.elements[ i ].name.length === 0 )        continue;
            else if ( element_form.elements[ i ].name.toLowerCase() == 'product_code' )                                        element_form.elements[ i ].name = 'Products[ 1 ]:code';
            else if ( element_form.elements[ i ].name.toLowerCase() == 'quantity' )                                            element_form.elements[ i ].name = 'Products[ 1 ]:quantity';
            else if ( element_form.elements[ i ].name.toLowerCase().indexOf( 'product_attributes[' ) == 0 )                    element_form.elements[ i ].name = 'Products[ 1 ]:attributes' + element_form.elements[ i ].name.substring( 18 );
        }

        if ( element_form.elements.Action.value != 'ATWL' )
        {
            element_form.elements.Action.value = 'ADPM';
        }

        return true;
    }

    document.addEventListener( 'DOMContentLoaded', function( event )
    {
        document.getElementById( 'AddToCartForm' ).addEventListener( 'submit', function( event ) { write_quantity_inputs(); return true; }, false );
    }, false );
```

### Step 4 -- Handling Price changes

At this stage, products will only update price for change in attributes, but the price of add-on products will not be added to the total displayed on the product page. We will have to modify the page template and use two scripts. One that handles price changes that is triggered by attribute change and another that is triggered when an add-on product checkbox is checked or unchecked.

**PROD Template change**

The default price output is in Home > User Interface > Templates > PROD > Product Display Layout and it looks something like this.

```xml
<p class="u-flex x-product-layout-purchase__pricing">
    <mvt:if expr="l.settings:product:base_price GT l.settings:product:price">
        <span class="x-product-layout-purchase__pricing-original">
            <s id="price-value-additional" itemprop="price" content="&mvt:product:base_price;">&mvt:product:formatted_base_price;</s>
        </span>     
        <span class="x-product-layout-purchase__pricing-current">
            <span id="price-value" data-main_price="&mvt:product:price;">&mvt:product:formatted_price;</span>
        </span>
    <mvt:else>
    <span class="x-product-layout-purchase__pricing-current">
        <span id="price-value" data-main_price="&mvt:product:price;" itemprop="price" content="&mvt:product:price;">&mvt:product:formatted_price;</span>
    </span>
    </mvt:if>
</p>
```
The Attribute Machine outputs current price of product based on discounts and attributes to "price-value" and the base price to "price-value-additional". We will still utilize these elements in our script but use a different element to visually output pricing. So we will make the default output hidden. 

```xml
<p class="u-flex x-product-layout-purchase__pricing">
    <mvt:if expr="l.settings:product:base_price GT l.settings:product:price">
        <span class="x-product-layout-purchase__pricing-original">
            <s id="price-value-additional" class="u-hidden">&mvt:product:formatted_base_price;</s>
            <s id="base_price_output" itemprop="price" content="&mvt:product:base_price;" >&mvt:product:formatted_base_price;</s>
        </span>
    <span class="x-product-layout-purchase__pricing-current">
        <span id="price-value" class="u-hidden">&mvt:product:formatted_price;</span>
        <span id="current-price">&mvt:product:formatted_price;</span>
    </span>
    <mvt:else>
        <span class="x-product-layout-purchase__pricing-current">
            <span id="price-value" class="u-hidden">&mvt:product:formatted_price;</span>
            <span id="current-price"  itemprop="price" content="&mvt:product:price;">&mvt:product:formatted_price;</span>
        </span>
    </mvt:if>
</p>
```

**Price Change due to Add-on Products**

This can be added as a javascript resource. This only triggers when a add-on product is checked or unchecked.

```js
$(document).ready(function () {

    // Get the output div
    let priceOutput = $('#current-price');


    // Function to update the total price
    function updateTotalPrice() {

        // Check if there a base price displayed.
        if ($('#base_price_output')) {

            let baseTotal = 0;

            // Add the price of checked add-on checkboxes to base total
            $('.c-form-checkbox--add-on:checked').each(function () {
                baseTotal += parseFloat($(this).data('add_on_price'));
            });

            // Get Base Price
            let baseCurrencyPrice = $('#price-value-additional').text();
            // Convert Price string to Number
            let baseNumberprice = Number(baseCurrencyPrice.replace(/[^0-9.-]+/g,""));
            

            baseTotal += baseNumberprice;

            // Format the total price with USD currency and two decimal places
            let formattedBaseTotal = new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(baseTotal);

            $('#base_price_output').text(formattedBaseTotal);

            }

        // start off with zero
        let total = 0; 

        // Add the price of checked add-on checkboxes to total
        $('.c-form-checkbox--add-on:checked').each(function () {
            total += parseFloat($(this).data('add_on_price'));
        });

        // Get price main product price
        let mainProductCurrencyPrice = $('#price-value').text();
        
        // convert main product price from currency string format to number
        let mainProductNumberPrice = Number(mainProductCurrencyPrice.replace(/[^0-9.-]+/g,""));

        // add main product price to total
        total += mainProductNumberPrice;

        // Format the total price with USD currency and two decimal places
        let formattedTotal = new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD',
            minimumFractionDigits: 2,
            maximumFractionDigits: 2
        }).format(total);

        // Update the price display
        priceOutput.text(formattedTotal);
    }

    // Add event listener add-on products
    $('.c-form-checkbox--add-on').change(function () {
        updateTotalPrice();
    })

});
```

**Price Change Due to Attribute Change**

This uses MivaEvents.SubscribeToEvent to listen for price change due to attributes. This must be placed in Home > User Interface > Templates > PROD > Attribute Machine > Head Template or Home > User Interface > Templates > PROD > Product Display Layout Image Machine > Head Template. 

```js
MivaEvents.SubscribeToEvent('price_changed', function (product_data) { 
        // Get the output div
        let priceOutput = $('#current-price');
    

        // Function to update the total price
        function attrUpdateTotalPrice() {

            //Check for base price
            if (product_data.additional_price) {

            let basePriceOutput = $('#base_price_output');
            let baseprice = product_data.additional_price;

            let baseTotal = 0;

                // Add the price of checked add-on checkboxes to total
            $('.c-form-checkbox--add-on:checked').each(function () {
                baseTotal += parseFloat($(this).data('add_on_price'));
            });

            baseTotal += baseprice;

            // Reformat price to USD
            let formattedBaseTotal = new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(baseTotal);

            // Update base price
            basePriceOutput.text(formattedBaseTotal);

            }
    
            let total = 0; 
    
            // Add the price of checked add-on checkboxes to total
            $('.c-form-checkbox--add-on:checked').each(function () {
                total += parseFloat($(this).data('add_on_price'));
            });
    
            // Get price main product price
            let mainProductPrice = product_data.price;
    
            // add main product price to total
            total += mainProductPrice;
    
            // Reformat total to USD
            let formattedTotal = new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(total);
    
            // Update the price display
            priceOutput.text(formattedTotal);
        }
    
    
        // Add initial price update
        attrUpdateTotalPrice();

}); 
```

### Step 5 -- AJAX request

In some miva themes, adding items to cart opens a minibasket that displays everything that has been added so far. This is done via an AJAX request in a script named "add-to-cart.js". Like before with the Attribute Machine, the default "add-to-cart-ajax.js" will only work with the form associated with the action ADPR. We have to modify it in order to also be able to handle the form associated with the action ADPM. If you don't already have this script you can create one and use theme.js to call it. Here is the modified script.  

```js
/**
 * When called from a `theme.js` file on a product page, this extension will
 * work with the default page code to add a product to the cart utilizing an
 * AJAX call to the form processor.
 *
 * The function contains internal error checking as well as a check to see which
 * page was reached and displaying messages accordingly. If the store is also
 * utilizing the `mini-basket` extension, said extension will be triggered for
 * display upon successfully adding a product to the cart.
 * 
 * CUSTOM -- This code has been modified to be able to handle multi-product add to cart
 */
(function (window, document, undefined) {
	'use strict';

	var purchaseButton = document.querySelector('[data-hook="add-to-cart"]');
	var purchaseButtonText = (purchaseButton.nodeName.toLowerCase() === 'input') ? purchaseButton.value : purchaseButton.textContent;
	var purchaseForm = document.querySelector('[data-hook="purchase"]');
	var purchaseFormActionInput = purchaseForm.querySelector('input[name="Action"]');
	var responseMessage = document.querySelector('[data-hook="purchase-message"]');
	var miniBasketCount = document.querySelectorAll('[data-hook~="mini-basket-count"]');
	var miniBasketAmount = document.querySelectorAll('[data-hook~="mini-basket-amount"]');
	var	element_quantity = document.getElementById('add_on_quantity'); // CUSTOM -- should this elemtent should only be present for add-on products


	purchaseForm.addEventListener('submit', function (evt) {
		if (purchaseFormActionInput.value === 'ADPR' || purchaseFormActionInput.value === 'ADPM') {

			evt.preventDefault();
			evt.stopImmediatePropagation();
	
			purchaseForm.action = purchaseButton.getAttribute('data-action');
			// CUSTOM -- Check for add-on products
			if (element_quantity) {
				purchaseFormActionInput.value = 'ADPM'
			} else {
				purchaseFormActionInput.value = 'ADPR';
			}
	
			var data = new FormData(purchaseForm);
			var request = new XMLHttpRequest(); // Set up our HTTP request
	
			purchaseForm.setAttribute('data-status', 'idle');
	
			if (purchaseForm.getAttribute('data-status') !== 'submitting') {
				purchaseForm.setAttribute('data-status', 'submitting');
				purchaseButton.setAttribute('disabled', 'disabled');
	
				if (purchaseButton.nodeName.toLowerCase() === 'input') {
					purchaseButton.value = 'Processing...';
				}
				else{
					purchaseButton.textContent = 'Processing...';
				}
	
				responseMessage.innerHTML = '';
	
				// Setup our listener to process completed requests
				request.onreadystatechange = function () {
					// Only run if the request is complete
					if (request.readyState !== 4) {
						return;
					}
	
					// Process our return data
					if (request.status === 200) {
						// What do when the request is successful
						var response = request.response;
	
						if (response.body.id === 'js-BASK') {
							var basketData = response.querySelector('[data-hook="mini-basket"]');
							var basketCount = basketData.getAttribute('data-item-count');
							var basketSubtotal = basketData.getAttribute('data-subtotal');
	
							if (miniBasketCount) {
								for (var mbcID = 0; mbcID < miniBasketCount.length; mbcID++) {
									miniBasketCount[mbcID].textContent = basketCount; // Update mini-basket quantity (display only)
								}
							}
	
							if (miniBasketAmount) {
								for (var mbaID = 0; mbaID < miniBasketAmount.length; mbaID++) {
									miniBasketAmount[mbaID].textContent = basketSubtotal; // Update mini-basket subtotal (display only)
								}
							}
	
							if (typeof miniBasket !== 'undefined') {
								document.querySelector('[data-hook="mini-basket"]').innerHTML = response.querySelector('[data-hook="mini-basket"]').innerHTML;
	
								setTimeout(function () {
									document.querySelector('[data-hook="open-mini-basket"]').click();
								}, 100);
							}
							else {
								responseMessage.innerHTML = '<div class="x-messages x-messages--success"><span class="u-icon-check"></span> Added to cart.</div>';
							}
	
							// Re-Initialize Attribute Machine (if it is active)
							if (typeof attrMachCall !== 'undefined') {
								attrMachCall.Initialize();
							}
						}
						else if(response.body.id === 'js-PATR') {
							var findRequired = purchaseForm.querySelectorAll('.is-required');
							var missingAttributes = [];
	
							for (var id = 0; id < findRequired.length; id++) {
								missingAttributes.push(' ' + findRequired[id].title);
							}
	
							responseMessage.innerHTML = '<div class="x-messages x-messages--warning">All <em class="u-color-red">Required</em> options have not been selected.<br />Please review the following options: <span class="u-color-red">' + missingAttributes + '</span>.</div>';
						}
						else if(response.body.id === 'js-PLMT') {
							responseMessage.innerHTML = '<div class="x-messages x-messages--warning">We do not have enough of the combination you have selected.<br />Please adjust your quantity.</div>';
						}
						else if(response.body.id === 'js-POUT') {
							responseMessage.innerHTML = '<div class="x-messages x-messages--warning">The combination you have selected is out of stock.<br />Please review your options or check back later.</div>';
						}
						else {
							responseMessage.innerHTML = '<div class="x-messages x-messages--warning">Please review your selection.</div>';
						}
	
						// Reset button text and form status
						purchaseButton.removeAttribute('disabled');
	
						if (purchaseButton.nodeName.toLowerCase() === 'input') {
							purchaseButton.value = purchaseButtonText;
						}
						else{
							purchaseButton.textContent = purchaseButtonText;
						}
	
						purchaseForm.setAttribute('data-status', 'idle');
					}
					else {
						// What do when the request fails
						console.log('The request failed!');
						purchaseForm.setAttribute('data-status', 'idle');
					}
				};
	
				/**
				 * Create and send a request
				 * The first argument is the post type (GET, POST, PUT, DELETE, etc.)
				 * The second argument is the endpoint URL
				 */
				request.open(purchaseForm.method, purchaseForm.action, true);
				request.responseType = 'document';
				request.send(data);
			}
		} else{
			return;
		}

	}, false);

})(window, document);
```

### Step 6 -- Error Handling

At this stage, user experience is essentially the same as the default shadows theme excpet for displaying errors such as out of stock messages etc.  We can add this script in Home > User Interface >  Global Settngs > Settings > Minibasket.

```xml
<mvt:if expr="l.settings:messages:error_message_count"> 
    <div class="x-messages x-messages--error"> 
        <mvt:foreach iterator="error" array="messages:error_messages"> 
	    <mvt:if expr="l.settings:error EQ 'Out Of Stock'">
		<mvt:if expr="ISNULL l.settings:out_of_stock_check">
			One or more of selected items is <strong>Out of Stock</strong>.
			<mvt:assign name="l.settings:out_of_stock_check" value="1" />
		</mvt:if>
        <!-- 

            You can add more custom error messages here

            <mvt:elseif expr="some other condition">

         -->
	    <mvt:else>
	    &mvt:error;
	    </mvt:if> 
        </mvt:foreach> 
    </div> 
</mvt:if> 
<mvt:if expr="l.settings:messages:information_message_count"> 
        <div class="x-messages x-messages--info"> 
            <mvt:foreach iterator="message" array="messages:information_messages"> 
                 &mvt:message; 
        </mvt:foreach> 
    </div> 
</mvt:if>
```

## Admin Workflow

### Setting up an Add-on Product

Go to the product that you would like to be an add-on product and click "more" and select "Add-ons (Child)".

![](./Assets/add-on-product-1.png)

![](./Assets/add-on-product-2.png)

(Optional) If you want an image to appear next to your add-on product checkbox click upload.

![](./Assets/add-on-product-3.png)

(Optional) If you want a shortened name to appear instead of the full product name. enter it here.

![](./Assets/add-on-product-4.png)

Click update when all changes are made.

Click Product Tab. Copy the Product Code. in This example Product code is test-add-on

![](./Assets/add-on-code.png)

### Setting up the Main Product

Go to your main product and click "more" and select "Add-ons (Parent)"

![](./Assets/add-on-product-1.png)

![](./Assets/add-on-product-5.png)

Simply paste the product code in and that's it! The add-on product will now appear with your main product. If you want to multiple add-on products then seperate each code with a comma.

![](./Assets/add-on-product-6.png)
