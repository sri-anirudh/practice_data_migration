
# F1 Data Engineering Pipeline (Practice Project)

This project demonstrates a complete data engineering pipeline using:
- **GitHub** (for raw JSON files)
- **MS SQL Server** (for tabular CSV files)
- **Azure Blob Storage**
- **Azure Databricks** (for transformations)
- **Snowflake** (for storing final tables)
- **Azure Data Factory** (for orchestration)

---

## 📁 Step 1: Split the Source Data

### GitHub (Push these files)
- `drivers.json`
- `results.json`
- `pit_stops.json`

### MS SQL Server (Insert these files into tables)
- `circuits.csv`
- `races.csv`

---

## ☁️ Step 2: Ingest into Azure Blob Storage

### A. From GitHub
Use Databricks or Python scripts to download and store in Blob:

```python
import requests

url = 'https://raw.githubusercontent.com/<your-repo>/drivers.json'
r = requests.get(url)
with open('/dbfs/mnt/blob/drivers.json', 'wb') as f:
    f.write(r.content)
```

### B. From MS SQL
Use Azure Data Factory to copy `circuits` and `races` tables into Blob Storage.

---

## 🔗 Step 3: Mount Azure Blob Storage in Databricks

```python
dbutils.fs.mount(
  source = "wasbs://<container>@<storage-account>.blob.core.windows.net/",
  mount_point = "/mnt/blob",
  extra_configs = {"fs.azure.account.key.<storage-account>.blob.core.windows.net": "<your-key>"}
)
```

---

## 🧠 Step 4: Transform Data in Databricks

Example using PySpark:

```python
drivers_df = spark.read.json("/mnt/blob/drivers.json")
races_df = spark.read.csv("/mnt/blob/races.csv", header=True)
# Merge, transform, and enrich as needed
```

Create fact and dimension tables:
- `dim_drivers`
- `dim_circuits`
- `dim_races`
- `fact_results`
- `fact_pitstops`

---

## ❄️ Step 5: Load into Snowflake

### Configure Snowflake connection:

```python
sfOptions = {
  "sfURL": "<account>.snowflakecomputing.com",
  "sfDatabase": "<database>",
  "sfSchema": "<schema>",
  "sfWarehouse": "<warehouse>",
  "sfRole": "<role>",
  "sfUser": "<user>",
  "sfPassword": "<password>"
}
```

### Write to Snowflake:

```python
final_df.write.format("snowflake") \
    .options(**sfOptions) \
    .option("dbtable", "fact_results") \
    .mode("overwrite") \
    .save()
```

---

## 🔁 Step 6: Use ADF to Connect to Snowflake (Optional)

- Set up Snowflake as a Linked Service in ADF.
- Use Copy Activity to bring Snowflake data into other sinks or visualization layers.

---

## ✅ Final Output Tables

- `dim_drivers`
- `dim_circuits`
- `dim_races`
- `fact_results`
- `fact_pitstops`
