---
title: Functional test suites | Commerce Testing
description: Learn how to run groups of tests based on specific conditions for Adobe Commerce and Magento Open Source code using the Functional Testing Framework.
---

# Suites

Suites are essentially groups of tests that run in specific conditions (preconditions and postconditions).
They enable including, excluding, and grouping tests for a customized test run.
You can form suites using separate tests, groups, and modules.

Each suite must be defined in the `<VendorName>/<ModuleName>/Test/Mftf/Suite` directory.

The tests for each suite are generated in a separate directory under `<magento 2 root>/dev/tests/acceptance/tests/functional/Magento/_generated/`.
All tests that are not within a suite are generated in the _default_ suite at `<magento 2 root>/dev/tests/acceptance/tests/functional/Magento/_generated/default`.

<InlineAlert variant="info" slots="text"/>

If a test is generated into at least one custom suite, it will not appear in the _default_ suite.

## Format

The format of a suite:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suites xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../dev/tests/acceptance/vendor/magento/magento2-functional-testing-framework/src/Magento/FunctionalTestingFramework/Suite/etc/suiteSchema.xsd">
    <suite name="">
        <before>
        </before>
        <after>
        </after>
        <include>
            <test name=""/>
            <group name=""/>
            <module name="" file=""/>
        </include>
        <exclude>
            <test name=""/>
            <group name=""/>
            <module name="" file=""/>
        </exclude>
    </suite>
</suites>
```

## Principles

-  A suite name:

    -  must not match any existing group value.
  For example, the suite `<suite name="ExampleTest">` will fail during test run if any test contains in annotations `<group value="ExampleTest">`.
    -  must not be `default` or `skip`. Tests that are not in any suite are generated under the `default` suite.
       The suite name `skip` is synonymous to including a test in the `<group value="skip"/>`.
    -  can contain letters, numbers, and underscores.
    -  should be upper camel case.

-  A suite must contain at least one `<include>`, or one `<exclude>`, or both.
-  Using `<before>` in a suite, you must add the corresponding `<after>` to restore the initial state of your testing instance.
-  One `<suite>` tag is allowed per suite XML file.

## Conditions

Using suites enables test writers to consolidate conditions that are shared between tests.
The code lives in one place and executes once per suite.

-  Set up preconditions and postconditions using [actions](test/actions.md) in [`<before>`](#before) and [`<after>`](#after) correspondingly, just similar to use in a [test](test/index.md).
-  Clean up after suites just like after tests.
The Functional Testing Framework enforces the presence of both `<before>` and `<after>` if either is present.

## Test writing

Since suites enable precondition consolidation, a common workflow for test writing is adding a new test to an existing suite.
Such test is generated in context of the suite that contains it.
You cannot isolate this test from preconditions of the suite; it cannot be used outside of the suite at the same time.

There are several ways to generate and execute your new test in the context of a suite:

-  Edit the appropriate `suite.xml` to include your test only and run:

  ```bash
  vendor/bin/mftf run:group <suiteName>
  ```

-  Temporarily add a group to your test like `<group value="foo">` and run:

  ```bash
  vendor/bin/mftf run:group foo
  ```

-  To limit generation to your suite/test combination, run in conjunction with the above:

  ```bash
  vendor/bin/mftf generate:suite <suite>
  ```

-  To generate any combination of suites and tests, use [`generate:tests`](commands/mftf.md#generatetests) with the `--tests` flag.

## Examples

### Enabling/disabling WYSIWYG in suite conditions

<!-- {% raw %} -->

```xml
<suites xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../dev/tests/acceptance/vendor/magento/magento2-functional-testing-framework/src/Magento/FunctionalTestingFramework/Suite/etc/suiteSchema.xsd">
    <suite name="WYSIWYG">
        <before>
            <actionGroup ref="LoginAsAdmin" stepKey="login"/>
            <amOnPage url="admin/admin/system_config/edit/section/cms/" stepKey="navigateToConfigurationPage" />
            <waitForPageLoad stepKey="wait1"/>
            <conditionalClick stepKey="expandWYSIWYGOptions" selector="{{ContentManagementSection.WYSIWYGOptions}}" dependentSelector="{{ContentManagementSection.CheckIfTabExpand}}" visible="true" />
            <waitForElementVisible selector="{{ContentManagementSection.EnableWYSIWYG}}" stepKey="waitForEnableWYSIWYGDropdown1" />
            <waitForElementVisible  selector="{{ContentManagementSection.EnableSystemValue}}" stepKey="waitForUseSystemValueVisible"/>
            <uncheckOption selector="{{ContentManagementSection.EnableSystemValue}}" stepKey="uncheckUseSystemValue"/>
            <selectOption selector="{{ContentManagementSection.EnableWYSIWYG}}" userInput="Enabled by Default" stepKey="selectOption1"/>
            <click selector="{{ContentManagementSection.WYSIWYGOptions}}" stepKey="collapseWYSIWYGOptions" />
            <click selector="{{ContentManagementSection.Save}}" stepKey="saveConfig" />
        </before>
        <after>
            <actionGroup ref="LoginAsAdmin" stepKey="login"/>
            <actionGroup ref="DisabledWYSIWYG" stepKey="disable"/>
        </after>
        <include>
            <group name="WYSIWYG"/>
        </include>
    </suite>
</suites>
```

<!-- {% endraw %} -->
This example declares a suite with the name `WYSIWYG`.
The suite enables WYSIWYG _before_ running tests.
It performs the following steps:

1. Log in to the backend.
2. Navigate to the **Configuration** page.
3. Enable **WYSIWYG** in the Adobe Commerce or Magento Open Source instance.

_After_ the testing, the suite returns the Adobe Commerce or Magento Open Source instance to the initial state disabling WYSIWYG:

1. Log back in.
2. Disable **WYSIWYG** in the Adobe Commerce or Magento Open Source instance.

This suite includes all tests that contain the `<group value="WYSIWYG"/>` annotation.

### Execute CLI commands in suite conditions

```xml
<suites xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:mftf:Suite/etc/suiteSchema.xsd">
    <suite name="Cache">
        <before>
            <magentoCLI stepKey="disableCache" command="cache:disable"/>
        </before>
        <after>
            <magentoCLI stepKey="enableCache" command="cache:enable"/>
        </after>
        <include>
            <test name="SomeCacheRelatedTest"/>
            <group name="CacheRelated"/>
        </include>
    </suite>
</suites>
```

This example declares a suite with the name `Cache`.

Preconditions:

1. It disables the Adobe Commerce or Magento Open Source instance cache entirely before running the included tests.
2. After the testing, it re-enables the cache.

The suite includes a specific test `SomeCacheRelatedTest` and every `<test>` that includes the `<group value="CacheRelated"/>` annotation.

### Change configurations in suite conditions

```xml
<suites xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:mftf:Suite/etc/suiteSchema.xsd">
    <suite name="PaypalConfiguration">
        <before>
            <createData entity="SamplePaypalConfig" stepKey="createSamplePaypalConfig"/>
        </before>
        <after>
            <createData entity="DefaultPayPalConfig" stepKey="restoreDefaultPaypalConfig"/>
        </after>
        <include>
            <module name="Catalog"/>
        </include>
        <exclude>
            <test name="PaypalIncompatibleTest"/>
        </exclude>
    </suite>
</suites>
```

This example declares a suite with the name `PaypalConfiguration`:

-  `<before>` block persists a Paypal Configuration enabling all tests in this suite to run under the newly reconfigured Adobe Commerce or Magento Open Source instance.
-  `<after>` block deletes the persisted configuration, returning Adobe Commerce or Magento Open Source to its initial state.
-  The suite includes all tests from the `Catalog` module, except the `PaypalIncompatibleTest` test.

## Elements reference

### suites

The root element for suites.

### suite

A set of "before" and "after" preconditions, and test filters to include and exclude tests in the scope of suite.

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Unique suite name identifier.
`remove`|boolean|optional|Removing the suite during merging.

It can contain `<before>`, `<after>`, `<include>`, and `<exclude>`.

### before

A suite hook with preconditions that executes once before the suite tests.

It may contain test steps with any [actions](test/actions.md) and [action groups](test/action-groups.md).

<InlineAlert variant="info" slots="text" />

Tests in the suite are not run and screenshots are not saved in case of a failure in the before hook.
To troubleshoot the failure, run the suite locally.

### after

A suite hook with postconditions executed once after the suite tests.

It may contain test steps with any [actions](test/actions.md) and [action groups](test/action-groups.md).

### include

A set of filters that you can use to specify which tests to include in the test suite.

It may contain filters by:

-  test which names a specific `<test>`.
-  group which refers to a declared `group` annotation.
-  module which refers to `test` files under a specific module.

The element can contain [`<test>`](#test), [`<group>`](#group), and [`<module>`](#module).

### exclude

A set of filters that you can use to specify which tests to exclude in the test suite.

There are two types of behavior:

1. Applying filters to the included tests when the suite contains [`<include>`](#include) filters.
   The Functional Testing Framework will exclude tests from the previously included set and generate the remaining tests in the suite.
2. Applying filters to all tests when the suite does not contain [`<include>`](#include) filters.
   The Functional Testing Framework will generate all existing tests except the excluded.
   In this case, the custom suite will contain all generated tests except excluded, and the _default_ suite will contain the excluded tests only.

It may contain filters by:

-  test which names a specific `<test>`.
-  group which refers to a declared `group` annotation.
-  module which refers to `test` files under a specific module.

The element may contain [`<test>`](#test), [`<group>`](#group), and [`<module>`](#module).

### test

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Filtering a test by its name.
`remove`|boolean|optional|Removing the filter during merging.

### group

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Filtering tests by the `<group>` annotation.
`remove`|boolean|optional|Removing the filter during merging.

### module

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Filtering tests by their location in the corresponding module.
`file`|string|optional|Filtering a specific test file in the module.
`remove`|boolean|optional|Removing the filter during merging.
