---
name: odoo-19-migration
description: Specialized skill for migrating Odoo 18 custom modules to Odoo 19. Covers all structural breaking changes, manifest strictness, model removals (hr.contract), and database constraints (res.groups, ir.actions.server) discovered during migration.
---

# Odoo 18 to 19 Migration Guide

Use this skill when upgrading custom modules from Odoo 18 to Odoo 19. Odoo 19 introduces several strict parsing rules and major structural changes to core modules (especially HR and Security).

## 1. HR Core: Removal of `hr.contract`
The `hr.contract` model has been completely removed in Odoo 19. All contract logic is now handled by `hr.version`.
- **Python Inheritance:** Change `_inherit = 'hr.contract'` to `_inherit = 'hr.version'`. Ensure you also check arrays: `_inherit = ['hr.version', 'mail.thread']`.
- **Relational Fields:** Change `fields.Many2one('hr.contract')` to `fields.Many2one('hr.version')`.
- **Report Bindings:** Change `'doc_model': 'hr.contract'` to `'doc_model': 'hr.version'` in Python and XML reports.
- **XML Views:** Any `inherit_id` pointing to `hr_contract.hr_contract_view_form` or `hr_contract.hr_contract_view_tree` must be removed or rewritten to target `hr.hr_version_view_form` or `hr.hr_version_list_view`.
- **Data References:** Any XML `<field name="model_id" ref="hr_contract.model_hr_contract"/>` must be updated to `ref="hr.model_hr_version"`.

## 2. HR Core: `hr.employee` Bank Accounts
In Odoo 19, the `bank_account_id` field on `hr.employee` has been deprecated/removed.
- It is replaced by a Many2many `bank_account_ids` and a Many2one `primary_bank_account_id`.
- **Migration:** Change all references from `employee.bank_account_id` to `employee.primary_bank_account_id` in Python logic and QWeb reports.

## 3. Strict Python Manifest Parsing (`__manifest__.py`)
Odoo 19's module registry loader uses extremely strict AST parsing for `__manifest__.py` files. A single syntax error will silently cause the module to be "not available" without throwing an immediate traceback.
- **Dangling Commas:** Remove any dangling commas inside empty lists or dictionaries (e.g., `'assets': { , }`).
- **Multiline Strings:** Always use triple quotes `"""` for multiline descriptions. Using a single quote `'` across multiple lines will cause a fatal `SyntaxError: unterminated string literal`.

## 4. Security: `res.groups` Category Removed
The `category_id` field has been removed from the `res.groups` model (replaced by `privilege_id`).
- **Migration:** Remove `<field name="category_id" ref="..."/>` from all `<record model="res.groups">` blocks in your `security.xml` files. Leaving it will result in `ValueError: Invalid field 'category_id' in 'res.groups'`.

## 5. Server Actions: Strict SQL Constraints
The `ir.actions.server` model now enforces a strict `NOT NULL` constraint on the `state` field at the PostgreSQL level.
- **Migration:** You MUST define `<field name="state">code</field>` (or the appropriate state) in every Server Action XML record. Omitting it will throw `psycopg2.errors.NotNullViolation: null value in column "state"`.

## 6. Python Logic: Handling Empty Dates
Fields in `hr.version` (like `contract_date_start`) can often be empty (`False`) during data migration.
- `fields.Date.from_string(False)` returns `None`.
- Attempting to access attributes like `.day` or use `relativedelta()` on `None` will crash the view render.
- **Migration:** Always wrap date calculations in `if contract.contract_date_start:` guards.
