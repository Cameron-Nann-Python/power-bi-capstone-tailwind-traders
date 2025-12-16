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
Another Excel spreadsheet prepared by Tailwind Traders is the `Purchases.xlsx` file, which contains purchase order data. The same process to transform the data in Power Query is applied. The columns **PurchaseID**, **OrderID**, **Return Policy (Days)**, and **Warranty (Months)** are updated to the **Whole Number** data type. The columns **Purchase Date** and **Last Visited** are updated to the **Date** data type. The columns **Supplier** and **ReturnStatus** are updaetd to the **Text** data type.

As Tailwind Traders is only concerned with generated revenue, returned orders should not appear in the report. Uncheck the box for **Returned**.

<img width="1302" height="685" alt="hide-returned-orders" src="https://github.com/user-attachments/assets/259d6f75-4be4-4159-9c9e-be24614a44da" />

### Configure Countries Data
The final Excel spreadsheet containing country data is the `Countries.xlsx` file. The same process to transform the data in Power Query is applied. The columns **Country ID** and **Exchange ID** are updated to the **Whole Number** data type and the column **Country** is updated to the **Text** data type. 

### Configure Historical Currency Exchange Data
A Python script will serve as the basis for currency conversion. To load the script, use the **Get Data** tab in the report home view. Choose the **More** option and Enter **Python** in the search bar so that the **Python Script** option appears. 

<img width="848" height="497" alt="python-currency-conversion" src="https://github.com/user-attachments/assets/c23deb99-8254-4bbb-9bad-72271f38536a" />

The script can be copied here:
```
import pandas as pd
from io import StringIO

data = """Exchange ID;ExchangeRate;Exchange Currency
1;1;USD
2;0,75;GBP
3;0,85;EUR
4;3,67;AED
5;1,3;AUD"""
df = pd.read_csv(StringIO(data), sep=';')

# Return the transformed dataframe
df
```

The script takes the raw string data and loads it into a pandas DataFrame. StringIO is used so that the `pd.read_csv()` function can read the string data like a file. Ensure the name of the table is **Exchange Data**.

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

## Technologies Used
- Microsoft Excel
- Power BI Desktop
- Python 3.13.3
- Pandas

## Files Included
- `Tailwind-Traders-Sales.xlsx`: Excel spreadsheet of company sales data for Tailwind Traders
- `Purchases.xlsx`: Excel spreadsheet of customer purchases for Tailwind Traders
- `Countries.xlsx`: Excel spreadsheet of countries and exchange rate ids
