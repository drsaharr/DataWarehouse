# Sales ETL & Data Warehouse â€” GitHub README

<div align="center">

**A modern SQL Server Data Warehouse project for Sales Analytics**

_T-SQL â€¢ Star Schema â€¢ Dimensionâ€“Fact Architecture â€¢ Full Load ETL_

</div>

---

## ðŸ“Œ Overview

This repository contains the complete source code and documentation for a **Sales Data Warehouse** built on **Microsoft SQL Server**.

The project implements a clean **Staging â†’ Dimension â†’ Fact** architecture with a denormalized star schema, enabling reliable, auditable, and analytics-ready sales reporting for BI tools such as Power BI and Grafana.

---

## ðŸ—‚ï¸ Repo Contents

| Path | Description |
|---|---|
| `sql/schema` | Database, schema, and table creation scripts (DDL) |
| `sql/etl` | Stored procedures for ETL loading |
| `sql/quality` | Data quality and validation queries |
| `data-catalog/` | Operational and business metadata documentation |
| `docs/` | Supplemental design and architecture notes |

```
.
â”œâ”€â”€ sql
â”‚   â”œâ”€â”€ schema/          # DDL for stg, dim, fact, etl layers
â”‚   â”œâ”€â”€ etl/             # ETL stored procedures (full load)
â”‚   â””â”€â”€ quality/         # Data quality & validation queries
â”œâ”€â”€ data-catalog/        # DATA_CATALOG.md
â”œâ”€â”€ docs/                # Architecture & design documentation
â””â”€â”€ README.md
```

---

## ðŸ—ï¸ Architecture

The warehouse follows a classic layered design:

```
Source Systems
      â”‚
      â–¼
  [stg] Staging Layer        â†’ Raw ingestion, minimal transformation
      â”‚
      â–¼
  [dim] Dimension Layer      â†’ Business entities (Product, Customer, Date)
      â”‚
      â–¼
  [fact] Fact Layer          â†’ Measurable transactions (Sales)
      â”‚
      â–¼
    BI / Reporting
```

### Key Design Decisions

- **Denormalized Star Schema** â€” the `Product` dimension includes category attributes directly, reducing required joins.
- **Full Load ETL** â€” dimensions and facts are rebuilt on each run for a consistent, reproducible state.
- **Surrogate Keys** â€” every dimension has a dedicated integer key (`ProductKey`, `CustomerKey`, etc.).
- **Unknown Members** â€” missing lookups map to surrogate key `0` to preserve referential integrity.
- **Atomic Transactions** â€” all ETL runs under transactional control with error handling.

---

## ðŸ§± Schema Layers

| Schema | Type | Responsibility |
|---|---|---|
| `stg` | Staging | Stores raw extracted source data |
| `dim` | Dimension | Stores descriptive attributes for analysis |
| `fact` | Fact | Stores measurable business transactions |
| `etl` | Operational | Stores ETL logs and execution metadata |

---

## ðŸ“š Core Tables

### Dimensions

- **`dim.Product`** â€” product attributes, including denormalized category and brand.
- **`dim.Customer`** â€” customer attributes and location.
- **`dim.Date`** â€” reusable calendar dimension.

### Fact

- **`fact.Sales`** â€” grain is one row per order line, with quantity, price, and amount calculations.

### Staging

- **`stg.SalesOrder`**, `stg.Customer`, `stg.Product` â€” raw source snapshots.

---

## ðŸ”§ ETL Pipeline

### Load Sequence

1. Load staging tables from source.
2. Clean and validate incoming records.
3. Load dimension tables.
4. Resolve surrogate keys.
5. Load the fact table.
6. Write execution logs.

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

    -- ETL operations here

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    THROW;
END CATCH;
```

---

## ðŸ“Š KPI Examples

| KPI | Formula |
|---|---|
| **Gross Sales** | `SUM(GrossAmount)` |
| **Net Sales** | `SUM(NetAmount)` |
| **Total Discount** | `SUM(DiscountAmount)` |
| **Total Quantity Sold** | `SUM(Quantity)` |
| **Order Count** | `COUNT(DISTINCT OrderKey)` |

---

## âœ… Data Quality

The pipeline includes validation queries for:

- Duplicate source lines
- Invalid or negative quantities
- Inconsistent financial calculations
- Orphan fact records
- Unknown member mapping rates

---

## ðŸ§° Prerequisites

- **Microsoft SQL Server** (2019+ recommended)
- SQL Server Management Studio (or equivalent)
- Permission to create databases, schemas, tables, and stored procedures

---

## ðŸš€ Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/sales-dwh.git
cd sales-dwh
```

### 2. Create the database and schema

Execute the DDL scripts under `sql/schema` against your target SQL Server instance.

### 3. Deploy the ETL procedures

Execute the stored procedures under `sql/etl`. Each procedure adopts the full load strategy.

### 4. Run the pipeline

```sql
EXEC etl.usp_load_dim_northwind;
EXEC etl.usp_load_fact_order_full;
```

### 5. Validate

Run the validation queries under `sql/quality` to confirm data integrity.

---

## ðŸ“– Documentation

| Document | Purpose |
|---|---|
| `DATA_CATALOG.md` | Complete metadata, definitions, and business rules |

---

## ðŸ› ï¸ Roadmap

- [ ] Add incremental load support for large-volume tables
- [ ] Introduce Slowly Changing Dimension (SCD) Type 2 for history
- [ ] Add automated monitoring in Grafana / Loki
- [ ] Publish curated reporting views
- [ ] Add CI checks for schema drift

---

## ðŸ‘¤ Maintainers

**sahar teimouri** ” Data analyst & Data scientist 

