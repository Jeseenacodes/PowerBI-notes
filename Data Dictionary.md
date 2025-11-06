### Data Dictionary DAX code

This DAX expression automatically generates a **Data Dictionary table** inside Power BI, listing all tables, columns, measures, and relationships in the model along with their descriptions and expressions combining metadata from:

* **Columns** (`INFO.VIEW.COLUMNS()`)
* **Measures** (`INFO.VIEW.MEASURES()`)
* **Tables** (`INFO.VIEW.TABLES()`)
* **Relationships** (`INFO.VIEW.RELATIONSHIPS()`)

Each part retrieves names, descriptions, locations, and expressions, filters out hidden or internal elements, and merges them using `UNION()` into a single reference table.

---

## Purpose

Data dictionaries help document Power BI model so its easy to:

* Understand table structures and relationships
* Review column and measure logic
* Track updates or hidden fields
* Maintain cleaner, well-documented datasets

```DAX
// Data Dictionary - Power BI DAX

Data Dictionary =
VAR _columns =
    SELECTCOLUMNS(
        FILTER(
            INFO.VIEW.COLUMNS(),
            [Table] <> "Data Dictionary"
                && NOT([IsHidden])
        ),
        "Type", "Column",
        "Name", [Name],
        "Description", [Description],
        "Location", [Table],
        "Expression", [Expression]
    )

VAR _measures =
    SELECTCOLUMNS(
        FILTER(
            INFO.VIEW.MEASURES(),
            [Table] <> "Data Dictionary"
                && NOT([IsHidden])
        ),
        "Type", "Measure",
        "Name", [Name],
        "Description", [Description],
        "Location", [Table],
        "Expression", [Expression]
    )

VAR _tables =
    SELECTCOLUMNS(
        FILTER(
            INFO.VIEW.TABLES(),
            [Name] <> "Data Dictionary"
                && [Name] <> "Calculations"
                && NOT([IsHidden])
        ),
        "Type", "Table",
        "Name", [Name],
        "Description", [Description],
        "Location", BLANK(),
        "Expression", [Expression]
    )

VAR _relationships =
    SELECTCOLUMNS(
        INFO.VIEW.RELATIONSHIPS(),
        "Type", "Relationship",
        "Name", [Relationship],
        "Description", BLANK(),
        "Location", [FromTable],
        "Expression", [Relationship]
    )

RETURN
    UNION(_columns, _measures, _tables, _relationships)
```

---

## How to Use in Power BI

1. **Open Power BI Desktop**
   Go to **Model View** or **Data View**.

2. **Create a New Table**

   * On the ribbon, select **Modeling → New Table**.
   * Paste the full DAX expression above.

3. **Rename the Table**
   to `Data Dictionary` (or any preferred name).

4. **View the Results**
   You’ll now see a table listing:

   * All **tables**, **columns**, **measures**, and **relationships**
   * Their **descriptions**, **locations**, and **expressions**

5. **(Optional)** Include this as a support table or document page in the Power BI file for project transparency.

---

## Example Output

| Type         | Name             | Description          | Location | Expression                                 |
| ------------ | ---------------- | -------------------- | -------- | ------------------------------------------ |
| Table        | Sales            | Sales fact table     |          |                                            |
| Column       | Sales Amount     | Total sales revenue  | Sales    | `[Quantity] * [Unit Price]`                |
| Measure      | Total Revenue    | Calculates total rev | Sales    | `SUM(Sales[Sales Amount])`                 |
| Relationship | Sales → Products | Join via ProductKey  | Sales    | `Sales[ProductKey] = Products[ProductKey]` |

---

## Notes

* `INFO.VIEW.*()` functions are part of **DAX metadata views**, available in Power BI Desktop (2024+) or Power BI Service.
* Hidden or system objects are automatically excluded.
* You can extend this by adding more metadata columns or adjusting filters.

---

