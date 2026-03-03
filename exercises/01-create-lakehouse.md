# Opprett et Microsoft Fabric Lakehouse

*Lab referert fra [Microsoft Learn DP-600: Create a Microsoft Fabric Lakehouse](https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/01-lakehouse.html)*

---

## Introduksjon

Storskala dataanalyse-løsninger har tradisjonelt blitt bygget rundt et datalager, der data lagres i relasjonstabeller og kan forespørres med SQL. Med veksten i “big data” – karakterisert ved høye volum, variasjon og hastighet – samt tilgjengeligheten av lavkost lagring og skybasert distribuert prosessering, har det oppstått en alternativ tilnærming: **data lake**.  

I en data lake lagres data som filer uten et fast definert skjema. Mange dataingeniører og analytikere ønsker nå å kombinere fordelene fra både datalager og data lake, noe som resulterer i en **data lakehouse**: Data lagres i filer i en data lake, men et relasjonelt skjema legges til som metadata, slik at dataene kan forespørres med SQL.

I Microsoft Fabric tilbyr et lakehouse svært skalerbar fil-lagring i OneLake (basert på Azure Data Lake Store Gen2) med en **metastore** for relasjonelle objekter som tabeller og visninger, basert på Delta Lake-formatet. Delta Lake lar deg definere et skjema for tabellene som kan forespørres med SQL.

> Estimert tid for å fullføre: 30 minutter  
> Merk: Du trenger en Microsoft Fabric-prøve for å gjennomføre øvelsen.

---

## Opprett et arbeidsområde

Før du kan jobbe med data i Fabric, må du opprette et arbeidsområde med Fabric trial aktivert.

1. Gå til [Microsoft Fabric-hjem](https://app.fabric.microsoft.com/home?experience=fabric) og logg inn med dine Fabric-legitimasjon.
2. Velg **Workspaces** i venstremenyen 🗇.
3. Opprett et nytt arbeidsområde med valgfritt navn og velg en lisensieringsmodus som inkluderer Fabric-kapasitet (Trial, Premium eller Fabric).
4. Når arbeidsområdet åpnes, skal det være tomt.

![New Workspace - Microsoft Learning](https://raw.githubusercontent.com/masahraei/microsoft-fabric-platform-enablement-hso/main/images/new-workspace.png)

*Source: Microsoft Learning (mslearn-fabric lab materials)*

---

## Opprett et lakehouse

1. Velg **Create** i venstremenyen, og under **Data Engineering** velg **Lakehouse**.
2. Gi lakehouset et unikt navn.
3. Sørg for at “Lakehouse schemas (Public Preview)” er deaktivert.
4. Etter ett minutt vil lakehouset bli opprettet.

![Plassholder for skjermbilde av nytt lakehouse](/assets/new-lakehouse.png)

**Utforsk Lakehouse:**
- **Tables**: Tabeller som kan forespørres med SQL, basert på Delta Lake-format.
- **Files**: Datafiler i OneLake, inkludert mulighet for snarveier til ekstern lagring.
- For øyeblikket finnes det ingen tabeller eller filer.

---

## Last opp en fil

1. Last ned `sales.csv` fra:  
   `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`
2. I Lakehouse Explorer, opprett en undermappe `data` under **Files**.
3. Last opp `sales.csv` til `Files/data`.

![Plassholder for skjermbilde av upload](/assets/upload-sales.png)

4. Velg filen for å forhåndsvise innholdet.

---

## Utforsk snarveier

- Velg **New shortcut** fra …-menyen under **Files**.
- Snarveier lar deg referere til eksterne datakilder uten å kopiere dem til lakehouset.

---

## Last fildata inn i en tabell

1. Gå til `Files/data` og velg `sales.csv`.
2. Velg **Load to Tables > New table**.
3. Sett tabellnavn til `sales` og bekreft.

![Plassholder for tabellvisning](/assets/sales-table.png)

4. Forhåndsvis tabellen og se de underliggende filene (`_delta_log`).

---

## Bruk SQL til å forespørre tabeller

1. Bytt fra Lakehouse til **SQL analytics endpoint**.
2. Opprett en ny SQL-forespørsel og kjør:

```sql
SELECT Item, SUM(Quantity * UnitPrice) AS Revenue
FROM sales
GROUP BY Item
ORDER BY Revenue DESC;

```

### Kjør SQL-forespørselen

Klikk på ▷ **Run**-knappen for å kjøre spørringen.

Resultatet skal vise total omsetning (Revenue) per produkt, sortert synkende etter høyest omsetning.

![Plassholder for skjermbilde av SQL-spørring med resultater](/assets/sql-query-results.png)

---

## Opprett en visuell forespørsel

Selv om mange datafagfolk er komfortable med SQL, kan analytikere med erfaring fra Power BI bruke Power Query til å lage visuelle forespørsler.

### 1. Opprett ny visuell forespørsel

1. På verktøylinjen, utvid **New SQL query**.
2. Velg **New visual query**.

Dra tabellen `sales` inn i det nye visuelle redigeringspanelet som åpnes.

![Plassholder for skjermbilde av Visual Query-editor](/assets/visual-query-editor.png)

---

### 2. Velg kolonner

1. I menyen **Manage columns**, velg **Choose columns**.
2. Velg kun følgende kolonner:
   - `SalesOrderNumber`
   - `SalesOrderLineNumber`

![Plassholder for skjermbilde av Choose columns-dialog](/assets/choose-columns.png)

---

### 3. Gruppér data

1. Gå til menyen **Transform**.
2. Velg **Group by**.
3. Bruk følgende innstillinger:

- **Group by:** `SalesOrderNumber`
- **New column name:** `LineItems`
- **Operation:** Count distinct values
- **Column:** `SalesOrderLineNumber`

Når du er ferdig, vil resultatpanelet vise antall linjeelementer per salgsordre.

![Plassholder for skjermbilde av Visual Query med resultater](/assets/visual-query-results.png)

---

## Rydd opp ressurser

I denne øvelsen har du:

- Opprettet et lakehouse
- Lastet opp data
- Opprettet en tabell
- Kjørt SQL-spørringer
- Brukt visuell spørring
- Utforsket hvordan lakehouse lagrer data i OneLake

Hvis du er ferdig med øvelsen, kan du slette arbeidsområdet:

1. Velg arbeidsområde-ikonet i venstremenyen.
2. Velg **Workspace settings** i verktøylinjen.
3. Under **General**, velg **Remove this workspace**.

Dette sikrer at testressurser ikke bruker kapasitet unødvendig.

---

> 📌 **Kreditering:**  
> Denne øvelsen er oversatt og tilpasset fra Microsoft Learn – DP-600:  
> https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/01-lakehouse.html
