# The DAX Language

DAX is the language of Power Pivot, Power BI, SSAS Tabular and it is simple to use, but takes effor to understand. The DAX formula resembles Excel formula because it was devleoped along with Power Pivot for Excel. But there are some important difference compared to Excel formulas; which are there is no concept of cells, rows or columns. There are many new functions and frequently new functions are added to this language.

DAX is designed for data models and business use cases.

## Functional Language

DAX is a functional language and the execution of any expression flows with function calls. Given belows is an example of a DAX expression:

```dax title="DAX Expression Example"
FMCG Sales Amount =
    SUMX (
        FILTER (
            Sales,
            Sales[Category] = "FMCG"
        ),
        Sales[Quantity] * Sales[Price]
    )
```

This expression first filters the `Sales` table where the `Category` field has value `FMCG`. The result of the `FILTER` function is a filtered table which is then passed to `SUMX` and the `SUMX` function then goes through each row in the filtered table and keeps totalling the result of `Quantity * Price` and finally returns the result. This result is stored in a measure named `FMCG Sales Amount`

This is just an example to demonstrate how DAX, as a functional language, uses pre-built functions to compute results.

## Data Types in DAX

DAX has a very minimal set of data types.

=== "Numeric Data Types"

    - Integer
    - Decimal
    - Currency
    - TRUE / FALSE (Boolean)
=== "Date & Time"
    - Date
    - DateTime

    Date and DateTime are stored as decimal numbers. Every Date or DateTime value has an integer part and a decimal part. The integer part is a number representing the number of days elapsed since 30th December 1899. The time part is stored as a fraction computed as follows:

    - 1 hour = $\frac{1}{24}$
    - 1 minute = $\frac{1}{24 \times 60} = \frac{1}{1440}$
    - 1 second = $\frac{1}{24 \times 60 \times 60} = \frac{1}{86400}$  
    - 1 millisecond = $\frac{1}{24 \times 60 \times 60 \times 1000} = \frac{1}{86400000}$  

    So any $Date + 1$ will be next day. Any $Date - 1$ will be previous day. Adding $\frac{5}{24} + \frac{30}{1440}$ will add $5.5 \: hours$ to the date and so on. This will be very useful in computations dealing with dates and datetimes.

=== "Other Data Types"

    - String
    - Binary Objects

## DAX Syntax

When we write DAX code, we will be using table names and column names to perform various computations on them. There is a syntax or format to be followed:

- The general format is to use `'Table Name'[Column]`.
- If the table name does not have multiple words, we can skip the single quotes and use `Table[Column]`.
- If we are writing a DAX formula on the current table, we can skip the table name and use the column name only. **DO NOT DO IT** though. Reading the code becomes difficult later on.
- The square brackets around the column names should always be used.

## Calculated Columns

- These are **columns** added to the tables using DAX
- The values in the column will be computed row by row.
- The column **`Product[Price]`** means:
    - The value of the `Price` column
    - In the `Product` table.
    - For the current row.
    - Different for each row.

## Calculated Columns and Measures

- Suppose our `Sales` table have 1000 rows.
- In each row, there is a `Sales[SalesAmount]` column and `Sales[ProductCost]` column.
- Suppose we want to find the `Gross Margin %` of all sales (1000 rows)
- We can add a **Calculated Column** in the `Sales` table to compute the `Gross Margin` for each row as `Gross Margin = Sales[SalesAmount] - Sales[ProductCost]`. This can be computed for each row. **THIS IS OKAY**.
- But if would be **WRONG** to add a calculated column as `Gross Margin % = Sales[Gross Margin] / Sales[SalesAmount]` and then sum it up. It won't give the `Gross Margin %` of total sales obviously.
- In this case, we need measures.
    - `TotalSalesAmount = SUMX(Sales,Sales[SalesAmount])`
    - `TotalGrossMargin = SUMX(Sales,Sales[GrossMargin])`
    - `Gross Margin % = [TotalGrossMargin] / [TotalSalesAmount]`
Now the `Gross Margin %` will be correct. 

The reason is $\sum_{rows=1}^{1000}  \frac{Margin}{Sales} \ne \frac{\sum Margin}{\sum Sales}$

## Measures

- Written using DAX
- Do not work row by row
- Instead, use tables and aggregators
- Do not have the «current row» concept
- Examples
    - `GrossMargin` is a **calculated column** but can be a **measure** too
    - `GrossMargin %` needs to be a **measure**

## Calculated Columns Vs Measures

- **Calculated Columns** belongs to a **TABLE**. So we always use table name along with the column name as `TableName[ColumnName]`
- **Measures** DO NOT belong to a **TABLE**. We can move a measure from one table to another. It will be listed under a table only for organization or grouping purpose. When we refer to a measure, avoid using the table name and just refer to it using the measure name in square brackets as `[MeasureName]`
- **Calculated Columns** are computed at the time of report refresh or data load. It occupies storage and once it is computed at report refresh time, it does not respond to what we select in filters or slicers. The values don't change.
- **Measures** are not stored in the tables and does not occupy any space. It consumes CPU power. When we use measures in our visualizations, depending on the slicers or filters we choose, the values of measures change dynamically based on the **FILTER CONTEXT** and **EVALUATION CONTEXT**.
