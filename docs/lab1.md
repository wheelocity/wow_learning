# Getting Started With DAX

In this exercise, you create some calculated columns and measures using Power BI to perform analysis of Sales.
The goal is to discover how to create calculated columns, measures and how to leverage them to solve a simple business scenario.

The data for this excercise is available in the following link:

[Download Data](files/lab1_data.xlsx){ .md-button }

## Lab 1 Activites



1. Open Power BI Desktop application.
2. Import data from excel sheets as tables.
3. Verify and and correct the data types of table fields as per the nature of data.
4. Load the tables.
5. Build relationship between tables.
6. Set the filter directions correctly in relationships.
7. Select `Enter Data` from the ribbon and give it a name called `_measures`
8. Add a **Calculated Column** to the `sales` table named `SalesAmount` with values as product of quantity and rate.
9. Create a **Measure** named `TotalSalesAmount` by summing up the `SalesAmount` column of `sales` table.
10. Delete the `SalesAmount` column from `sales` table.
11. Fix the error on the `TotalSalesAmount` measure. Can you fix this error without adding a column to `sales` table?
10. Create a **Measure** named `SalesSKUCount` that shows the distinct count of SKUs in `Sales` Table.
11. Create a **Measure** named `CatalogueSKUCount` that shows the distinct count of SKUs in `products` Table.
12. Add a **Calculated Column** named `TransactionCount` to product master
13. Add a **Calculated Column** named `SalesValue` to product master



