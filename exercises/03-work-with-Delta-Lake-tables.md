# Work with Delta Lake tables in Microsoft Fabric (Øvelse)

Tabeller i en Microsoft Fabric Lakehouse er basert på det åpne kildekodeformatet Delta Lake. Delta Lake gir støtte for relasjonelle egenskaper for både batch- og strømmedata. I denne øvelsen skal du opprette Delta-tabeller og utforske data ved hjelp av SQL-spørringer.

⏱️ **Estimert tid:** 45 minutter  
📌 **Forutsetning:** Du trenger tilgang til en Microsoft Fabric tenant

---

## Opprett et workspace

Gå til Microsoft Fabric:  
https://app.fabric.microsoft.com

1. Logg inn
2. Velg **Workspaces**
3. Klikk **New workspace**
4. Velg en kapasitet (Trial / Premium / Fabric)
5. Åpne workspace

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)

---

## Opprett en Lakehouse

1. Klikk **New item**
2. Velg **Lakehouse**
3. Gi den et navn
4. **Deaktiver "Lakehouse schemas (Public Preview)"**

⚠️ Viktig: Dette kan ikke endres senere

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/02-new-lakehouse.png)

---

## Last opp data

1. Last ned data:  
[products.csv](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)

2. Lage en mappe og gi den dette navnet "products"
3. Last opp filen **products.csv** til Lakehouse → Files → products

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/03-upload-products.png)

---

## Opprett Notebook

1. Klikk **Create → Notebook**

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/04-new-notebook.png)

2. Gi navn (f.eks. `DeltaLakeLab`)
3. Gjør første celle om til Markdown og legg inn:
```python
# Delta Lake tables
Use this notebook to explore Delta Lake functionality
```
4. Legg til lakehouse via OneLake catalog
5. Legg til en kodecelle:
```python
from pyspark.sql.types import StructType, IntegerType, StringType, DoubleType

schema = StructType() \
.add("ProductID", IntegerType(), True) \
.add("ProductName", StringType(), True) \
.add("Category", StringType(), True) \
.add("ListPrice", DoubleType(), True)

df = spark.read.format("csv").option("header","true").schema(schema).load("Files/products/products.csv")

display(df)
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/05-products-schema.png)

---

## Opprett Delta-tabeller

Managed tabell
```python
df.write.format("delta").saveAsTable("managed_products")
```

External tabell
1. Kopier ABFS-path
2. Bruk koden:
```python
df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
```
---

## Sammenlign tabeller

```sql
%%sql
DESCRIBE FORMATTED managed_products;

%%sql
DESCRIBE FORMATTED external_products;

%%sql
DROP TABLE managed_products;
DROP TABLE external_products;
```

---

## Opprett Delta-tabell med SQL

```sql
%%sql
CREATE TABLE products
USING DELTA
LOCATION 'Files/external_products';

%%sql
SELECT * FROM products;
```


---

## Oppdater data (SQL)

```sql
%%sql
UPDATE salesorders
SET Tax = Tax * 1.1
WHERE Quantity > 1;
```

---

## Versjonering av tabeller

```sql
%%sql
UPDATE products
SET ListPrice = ListPrice * 0.9
WHERE Category = 'Mountain Bikes';

%%sql
DESCRIBE HISTORY products;

delta_table_path = 'Files/external_products'
current_data = spark.read.format("delta").load(delta_table_path)
display(current_data)

original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
display(original_data)
```

---

## Analyser data med SQL

```sql
%%sql
CREATE OR REPLACE TEMPORARY VIEW products_view
AS
SELECT Category, COUNT(*) AS NumProducts, MIN(ListPrice) AS MinPrice, MAX(ListPrice) AS MaxPrice, AVG(ListPrice) AS AvgPrice
FROM products
GROUP BY Category;

SELECT * FROM products_view ORDER BY Category;
```

```sql
%%sql
SELECT Category, NumProducts
FROM products_view
ORDER BY NumProducts DESC
LIMIT 10;
```

Når dataene er returnert, velg + New chart for å vise et av de foreslåtte diagrammene.

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/06-sql-select.png)

```python
from pyspark.sql.functions import col

df_products = spark.sql("SELECT Category, MinPrice, MaxPrice, AvgPrice FROM products_view").orderBy(col("AvgPrice").desc())
display(df_products.limit(6))
```


---

## Streaming med Delta

```python
from notebookutils import mssparkutils
from pyspark.sql.types import *

inputPath = 'Files/data/'
mssparkutils.fs.mkdirs(inputPath)

jsonSchema = StructType([
StructField("device", StringType(), False),
StructField("status", StringType(), False)
])

iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)
```

```python
device_data = '''{"device":"Dev1","status":"ok"}
{"device":"Dev2","status":"error"}'''

mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
```

```python
delta_stream_table_path = 'Tables/iotdevicedata'
checkpointpath = 'Files/delta/checkpoint'

deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
```

```python
delta_stream_table_path = 'Tables/iotdevicedata'
checkpointpath = 'Files/delta/checkpoint'

deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
```

```sql
%%sql
SELECT * FROM IotDeviceData;
```

```python
# Add more data to the source stream
more_data = '''{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"error"}
{"device":"Dev2","status":"error"}
{"device":"Dev1","status":"ok"}'''

mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
```

```sql
%%sql
SELECT * FROM IotDeviceData;
```

```python
deltastream.stop()
```

---

## Rydd opp ressurser

I denne øvelsen har du lært:

- Gå til workspace
- Åpne innstillinger
- Velg Remove this workspace


---
