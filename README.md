# dbml-renderer-enhanced

Forked from [@softwaretechnik/dbml-renderer](https://github.com/softwaretechnik-berlin/dbml-renderer) v1.0.31.

## What's Different

This fork adds two features for SA (System Analysis) document ERDs:

### 1. Column-Level Color Highlighting

Add `[color: #hex]` to any column to highlight its row background. This enables reviewers to see exactly which columns changed — not just which table changed.

```dbml
Table iex_merchant_config {
  config_id varchar(64) [pk, not null]
  merchant_id varchar(64) [not null]
  settlement_currency varchar(3) [not null, color: #DCFCE7]  // green = modified
  new_field varchar(255) [color: #FEE2E2]                    // light red = new
  deleted tinyint [not null, default: 0]                     // default = existing
}
```

**Color convention for SA ERDs:**

| Scenario | Header | Column Background |
|----------|--------|-------------------|
| Existing table, no change | Default blue `#1d71b8` | Default grey `#e7e2dd` |
| New table | Red `#DC2626` | All rows: `#FEE2E2` (light red) |
| Existing table, new column | Default blue `#1d71b8` | New column only: `#FEE2E2` |
| Existing table, modified column | Default blue `#1d71b8` | Modified column only: `#DCFCE7` (light green) |

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

// New table — red header + all columns light red
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

// Modified table — default header, changed column colored green
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
- Fork changes: column-level color support, NN marker