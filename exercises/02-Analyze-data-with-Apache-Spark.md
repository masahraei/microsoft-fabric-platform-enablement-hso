# Øvelse – Analyser data med Apache Spark i Microsoft Fabric

**Attribution:** This exercise is based on Microsoft Learning's mslearn-fabric labs and is used with attribution.

I denne øvelsen skal du:
- Laste inn data i en lakehouse
- Bruke PySpark til å analysere data
- Transformere og lagre data
- Bruke SQL og visualisering

⏱️ Estimert tid: 45 minutter

---

## 1. Opprett workspace

1. Gå til Microsoft Fabric og logg inn  
2. Velg **Workspaces**  
3. Opprett et nytt workspace med Fabric capacity (Trial/Premium)  
4. Åpne workspace
   
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/1-new-workspace.png)

---

## 2. Opprett Lakehouse og last opp data

1. Velg **New item → Lakehouse**  
2. Gi lakehouse et navn  
3. Deaktiver **Lakehouse schemas**
   
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/2-new-lakehouse.png)

### Last opp data
1. Last ned data ([orders.zip](https://github.com/masahraei/microsoft-fabric-platform-enablement-hso/blob/main/datasets/orders.zip))  
2. Pakk ut → du får:
   - 2019.csv  
   - 2020.csv  
   - 2021.csv  

3. I Lakehouse:
   - Gå til **Files**
   - Velg Upload → Upload folder
   - Last opp hele `orders`-mappen
     
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/3-uploaded-files.png)

---

## 3. Opprett Notebook

1. Velg **Create → Notebook**  
2. Gi den navn: `Sales analysis`
   
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/4-new-notebook.png)

### Legg til introduksjon (Markdown)

```markdown
# Sales order data exploration
Use this notebook to explore sales order data
```
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/5-name-notebook-markdown.png)

---

## 4. Last inn data (DataFrame)

Velg 2019.csv → Load data → Spark

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/6-explorer-notebook-view.png)

```python
df = spark.read.format("csv").option("header","true").load("Files/orders/2019.csv")
display(df.limit(100))
```
Kjør cellen ▶️

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/7-auto-generated-load.png)

---


## 5. Riktig håndtering av header
Endre koden:
```python
df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
```
Kjør på nytt.


---


## 6. Definer schema

```python
from pyspark.sql.types import *

orderSchema = StructType([
    StructField("SalesOrderNumber", StringType()),
    StructField("SalesOrderLineNumber", IntegerType()),
    StructField("OrderDate", DateType()),
    StructField("CustomerName", StringType()),
    StructField("Email", StringType()),
    StructField("Item", StringType()),
    StructField("Quantity", IntegerType()),
    StructField("UnitPrice", FloatType()),
    StructField("Tax", FloatType())
])

df = spark.read.format("csv").schema(orderSchema).load("Files/orders/2019.csv")

display(df)
```
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/8-define-schema.png)

---

## 7. Les alle filer

```python
df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")

display(df)
```

---

## 8. Utforsk data

Filtrer kolonner
```python
customers = df['CustomerName', 'Email']

print(customers.count())
print(customers.distinct().count())

display(customers.distinct())
```

Alternativ metode:
```python
customers = df.select("CustomerName", "Email")
```

Filtrer spesifikt produkt:
```python
customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')

print(customers.count())
print(customers.distinct().count())

display(customers.distinct())
```

---
## 9. Aggreger og grupper data

```python
productSales = df.select("Item", "Quantity").groupBy("Item").sum()

display(productSales)
```
```python
from pyspark.sql.functions import *

yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")

display(yearlySales)
```
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/9-spark-sql-dataframe.png)

---

## 10. Transformér data

Filtrer kolonner
```python
from pyspark.sql.functions import *

transformed_df = df.withColumn("Year", year(col("OrderDate"))) \
                   .withColumn("Month", month(col("OrderDate")))

transformed_df = transformed_df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)) \
                               .withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

display(transformed_df.limit(5))
```

---

## 11. Lagre data (Parquet)

Filtrer kolonner
```python
transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')

print("Data lagret!")
```

---


## 12. Les Parquet-data

Filtrer kolonner
```python
orders_df = spark.read.format("parquet").load("Files/transformed_data/orders")
display(orders_df)
```
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/10-parquet-files.png)

---


## 13. Partisjonering (bedre ytelse)

Filtrer kolonner
```python
orders_df.write.partitionBy("Year","Month") \
    .mode("overwrite") \
    .parquet("Files/partitioned_data")
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/11-partitioned-data.png)

Les data for 2021:
```python
orders_2021_df = spark.read.format("parquet") \
    .load("Files/partitioned_data/Year=2021/Month=*")

display(orders_2021_df)
```

---


## 14. Jobb med tabeller og SQL

Filtrer kolonner
```python
# Create a new table
df.write.format("delta").saveAsTable("salesorders")

# Get the table description
spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
```
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/12-salesorder-table.png)

---


## 15. Kjør SQL

Filtrer kolonner
```sql
%%sql

SELECT YEAR(OrderDate) AS OrderYear,
       SUM((UnitPrice * Quantity) + Tax) AS Revenue
FROM salesorders
GROUP BY YEAR(OrderDate)
ORDER BY OrderYear
```


---


## 16. Visualisering
```sql
%%sql
SELECT * FROM salesorders
```
Enkel graf i notebook

 - likk på resultat → New chart
  
 - Velg:
  
   - Bar chart
      
   - X: Item
      
   - Y: Quantity
     
![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-2/13-built-in-chart.png)

Visualisering med Python
 - Matplotlib
```python
from matplotlib import pyplot as plt

df_sales = spark.sql("""
SELECT YEAR(OrderDate) AS Year,
       SUM((UnitPrice * Quantity) + Tax) AS Revenue
FROM salesorders
GROUP BY YEAR(OrderDate)
""").toPandas()

plt.bar(df_sales['Year'], df_sales['Revenue'])
plt.title("Revenue per Year")
plt.show()
```

 - Seaborn
```python
import seaborn as sns

plt.clf()

ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)

plt.show()
```


---


## 17. Rydd opp

 - Stopp notebook session

 - Slett workspace hvis ferdig


---


## Oppsummering

I denne øvelsen har du:

 - Lest data med Spark

 - Brukt DataFrames

 - Transformert data

 - Lagret som Parquet

 - Brukt SQL

 - Visualisert data


---

> 📌 **Kreditering:**  
> Denne øvelsen er oversatt og tilpasset fra Microsoft Learn – DP-600:  
> https://learn.microsoft.com/en-us/training/modules/use-apache-spark-work-files-lakehouse/



