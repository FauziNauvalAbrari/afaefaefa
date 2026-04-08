# Kalla ICT Odoo Development Standards

> **Document Version:** 1.2
> **Last Updated:** 2026-04-08
>
> **Changelog:**
> - **v1.2** — Added `kict_` prefix purpose statement; added inherited model scenario with file/field/method naming rules; clarified XML ID override rules; added module structure template and `__manifest__.py` template; replaced upgrade procedure with CLI as primary method
> - **v1.1** — Initial release

## Naming Conventions

All custom development must follow these naming conventions to ensure consistency and maintainability:

### Prefix Guidelines

The prefix used in all custom development identifies the organization or vendor responsible for the customization.

> **Purpose:** The `kict_` prefix exists to create a clear, visible boundary between Kalla ICT custom code and original Odoo code, vendor modules, and third-party addons. Any file, model, field, method, or XML ID carrying this prefix was written or owned by Kalla ICT — not Odoo, not a vendor. This makes it immediately obvious what is safe to modify, what is under our version control, and what should be reviewed during upgrades.

- **kict\_** - Used for Kalla ICT (Kalla Information and Communication Technology)
  - This is the standard prefix for all internal Kalla ICT custom development
  - Example: `kict_inventory_management`, `kict_sales_order`

- **Vendor Prefixes** - For modules customized by external vendors
  - Each vendor must use a unique abbreviation that describes their organization
  - The abbreviation should be short (2-5 characters), lowercase, and meaningful
  - Format: `<vendor_abbr>_<module_purpose>`
  - Examples:
    - `abc_` for ABC Solutions: `abc_custom_reports`
    - `xyz_` for XYZ Technologies: `xyz_integration_module`
    - `tk_` for TechKnow: `tk_advance_vehicle_repair`

**Important:** Once a vendor prefix is established, it must be used consistently across all their custom modules to maintain clear ownership and traceability.

### Module Names

- **Prefix:** `kict_`
- **Format:** `kict_<module_purpose>`
- **Example:** `kict_inventory_management`

### Model Names

- **Prefix:** `kict.`
- **Format:** `kict.<model_name>`
- **Example:** `kict.sales.order`

### Field Names

- **Prefix:** `kict_` (only for custom (not original) fields in inherited models)
- **Format:** `kict_<field_name>`
- **Example:** `kict_delivery_date`

### Method Names

- **Prefix:** `kict_` (only for custom (not original) methods in inherited models)
- **Format:** `kict_<method_name>`
- **Example:** `kict_calculate_discount()`

### XML IDs

- **Prefix:** `kict_`
- **Format:** `kict_<xml_id>`
- **Example:** `kict_view_invoice_form`

**New records (new models):** Always use the `kict_` prefix.

```xml
<record id="kict_driver_profile_form_view" model="ir.ui.view">
    ...
</record>
```

**Inherited/override records (extending existing Odoo views):** The `kict_` prefix applies to the ID of the *new* inherited view you are creating. The `inherit_id` reference keeps the original Odoo ID unchanged — never add `kict_` to an Odoo-owned ID.

```xml
<!-- Correct -->
<record id="kict_fleet_vehicle_form_view_inherit" model="ir.ui.view">
    <field name="inherit_id" ref="fleet.fleet_vehicle_form_view"/>  <!-- original Odoo ID, no prefix -->
    ...
</record>

<!-- Wrong — do not prefix the reference to an existing Odoo record -->
<field name="inherit_id" ref="fleet.kict_fleet_vehicle_form_view"/>
```

### File Names

- **XML Files:** `kict_<filename>.xml`
- **Python Files:** `kict_<filename>.py`

### Inherited Model Scenario

When extending an existing Odoo model using `_inherit`, different rules apply compared to creating a new model. The table below summarises what to prefix and what to leave unchanged:

| Element | Rule | Example |
|---|---|---|
| `_name` | Do **not** redefine — keep Odoo's original | `fleet.vehicle` |
| New fields | Prefix with `kict_` | `kict_driver_id` |
| New methods | Prefix with `kict_` | `kict_validate_licence()` |
| Python file | `kict_<original_model>_inherit.py` | `kict_fleet_vehicle_inherit.py` |
| XML view file | `kict_<original_model>_views_inherit.xml` | `kict_fleet_vehicle_views_inherit.xml` |

**Example — extending `fleet.vehicle`:**

```python
# File: kict_fleet_vehicle_inherit.py
class FleetVehicleInherit(models.Model):
    _inherit = 'fleet.vehicle'          # keep original _name, do NOT set _name

    kict_driver_id = fields.Many2one(   # kict_ prefix on new field
        'kict.driver.profile',
        string='Assigned Driver',
    )

    def kict_validate_assignment(self):  # kict_ prefix on new method
        """
        DESC: Validates driver assignment rules before saving
        INPUT: none
        OUTPUT: bool — True if valid
        """
        ...
```

> **Key Rule:** The `kict_` prefix on fields and methods marks them as Kalla ICT additions. Original Odoo fields and methods on the same model are left unchanged and unprefixed. Never rename or re-prefix original Odoo fields.

## Code Documentation

### Method Comments

Every method must include a docstring with:

- **DESC:** Brief description of the method's purpose
- **INPUT:** List of parameters with types and descriptions
- **OUTPUT:** Description of return value(s) with types

**Example:**

```python
def kict_calculate_total(self, amount, discount):
    """
    DESC: Calculates the total amount after applying discount
    INPUT:
        - amount (float): Original amount
        - discount (float): Discount percentage
    OUTPUT:
        - total (float): Final amount after discount
    """
```

## Changelog Management

Every module must maintain a `CHANGELOG.md` file in the module's root directory to track all modifications.

### Changelog Format

```markdown
# Changelog - [Module Name]

## [Version] - YYYY-MM-DD

### Added

- New features or functionality

### Changed

- Changes to existing functionality

### Fixed

- Bug fixes

### Removed

- Removed features or functionality

### Security

- Security-related changes
```

### Changelog Rules

- **Mandatory:** Update changelog for every change to the module
- **Location:** `CHANGELOG.md` in module root directory
- **Version Format:** Follow Odoo versioning format `17.0.X.Y.Z` (see [Module Versioning](#module-versioning) below)
- **Date Format:** YYYY-MM-DD
- **Entry Details:** Include ticket/issue reference numbers
- **Before Commit:** Always update changelog before committing changes
- **Categories:** Use appropriate category (Added, Changed, Fixed, Removed, Security)

**Example:**

```markdown
# Changelog - kict_inventory_management

## [17.0.1.2.0] - 2024-01-15

### Added

- New field `kict_stock_location` in product form [TICKET-123]
- Method `kict_validate_stock()` for inventory validation

### Fixed

- Corrected calculation in `kict_calculate_total()` [TICKET-125]

## [17.0.1.1.0] - 2024-01-10

### Changed

- Updated inventory report layout [TICKET-120]
```

## Additional Best Practices

### Security

- Always define access rights in `ir.model.access.csv`
- Implement record rules where necessary
- Validate user permissions in critical methods

### Performance

- Use `@api.depends()` for computed fields
- Avoid loops in ORM operations when possible
- Use SQL queries sparingly and only when necessary

### Code Quality

- Follow PEP 8 style guidelines
- Keep methods focused on single responsibility
- Use meaningful variable names
- Avoid hardcoded values; use configuration parameters

### Dependency Management

- Always include a `requirements.txt` file in the module root directory
- List all Python packages/plugins required by the module
- Specify package versions to ensure consistency (e.g., `requests==2.28.1` or `pandas>=1.5.0`)
- Update `requirements.txt` whenever adding new Python dependencies
- Test installation from `requirements.txt` before committing
- Document any system-level dependencies in module README
- Avoid using unnecessary packages; minimize dependencies

### Module Versioning

All custom modules must use the Odoo 5-segment version format:

```
17 . 0 . X . Y . Z
│    │   │   │   └── Patch  — bug fixes, typo corrections, minor tweaks (no migration needed)
│    │   │   └────── Minor  — new features, backward-compatible additions (reset Z to 0)
│    │   └────────── Major  — breaking changes, field renames/removals, model restructure (reset Y and Z to 0)
│    └────────────── Odoo minor — always 0 for Odoo 17
└─────────────────── Odoo major — must match your Odoo version (17)
```

#### When to Bump Each Segment

| Segment       | When to Increment                                            | Examples                                                   | Migration Script?                                        |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Z** (Patch) | Bug fixes, view corrections, label changes                   | Fix wrong calculation, fix broken domain                   | Not required                                             |
| **Y** (Minor) | New fields with defaults, new views, new menus, new reports  | Add `kict_delivery_date` field, add new action             | Usually not required; add if populating existing records |
| **X** (Major) | Rename/remove fields, change field types, restructure models | Rename `kict_old_field` → `kict_new_field`, remove a model | **Required** — add script under `migrations/17.0.X.0.0/` |

#### Rules

- **Never change** the first two segments (`17.0`) — they are locked to the Odoo version
- When bumping **Y**, always reset **Z** to `0`
- When bumping **X**, always reset **Y** and **Z** to `0`
- Every module must start at `17.0.1.0.0` at initial release
- A high patch counter (e.g. `Z > 10`) is a signal to review whether some of those changes should have bumped **Y** instead
- Avoid short formats like `0.1` or `2.6` — these do not follow the Odoo standard and must be corrected

#### Migration Scripts

When bumping **X** (major), a migration script is required:

```
module_name/
  migrations/
    17.0.2.0.0/
      pre-migrate.py    ← runs before ORM loads (rename columns, drop constraints)
      post-migrate.py   ← runs after ORM loads (data migration, recompute fields)
```

### Module Structure Template

Every new KICT module must follow this folder structure. Folders marked *(required)* must always exist; others are created only when needed.

```
kict_<module_name>/
├── __init__.py                  (required)
├── __manifest__.py              (required)
├── CHANGELOG.md                 (required)
├── models/                      (required)
│   └── __init__.py
├── views/                       (required)
│   └── menus.xml
├── security/                    (required)
│   └── ir.model.access.csv
├── data/                        when seed/config data is needed
│   └── sequence_data.xml
├── report/                      when printable reports are needed
├── wizard/                      when transient models are needed
├── controllers/                 when REST/JSON endpoints are needed
└── static/description/
    └── icon.png
```

### Manifest Template

Every module's `__manifest__.py` must include all fields below. Do not omit any — inconsistent manifests cause upgrade and dependency resolution issues.

```python
# -*- coding: utf-8 -*-
{
    'name': 'KICT Module Display Name',
    'version': '17.0.1.0.0',           # always start here; follow versioning rules
    'category': 'Category/Subcategory', # see Odoo category list
    'summary': 'One-line description of what this module does',
    'description': """
        Longer description if needed.
        Can be multi-line.
    """,
    'author': 'Kalla ICT',
    'website': 'https://kalla.co.id',
    'license': 'LGPL-3',               # always LGPL-3 for internal modules
    'depends': [
        'base',
        # list all direct dependencies explicitly
    ],
    'data': [
        # load order matters — security first, then data, then views
        'security/ir.model.access.csv',
        'security/<module>_security.xml',
        'data/sequence_data.xml',
        'views/<model>_views.xml',
        'views/menus.xml',
    ],
    'demo': [],
    'application': False,              # True only for top-level app modules
    'installable': True,
    'auto_install': False,
}
```

> **Note on `application`:** Set to `True` only if the module is a standalone top-level application that appears in the Odoo home app list (e.g. a full Driver Hub app). Feature extension modules must always be `False`.

### Version Control

- Commit frequently with clear messages
- Reference ticket/issue numbers in commits
- Keep commits atomic and focused

## Code Review Guidelines

All code changes must undergo a thorough review process before being merged into the main branch. The following checklist must be verified during code review:

### 1. Naming Convention Compliance

- [ ] All modules, models, fields, and methods follow the correct prefix convention (`kict_` or vendor-specific)
- [ ] XML IDs use appropriate prefixes
- [ ] File names follow the naming standards
- [ ] Variable names are meaningful and follow Python conventions

### 2. Documentation Requirements

- [ ] All methods include proper docstrings with DESC, INPUT, and OUTPUT
- [ ] CHANGELOG.md is updated with all changes
- [ ] Version number is incremented appropriately using the `17.0.X.Y.Z` Odoo format (see [Module Versioning](#module-versioning))
- [ ] Correct segment bumped: Z for bug fixes, Y for new features, X for breaking changes
- [ ] When X is bumped, migration script is provided under `migrations/17.0.X.0.0/`
- [ ] Ticket/issue references are included in changelog entries
- [ ] Complex logic has inline comments explaining the approach

### 3. Code Quality

- [ ] Code follows PEP 8 style guidelines
- [ ] No hardcoded values; configuration parameters or constants are used
- [ ] Methods have single responsibility and are not overly complex
- [ ] No commented-out code or debug statements (e.g., `print()`, `pdb`)
- [ ] No duplicate code; common logic is extracted into reusable methods
- [ ] Error handling is implemented where necessary

### 4. Security Checks

- [ ] Access rights are defined in `ir.model.access.csv`
- [ ] Record rules are implemented where necessary
- [ ] User permissions are validated in critical operations
- [ ] No SQL injection vulnerabilities (use parameterized queries)
- [ ] Sensitive data is not logged or exposed
- [ ] Input validation is performed for user-supplied data

### 5. Performance Considerations

- [ ] Computed fields use `@api.depends()` decorator properly
- [ ] Bulk operations are used instead of loops where possible
- [ ] Database queries are optimized (no N+1 queries)
- [ ] Unnecessary database calls are avoided
- [ ] Large datasets are handled with pagination or lazy loading

### 6. Functionality Verification

- [ ] Code has been tested in development environment
- [ ] All existing functionality still works (no regression)
- [ ] New features work as expected
- [ ] Edge cases and error scenarios are handled
- [ ] Module dependencies are correctly declared in `__manifest__.py`
- [ ] All required Python packages are listed in `requirements.txt`
- [ ] Package versions are specified in `requirements.txt` (e.g., `package==1.2.3` or `package>=1.2.0`)

### 7. Database Changes

- [ ] Migration scripts are provided for schema changes
- [ ] Data migration preserves existing records
- [ ] Migration scripts are tested and reversible when possible
- [ ] Default values are set for new required fields

### 8. User Interface

- [ ] Views follow Odoo UI/UX guidelines
- [ ] Field labels are clear and user-friendly
- [ ] Required fields are properly marked
- [ ] Help text is provided for complex fields
- [ ] Responsive design is maintained

### 9. Testing

- [ ] Unit tests are included for critical business logic
- [ ] Test coverage is adequate
- [ ] All tests pass successfully
- [ ] Manual testing has been performed

### 10. Git Practices

- [ ] Commit messages are clear and descriptive
- [ ] Commits are atomic and focused on single changes
- [ ] No merge conflicts exist
- [ ] Branch is up to date with target branch
- [ ] No unnecessary files are committed (e.g., `__pycache__`, `.pyc` files)

### Review Process

1. **Developer:** Create pull/merge request with clear description
2. **Reviewer:** Go through the checklist and test the changes
3. **Feedback:** Provide constructive feedback with specific suggestions
4. **Revision:** Developer addresses all feedback
5. **Approval:** Reviewer approves after all issues are resolved
6. **Merge:** Authorized person merges the approved changes

### Review Best Practices

- **Be Constructive:** Focus on improvement, not criticism
- **Be Specific:** Point to exact lines and provide suggestions
- **Be Timely:** Review requests should be addressed within 24 hours
- **Be Thorough:** Don't rush through reviews; quality is paramount
- **Ask Questions:** If something is unclear, ask for clarification
- **Test Locally:** Pull the code and test it in your environment when necessary

## Module Upgrade Procedure

To ensure smooth deployment and avoid errors when upgrading edited modules, follow this exact sequence:

### Upgrade Steps (Must Follow This Order)

1. **Merge the Branch**
   - Complete the code review process
   - Merge the approved branch into the target branch (development/production)
   - Ensure the merge is successful with no conflicts
   - Wait for the code to be deployed to the server (if using CI/CD)

2. **Upgrade via CLI (Primary Method)**

   The CLI is the reliable, repeatable upgrade method and must be used as the standard approach:

   ```bash
   docker exec -it <odoo_container_name> odoo -u <module_name> -d <database_name>
   ```

   **Example:**
   ```bash
   docker exec -it odoo17 odoo -u kict_driver_hub_management -d kalla_production
   ```

   To upgrade multiple modules at once:
   ```bash
   docker exec -it odoo17 odoo -u kict_driver_hub_management,kict_fleet_transporter_extend -d kalla_production
   ```

3. **Upgrade via UI (Secondary Method)**

   Use the UI only when CLI access is unavailable:
   - Navigate to Apps menu in Odoo
   - Search for the module
   - Click the **Upgrade** button on the module card
   - Wait for the upgrade to complete and verify no errors appear

### Important Notes

- **CLI is preferred** — it provides clear error output in the terminal and is not affected by browser session state
- **Verify Changes:** After upgrade, test the new functionality to ensure it works as expected
- **Rollback Plan:** Always have a rollback strategy (database backup) before upgrading in production

### Troubleshooting

If errors occur during upgrade:

1. Check server logs for detailed error messages
2. Verify all dependencies are properly installed
3. Ensure database migrations completed successfully
4. If upgrade fails, do not retry without investigating the cause
5. Contact the development team for assistance if needed

### Post-Upgrade Checklist

- [ ] Module upgraded successfully without errors
- [ ] New features are working as expected
- [ ] Existing functionality remains intact (no regression)
- [ ] CHANGELOG.md reflects the current version
- [ ] Users are notified of new features/changes if applicable
