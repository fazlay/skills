---
name: odoo-20-migration
description: >-
  Specialized skill for migrating Odoo 19 (and earlier) custom modules to Odoo 20.
  Covers all structural breaking changes, manifest strictness, model removals (e.g. res.bank),
  Javascript OWL 2.4 updates (proxy, strict this context, removed hooks), and security data file requirements (ir.access.csv strictness).
---

# Odoo 20 Migration Guide

When the user asks you to migrate modules to Odoo 20, or you encounter errors during Odoo 20 development/upgrades, strictly apply the following knowledge base:

## 1. Manifest Files (`__manifest__.py`)
Odoo 20 enforces strict typing on manifest dictionary keys.
- **Booleans**: Fields like `installable`, `application`, and `auto_install` **MUST** be boolean values (`True` or `False`). String representations like `"True"` or `"False"` will crash the registry with an `AssertionError`.

## 2. Security Files (`ir.access.csv`)
Odoo 20 no longer accepts loosely formatted CSV files.
- **No Blank Permissions**: You can no longer leave permission columns blank (e.g., `1,1,,`). They must be explicitly set to `1` or `0`.
- **Headers**: The header row must perfectly align with the 7 required columns: `id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink`.

## 3. Python Model & Field Changes
- **`res.bank` removed**: The `res.bank` model has been removed entirely. Its fields (like `bank_name`) have been flattened into `res.partner.bank`. Update all `Many2one` relations from `res.bank` to `res.partner.bank`.
- **HR Module Changes**: 
  - `hr.contract` in Odoo 18/19 was refactored heavily. Much of its core logic was moved to `hr.version`.
  - `hr.version.type` has been refactored/replaced by `hr.employee.type`.
  - `hr.departure.wizard` has been removed.
- **Strict Field Kwargs**: Arbitrary kwargs (like `track_visibility`, `states`, `size`) on fields are no longer allowed out-of-the-box. They will cause `unknown parameter` warnings/errors. You must either remove them, use the standard alternatives (e.g. `tracking=True`), or override `_valid_field_parameter` on the model.
- **Field Renames**: `groups_id` has been renamed to `group_ids` across the codebase.

## 4. OWL / Javascript Frontend (OWL 2.4+)
Odoo 20 uses an updated version of OWL which fundamentally drops support for some older syntaxes.
- **`useState` Removed**: The `useState` hook is completely removed from `@odoo/owl` for class components. You must use `proxy` (or `reactive`) instead.
  - **Before**: `this.state = useState({ isVisible: false });`
  - **After**: `this.state = proxy({ isVisible: false });`
- **Strict Template Context (`this.`)**: QWeb template evaluation contexts are now strictly bound to the component instance. You can no longer access component state or methods without the `this.` prefix.
  - **Before**: `<div t-if="state.isVisible" t-on-click="onClick">`
  - **After**: `<div t-if="this.state.isVisible" t-on-click="this.onClick">`
- **`useExternalListener` Removed**: You can no longer import `useExternalListener`. Instead, bind window/document events inside `onMounted` and `onWillUnmount`.
  ```javascript
  const onKeydown = this.onKeydown.bind(this);
  onMounted(() => window.addEventListener("keydown", onKeydown));
  onWillUnmount(() => window.removeEventListener("keydown", onKeydown));
  ```

## 5. QWeb Views & XML
- **`t-esc` deprecated**: Replace all instances of `t-esc` with `t-out`.
- **Removed Xpaths**: Ensure xpath targets still exist in the base Odoo 20 views (e.g., `tz` in `resource.calendar` was replaced by `company_id`).

## 6. Static Description Files (`index.html`)
- **No XML Declarations**: Odoo 20 parses `static/description/index.html` files strictly. You must remove the `<?xml version="1.0" encoding="utf-8"?>` declaration from the top of HTML files, otherwise the module parser will throw a `ParseError`.
