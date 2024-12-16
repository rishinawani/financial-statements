# financial-statements
## Project Overview
For this project, we created financial reports leveraging data from a data warehouse hosted on SQL Server and creatind visualisations and data model in powerbi. Below is an overview of the process:
## 1. Data Preparation:
Created a view containing all relevant columns required for the reports by joining multiple tables.
This view was ingested into Power BI for further processing.
## 2.Column Selection
Created a view containing all relevant columns required for the reports by joining multiple tables.
This view was ingested into Power BI for further processing.
## 3. Dimensions Tables:
![model Logo](https://github.com/rishinawani/financial-statements/blob/main/model.PNG)

The view was a denormalized table and was transformed into a star schema with four dimension tables:
Dim_Account
Dim_Region
Dim_StoreName
Removed duplicates and unnecessary columns by referencing the main table.
## 4. Custom Tables
Created a customer category table to correctly categorize income statement data.

Added a headers table for flexibility in adding or removing line items for the income statement.
## Best Practices Followed
Always create a date dimension table.

Build one-to-many relationships between fact and dimension tables.

Aim for a star schema for optimized performance.

Hide the "many" side of relationships in the modeling tab of Power BI.

Use dates from the date dimension table rather than from fact tables for consistency.
## Dax Measures
### 1. Headers Column:
Added a column for "Headers" to structure the income statement visualization.
### 2. Key Measures
Key Measures:

### Total Amount:

SumAmount = SUM(Fact_GLTrans[GLTranAmount])

Used to calculate the total amount in the income statement by excluding the balance sheet category.

### Income Statement Amount:

I/S Amount = CALCULATE(ABS([SumAmount]), Dim_Headers[Statement] = "Income Statement")

Filters the context to include only rows where the statement is "Income Statement."

### Running Subtotal:

I/S Subtotal = CALCULATE([I/S Amount], FILTER(ALL(Dim_Headers), Dim_Headers[Sort] < MAX(Dim_Headers[Sort])))

Creates a running total for subtotal rows where specific amounts were not populated.

Income Statement Measure:

Income Statement =
SWITCH(
    TRUE(),
    SELECTEDVALUE(Dim_Headers[MeasureName]) = "Subtotal", [I/S Subtotal],
    [I/S Amount]
)

Dynamically switches between subtotal and specific amounts based on selected headers.
![Netflix Logo](https://github.com/rishinawani/financial-statements/blob/main/explain.PNG)

### Percentage of Revenue:

% of Revenue =
VAR Revenue = CALCULATE([I/S Amount], FILTER(ALL(Dim_Headers), Dim_Headers[Category] = "Revenue"))
RETURN DIVIDE([I/S Subtotal], Revenue, 0)

Calculates the percentage of each subtotal relative to revenue.
## Financial Logic 
Income Statement =
VAR Display_Filter = NOT ISFILTERED(DIM_GL_Accnt[Subcategory])
RETURN
SWITCH(
    TRUE(),
    SELECTEDVALUE(Dim_Headers[MeasureName]) = "Subtotal" && Display_Filter, [I/S Subtotal],
    SELECTEDVALUE(Dim_Headers[MeasureName]) = "% of Revenue" && Display_Filter, [% of Revenue],
    [I/S Amount]
)
## Additional Measures
### Gross Profit Margin:
Gross Profit Margin =
DIVIDE(
    CALCULATE([Income Statement], FILTER(Dim_Headers, Dim_Headers[Category] = "Gross Profit")),
    CALCULATE([Income Statement], FILTER(Dim_Headers, Dim_Headers[Category] = "Revenue")),
    0
)
### operating  Margin:
Operating Margin Ratio =
VAR Operating_Margin = CALCULATE([I/S Subtotal], Dim_Headers[Category] = "EBIT")
VAR Revenue = CALCULATE([I/S Amount], Dim_Headers[Category] = "Revenue")
RETURN DIVIDE(Operating_Margin, Revenue, 0)

![ Logo](https://github.com/rishinawani/financial-statements/blob/main/refresh%20dae.PNG)

