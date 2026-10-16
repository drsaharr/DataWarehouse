# Sales ETL & Data Warehouse — Northwind DW

<div align="center">

**A modern SQL Server Data Warehouse project for Sales Analytics**

_T-SQL • Star/Snowflake Architecture • Dimension–Fact Modeling • Full Load ETL_

</div>

---

## 📌 Overview

This repository contains the complete source code and documentation for an enterprise-ready **Sales Data Warehouse** built on **Microsoft SQL Server** using the Northwind dataset.

The project implements an auditable **Staging → Dimension → Fact** pipeline with a rich dimensional model (including Outrigger Geographies and a Shared Persian/Gregorian Date Dimension), optimized for analytical workloads, OLAP reporting, and BI tools such as Power BI.

---

## 🗂️ Repo Contents

| Path | Description |
|---|---|
| `scripts/` | Database DDL, staging tables, and ETL stored procedures |
| `dataset/` | Source database initialization files |
| `documents/` | Architecture blueprints, Data Flow diagrams, and Data Catalog |

```text
.
├── dataset/             # Source data initialization scripts
├── documents/           # Architecture diagrams, Blueprint & Data Catalog
├── scripts/             # DDL & ETL Stored Procedures
└── README.md
```

---

## 🏗️ Architecture & Blueprint

The architecture is derived directly from the **Architecture Blueprint**, linking dimensional hierarchies and outriggers:

```text
       [Dim_Geography]
        ▲     ▲     ▲
        │     │     │ (FK: geography_key)
        │     │  [Dim_Supplier]
        │     │     ▲
        │     │     │ (FK: supplier_key)
[Dim_Customer]│  [Dim_Product]   [Dim_Employee]   [Dim_Shipper]   [Dim_Date (Shared)]
       │      │        │                │               │          (3 Date Roles)
       │      └────────┼────────────────┼───────────────┼─────────────────┘
       ▼               ▼                ▼               ▼
 ┌────────────────────────────────────────────────────────────────────────┐
 │                              Fact_Order                                │
 │ (Grain: One row per Order ID + Product ID)                             │
 │ Keys: product_key, customer_key, employee_key, shipper_key, date_keys  │
 │ Measures: quantity, unit_price, discount_amount, freight, total_amount │
 └────────────────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

- **Shared Date Dimension (`Dim_Date`)** — Supports dual Gregorian (`MiladiDate`) and Solar Hijri (`ShamsiDate`) calendars, serving role-playing dates: `OrderDate`, `RequiredDate`, and `ShippedDate`.
- **Outrigger Geography Hierarchy** — Normalized `Dim_Geography` dimension referenced directly by Customer, Employee, and Supplier entities.
- **Supplier-to-Product Outrigger** — Products maintain reference to their corresponding suppliers via `supplier_key`.
- **Surrogate Keys & Default Unknown Members** — All dimensions utilize auto-incrementing integer keys; unmatched lookups default to surrogate key `0`.
- **Transactional ETL Control** — ETL stored procedures employ `XACT_ABORT ON` and atomic `TRY...CATCH` blocks with rollback protection.

---

## 🧱 Schema Layers

| Layer | Schema | Responsibility |
|---|---|---|
| **Staging** | `stage` | Raw extracted source tables; temporary holding area |
| **Dimension** | `dds` / `dbo` | Clean, standardized master entities and outrigger dimensions |
| **Fact** | `dds` / `dbo` | Atomic business transactions at Order Line granularity |

---

## 📚 Core Tables & Dimensions

### Dimension Tables

- **`Dim_Geography`** — Common geographical attributes (`geography_key`, `country`, `region`, `city`, `postal_code`).
- **`Dim_Customer`** — Customer profiles and contacts (`customer_key`, `customer_id`, `customer_name`, `geography_key`).
- **`Dim_Supplier`** — Vendor details (`supplier_key`, `supplier_id`, `supplier_name`, `contact_name`, `geography_key`).
- **`Dim_Product`** — Product catalog details (`product_key`, `product_id`, `product_name`, `category_name`, `unit_price`, `supplier_key`).
- **`Dim_Employee`** — Sales representatives (`employee_key`, `employee_id`, `first_name`, `last_name`, `title`, `geography_key`).
- **`Dim_Shipper`** — Logistics carriers (`shipper_key`, `shipper_id`, `shipper_name`, `phone`).
- **`Dim_Date`** — Shared enterprise calendar (`MiladiDateKey`, `MiladiDate`, `ShamsiDate`, `Year`, `Month`, `Day`, `Quarter`, `DayOfWeek`).

### Fact Table

- **`Fact_Order`**
  - **Grain:** One row per Order ID + Product ID.
  - **Metrics & Measures:** `quantity`, `unit_price`, `discount_amount`, `freight_amount`, `total_amount`.
  - **Role-playing Keys:** `order_date_key`, `required_date_key`, `shipped_date_key`.

---

## 🔧 ETL Pipeline

### Load Sequence

1. Truncate / Ingest source data into Staging tables.
2. Load Outrigger dimension: `Dim_Geography`.
3. Load primary dimensions: `Dim_Date`, `Dim_Supplier`, `Dim_Customer`, `Dim_Employee`, `Dim_Shipper`.
4. Load dependent dimension: `Dim_Product` (resolving `supplier_key`).
5. Resolve all surrogate keys and insert into `Fact_Order`.

### Standard Cleansing Pattern

```sql
COALESCE(
    NULLIF(LTRIM(RTRIM(SourceColumn)), N''),
    N'Unknown'
)
```

### Transaction Pattern

```sql
SET XACT_ABORT ON;

BEGIN TRY
    BEGIN TRANSACTION;

    -- Staging / Dimension / Fact ETL Load logic

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    THROW;
END CATCH;
```

---

## 📊 Key Business KPIs

| KPI | Description |
|---|---|
| **Gross Sales Amount** | Total sales volume before deductions and discounts |
| **Net Sales Amount** | Revenue realized after discount deductions |
| **Total Freight** | Total logistics and freight costs |
| **Order Line Volume** | Count of distinct transactional items processed |

---

## 🧰 Prerequisites

- **Microsoft SQL Server** (2019 or later recommended)
- SQL Server Management Studio (SSMS) or Azure Data Studio
- Northwind sample database

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/hossein-khajvand/northwind_DW.git
cd northwind_DW
```

### 2. Prepare Source Database

Execute the script inside `dataset/NorthWind.txt` to initialize the Northwind operational database.

### 3. Deploy Staging & DDL

Execute the scripts located in `scripts/`:

- `Stage_CreateTables.sql`
- `DDS_northwind_sale_createTables.sql`

### 4. Run the ETL Procedures

Execute ETL stored procedures in dependency order:

```sql
EXEC dbo.Stage_sp_LoadStage;
EXEC dbo.DDS_Usp_load_dim_northwind;
EXEC dbo.Usp_load_fact_order_full;
```

---

## 📖 Documentation

Additional technical specifications are available in `documents/`:

- **Architecture Blueprint:** Detailed physical design in `Northwind_Data_Warehouse_Architecture_Blueprint.pdf`.
- **Data Flow Diagrams:** Visual flow representations in `Data Flow.drawio.pdf`.
- **Data Catalog:** Data dictionary, field types, and business definitions.

---

## 🛠️ Roadmap

- [ ] Add automated monitoring and data testing.
- [ ] Add Slowly Changing Dimensions (SCD Type 2) support.
- [ ] Build connected Power BI executive dashboards.

---

## 👤 Maintainer

**sahar teimouri** — Data Analyst &Data scientist  
