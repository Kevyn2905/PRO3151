## Marketplace Seller Management Web Application

A data‑driven web application for marketplace sellers to manage daily operations, understand profitability, and make informed decisions. The app ingests marketplace order data (products, quantities, prices, net profit, margins, taxes, fees, etc.), transforms it into an analytics‑ready model, and exposes interactive dashboards via Streamlit on top of a structured Python backend.

---

### 1. Project Description

**Goal**: Provide marketplace sellers with a single pane of glass to monitor sales, profitability, and tax exposure across products, orders, and time periods.

**Key capabilities**:
- **Centralized data ingestion** from marketplace exports (e.g., CSV/Excel/API).
- **Automated ETL** to clean, normalize, and enrich order data.
- **Interactive dashboards** built with Streamlit for sales, profitability, and tax analysis.
- **Configurable business rules** (e.g., tax rates, fees, cost assumptions) applied consistently in the backend.

The application is built in Python with a clear separation between **backend (data & domain logic)** and **frontend (Streamlit UI)**.

---

### 2. Functional Requirements

- **Order Data Import**
  - Upload order data files (CSV/Excel) exported from one or more marketplaces.
  - Validate file structure (required columns, data types, date formats).
  - Map marketplace‑specific columns to an internal canonical schema.
  - Store raw files and normalized tables for traceability.

- **Sales Metrics & KPIs**
  - Aggregate and visualize:
    - Total revenue, orders, units sold.
    - Average order value (AOV).
    - Sales by product, category, marketplace, country, channel.
    - Time‑series trends (daily/weekly/monthly).
  - Filter and slice metrics by:
    - Date range.
    - Marketplace / country / currency.
    - Product / SKU / category.

- **Profitability Analysis**
  - Compute net profit and margin at:
    - Order level.
    - Product / SKU level.
    - Marketplace / channel level.
    - Time period (day/week/month/quarter).
  - Break down profitability into:
    - Gross revenue.
    - Discounts, refunds, and returns.
    - Marketplace fees and commissions.
    - Shipping income and shipping costs.
    - Product cost of goods sold (COGS).
  - Display:
    - Net profit (absolute).
    - Margin percentage.
    - Contribution margin by product and category.
    - Top/bottom performing products and marketplaces.

- **Tax Breakdown & Compliance Support**
  - Show tax amounts per:
    - Order.
    - Product / SKU.
    - Jurisdiction (country, state, VAT region, etc.).
  - Support multiple tax types:
    - VAT / GST / sales tax.
    - Marketplace‑withheld taxes (if provided in exports).
  - Aggregate tax amounts for reporting:
    - Tax collected vs. tax payable (where applicable).
    - Tax by period and jurisdiction.
  - Export tax summaries in CSV/Excel for accounting or tax filing.

- **Data Exploration & Reporting**
  - Interactive tables with sorting, filtering, and search.
  - Downloadable reports (CSV/Excel) for:
    - Orders.
    - Products.
    - Profitability and tax breakdowns.
  - Basic anomaly checks (e.g., negative margins, missing costs, inconsistent tax rates).

---

### 3. Non‑Functional Requirements

- **Performance**
  - Handle at least **hundreds of thousands of order records** without freezing the UI.
  - Use efficient in‑memory operations (Pandas, NumPy) and incremental loading where possible.
  - Cache expensive aggregations to reduce recomputation between interactions.

- **Security**
  - Do not log or expose sensitive customer data unnecessarily.
  - Option to mask or omit personally identifiable information (PII) in the UI and exports.
  - Secure configuration management for credentials (if any API integrations are added), using environment variables or secrets.
  - Clearly document data storage locations (local files, database).

- **Scalability**
  - Architecture supports evolving from:
    - Local single‑user deployments (Streamlit run locally).
    - To shared internal deployments (Streamlit on a server).
    - To a multi‑user environment (with authentication, database backend) if required later.
  - Modular ETL and analytics layers to plug in additional marketplaces or data sources.

- **Usability**
  - Clean, intuitive navigation between pages (overview, sales, profitability, taxes, data management).
  - Clear labels, tooltips, and descriptions for metrics and filters.
  - Responsive layouts that adapt to different screen sizes.
  - Meaningful error messages and status indicators (e.g., when uploads fail or data is incomplete).

---

### 4. Solution Architecture

The application follows a **backend/frontend architecture**, both implemented in Python.

- **Data Ingestion Layer**
  - Accepts order data from:
    - File uploads (CSV, Excel).
    - (Optional future) marketplace APIs or scheduled jobs.
  - Responsibilities:
    - File validation (schema, required columns).
    - Standardization of date/time, currencies, numeric fields.
    - Logging of imports (file meta, timestamps, status).

- **ETL & Data Processing Layer**
  - Built using **Pandas** and **NumPy**.
  - Core steps:
    - **Extract**: Read raw files into dataframes.
    - **Transform**:
      - Clean and normalize raw fields.
      - Map marketplace columns to canonical schema (`order_id`, `sku`, `quantity`, `unit_price`, `fees`, `tax`, etc.).
      - Enrich with:
        - Product master data (cost, category, brand).
        - Exchange rates (if multi‑currency).
        - Fee and tax rules.
      - Compute derived metrics:
        - Revenue, COGS, fees, net profit, margin.
    - **Load**:
      - Store processed datasets in:
        - In‑memory data structures (for local mode).
        - Or a database / Parquet files (for scalable mode).
  - Provide a data access layer (e.g., repository/service functions) for the Streamlit frontend to query aggregated views.

- **Backend Services / Domain Logic**
  - Encapsulate business logic for:
    - KPI calculation.
    - Profitability breakdown.
    - Tax rules and jurisdiction logic.
  - Expose a simple interface to the frontend, e.g.:
    - `get_sales_summary(filters)`.
    - `get_profitability_by_sku(filters)`.
    - `get_tax_summary(filters)`.

- **Streamlit Frontend (UI Layer)**
  - Organized into logical pages/tabs:
    - **Overview Dashboard**
      - Key KPIs, quick filters, high‑level charts.
    - **Sales Analytics**
      - Time‑series charts, breakdown by SKU, category, marketplace.
    - **Profitability Analytics**
      - Margin and profit breakdowns, top/bottom lists.
    - **Tax Dashboard**
      - Tax amounts by jurisdiction, type, period, and export options.
    - **Data Management**
      - File upload, data quality checks, import history.
  - Uses Streamlit components for:
    - Filters (date range, marketplace, product, country).
    - Charts (line, bar, pie, tables).
    - Download buttons for reports.

- **Configuration & Extension Points**
  - Central configuration for:
    - Marketplace mappings.
    - Tax rates and fee rules.
    - Currency and localization settings.
  - Easily extendable to:
    - Additional marketplaces (new mapping config + ETL adapter).
    - Additional dashboards or analytics modules.

---

### 5. Technology Stack

- **Programming Language**
  - Python 3.10+ (recommended).

- **Core Libraries**
  - **Pandas**: Tabular data manipulation, ETL, and analytics.
  - **NumPy**: Numerical computations and vectorized operations.
  - **Streamlit**: Frontend framework for data apps and dashboards.

- **Optional Libraries (future enhancements)**
  - **SQLAlchemy** / database driver (PostgreSQL, SQLite, etc.) for persistent storage.
  - **PyArrow** for fast columnar storage (e.g., Parquet files).
  - **Plotly / Altair** for advanced interactive visualizations.

---

### 6. Project Structure

A suggested high‑level project layout:

```text
lab-web-app/
├─ backend/
│  ├─ __init__.py
│  ├─ config/
│  │  ├─ __init__.py
│  │  ├─ settings.py          # Global settings (paths, env, feature flags)
│  │  └─ tax_rules.py         # Tax rates & rules by jurisdiction
│  ├─ ingestion/
│  │  ├─ __init__.py
│  │  ├─ file_loader.py       # File upload handling & parsing (CSV/Excel)
│  │  └─ schema_mapping.py    # Marketplace -> internal schema mapping
│  ├─ etl/
│  │  ├─ __init__.py
│  │  ├─ transform.py         # Cleaning, normalization, enrichment
│  │  └─ metrics.py           # KPI, profitability & tax calculations
│  ├─ repositories/
│  │  ├─ __init__.py
│  │  └─ data_access.py       # Read/write processed data (memory/db/files)
│  └─ services/
│     ├─ __init__.py
│     ├─ sales_service.py     # Sales metrics service functions
│     ├─ profit_service.py    # Profitability service functions
│     └─ tax_service.py       # Tax breakdown & summaries
│
├─ frontend/
│  ├─ __init__.py
│  ├─ app.py                  # Streamlit entry point, page routing
│  ├─ pages/
│  │  ├─ overview_page.py     # Overview dashboard
│  │  ├─ sales_page.py        # Sales analytics
│  │  ├─ profit_page.py       # Profitability analytics
│  │  ├─ tax_page.py          # Tax dashboards
│  │  └─ data_page.py         # Uploads, data quality, import history
│  └─ components/
│     ├─ filters.py           # Shared filter widgets (date, marketplace, etc.)
│     └─ charts.py            # Shared chart/table rendering helpers
│
├─ data/
│  ├─ raw/                    # Raw marketplace export files
│  ├─ processed/              # Cleaned & normalized datasets
│  └─ exports/                # Generated reports (CSV/Excel)
│
├─ tests/
│  ├─ __init__.py
│  ├─ test_ingestion.py
│  ├─ test_etl.py
│  ├─ test_metrics.py
│  └─ test_services.py
│
├─ requirements.txt           # Python dependencies
└─ README.md                  # Project documentation (this file)
```

You can adapt this structure based on deployment target (local only vs. server), but keeping a clear separation between `backend` and `frontend` modules will make the application easier to test, extend, and maintain.

