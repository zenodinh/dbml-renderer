# dbml-renderer-enhanced

Forked from [@softwaretechnik/dbml-renderer](https://github.com/softwaretechnik-berlin/dbml-renderer) v1.0.31.

## What's Different

This fork adds two features for SA (System Analysis) document ERDs:

### 1. Column-Level Color Highlighting

Add `[color: #hex]` to any column to change the **text color** of that column name. Column cell background stays **white** — clean and readable. This lets reviewers see exactly which columns changed at a glance.

```dbml
Table iex_merchant_config {
  config_id varchar(64) [pk, not null]
  merchant_id varchar(64) [not null]
  settlement_currency varchar(3) [not null, color: #DCFCE7]  // light green = modified
  new_field varchar(255) [color: #FEE2E2]                    // light red = new
  deleted tinyint [not null, default: 0]                      // default text color
}
```

**Color convention for SA ERDs:**

| Element | Color | Hex | Usage |
|---------|-------|-----|-------|
| Table header (existing) | Blue | `#1d71b8` | Default — no setting needed |
| Table header (new) | Red | `#DC2626` | New table added in this SA |
| Table header (modified) | Green | `#16A34A` | Existing table with column changes |
| Column cell background | White | `#ffffff` | All column cells — clean and readable |
| Column text (existing) | Dark grey | `#29235c` | Default — no setting needed |
| Column text (new) | Light red | `#FEE2E2` | New column in any table |
| Column text (modified) | Light green | `#DCFCE7` | Modified column in any table |

**Visual style:** White column cells with clean borders, matching dbdiagram.io's clean visual theme.

### 2. NN Marker for NOT NULL

NOT NULL columns display `NN` instead of `(!)` — matching the dbdiagram.io convention.

```
// Before (upstream):  varchar(64) (!)
// After (this fork):  varchar(64) NN
```

## Installation

```bash
npm install dbml-renderer-enhanced
```

Or use directly via npx:

```bash
npx dbml-renderer-enhanced -i schema.dbml -o schema.svg
```

## Usage

```bash
# Render SVG (default)
dbml-renderer-enhanced -i schema.dbml -o output.svg

# Render DOT
dbml-renderer-enhanced -i schema.dbml -f dot -o output.dot

# Output to stdout
dbml-renderer-enhanced -i schema.dbml
```

## Full SA ERD Example

```dbml
Project merchant_sa {
  database_type: 'MySQL'
}

// Existing table — no color changes
Table iex_merchant {
  merchant_id varchar(64) [pk, not null]
  merchant_name varchar(255) [not null]
  status varchar(32) [not null]
  deleted tinyint [not null, default: 0]
  gmt_create timestamp [not null, default: `now()`]
  gmt_modified timestamp [not null, default: `now()`]
}

// New table — red header + all columns have light red text
Table iex_merchant_product [headercolor: #DC2626] {
  product_id varchar(64) [pk, not null, color: #FEE2E2]
  merchant_id varchar(64) [not null, color: #FEE2E2]
  product_name varchar(255) [not null, color: #FEE2E2]
  product_code varchar(64) [unique, not null, color: #FEE2E2]
  deleted tinyint [not null, default: 0, color: #FEE2E2]
  gmt_create timestamp [not null, default: `now()`, color: #FEE2E2]
  gmt_modified timestamp [not null, default: `now()`, color: #FEE2E2]

  Indexes {
    (merchant_id, product_code) [unique]
    merchant_id
  }
}

// Modified table — default header, changed column has green text
Table iex_merchant_config {
  config_id varchar(64) [pk, not null]
  merchant_id varchar(64) [not null]
  settlement_currency varchar(3) [not null, color: #DCFCE7]
  deleted tinyint [not null, default: 0]
  gmt_create timestamp [not null, default: `now()`]
  gmt_modified timestamp [not null, default: `now()`]
}

Ref: iex_merchant_product.merchant_id > iex_merchant.merchant_id [delete: cascade]
Ref: iex_merchant_config.merchant_id > iex_merchant.merchant_id [delete: cascade]
```

## Credits

- Original: [softwaretechnik-berlin/dbml-renderer](https://github.com/softwaretechnik-berlin/dbml-renderer) — MIT License
- Fork changes: column-level color (text), NN marker, white cell backgrounds