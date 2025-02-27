---
title: Input testing data | Commerce Testing
description: Define data entities for the Functional Testing Framework on Adobe Commerce and Magento Open Source projects.
---

# Input testing data

The Functional Testing Framework enables you to specify and use `<data>` entities defined in XML. Default `<data>` entities are provided for use and as templates for entity creation and manipulation.
The following diagram shows the XML structure of a data object:

![Data Object](../_images/functional-testing/data-dia.svg)

The `<data>` entities are stored in `<module_dir>/Test/Mftf/Data/`.

## Supply data to test by reference to a data entity

Test steps requiring `<data>` input in an action, like filling a field with a string, may reference an attribute from a data entity:

```xml
userInput="{{SimpleSubCategory.name}}"
```

In this example:

*  `SimpleSubCategory` is an entity name.
*  `name` is a `<data>` key of the entity. The corresponding value will be assigned to `userInput` as a result.

The following is an example of the usage of `<data>` entity in the `Magento/Customer/Test/Mftf/Test/AdminCustomersAllCustomersNavigateMenuTest.xml` test:

```xml
<actionGroup ref="AdminNavigateMenuActionGroup" stepKey="navigateToAllCustomerPage">
    <argument name="menuUiId" value="{{AdminMenuCustomers.dataUiId}}"/>
    <argument name="submenuUiId" value="{{AdminMenuCustomersAllCustomers.dataUiId}}"/>
</actionGroup>
```

In the above example:

*  `AdminMenuCustomers` is an entity name.
*  `dataUiId` is a `<data>` key of the entity.

### Environmental data

```xml
userInput="{{_ENV.MAGENTO_ADMIN_USERNAME}}"
```

In this example:

*  `_ENV` is a reference to the `dev/tests/acceptance/.env` file, where basic environment variables are set.
*  `MAGENTO_ADMIN_USERNAME` is a name of an environment variable.
   The corresponding value will be assigned to `userInput` as a result.

The following is an example of the usage of `_ENV` in the `Magento/Braintree/Test/Mftf/ActionGroup/AdminDeleteRoleActionGroup.xml` action group:

```xml
<fillField stepKey="TypeCurrentPassword" selector="{{AdminDeleteRoleSection.current_pass}}" userInput="{{_ENV.MAGENTO_ADMIN_PASSWORD}}"/>
```

### Sensitive data

```xml
userInput="{{_CREDS.my_secret_token}}"
```

In this example:

*  `_CREDS` is a constant to reference to the `dev/tests/acceptance/.credentials` file, where sensitive data and secrets are stored for use in a test.
*  `MY_SECRET_TOKEN` is the name of a key in the credentials variable.
  The corresponding value of the credential will be assigned to `userInput` as a result.
*  The decrypted values are only available in the `.credentials` file in which they are stored.

Learn more in [Credentials][].

The following is an example of the usage of `_CREDS` in the `Magento/Braintree/Test/Mftf/Data/BraintreeData.xml` data entity:

```xml
<entity name="MerchantId" type="merchant_id">
    <data key="value">{{_CREDS.magento/braintree_enabled_fraud_merchant_id}}</data>
</entity>
```

## Persist a data entity as a prerequisite of a test

A test can specify an entity to be persisted (created in the database) so that the test actions could operate on the existing known data.

Example of referencing `data` in a test:

```xml
userInput="$createCustomer.email$"
```

In this example:

*  `createCustomer` is a step key of the corresponding test step that creates an entity.
*  `email` is a data key of the entity.
  The corresponding value will be assigned to `userInput` as a result.

The following is an example of the usage of the persistant data in `Magento/Customer/Test/Mftf/Test/AdminCreateCustomerWithCountryUSATest.xml` test:

```xml
<actionGroup ref="AdminFilterCustomerByEmail" stepKey="filterTheCustomerByEmail">
    <argument name="email" value="$$createCustomer.email$$"/>
</actionGroup>
```

<InlineAlert variant="info" slots="text" />

As of Functional Testing Framework 2.3.6, you no longer need to differentiate between scopes (a test, a hook, or a suite) for persisted data when referencing it in tests.

The Functional Testing Framework now stores the persisted data and attempts to retrieve it using the combination of `stepKey` and the scope of where it has been called.
The current scope is preferred, then widening to _test > hook > suite_ or _hook > test > suite_.

This emphasizes the practice for the `stepKey` of `createData` to be descriptive and unique, as a duplicated `stepKey` in both a `<test>` and `<before>` prefers the `<test>` data.

## Use data returned by test actions

A test can also reference data that was returned as a result of [test actions][], like the action `<grabValueFrom selector="someSelector" stepKey="grabStepKey>`.

Further in the test, the data grabbed by the `someSelector` selector can be referenced using the `stepKey` value. In this case, it is `grabStepKey`.

The `stepKey` value can only be referenced within the test scope that it is defined in (`test`, `before/after`).

The following example shows the usage of `grabValueFrom` in testing, where the returned value is used by action's `stepKey`:

```xml
<grabValueFrom selector="someSelector" stepKey="grabStepKey"/>
<fillField selector=".functionalTestSelector" userInput="{$grabStepKey}" stepKey="fillFieldKey1"/>
```

The following is an example of the `Magento/Catalog/Test/Mftf/ActionGroup/AssertDiscountsPercentageOfProductsActionGroup.xml` test:

```xml
<grabValueFrom selector="{{AdminProductFormAdvancedPricingSection.productTierPricePercentageValuePriceInput('0')}}" stepKey="grabProductTierPriceInput"/>
<assertEquals stepKey="assertProductTierPriceInput">
    <expectedResult type="string">{{amount}}</expectedResult>
    <actualResult type="string">$grabProductTierPriceInput</actualResult>
</assertEquals>
```

## Hard-coded data input

The data to operate against can be included as literals in a test. Hard-coded data input can be useful in assertions.

See also [Actions][].

```xml
userInput="We'll email you an order confirmation with details and tracking info."
```

## Format

The format of the `<data>` entity is:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:mftf:DataGenerator/etc/dataProfileSchema.xsd">
    <entity name="" type="">
        <data key=""></data>
    </entity>
    <entity name="" type="">
        <data key="" unique=""></data>
        <var key="" entityType="" entityKey=""/>
    </entity>
</entities>
```

## Principles

The following conventions apply to `<data>` entities:

*  A `<data>` file may contain multiple data entities.
*  Camel case is used for `<data>` elements. The name represents the `<data>` type. For example, a file with customer data is `CustomerData.xml`. A file for simple product would be `SimpleProductData.xml`.
*  Camel case is used for the entity name.
*  The file name must have the suffix `Data.xml`.

## Example

Example (`Magento/Catalog/Test/Mftf/Data/CategoryData.xml` file):

```xml
<?xml version="1.0" encoding="UTF-8"?>

<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:mftf:DataGenerator/etc/dataProfileSchema.xsd">
    <entity name="_defaultCategory" type="category">
        <data key="name" unique="suffix">simpleCategory</data>
        <data key="name_lwr" unique="suffix">simplecategory</data>
        <data key="is_active">true</data>
    </entity>
    <entity name="SimpleSubCategory" type="category">
        <data key="name" unique="suffix">SimpleSubCategory</data>
        <data key="name_lwr" unique="suffix">simplesubcategory</data>
        <data key="is_active">true</data>
        <data key="include_in_menu">true</data>
    </entity>
</entities>
```

This example declares two `<data>` entities: `_defaultCategory` and `SimpleSubCategory`. They set the data required for [category creation][].

All entities that have the same name will be merged during test generation. Both entities are of the `category` type.

`_defaultCategory` sets three data fields:

*  `name` defines the category name as `simpleCategory` with a unique suffix. Example: `simpleCategory598742365`.
*  `name_lwr` defines the category name in lowercase format with a unique suffix. Example: `simplecategory697543215`.
*  `is_active` sets the enable category to `true`.

`SimpleSubCategory` sets four data fields:

*  `name` that defines the category name with a unique suffix. Example: `SimpleSubCategory458712365`.
*  `name_lwr` that defines the category name in lowercase format with a unique suffix. Example: `simplesubcategory753698741`.
*  `is_active` sets the enable category to `true`.
*  `include_in_menu` that sets the include in the menu to `true`.

The following is an example of a call in test:

```xml
<fillField selector="{{AdminCategoryBasicFieldSection.categoryNameInput}}" userInput="{{_defaultCategory.name}}" stepKey="enterCategoryName"/>
```

This action inputs data from the `name` of the `_defaultCategory` entity (for example, `simpleCategory598742365`) into the field with the locator defined in the selector of the `categoryNameInput` element of the `AdminCategoryBasicFieldSection`.

You can also call data from the xml definition of a `data` tag directly:

```xml
<entity name="NewAdminUser" type="user">
    <data key="username" unique="suffix">admin</data>
    <data key="current_password">{{AnotherUser.current_password}}</data>  <!-- Data from another entity -->
    <data key="current_password">{{_ENV.MAGENTO_ADMIN_PASSWORD}}</data>  <!-- ENV file reference -->
</entity>
```

## Reference

### entities

`<entities>` is an element that contains all `<entity>`  elements.

### entity

`<entity>` is an element that contains `<data>` elements.

Attributes|Type|Use|Description
---|---|---|---
`name`|string|optional|Name of the `<entity>`. Use camel case for entity names.
`type`|string|optional|Node containing the exact name of `<entity>` type. Used later to find specific Persistence Layer Model class. `type` in `<data>` can be whatever the user wants; There are no constraints. It is important when persisting data, depending on the `type` given, as it will try to match a metadata definition with the operation being done. Example: A `myCustomer` entity with `type="customer"`, calling `<createData entity="myCustomer"/>`, will try to find a metadata entry with the following attributes: `<operation dataType="customer" type="create">`.
`deprecated`|string|optional|Used to warn about the future deprecation of the data entity. String will appear in Allure reports and console output at runtime.

`<entity>` may contain one or more [`<data>`][], [`<var>`][], [`<required-entities>`][], or [`<array>`][] elements in any sequence.

### data

`<data>` is an element containing a data/value pair.

Attributes|Type|Use|Description
---|---|---|---
`key`|string|optional|Key attribute of data/value pair.
`unique`|enum: `"prefix"`, `"suffix"`|optional|Add suite or test wide unique sequence as "prefix" or "suffix" to the data value if specified.

Example:

```xml
<data key="name" unique="suffix">simpleCategory</data>
```

### var

`<var>` is an element that can be used to grab a key value from another entity. For example, when creating a customer with the `<createData>` action, the server responds with the auto-incremented ID of that customer. Use `<var>` to access that ID and use it in another data entity.

Attributes|Type|Use|Description
---|---|---|---
`key`|string|optional|Key attribute of this entity to assign a value to.
`entityType`|string|optional|Type attribute of referenced entity.
`entityKey`|string|optional|Key attribute of the referenced entity from which to get a value.
`unique`|--|--|_This attribute hasn't been implemented yet._

Example:

```xml
<var key="parent_id" entityType="category" entityKey="id" />
```

### requiredEntity

`<requiredEntity>` is an element that specifies the parent/child relationship between complex types.

Example: You have customer address info. To specify that relationship:

```xml
<entity name="CustomerEntity" type="customer">
    ...
    <requiredEntity type="address">AddressEntity</requiredEntity>
    ...
</entity>
```

Attributes|Type|Use|Description
---|---|---|---
`type`|string|optional|Type attribute of `<requiredEntity>`.

### array

`<array>` is an element that contains a reference to an array of values.

Example:

```xml
<entity name="AddressEntity" type="address">
    ...
    <array key="street">
        <item>7700 W Parmer Ln</item>
        <item>Bld D</item>
    </array>
    ...
</entity>
```

Attributes|Type|Use|Description
---|---|---|---
`key`|string|required|Key attribute of this entity in which to assign a value.

`<array>` may contain [`<item>`][] elements.

### item

`<item>` is an individual piece of data to be passed in as part of the parent `<array>` type.

Attributes|Type|Use|Description
---|---|---|---
`name`|string|optional|Key attribute of <item/> entity in which to assign a value. By default numeric key will be generated.

<!-- Link Definitions -->
[`<array>`]: #array
[`<data>`]: #data
[`<item>`]: #item
[`<required-entities>`]: #requiredentity
[`<var>`]: #var
[Actions]: test/actions.md
[category creation]: https://docs.magento.com/user-guide/catalog/category-create.html
[Credentials]: credentials.md
[test actions]: test/actions.md#actions-returning-a-variable
