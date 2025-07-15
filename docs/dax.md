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