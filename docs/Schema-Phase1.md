# STOCKR - Schema Definition
**Phase 1 · SQLite via SQLAlchemy · 11 tables · v5 final**

> Images are excluded from Phase 1 — the terminal cannot display them. An `image_path` column will be added to `items` in Phase 2 when file serving is available via the API.

---

## Tables

- [users](#users)
- [sessions](#sessions)
- [categories](#categories)
- [items](#items)
- [sales](#sales)
- [orders](#orders)
- [order_items](#order_items)
- [inventory_adjustments](#inventory_adjustments)
- [audit_log](#audit_log)
- [activity_logs](#activity_logs)
- [indexes](#indexes)

---

## Auth

### users
> Login credentials and role assignment

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | Surrogate key |
| username | TEXT | UQ | NOT NULL, UNIQUE | Used for login |
| password_hash | TEXT | | NOT NULL | bcrypt hash — never store plain text |
| role | TEXT | | NOT NULL, CHECK IN ('management', 'restocker', 'customer') | Drives all access control |
| created_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

### sessions
> Tracks active and past login sessions per user

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| user_id | INTEGER | FK → users.id | NOT NULL | Who is logged in |
| logged_in_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| logged_out_at | DATETIME | | NULLABLE | NULL = session still active |

---

## Organization

### categories
> Product groupings for filtering and reporting — replaces departments

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | Surrogate key |
| name | TEXT | UQ | NOT NULL, UNIQUE | e.g. "Electronics", "Grocery" |
| description | TEXT | | NULLABLE | Optional category description |
| created_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

## Inventory

### items
> Every product tracked in the system

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| name | TEXT | | NOT NULL | Display name of the product |
| brand | TEXT | | NULLABLE | Optional brand name |
| upc | TEXT | UQ | NULLABLE, UNIQUE | Barcode — unique when present |
| description | TEXT | | NULLABLE | |
| category_id | INTEGER | FK → categories.id | NULLABLE | Items can exist without a category |
| location | TEXT | | NOT NULL | e.g. "Aisle 3, Shelf B" |
| price | REAL | | NOT NULL, CHECK >= 0 | Sell price in dollars |
| stock_quantity | INTEGER | | NOT NULL, DEFAULT 0, CHECK >= 0 | Units currently on the floor |
| in_storage | BOOLEAN | | NOT NULL, DEFAULT FALSE | TRUE = held in storage, not yet on floor |
| is_active | BOOLEAN | | NOT NULL, DEFAULT TRUE | FALSE = soft-deleted, history preserved |
| deactivation_reason | TEXT | | NULLABLE | Required when is_active = FALSE — enforced in service layer |
| created_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| updated_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Updated on every write via service layer |

---

## Transactions

### sales
> Auto-logged on stock decrease — manual entries and corrections by managers

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| item_id | INTEGER | FK → items.id | NOT NULL | Which item was sold |
| quantity_sold | INTEGER | | NOT NULL, CHECK > 0 | Units sold in this entry |
| price_at_sale | REAL | | NOT NULL | Snapshot — preserved if item price changes later |
| source | TEXT | | NOT NULL, CHECK IN ('auto', 'manual', 'correction') | auto = stock trigger · manual/correction = manager |
| user_id | INTEGER | FK → users.id | NULLABLE | NULL for auto entries, set for manual/correction |
| sold_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Used for daily/weekly/monthly/yearly report grouping |

---

### orders
> Restock order headers - created and managed by managers only

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| status | TEXT | | NOT NULL, CHECK IN ('pending', 'complete', 'cancelled') | |
| user_id | INTEGER | FK → users.id | NOT NULL | Manager who created the order |
| created_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| completed_at | DATETIME | | NULLABLE | Set when status → complete |

---

### order_items
> Line items linking orders to inventory - supports multiple items per order

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| order_id | INTEGER | FK → orders.id | NOT NULL | |
| item_id | INTEGER | FK → items.id | NOT NULL | |
| quantity | INTEGER | | NOT NULL, CHECK > 0 | Units being ordered |

---

## Corrections

### inventory_adjustments
> Stock changes that are not sales - corrections, damage, restock received

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| item_id | INTEGER | FK → items.id | NOT NULL | |
| user_id | INTEGER | FK → users.id | NOT NULL | Restocker or manager who made the change |
| quantity_change | INTEGER | | NOT NULL | Positive = stock added · negative = stock removed |
| reason | TEXT | | NOT NULL, CHECK IN ('correction', 'restock', 'damage', 'other') | |
| adjusted_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

## Auditing

### audit_log
> Field-level changes to items — immutable, append-only, never updated or deleted

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | |
| item_id | INTEGER | FK → items.id | NOT NULL | |
| user_id | INTEGER | FK → users.id | NOT NULL | Who made the change |
| action | TEXT | | NOT NULL, CHECK IN ('created', 'updated', 'deleted') | |
| field_changed | TEXT | | NULLABLE | e.g. "price", "stock_quantity" — NULL for created/deleted |
| old_value | TEXT | | NULLABLE | Stored as text regardless of original type |
| new_value | TEXT | | NULLABLE | Stored as text regardless of original type |
| changed_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

### activity_logs
> Broader user action tracking across all tables — immutable, append-only

| Column | Type | Key | Constraints | Notes |
|---|---|---|---|---|
| id | INTEGER | PK | AUTO INCREMENT, NOT NULL | Surrogate key |
| user_id | INTEGER | FK → users.id | NOT NULL | User who performed the action |
| action | TEXT | | NOT NULL | e.g. CREATE_ITEM, DELETE_ITEM, UPDATE_STOCK |
| target_table | TEXT | | NOT NULL | Table affected by the action |
| target_id | INTEGER | | NOT NULL | ID of the affected record |
| details | TEXT | | NULLABLE | Optional extra information |
| created_at | DATETIME | | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

## Performance

### indexes
> indexes for search and reporting performance

| Table | Column | Type | Notes |
|---|---|---|---|
| items | name | INDEX | Search by product name |
| items | upc | UNIQUE INDEX | Enforced by UNIQUE constraint |
| items | brand | INDEX | Filter/search by brand |
| items | category_id | INDEX | Filter by category |
| sales | sold_at | INDEX | Fast grouping for daily/weekly/monthly/yearly reports |
| audit_log | item_id | INDEX | Fast lookup of all changes for a given item |
| activity_logs | user_id | INDEX | Fast lookup of all actions by a given user |

---

*STOCKR · SCHEMA.md · Phase 1 · SQLite + SQLAlchemy · 11 tables*