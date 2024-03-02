# Miva Merchant - ADPM (Add-Product Multiple) on PROD Page

## Table of Contents
- [Description](#description)
- [Goals] (#Purpose and Goals)


## Description

This is a walk through on how to implement add-on products onto main product pages (PROD Page) in Miva Merchant eccomerce stores that use shadows readytheme. This works in tandem with the attribute machine and retains an "out-of-box" feel of shadows readytheme.

## Goals

- Create an easier, more efficient and enjoyable user experience. 
The main goal of this project is to allow customers to add recommended or related products to their basket alongside a main product all in a single form submit. This makes the user shopping experience quicker, more efficient and more enjoyable. More items added to cart lead to increase sales average.

- More accurate inventory tracking and discount/tax calculation 
Prior to this project, we used product attributes to imnplement add-on products. This caused the add-on products to inherit certain characteristics from the main product like tax status, discounts, etc. By using the ADPM action each product becomes their own line item in the basket and thus doesn't inherit anything from the main product. Because of this inventory tracking and  discount calculations are more accurate.

- Retain Attribute Machine Functionality
ADPM does not work with the attribute machine at the time of writing. This project works around this limitation to keep the same user experience as before.


