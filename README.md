# Data-Miners

Graduate group coursework (MSDS 432) building an Azure SQL Server–hosted analytics warehouse for San Francisco housing policy data: open-data ingestion, dimensional modeling, and EDA across evictions, buyout agreements, address reference data, and zip-level demographics.

---

## 1. Title and Summary

**Data Miners (Azure ETL / Dimensional Modeling)**  
Northwestern University M.S. in Data Science (Data Engineering specialization): extract public datasets from SF Open Data and zip atlas CSVs, load raw tables into Azure SQL Database, profile and harmonize location keys, build Kimball-style dimensions and a fact table, and explore buyout/eviction patterns in Jupyter.

---

## 2. Concepts and Methods

- **Open-data ingestion (Socrata API):** `sodapy` client against `data.sfgov.org`; pull eviction notices (`5cei-gny5`, ~43k rows) and buyout agreements (`wmam-7g8d`, ~5.9k rows); strip computed columns; cast to string for staging (`Upload_Raw_Data.ipynb`)
- **Bulk load to Azure SQL Database:** SQLAlchemy + `pyodbc` (ODBC Driver 18); `to_sql` for smaller tables; chunked `insert_with_progress` for large eviction/address loads; row-count validation after load (`Upload_Raw_Data.ipynb`)
- **Reference / demographic sources:** zip atlas CSVs (median age, household income, population density) loaded to `Zip_Atlas_*_Raw` tables; address-with-units reference dataset (`SF_Addresses_With_Units_Raw`)
- **Raw-layer inventory:** `SF_Eviction_Notices_Raw`, `SF_Buyout_Agreements_Raw`, `SF_Addresses_With_Units_Raw`, zip atlas raw tables; relational staging tables `Rel_*` for transformed joins (`Location_EDA.ipynb`, table listings across notebooks)
- **Location harmonization EDA:** SQL reads from raw tables; compare address/zip/neighborhood/supervisor-district fields across evictions, buyouts, and address reference; schema introspection via `sys.tables` / `sys.columns` (`Location_EDA.ipynb`)
- **Zip atlas wrangling:** SQL pull of atlas tables; column normalization; outer merges on `Zip_Code`; population field reconciliation; skew/correlation/histogram profiling (`atlas-eda.ipynb`)
- **Dimensional model DDL:** `DIM_Eviction_Reason`, `DIM_Demographics`, `DIM_District`, `Date`, bridge `BR_Reason`, fact `FACT_SanFrancisco` with foreign keys to location/dimension keys (`Create_DIM_Tables.ipynb`)
- **Downstream EDA:** buyout amount distributions and boxplots (`EDA_Buyouts_Dataset.ipynb`); eviction-focused exploration (`Evictions_EDA.ipynb`); buyout ETL validation against warehouse tables (`Buyouts ETL.ipynb`)
- **Azure Synapse workspace artifacts:** linked services for Azure SQL Database, Azure Blob FS, and Azure SQL DW; integration runtime and factory metadata (`linkedService/`, `factory/`, `integrationRuntime/`)

**Credentials:** prefer `AZURE_MSDS432_USERNAME` / `AZURE_MSDS432_PASSWORD` env vars in most notebooks; some cells use inline connection strings

---

## 3. Stack

| Layer | Tools |
|-------|-------|
| Language | Python 3 |
| Environment | Jupyter Notebook |
| Cloud DB | Azure SQL Database (`mysqlserver-432.database.windows.net` / `mySampleDatabase`) |
| Connectivity | SQLAlchemy, pyodbc, pymssql, ODBC Driver 17/18 for SQL Server |
| Ingestion | sodapy (Socrata), pandas |
| Azure platform | Synapse linked services (SQL DB, ADLS Gen2, SQL DW), system-assigned managed identity metadata |
| Data domain | SF evictions, tenant buyout agreements, addresses, zip-level demographics |

---

## 4. Structure

```
Data-Miners/
├── EDA/
│   ├── Upload_Raw_Data.ipynb
│   ├── Location_EDA.ipynb
│   ├── atlas-eda.ipynb
│   ├── Create_DIM_Tables.ipynb
│   ├── CreateTables.ipynb
│   ├── Buyouts ETL.ipynb
│   ├── EDA_Buyouts_Dataset.ipynb
│   ├── Evictions_EDA.ipynb
│   ├── median-age.csv
│   ├── median-household-income.csv
│   ├── population-density.csv
│   └── Buyout_Agreements.csv
├── linkedService/          # Synapse linked service definitions
├── factory/                # Synapse workspace factory metadata
├── integrationRuntime/
├── credential/
└── README.md
```

- **Organization:** pipeline staged as ingest → location EDA → demographics merge → dimensional DDL → subject-matter EDA notebooks
- **Reusable modules:** `insert_with_progress` chunk loader in upload notebook; otherwise SQL/pandas inline
- **Engineering practice:** raw vs. relational vs. dimensional layers; progress-tracked bulk insert for high-row-count tables; cross-source address/zip reconciliation before star-schema load; open-data API limits and column hygiene (`@computed` drops)

---

**Course context:** Northwestern University, M.S. in Data Science, Data Engineering specialization (MSDS 432)  
**Repository:** https://github.com/EAName/Data-Miners  
**Upstream fork base:** [kaileen-silva-northwestern/Data-Miners](https://github.com/kaileen-silva-northwestern/Data-Miners)
