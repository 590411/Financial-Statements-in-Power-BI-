
# Performance  Analysis of a retail company by store , category and brand 

## Company activity/ field : Retail Company - Head Rest Ltd

## Table of Content
-  [Project Overview](#project-overview)
-  [Data Sources](#data-sources)
-  [Tools](#tools)
-  [Data cleaning ](#data-cleaning )
-  [Exploratory data analysis](#exploratory-data-analysis)
-  [Data analysis](#data-analysis)
-  [Findings](#findings)
  
### Project overview
---

---- **Company information**

    - The company sells beds and sleep related accessories . It has several stores ,each of which have a home manager
    - Each store sells 4 brands of products including our own brand, Head Rest Beds
    - Each brand offers products in the categories of mattresses, bed frames, pillows and bed linen
    - Managers earn commission based on sales and their time in service
 
---- **Delivrables**

   - Create a data model for analysis
   - Find keys problem within the business
   - Build a business overview dashboard
   - Build a store performance dashboard
  
![1](https://github.com/user-attachments/assets/14b7a46a-0053-41e6-8c67-7adebbfba884)

![2](https://github.com/user-attachments/assets/5ebd90a0-e29d-483b-add8-75b756c041af)

![3](https://github.com/user-attachments/assets/ad29c707-c4d1-443a-9807-f150e69a9c5b)

![4](https://github.com/user-attachments/assets/9ac660cd-97e6-485a-a146-d27eff566d3c)

![5](https://github.com/user-attachments/assets/f4cff69b-3db1-4738-830c-4cdd7138462f)


 ### Data sources
 ---

*Company  Data  ,the dataset use for this analysis is "SalesData.xlsx"  file . we have data on sales , promotions , products , stores and managers*

### Tools
---

  - Excel-power query : Data cleaning
  - Excel-power pivot : Data model 
  - Excel-power pivot-DAX : Calculating mesures
  - Excel-pivot table : Dynamic tables
  - Excel-pivot chart : Dynamic visualisation
      - [Download Excel here](https://microsoft.com)

### Data cleaning 
---

    1.No duplicate found
	2.No Null/NA value found
	3.ALL columns had correct value format
	  
We just had to transform the data on sales , promotions , products , stores and managers into tables

### Exploratory data analysis
---

EDA involved exploring  the sales data to  answer key question , such as :

  - What is the overrall  sales trends ?
  - what is the Sales units, growth sales and margin per store  ?
  - what is the Sales units, growth sales and margin per category  ?
  - what is the Sales units, growth sales and margin per brand  ?

### Data analysis
---

```DAX
To create the model , the source data has been normalized  in order to Reduced storage space , Easier maintenance and Improved query speed . The normalization gave
	  - One fact table ; Sales Fact Table
	  - Five dimension tables  ; Product Dimension Table , Store Dimension Table , Date Dimension Table , Manager Dimension Table and Commission Dimension
        Table
	  - Created relationship between Sales Fact Table and dimension tables to build the model
	  - Creating implicit measures

--Measure to calculate units sold :: Units:=sum(Sales[Units Sold])
--Measure to calculate Margin Amount :: MarginAmt:=sum([MarginDollars])
--Measure to calculate sales :: Sales:=SUMX(Sales,[Units Sold]*[UnitPrice])
--Measure to calculate margin percentage :: MarginPct:=[MarginAmt]/[Sales]
--Measure to calculate sales days  ::  SalesDays:=DISTINCTCOUNT([DateID])
--Measure to calculate sales per days :: SalesPerDay:=[Sales]/[SalesDays]
--Calculating Sales percentage of total of product :: Pdt_Sales_PCT:=[Sales]/CALCULATE([Sales],ALL(Dim_Products))
--Calculating Sales percentage of total of managers  :: Mger_sales_PCT:= [Sales] / CALCULATE([Sales],ALL(Dim_Managers))
--Calculating Sales percentage of total of days  :: Day_Sales_PCT:=
   var TotalSales = CALCULATE([Sales],ALLSELECTED(Dim_Dates))
   var Daysales =[Sales]
   return  TotalSales /  Daysales
--calculating promotion sales  :: PromoSales:=CALCULATE([Sales],Sales[Promo]<>"")
--Calculating promotion sales percentage :: Promo_sales_PCT:=[PromoSales] /[Sales]
--Calculating sales of previous year :: SalesPY:=CALCULATE([Sales],SAMEPERIODLASTYEAR(Dim_Dates[Date]))
--Calculating sales growth :: SalesGrowth:=IFERROR([Sales]/[SalesPY]-1, "NA")
--Calculating margin growth ::MarginGrowth:=
    var MarginPY = CALCULATE([MarginPct],SAMEPERIODLASTYEAR(Dim_Dates[Date])) 
    return [MarginPct] - MarginPY		
--Calculating sales per day YTD :: SalesPerdayYTD:=CALCULATE([SalesPerDay],DATESYTD(Dim_Dates[Date]))



 ##Function use
	 
		-- Creating a calculated column for the store size using the switch function :: =SWITCH(Dim_Stores[Store Type],"SM","Small","MED","Medium","WAREHOUSE","WAREHOUSE","OTHER")
			
	  - Other calculation
			
		-- Creating a calculated column for the MarginDollar  :: =[Units Sold]*[UnitPrice]*[RawMargin]
		-- Creating a calculated commission column in the sales table :: =RELATED(Dim_Commission[Commission])*[Units Sold]*[UnitPrice]	
		
## Renaming of  Measures name in pivot table
		
		   YoY sales(Salegrowth) , means year over year sales
		   pdt_sales_PCT share(share) in the business overview
		   Margin(MarginPct)
		   MarginGrowth(Yoy Margin)
		   promo_sales_PCT (Promo%)
		   Day_Sales_PCT(share)in the store performance
		   SalesPerDay(Sales/Day)
		   SalesPerdayYTD (Sales/day YTD)
		   SalesDays(days worked)

		
```

### Findings
---

-- On the business over view we are able to dive in the performance by store , category and brand of bed that the company sells.
 
-- On the store  performance overview , we are able to see which day of the week generated the most sale , performance of each category on the business . 
   Also we can assess the performance of each manager responsible for the store at any given period
 

💻💻  


