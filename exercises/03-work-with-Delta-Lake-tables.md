# Work with Delta Lake tables in Microsoft Fabric (Øvelse)

I denne øvelsen vil du lære hvordan du jobber med Delta Lake-tabeller i Microsoft Fabric ved hjelp av Spark.

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
https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip

2. Pakk ut filen
3. Last opp mappen **orders** til Lakehouse → Files

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/03-upload-products.png)

---

## Opprett Notebook

1. Klikk **Create → Notebook**
2. Gi navn (f.eks. `DeltaLakeLab`)

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/04-new-notebook.png)

---

## Les data med Spark

```python
df = spark.read.format("csv").option("header","true").load("Files/orders/*.csv")
display(df)
```

---

## Lag Delta-tabell

```python
df.write.format("delta").saveAsTable("salesorders")
```

---

## Les fra Delta-tabell

```python
df = spark.sql("SELECT * FROM salesorders LIMIT 1000")
display(df)
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)

---

## Oppdater data (SQL)

```sql
%%sql
UPDATE salesorders
SET Tax = Tax * 1.1
WHERE Quantity > 1;
```

---

## Slett data

```sql
%%sql
DELETE FROM salesorders
WHERE Quantity = 0;
```

---

## Time Travel (historikk)

```sql
%%sql
DESCRIBE HISTORY salesorders;
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)

---

## Les tidligere versjon

```python
df_old = spark.read.format("delta").option("versionAsOf", 0).table("salesorders")
display(df_old)
```


---

## Optimaliser tabell

Gå til:

- Lakehouse Explorer
- Klikk på ... ved tabellen
- Velg Maintenance → Run OPTIMIZE

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)


---

## VACUUM (rydde gamle filer)

```sql
%%sql
VACUUM salesorders RETAIN 168 HOURS;
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)


---

## Partisjonering

```python
df.write.format("delta").partitionBy("Item").saveAsTable("salesorders_partitioned")
```

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/workshop-3/01-new-workspace.png)


---

## Oppsummering

I denne øvelsen har du lært:

- Hva Delta Lake er
- Hvordan lage Delta-tabeller
- Hvordan oppdatere og slette data
- Hvordan bruke time travel
- Hvordan optimalisere data
- Hvordan partisjonering fungerer


---
