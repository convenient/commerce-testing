---
title: Merge points for testing extensions | Commerce Testing
description: Learn how to avoid duplicating code when using the Functional Testing Framework to test Adobe Commerce and Magento Open Source projects.
---

# Merge points for testing extensions

The Functional Testing Framework allows great flexibility when writing XML tests for extensions.
All parts of tests can be used, reused, and merged to best suit your needs and cut down on needless duplication.

Extension developers can utilitze these merge points to leverage existing tests and modify just the parts needed to test their extension. For instance, if your extension adds a form field to a Catalog admin page, you can modify the existing Catalog tests and add actions, data, etc as needed to test the custom field.
This topic shows how to merge and reuse test elements when testing extensions.

## Merging

Follow the links below for an example of how to merge:

- [Merge Action Groups][]
- [Merge Data][]
- [Merge Pages][]
- [Merge Sections][]
- [Merge Tests][]

## Extending

Only Test, Action Group, and Data objects in the the Functional Testing Framework can be extended.
Extending can be very useful for extension developers since it will not affect existing tests.

Consult [when to use Extends][] to use extends when deciding whether to merge or extend.

- [Extend Action Groups][]
- [Extend Data][]
- [Extend Tests][]

<!-- Link definitions -->
[when to use Extends]: ../test-writing/best-practices.md#when-to-use-extends
[Merge Action Groups]: merge-action-groups.md
[Merge Data]: merge-data.md
[Merge Pages]: merge-pages.md
[Merge Sections]: merge-sections.md
[Merge Tests]: merge-tests.md
[Extend Action Groups]: extend-action-groups.md
[Extend Data]: extend-data.md
[Extend Tests]: extend-tests.md
