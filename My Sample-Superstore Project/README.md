## Power Query Overview  
The following steps outline my approach to cleaning and transforming raw data for analysis using Power Query.  
- Imported raw data files into Power Query. Renamed the sheet FactOrders.
  
![1  Fact_Order_Label](https://github.com/user-attachments/assets/43f775f3-5164-4f4a-b509-4b6c031ef40b)
![2  Fact_Order_Table](https://github.com/user-attachments/assets/8d37cdaa-ad31-42c5-9f0a-5f32424a1900)  

- Referenced the FactOrders table three times to create the following Dimensions tables: Dim Customers, Dim Products, and Dim Regions.
  All unnecessary columns were removed, retaining only essential fields for analysis, and data modeling.  

![3  Dim_Customers_Label](https://github.com/user-attachments/assets/ded91718-965a-4863-a76f-70d41bf5354c)  
![4  Dim_Customers_Table](https://github.com/user-attachments/assets/b8e60e8b-cd82-4a6b-bd44-3585cfa1e015)  

![5  Dim_Products_Label](https://github.com/user-attachments/assets/bba1d1b2-0fa4-46b7-b4cf-bca2f5138211)  
![6  Dim_Products_Table](https://github.com/user-attachments/assets/a1f06b17-95b2-42fc-b562-e07ffbb6310d)  

![7  Dim_Regions_Label](https://github.com/user-attachments/assets/d5b85e1d-7a19-447d-bef4-0d5a4af66bdb)  
![8  Dim_Regions_Table](https://github.com/user-attachments/assets/0f339800-f43b-4946-9c3b-60b70d140b2f)  

### Data Cleaning  

- Trimmed whitespace and standardized text formatting across multiple columns to improve consistency.
- Applied data type conversions for key columns (e.g., converting text to numeric or date formats).
- Filtered out rows containing null values to refine the dataset.
- Merged tables using inner joins to combine data from multiple sources.
- Created custom columns using advanced transformations and applied conditional logic.
- Implemented dynamic sorting and applied custom query parameters to enhance usability.

### Date Table  

A separate date table is crucial in Power BI modeling because it supports efficient time intelligence calculations. It enables consistent filtering, sorting, and grouping of data by various date attributes (e.g., year, quarter, month) across fact tables. This approach also improves performance by avoiding fragmented calculations and ensures compatibility with advanced DAX functions like rolling totals, year-to-date, and Month-over-Month growth metrics.  

For the date table in my Power BI model, I utilized a script created by Devin Knight, ensuring an optimized and comprehensive structure. Full credit for the code goes to Devin Knight  

- In Power Query select From Other Sources > Blank Query. This will launch the Power Query Editor.
- Select Advanced Editor in either the Home or View tab of the editor.
- Remove any code that the editor is currently story and replace it with the following:
```
//Create Date Dimension
(StartDate as date, EndDate as date)=>

let
    //Capture the date range from the parameters
    StartDate = #date(Date.Year(StartDate), Date.Month(StartDate), 
    Date.Day(StartDate)),
    EndDate = #date(Date.Year(EndDate), Date.Month(EndDate), 
    Date.Day(EndDate)),

    //Get the number of dates that will be required for the table
    GetDateCount = Duration.Days(EndDate - StartDate),

    //Take the count of dates and turn it into a list of dates
    GetDateList = List.Dates(StartDate, GetDateCount, 
    #duration(1,0,0,0)),

    //Convert the list into a table
    DateListToTable = Table.FromList(GetDateList, 
    Splitter.SplitByNothing(), {"Date"}, null, ExtraValues.Error),

    //Create various date attributes from the date column
    //Add Year Column
    YearNumber = Table.AddColumn(DateListToTable, "Year", 
    each Date.Year([Date])),

    //Add Quarter Column
    QuarterNumber = Table.AddColumn(YearNumber , "Quarter", 
    each "Q" & Number.ToText(Date.QuarterOfYear([Date]))),

    //Add Week Number Column
    WeekNumber= Table.AddColumn(QuarterNumber , "Week Number", 
    each Date.WeekOfYear([Date])),

    //Add Month Number Column
    MonthNumber = Table.AddColumn(WeekNumber, "Month Number", 
    each Date.Month([Date])),

    //Add Month Name Column
    MonthName = Table.AddColumn(MonthNumber , "Month", 
    each Date.ToText([Date],"MMMM")),

    //Add Day of Week Column
    DayOfWeek = Table.AddColumn(MonthName , "Day of Week", 
    each Date.ToText([Date],"dddd"))

in
    DayOfWeek
```
- Click OK. This query is actually a function that accepts parameters so you will see that it’s waiting for you to invoke it with values.  

![9  Date_Invoke](https://github.com/user-attachments/assets/ada5e6e8-171d-400a-b709-708bd10e7148)

Click Invoke and provide the range of dates that you would like the date table to return back. Then click OK
The results can now be integrated into your solution.  For example, you may add this to an existing Power Pivot data model by selecting Close & Load To.

These steps directly supported the design of a star schema by establishing a clean, well-structured dataset, with a centralized fact table connected to multiple dimension tables. This structure laid the groundwork for efficient querying and analysis within Power BI, ensuring faster performance and a user-friendly dashboard experience.  

![10  Star Schema](https://github.com/user-attachments/assets/8882680f-b46d-4429-8fd0-03f6886e2f8f)  


## Power BI Report Overview  
This report was designed to provide insights into sales revenue trends for the year 2020 thru 2023, with dynamic filtering by category, month, and year.  
![1   Report](https://github.com/user-attachments/assets/e237121d-5340-43e6-bc63-8c58c9df630f)


## Key features include   
### 1. High-Level Metrics:
- High-Level Metrics:- Total Sales: Shows $237,420, with a positive trend (+127.5%) compared to the previous month.
- Average Sale: Highlights an average value of $1,457, demonstrating spending trends.
- Total Orders: Displays 163 orders, using an upward arrow for visual emphasis on growth.

### 2. Interactive Elements   
- Filters for months and years allow users to adjust the view dynamically, focusing on specific time periods.
- Visual alignment and drill-down capabilities make it easy to explore deeper insights.

### 3. Key Visuals  
- Rounded-Edge Bar Chart Using Deneb: To enhance the visual design of my Power BI report, I implemented a custom bar chart using an add-on called Deneb, which leverages Vega-Lite (Jason). Unlike native Power BI visuals, this approach allowed me to create a bar chart with rounded edges, adding a sleek and modern touch to the visualization.

![2  Deneb_Barchart](https://github.com/user-attachments/assets/1642ccc5-d3e9-4ece-a6aa-929a723cc88f)  

### DAX Measure: Total Sales
This measure was created to calculate the total revenue generated by orders in the Fact Orders table. It uses the `SUMX` function to iterate through each row, performing row-level calculations before aggregating the results.
```
Total Sales = 
SUMX(
    'Fact Orders',
    'Fact Orders'[Quantity] * 'Fact Orders'[Sales]
)
```
**Purpose:**  
- Granular Calculation: Accurately computes total sales by multiplying the quantity of each item sold by its sales on a row-by-row basis.
- Flexible Integration: Allows for seamless use across visuals and KPIs, supporting sales trend analysis and comparative metrics.

## Total Sales Card  
One of the highlights of this report is the Total Sales Card which not only displays the overall sales amount but also includes interactive buttons. These buttons allow users to toggle views and compare Total sales vs Previous Month, and Total Profit.

![Card_Anima](https://github.com/user-attachments/assets/25436a0d-61ee-4bb3-9847-74a71dace1de)  

### KPI Card: Total Sales Display
To showcase Total Sales ($237,420) I utilized the subtitle field of the KPI card, I applied conditional formatting using the following measure:  
```
KPI Total Sales = FORMAT([Total Sales],"$#,0") & ""
```
**Explanation:**  
- **Purpose:** The subtitle field only accepts text input, so this measure converts the numeric value of `Total Sales` into a formatted string.
- **Details:** The `FORMAT` function applies currancy formatting `("$#,0")` to display the total sales amount.

This approach enables the KPI card to display key metrics dynamically while adhering to the formatting restrictions of Power BI.  

### KPI Enhancements: Dynamic Comparisons with Field Parameters
To provide dynamic comparisons under the Total Sales KPI card, I created two custom DAX measures to calculate performance against:  
- Previous Month Sales  
- Total Profit  

DAX Measure Example: KPI Total Sales vs Previous Month  
```
KPI Total Sales vs Previous Month = 
VAR _var = [Total Sales] - [Previous Month Sales]
VAR _pct = DIVIDE([Total Sales], [Previous Month Sales]) - 1
VAR _sign = IF( _var > 0, "+", "")
RETURN
_sign & FORMAT(_pct, "#0.0%") & " | " & _sign & FORMAT( _var, "#0,#" )
```
Measure Logic:  
- `_var:` Calculates the difference between current Total Sales and Previous Month Sales.  
- `_pct:` Computes the percentage growth using the DIVIDE function.  
- `_sign:` Adds a "+" for positive growth or no sign for negative or zero values.  
Return Statement: Combines the sign, percentage change, and absolute difference into a single formatted string for display.

Field Parameters and Slicer Integration:  
- These measures were added to a field parameter, enabling the user to toggle between different comparative metrics (e.g., vs Previous Month and vs Total Profit).  
- A slicer was added to the report, allowing users to dynamically switch between these views, making the report more interactive and user-centric.





















