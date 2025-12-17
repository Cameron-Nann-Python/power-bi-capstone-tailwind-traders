# Power BI Capstone Project: Tailwind Traders

## Overview
This capstone project is the final project of the Microsoft Power BI Professional Data Analyst Certificate Program. I take on the role of a junior data analyst where Tailwind Traders is fictious manufacturing company that needs a report on its latest sales data. The report must compare the gross and net sales. A proper report completion will lead the company to drive insights that boost prospective sales.

## Step 1: Preparing Excel Data
The company sales data is stored in Excel file called `Tailwind-Traders-Sales.xlsx`. The file initially does not contain data on gross sales, so a calculated column named "Gross Revenue" can be added using **=E2*G2**.

<img width="1325" height="422" alt="add-gross-revenue-column" src="https://github.com/user-attachments/assets/13d0e05d-cd1b-49df-9ee9-dc14ac13e8fc" />
<br></br>
To calculate net revenue, a tax column needs to be added. This is done by creating calculated column named **Total Tax** by multiplying the **Tax per Product** column values with the "Quantity Purchased" column values. The net revenue can be found in a calculated column named **Net Revenue** by subtracting the **Total Tax column** column values from the "Gross Revenue column values. These are the primary values needed to complete the report.
<br></br>
<img width="1636" height="620" alt="add-net-revenue-column" src="https://github.com/user-attachments/assets/2bb63cb9-e4f9-4b96-8dfe-d5cdc1cd83f7" />


## Step 2: Configure Data Sources
Before analysis on the data can start, the column data types need to be adjusted in Power Query. In Power BI Desktop, a new report is opened. With the **Get Data** tab, the table from Excel can be selected and the transform option loads the Power Query editor. 

### Configure Sales Data
The columns **Gross Product Price**, **Tax per Product**, and **Rating** are updated to the **Fixed Decimal Number** data type. 
<br></br>
<img width="1917" height="973" alt="fixed-decimal-column-conversion" src="https://github.com/user-attachments/assets/9f5e1d7c-2081-4304-a620-836e67a561f0" />
<br></br>
The columns **Quantity Purchased**, **Loyalty Points**, and **Stock** are updated to the **Whole number** data type and the column **Product Category** is updated to the **Text** data type. Now that the data types are set, the data can be validated. Navigating to the **View** tab, the Order ID shows that each entry is valid.
<br></br>
<img width="1283" height="781" alt="order-id-valid" src="https://github.com/user-attachments/assets/c3cda3a9-2d65-4c05-bf72-9f8f80e875c4" />
<br></br>
### Configure Purchase Data
Another Excel spreadsheet prepared by Tailwind Traders is the `Purchases.xlsx` file, which contains purchase order data. The same process to transform the data in Power Query is applied. The columns **PurchaseID**, **OrderID**, **Return Policy (Days)**, and **Warranty (Months)** are updated to the **Whole Number** data type. The columns **Purchase Date** and **Last Visited** are updated to the **Date** data type. The columns **Supplier** and **ReturnStatus** are updated to the **Text** data type.

As Tailwind Traders is only concerned with generated revenue, returned orders should not appear in the report. Uncheck the box for **Returned**.

<img width="1302" height="685" alt="hide-returned-orders" src="https://github.com/user-attachments/assets/259d6f75-4be4-4159-9c9e-be24614a44da" />

### Configure Countries Data
The final Excel spreadsheet containing country data is the `Countries.xlsx` file. The same process to transform the data in Power Query is applied. The columns **Country ID** and **Exchange ID** are updated to the **Whole Number** data type and the column **Country** is updated to the **Text** data type. 

### Configure Historical Currency Exchange Data
A Python script will serve as the basis for currency conversion. To load the script, use the **Get Data** tab in the report home view. Choose the **More** option and Enter **Python** in the search bar so that the **Python Script** option appears. 

The script can be seen below:
```
import pandas as pd
from io import StringIO

data = """Exchange ID;ExchangeRate;Exchange Currency
1;1;USD
2;0.75;GBP
3;0.85;EUR
4;3.67;AED
5;1.3;AUD"""
df = pd.read_csv(StringIO(data), sep=';')

# Return the transformed dataframe
df
```

The script takes the raw string data and loads it into a pandas DataFrame. StringIO is used so that the `pd.read_csv()` function can read the string data like a file. Ensure the name of the table is **Exchange Data** and the **ExchangeRate** column has data type **Fixed Decimal Number**. 

## Step 3: Design the Data Model

### Configure Model Relationships
Go to the **Model View** to set up relationships between the tables. The relationship between the **Countries** and **Exchange Data** tables should be set on **Exchange ID** with a one-to-one cardinality. The cross-filter direction should be set to both, and this is the active relationship.

<p align="center">
  <img width="446" height="992" alt="countries-exchange-data-relationship" src="https://github.com/user-attachments/assets/1588bba8-e9d3-4049-9203-9cab530e6e8a" />
</p>

The relationship between **Sales** and **Countries** should be set on **Country ID** with a many-to-one cardinality. The cross-filter direction should be set to both, and this is the active relationship.

<p align="center">
  <img width="822" height="942" alt="sales-countries-relationship" src="https://github.com/user-attachments/assets/52c6f5d1-ffc7-4d68-8d84-c0a666cb8100" />
</p>

The relationship betweeen **Purchases** and **Sales** should be set on **OrderID**  with a one-to-one cardinality. The cross-filter direction should be set to both, and this is the active relationship.

<p align="center">
  <img width="830" height="947" alt="purchases-sales-relationship" src="https://github.com/user-attachments/assets/9e4f2a1e-50eb-4cc0-ab09-0b2f5a86b27a" />
</p>

### Add a Calendar Table
To track dates of interest, a calendar table can be implemented with DAX expressions. In the **Model View** tab, navigate to the **Calculations** tab and select **New Table**. Use the follow code to implement calendar table named "CalendarTable". To perform data aggregations later, create **Month**, **Quarter**, **Weekday**, and **Day** columns.
```
CalendarTable = 
ADDCOLUMNS(
CALENDAR(DATE(2020, 1, 1), DATE(2023, 12, 31)),
"Year", YEAR([Date]),
"Month Number", MONTH([Date]),
"Month", FORMAT([Date], "MMMM"),
"Quarter", QUARTER([Date]),
"Weekday", WEEKDAY([Date]),
"Day", DAY([Date])
)
```

Make sure the date formats between the **Date** column in **CalendarTable** match with the **Purchase Date** column in **Purchases** for consistency.

<p align="center">
  <img width="896" height="970" alt="match-date-formats" src="https://github.com/user-attachments/assets/41701e13-0817-4e97-9a85-6ee0d12d4ba1" />
</p>

A relationship between the calendar and **Purchases** should be set on the **Date** field with a many-to-one cardinality. The cross-filter direction should be set to both, and this is the active relationship.

<p align="center">
  <img width="832" height="928" alt="calendar-purchases-relationship" src="https://github.com/user-attachments/assets/f4034189-1b74-4dff-8acb-b64be8aa8b83" />
</p>

### Sales in USD Table
This table serves to collect global sales for Tailwind Traders and convert them to USD to understand culminative sales performance. There should be many-to-one and one-to-one relationships across the tables, allowing for data to be pulled from multiple tables. Select the **New Table** option from the **Calculations** tab. The table creation methodology can be seen below: 
- Use the ADDCOLUMNS() expression to gather column data
- Start with the **Sales** table as a basis on which to add more columns.
- Add a column named **Country Name** pulling **Country** column values from the **Countries** table to examine where a purchase was made.
- Add a column named **Exchange Rate** pulling **Exchange Rate** column values from the **Exchange Data** table for currency conversion.
- Add a column named **Exchange Currency** pulling **Exchange Currency** column values frome the **Exchange Data** table for currency type.
- Add a calculated column named **Gross Revenue USD** which is derived from multiplying the **Gross Revenue** column values from the **Sales** table with the **Exchange Rate** column values from the **Exchange Data** table to get the gross revenue in USD.
- Add a calculated column named **Net Revenue USD** which is derived from multiplying the **Net Revenue** column values from the **Sales** table with the **Exchange Rate** column values from the **Exchange Data** table to get the net revenue in USD.
- Add a calculated column named **Total Tax USD** which is derived from multiplying the **Total Tax** column values from the **Sales** table with the **Exchange Rate** column values from the **Exchange Data** table to get the total tax in USD.

The full DAX expression can be seen below:
```
Sales in USD = 
ADDCOLUMNS(
    Sales,
    "Country Name", RELATED(Countries[Country]),
    "Exchange Rate", RELATED('Exchange Data'[ExchangeRate]),
    "Exchange Currency", RELATED('Exchange Data'[Exchange Currency]),
    "Gross Revenue USD", [Gross Revenue] * RELATED('Exchange Data'[ExchangeRate]),
    "Net Revenue USD", [Net Revenue] * RELATED('Exchange Data'[ExchangeRate]),
    "Total Tax USD", [Total Tax] * RELATED('Exchange Data'[ExchangeRate])
)
```
A relationship between the **Sales** and **Sales in USD** can be set on the **OrderID** column with a one-to-one cardinality. The cross-filter is set to both, and this is the active relationship.

<p align="center">
  <img width="827" height="932" alt="image" src="https://github.com/user-attachments/assets/b6e3db10-b9f9-43a5-b6a5-e1dcf6eea78b" />
</p>

## Step 4: Data Aggregations
With a complete data model, calculations on the tables can be performed. 

### Yearly Profit Margin
To get the yearly profit margin, navigate to the **Model View**, click on the ellipsis next to the **Sales in USD** table name, and select **New Measure**. Divide the **Gross Revenue USD** column sum by the **Net Revenue USD** column sum to perform the calculation. The DAX logic can be seen below:

```
YearlyProfitMargin = DIVIDE(
  'Sales in USD'[Gross Revenue USD],
  'Sales in USD'[Net Revenue USD]
)
```
Format this measure as percentage.

### Quarterly Profit Margin
To get the quarterly profit, aggregate the gross and net sales by quarter to date using the `DATESQTD()` function. The DAX logic can be seen below:
```
QTD Profit = DIVIDE(
    CALCULATE(SUM('Sales in USD'[Gross Revenue USD]), DATESQTD('CalendarTable'[Date])),
    CALCULATE(SUM('Sales in USD'[Net Revenue USD]), DATESQTD('CalendarTable'[Date]))
)
```
Format this measure as percentage.

### Year-to-Date Profit Margin
To get the year-to-date profit, use the `TOTALYTD()` function passing **YearlyProfitMargin** as the expression parameter and **Date** from **CalendarDate** table as the dates parameter. The DAX logic can be seen below:
```
YTD Profit = TOTALYTD([YearlyProfitMargin], 'CalendarTable'[Date])
```
Format this measure as percentage.

### Median Sales
The total sales can be calculated using the **Gross Revenue USD** column from the **Sales in USD** table:
```
MedianSales = MEDIAN('Sales in USD'[Gross Revenue USD])
```

### Evaluate Measure Performance
To ensure that the measures run effectively, their performance will be evaluated using Power BI Desktop's Performance Analyzer. Navigate to the report view and create a card visual for the four measures. In the **View** tab, select **Performance Analyzer**. Start recording and refresh the visuals to examine their performance. The visuals should load under 200 ms as shown below:

<img width="521" height="395" alt="performance-analyzer-on-measures" src="https://github.com/user-attachments/assets/fef72a42-af0d-4cfc-a2dd-44fb92842af6" />

Remove all card visuals from the report view after verifying satisfactory loading performance.

## Step 5: Create a Sales Report
Navigate to the report view. Rename Page 1 as "Sales Overview". 

### Loyalty Ponts by Country Bar Chart
Select the **Clustered bar chart** visual from the **Visualizations pane. Set **Loyalty Points** from the **Sales in USD** table as the x-axis and **Country** from the **Countries** table as the y-axis. Title the chart "Loyalty Points by Country" and turn on data labels.

## Technologies Used
- Microsoft Excel
- Power BI Desktop
- Python 3.13.3
- Pandas

## Files Included
- `Tailwind-Traders-Sales.xlsx`: Excel spreadsheet of company sales data for Tailwind Traders
- `Purchases.xlsx`: Excel spreadsheet of customer purchases for Tailwind Traders
- `Countries.xlsx`: Excel spreadsheet of countries and exchange rate ids
