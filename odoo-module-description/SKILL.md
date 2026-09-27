---
name: odoo-module-description
description: >-
  Use this skill whenever you need to create, modify, or structure an Odoo module's static description file (static/description/index.html). It contains the standard structure, CSS guidelines, and layout requirements for Odoo App Store presentation.
---

# Odoo Module Description Generation

This skill guides the creation and modification of Odoo module description files (`static/description/index.html`). This file is used by Odoo to display the module's details, features, and instructions in the Apps menu.

## File Location & Purpose
- **Path**: `static/description/index.html` (relative to the module root).
- **Images**: Place images like `banner.png` and `icon.png` in the same `static/description/` directory.
- **Purpose**: Acts as the "App Store" listing page or instruction manual within Odoo.

## Architectural Structure
The file should be a single HTML snippet that relies heavily on inline CSS and Odoo-specific Bootstrap classes. Do NOT use external CSS files, as they might conflict with Odoo's core styles or fail to load.

### Wrapper Classes
Always wrap the main content in these standard Odoo classes:
```html
<section class="oe_container">
  <div class="oe_row oe_spaced">
    <!-- Content goes here -->
  </div>
</section>
```

### Styling Approach
- **Inline CSS**: Use inline `style="..."` attributes for almost all styling to prevent bleeding into Odoo's core CSS (e.g., `style="color: #1a3c5e; font-size: 28px; font-weight: 700;"`).
- **Responsive Layouts**: Use modern CSS features like CSS Grid for feature lists. Example: `display: grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); gap: 20px;`.
- **Icons**: Use Unicode HTML entities (e.g., `&#x1F3A8;` for 🎨, `&#x26A1;` for ⚡) instead of external icon libraries like FontAwesome, ensuring they render correctly everywhere.

## Common Sections
When generating a complete description, structure the HTML with these visual sections:
1. **Banner**: An `<img>` tag loading `banner.png` (e.g., `<img src="banner.png" style="max-width:100%;">`).
2. **Hero Headline**: A boxed area introducing the module with badges for key highlights (e.g., Odoo version, license).
3. **Key Features**: A responsive grid highlighting the main selling points.
4. **Quick Start / Instructions**: A step-by-step numerical guide on how to install and use the module.
5. **Technical Specs / Dependencies**: Important warnings (like pip install requirements) or standard HTML `<table>` elements providing module metadata.

## Best Practices
- **No full HTML document tags**: Do not include `<html>`, `<head>`, or `<body>` tags. The content is injected directly into Odoo's DOM.
- **Self-contained**: Ensure all referenced images are in the `static/description` folder and referenced with relative paths (e.g., `src="icon.png"`).

## User-Centric Documentation Standards
When writing the "Quick Start" or "How to Use" sections, you must adhere to the following standards to ensure the guide is bullet-proof for implementers and end-users:

- **Explicit Menu Paths:** Never write vague instructions like "Go to Settings". Always provide the exact Odoo breadcrumb path (e.g., *Sales > Configuration > Settings > Discounts*).
- **Comprehensive Workflow Coverage:** Document all possible entry points. If a feature applies to Sales Orders, but also works directly on standalone Invoices or Vendor Bills, explicitly document those standalone workflows as well.
- **Precise UI Terminology:** Guide users exactly how to find hidden elements. For example, instead of saying "enable the column," write "click the 'optional columns' icon (three dots/slider) on the far right of the list headers to show the field."
- **Security & Edge Cases Section:** Always include a dedicated, highlighted section (e.g., a yellow warning box) that explicitly lists any Odoo Access Rights (Groups) or configuration dependencies (like `sale.group_discount_per_so_line`) that control the visibility of the new features. Explain *why* a user might not see a field if they haven't configured it properly.
