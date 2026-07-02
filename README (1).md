# Project Title: Retail Sales Analysis — A Power BI Project 📊

The project's objective is to use a real simulated retail sales dataset to build a Power BI Dashboard. The goal is to demonstrate the ability to clean, analyze, transform, model, visualize, and extract insights driving actionable recommendations using Power BI.

## Data Source 📂🔢

The project starts from a single source file, **RetailData** (Excel/CSV), containing transaction-level retail sales data with the following fields: Order Id, Order_Date, item_id, Category, Sub_Category, Sub_Sub_Category, Brand, Unit_Price, Qty, Discount_Percent, Unit_Cost, City, State, Region, loc_id, Order_Type, Payment_Method, and Customer_id.

**Objective:** Analyze retail sales transactions and build a star schema model to enable fast, flexible reporting on revenue, margin, products, customers, locations, and payment behavior.

**Key Focus Areas:** Sales and margin trends over time, product category/brand performance, regional distribution, customer behavior, and payment method preferences.

## Data Model — Star Schema ⭐

The raw dataset was profiled, cleaned, and transformed into a star schema composed of one fact table and six dimension tables:

**FACT_SALES** (176,964 rows)
- Order_Id, date_id (FK), item_id (FK), loc_id (FK), Customer_id (FK), region_id (FK), payment_id (FK), Order_Type, Unit_Price, Qty, Discount_Percent, Unit_Cost, Montant_Brut, Montant_Net, Marge

**DIM_ITEM_PRODUIT** (2,644 rows) — item_id, Category, Sub_Category, Sub_Sub_Category, Brand

**DIM_LOCALISATION** (299 rows) — loc_id, City, State

**DIM_CUSTOMERS** (72,270 rows) — Customer_id

**DIM_DATES** (1,461 rows) — date_id, Order_Date, Day Name, Month Name, Quarter, Year

**DIM_PAYMENT** (6 rows) — payment_id, Payment_Method

**DIM_REGION** (4 rows) — region_id, Region

> **Data quality note:** Region was found to be inconsistent at the location level (a single loc_id could map to multiple regions), so it was modeled as its own dimension linked directly to the fact table rather than as an attribute of DIM_LOCALISATION.

## Installation Instructions ⚙️📥

1. Clone the repository: `git clone https://github.com/YOUR-USERNAME/project-name.git`
2. Install Power BI Desktop from the official website: Power BI Desktop
3. Open Power BI Desktop and load the CSV files (`fact_sales.csv` + the 6 `dim_*.csv` files)
4. Build relationships (1-to-many) from each dimension table to `fact_sales` using the corresponding keys
5. Explore the dashboard, interact with the visualizations, and gain insights from the data 📊🔍

## Features 🛠️📊

**Data Exploration & Understanding**: Data profiling, summary statistics, and functional dependency checks (e.g. loc_id ↔ Region) to validate schema design decisions. 📈🔎

**Data Cleaning & Preparation**: Deduplication, key generation, and consistency checks (e.g. loc_id → City/State validated as 100% unique) using Python/pandas. 🧹🔧

**Data Modelling**: Star schema design with a central fact table and conformed dimensions, including surrogate keys for Date, Payment, and Region. 🗂️🔗

**Data Transformation**: Calculated columns (Montant_Brut, Montant_Net, Marge) and DAX measures for dynamic aggregation (SUMX, DIVIDE) at report time. 🔄🔀

**Visualization Design**: Charts and visuals to communicate sales, margin, and customer insights. 🎨📊

**Dashboard Development**: Interactive, user-friendly dashboard for exploring the retail sales data. 📲🖥️

**Dynamic Filtering**: Slicers by Date, Region, Category, Payment Method, and Order Type. 🔍⚙️

**Insights**: Actionable recommendations on product, regional, and customer performance. 💡🔍

## Technologies Used 💻🔧

Microsoft Power BI · Microsoft Excel · Python (pandas) · Power Query · DAX

## Deliverables 📦🎁

The main deliverable is a Power BI Dashboard presenting sales, margin, and customer insights built on top of the star schema described above, including interactive visuals such as charts, graphs, and tables.

## Contributions 🤝🌟

Contributions are welcome — feel free to open an issue or submit a pull request with ideas, bug fixes, or feature suggestions.

Thanks for your interest in this project! 💖💻
