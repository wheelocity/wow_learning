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
- Use **Measures** when you want to compute percentages, ratios, complex aggregations and want the results to be dynamic based on the slicers and filters.

## Handling Errors

- An expression like `Sales[Amount] / Sales[Quantity]` fail if
    - `Sales[Quantity]` is zero
    - `Sales[Quantity]` is a string and cannot be converted to number.
- Cause of errors
    - Conversion errors
    - Arithmetical operations
    - Empty or missing values

## ISERROR(Expression)

The `ISERROR()` function takes any expression and evaluates it and returns `TRUE` or `FALSE` depending on the presence of an error during evaluation.

```dax title="ISERROR Example"
ISERROR (
    Sales[GrossMargin] / Sales[Amount]
)
```

## IFERROR(Expression,Alternative)
In case of error returns *Alternative*. Useful to avoid writing expression twice.

```dax title="IF & ISERROR Wrong example"
IF(
    ISERROR (
        Sales[GrossMargin] / Sales[Amount]
    ),
    0,
    Sales[GrossMargin] / Sales[Amount]
)
```
This can be written better as
```dax title="IFERROR example"
IFERROR (
    Sales[GrossMargin] / Sales[Amount],
    0
)
```

## Aggregate Functions

- Useful to aggregate values
    - `SUM(Table[Column])`
    - `AVERAGE(Table[Column])`
    - `MIN(Table[Column])`
    - `MAX(Table[Column])`
- Work only on numeric columns
- Aggregate only one column
    - `SUM(Sales[Amount])`
    - <code style="color:red">SUM(Sales[Quantity] * Sales[Rate])</code> This won't work.

## The X Aggregation Functions

- Iterators: useful to aggregate formulas
    - SUMX
    - AVERAGEX
    - MINX
    - MAXX
- Iterate over the table and evaluate the expression for each row
- Always receive two parameters
    - Table to iterate
    - Formula to evaluate for each row

```dax title="SUMX Example"
SalesAmount = SUMX(
    Sales,
    Sales[Price] * Sales[Quantity]
)
```
Other functions like `AVERAGEX`, `MINX` and `MAXX` work the same way.

## Counting Values

- Useful to count values
    - `COUNT` (only numeric columns)
    - `COUNTA` (counts anything except blanks)
    - `COUNTBLANK` (counts blanks)
    - `COUNTROWS` (counts rows in a table)
    - `DISTINCTCOUNT` (performs distinct count)
- The X aggregation versions.
    - `COUNTX`
    - `COUNTAX`
    - Works similar to other X aggregators like `SUMX`, `AVERAGEX` etc...


```dax title="COUNTROWS Example"
COUNTROWS(Sales) = 
    COUNTA(Sales[SaleID]) + 
    COUNTBLANK(Sales[SaleID])
```

## Logical Functions

Provides boolean logic.

- `AND` or `&&`
- `OR` or `||`
- `NOT`
- `IF`
- `IFERROR`

```dax title="Logical Functions Example"
Yield % = 
    IF(
        AND(
            Sales[Category] = "Soft",
            Sales[PitstopGroup] = "NanoPod"
        ),
        0.90,
        0
    )
```

OR

```dax title="Logical Functions Example"
Yield % = 
    IF(
        Sales[Category] = "Soft" &&
        Sales[PitstopGroup] = "NanoPod",
        0.90,
        0
    )
```

## The SWITCH Function

Makes it easier to perform nested IF Internally, it is converted into a set of nested IF.

```dax title="Nested IF Example"
Discount =
IF (
    Sales[Category] = "M",
    IF (
        Sales[Quantity] > 15,
        0.10,
        IF ( Sales[Quantity] > 10, 0.08, IF ( Sales[Quantity] > 5, 0.06, 0 ) )
    ),
    IF (
        Sales[Category] = "B",
        IF (
            Sales[Quantity] > 15,
            0.15,
            IF ( Sales[Quantity] > 10, 0.10, IF ( Sales[Quantity] > 5, 0.05, 0 ) )
        )
    )
)

```
This nested IF statement can be written using SWITCH as
```dax title="SWITCH Example"
Discount = 
    SWITCH(
        TRUE(),
        Sales[Category] = "M" && Sales[Quantity] > 15, 0.10,
        Sales[Category] = "M" && Sales[Quantity] > 10, 0.08,
        Sales[Category] = "M" && Sales[Quantity] > 5, 0.06,
        Sales[Category] = "B" && Sales[Quantity] > 15, 0.15,
        Sales[Category] = "B" && Sales[Quantity] > 10, 0.10,
        Sales[Category] = "B" && Sales[Quantity] > 5, 0.05,
        0
    )
```
Another example:
```dax title="SWITCH Another Example"
TerminalGroup = 
    SWITCH(
        LEFT(Sales[TerminalID],4),
        "PBTN", "WoW",
        "PBLD", "Training",
        "LTTN", "Liquidation",
        "Testing"
    )
```

## Information Functions

Provide information about expressions or values.

- ISBLANK
- ISNUMBER
- ISTEXT
- ISNONTEXT
- ISERROR

## MAX and MIN


```dax title="MAX & MIN"
MAX(Sales[Amount])
// OR
MAXX(Sales,Sales[Amount])
// Computes the maximum value in the Amount column of Sales Table

MAX(Sales[Amount],Sales[ListPrice])
//OR
IF(Sales[Amount]>Sales[ListPrice],Sales[Amount],Sales[ListPrice])
// Computes maximum of these two values 
// in each row when used in other expressions.
```

## Mathematical Functions

- `ABS`
- `EXP`
- `FACT`
- `LN`
- `LOG`
- `LOG10`
- `MOD`
- `PI`
- `POWER`
- `QUOTIENT`
- `SIGN`
- `SQRT`

## The DIVIDE Function

Divide is useful to avoid using `IF` inside an expression to check for zero denominators

```dax title="Error Handling using IF"
IF(
    Sales[Quantity] <> 0,
    Sales[Amount] / Sales[Quantity],
    0
)
```

You can write it better with `DIVIDE`

```dax title="DIVIDE example"
DIVIDE(Sales[Amount],Sales[Quantity],0)
```
This returns `0` if the division fails.

## Using Variables

Very useful to avoid repeating subexpressions in your code.

```dax title="Repeating Sub Expressions"
IF(
    SUM(Sales[Quantity]) > 1000,
    SUM(Sales[Quantity]) * 0.95,
    SUM(Sales[Quantity]) * 1.25
)
```

This is inefficient because the SUM formula will have to compute the sum of Quantity field from Sales table three times. We can write it better using variables.

```dax title="Repeating Sub Expressions"
VAR TotalQuantity = SUM(Sales[Quantity])
IF(
    TotalQuantity > 1000,
    TotalQuantity * 0.95,
    TotalQuantity * 1.25
)
```
Here it is evaluated only once. Better and faster.

## Rounding Functions

Many different rounding functions:

- `FLOOR ( Value, 0.01 )`
- `TRUNC ( Value, 2 )`
- `ROUNDDOWN ( Value, 2 )`
- `MROUND ( Value, 0.01 )`
- `ROUND ( Value, 2 )`
- `CEILING ( Value, 0.01 )`
- `ISO.CEILING ( Value, 0.01 )`
- `ROUNDUP ( Value, 2 )`
- `INT ( Value )`

## Text Functions

Let’s assume we have the following sample values:
```dax title="Sample Values"
Customer[FirstName] = "John"
Customer[LastName] = "Doe"
Customer[Code] = "ABC123"
Customer[Amount] = 12345.678
```

```dax title="Sample Values"
= CONCATENATE(Customer[FirstName], " ", Customer[LastName]) 
// Result: "John Doe"

= CONCATENATEX(VALUES(Customer[City]), Customer[City], ", ")
// Result: "Mumbai, Delhi, Chennai"

= FIND("C", Customer[Code], 1)   
// Result: 3 (position of "C" in "ABC123")

= SEARCH("c", Customer[Code], 1)
// Result: 3 (case-insensitive unlike FIND)

= LEFT(Customer[Code], 3)   
// Result: "ABC"

= RIGHT(Customer[Code], 3)  
// Result: "123"

= MID(Customer[Code], 2, 3)
// Result: "BC1"

= LEN(Customer[Code])
// Result: 6

= LOWER(Customer[Code])   
// Result: "abc123"

= UPPER(Customer[FirstName])  
// Result: "JOHN"

= TRIM("  John  ")  
// Result: "John" (removes extra spaces)

= EXACT("John", "john")
// Result: FALSE (case-sensitive)

= REPLACE(Customer[Code], 4, 3, "999")
// Result: "ABC999"

= SUBSTITUTE("red,red,blue", "red", "green")
// Result: "green,green,blue"

= FIXED(Customer[Amount], 2, TRUE)
// Result: "12345.68"

= FORMAT(Customer[Amount], "#,##0.00")
// Result: "12,345.68"

= REPT("*", 5)  
// Result: "*****"

= VALUE("123.45")  
// Result: 123.45 as number

```

## Date Functions

```dax title="Date Functions"
= DATE(2023, 12, 25)   
// Returns: 25-Dec-2023

= DATEVALUE("2023-12-25")
// Returns: 25-Dec-2023

= TIME(14, 30, 0)
// Returns: 2:30:00 PM

= TIMEVALUE("14:30:00")
// Returns: 2:30:00 PM

= HOUR(Sales[OrderDate])
// Extracts the hour part (0–23)

= MINUTE(Sales[OrderDate])
// Extracts the minute part (0–59)

= SECOND(Sales[OrderDate])
// Extracts the second part (0–59)

= DAY(Sales[OrderDate])
// Extracts day of the month (1–31)

= MONTH(Sales[OrderDate])
// Extracts month (1–12)

= YEAR(Sales[OrderDate])
// Extracts year (e.g., 2023)

= NOW()
// Returns current date and time

= TODAY()
// Returns current date (without time)

= EDATE(Sales[OrderDate], 2)
// Date after 2 months

= EOMONTH(Sales[OrderDate], 0)
// End of the current month

= WEEKDAY(Sales[OrderDate], 2)
// Returns day of week (1=Monday to 7=Sunday)

= WEEKNUM(Sales[OrderDate], 2)
// Returns week number of the year

= YEARFRAC(DATE(2023,1,1), Sales[OrderDate])
// Returns fraction of the year completed from Jan 1, 2023 to OrderDate
```

## Relational Functions

### `RELATED`
Follows relationships and returns the value of a column.

```dax title="Date Functions"
bf_ops_user_roster['RosterStatus'] = RELATED(bf_ops_roster['Status'])
```
If there is a relationship between user roster and roster, this fetches the roster status from `bf_ops_roster` into `bf_ops_user_roster`. This function is written in the table on many-side of the relationship to get the data from one-side. That is for each row, there can be only one row in the relation target.

### `RELATED_TABLE`
Follows relationships and returns the rows that are related to the current row.

```dax title="Date Functions"
SalesHeader['SKU Count'] = COUNTROWS(RELATEDTABLE(SalesItems))
```

For each sales header, there will be multiple rows (one row for each SKU in the invoice) on the sales items table. The related table used on the header table returns all the rows from sales items table that matches the relationship keys.

!!! note "Brief note on function types"

    In DAX, there are two kinds of functions. 

    - **Scalar Functions**: Functions that return a single value. Like `SUMX`, `MAX`, `RELATED` etc...
    - **Table Functions**: Functions that return a table. Like `ALL`, `FILTER`, `RELATEDTABLE` etc...



