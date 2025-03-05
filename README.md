
# Financial Statement Automation for GL retail corporation

## Company activity/ field :  
 
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

 GL retail corporation has a head office location and 5 retail stores , has been operating for several years  and need a data driven approach to turn good performance into great
 
 
---- **Delivrables**

-Automate and visual current reporting of financial statement
-Improve efficiency and reporting integrity
-Develop new insights to provide comparative advantage
-Build the data model in power query
-Build income statement power BI
-Build  balance sheet in power BI

 There are three sets of reports that need to  be automate using Power BI.

-AR & AP Ageing Reports : these reports take some time for the team to create in excel , this is mainly because the data is been  collected  manually from multiple sources, as well
as making multiple manual adjustments. So we need to replicate the current format, and able to slice by customer, supplier and region.

-Financial Statements : Automating these would be a big time saver . From the income statement perspective, a high level summary would be great. The most important metrics are
 Gross Profit %, EBIT % and Net Income %. For the balance sheet, we need the following ratios the Current Ratio between Current Assets and Liabilities. Build a means
to  have an easy way to see those trends that would be ideal. 

-Sales & Inventory : These report will be really valuable for both the finance and category teams. Whilst the total sales are correct, there is some issues in terms of product and category being misallocated. we shall try  to reallocate those items.

For all of these reports ,  make sure that we know when the data was last updated. it will help the team to understand what they’re looking at. 

  



 ### Data sources
 ---

*Company central database - GLRetail_FinanceDW  database*

### Tools
---
  - Azure Data Studio-SQL :  Data extraction and loading from the database
  - Power Query : Data transformation
  - Power BI : Data model and visualization
  - DAX : Calculating mesures

    
   
### Data cleaning 
---

Query to extract data  we need from the GLRetail_FinanceDW  database . A view has be created to avoid writing the query each time.

```SQL

CREATE VIEW vwGLTrans

-- This view contains all the information required to create a financial statements.

AS

SELECT 

--FactGLTran
    gl.FactGLTranID,
    gl.GLTranAmount,
    gl.JournalID,
    gl.GLTranDescription,
    gl.GLTranDate,

-- GLAcctID
    acc.AlternateKey as 'GLAcctNum',
    acc.GLAcctName,
    acc.Statement,
    acc.Category,
    acc.Subcategory,

--Stores
    sto.AlternateKey as 'StoreNum',
    sto.StoreName,
    sto.ManagerID,
    sto.PreviousManagerID,
    sto.ContactTel,
    sto.AddressLine1,
    sto.AddressLine2,
    sto.ZipCode,

--Region
    reg.AlternateKey 'RegionNum',
    reg.RegionName,
    reg.SalesRegionName,

--Last Refresh Date

    CONVERT(datetime2, GETDATE() at time zone 'UTC' at time zone 'Central Standard Time') AS 'LastRefreshDate'

  FROM [dbo].[FactGLTran] as gl

    join dimGLAcct  acc on  gl.[GLAcctID] = acc.[GLAcctID]
    join  dimStore as sto  on  gl.StoreID   =  sto.StoreID 
    join  dimRegion as reg  on sto.RegionID = reg.RegionID

  GO

```

* After importing the data in power query , a created the  fact and dimension tables again as the we import all the  data from the server as a view consisting of one table.
 		- The source table imported was made a staging table as we disenable load
		- The fact and dimension tables created were reference to the  source table

* The data model was created in power BI in the model tab , by linking the fact table (FactGLTrans) to the different dimension tables(DimDate, DimStore, DimGLAccts) as well as between DimStore and DimRegion

* Is best practice to populate values using a dax measure and not column. So i have created a dax measure that will be use in substitution of the GLTranAmount column
		SumAmount = SUM(FactGLTrans[GLTranAmount])

* Created header(element of income statement ad balance sheet) table in excel and imported to power bi  for more flexibility in the building of the income statement and balancesheet 



### Exploratory data analysis
---

EDA involved exploring  the data in the database , to identify null values , fact and de=imension tables:

*I have use the where and is not null clause to check if all value of column are null , especially for the  ProductCategory in the FactGLTran table

*From the GLRetail_FinanceDW  database of the company  and the  Entity relation diagram, We can identify the following :

Fact table  : FactGLTran has as primary key FactGLTranID  and 3 foreign keys : StoreID , JournalID  and GLAcctID
        
Dimension tables : dimGLAcct (primary key : GLAcctID ) , dimJournal (primary key :JournalID) , dimStore (primary key :StoreID) , dimRegion (primary key : RegionID) , dimEmployee (primary key : EmployeeID )


### Data analysis
---

**Building the income statement**

```
* To only populate elements of the income statement excluding the balance sheet element, i use the below dax measure
          I/S Amount = CALCULATE([SumAmount],Dimheader[Statement] ="Income Statement")

* The different subtotal represent Gross profit , EBIT , EBT and net income .  I use the below dax measure to have the different subtotal 
	I/S subtotal = CALCULATE([I/S Amount],FILTER(ALL(Dimheader),Dimheader[Sort] < MAX(Dimheader[Sort]))) 

*Calculating the percentage of revenue ( gross profit % , EBIT% , Net income %)
		staging - % of revenue = var revenue = CALCULATE([I/S Amount], FILTER(ALL(Dimheader),Dimheader[Category] ="Revenue")) return DIVIDE([I/S subtotal],revenue,0)

* Creating the income statement measure by combining the I/S Amount and I/S subtotal  measure  by using switch and selectedvalue functions
      	Income statement = SWITCH(TRUE(),SELECTEDVALUE(Dimheader[MeasureName])="Subtotal",[I/S subtotal], SELECTEDVALUE(Dimheader[MeasureName]) ="Per_Of_Revenue",[% of revenue],[I/S  Amount])

* Formatting the % of revenue to have the same format as the others
	% of revenue = FORMAT([staging - % of revenue],"0.00%")

*Using the ABS function to remove the negative signs 
		I/S Amount = CALCULATE(ABS([SumAmount]),Dimheader[Statement] ="Income Statement")

*Create sub category display filter
		var display_filter = not ISFILTERED(DimGLAccts[Subcategory])
		return
		SWITCH(TRUE(),
		SELECTEDVALUE(Dimheader[MeasureName])="Subtotal" && display_filter ,[I/S subtotal],
		SELECTEDVALUE(Dimheader[MeasureName]) ="Per_Of_Revenue" && display_filter ,[% of revenue],[I/S Amount]) 

* TO obtain Gross Margin Ratio , the below dax measure was used
		Gross Margin Ratio = 
 		var Gross_Profit = CALCULATE([I/S subtotal], Dimheader[Category]="Gross Profit")
 		var Revenue = CALCULATE([I/S Amount],Dimheader[Category]="Revenue")
 		return DIVIDE( Gross_Profit,Revenue,0)

* TO obtain Operating Margin Ratio , the below dax measure was used
		Operating Margin Ratio = 
 		var Operating_Expenses = CALCULATE([I/S subtotal], Dimheader[Category]="EBIT")
 		var Revenue = CALCULATE([I/S Amount],Dimheader[Category]="Revenue")
 		return DIVIDE( Operating_Expenses ,Revenue,0)

```

**Building the balance sheet**

```
* To only populate elements of the balance sheet excluding the income statement element, i use the below dax measure
		B/S Amount = CALCULATE([SumAmount], Dimheader[Statement]= "Balance Sheet")

* To populate the cumulative amount of the balance sheet each year , the below dax measure was use
		Cumulative Amount = CALCULATE([B/S Amount], FILTER(ALL(DimDate),DimDate[Date] <= MAX(DimDate[Date])))

*The different subtotal represent total asset, total liability and total equity .  I use the below dax measure to have the different subtotal 
		 B/S subtotal = CALCULATE([B/S Amount], ALL(Dimheader), Dimheader[Balance Sheet Section] in VALUES(Dimheader[Balance Sheet Section]))

* calculating the balance sheet column
		Balance sheet = SWITCH(TRUE(), selectedvalue(Dimheader[MeasureName]) ="Section_Subtotal", [B/S subtotal], [Cumulative Amount])

* calculating the Opening retained Earnings
		Opening Retained Earnings = CALCULATE(ABS([SumAmount]), FactGLTrans[GLAcctNum] =4100, ALL(DimDate), ALL(Dimheader))
  
* calculating the retained Earnings
		Retained Earnings = [Opening Retained Earnings] + CALCULATE(ABS([SumAmount]),
                FILTER(ALL(DimDate),DimDate[Date]<= MAX(DimDate[Date])), 
                FILTER(ALL(Dimheader),  Dimheader[Statement] = "Income statement"))

* Total Equity
	Total Equity = [B/S subtotal] + [Retained Earnings]

* calculating total liability and equity
		Total Liabilities & Equity = CALCULATE( [Cumulative Amount], ALL(Dimheader) ,Dimheader[Balance Sheet Section] ="Total Liabilities" || 
                Dimheader[Balance Sheet Section] = "total Equity") +[Retained Earnings]

* Final DAX measure  , in order to populate all the  value of the balance sheet
		Balance sheet = SWITCH(TRUE(),
 		selectedvalue(Dimheader[MeasureName]) ="Section_Subtotal", [B/S subtotal],
		selectedvalue(Dimheader[MeasureName]) ="Retained_Earnings",[Retained Earnings],
		selectedvalue(Dimheader[MeasureName]) ="Total_Equity",[Total Equity],
		selectedvalue(Dimheader[MeasureName]) ="Total_LE",[Total Liabilities & Equity],
 		[Cumulative Amount])

*Create sub category display filter , to remove undesired subcategory item in some category
		 Balance sheet = 
	var display_filter = not ISFILTERED(DimGLAccts[Subcategory])
	return 
	SWITCH(TRUE(),
 	selectedvalue(Dimheader[MeasureName]) ="Section_Subtotal" && display_filter, [B/S subtotal],
	selectedvalue(Dimheader[MeasureName]) ="Retained_Earnings" && display_filter ,[Retained Earnings],
	selectedvalue(Dimheader[MeasureName]) ="Total_Equity" && display_filter ,[Total Equity],
	selectedvalue(Dimheader[MeasureName]) ="Total_LE" &&display_filter ,[Total Liabilities & Equity],
 	[Cumulative Amount])

*Calculating current ratio 
		Current Ratio = 
 		var CurrentAssets = CALCULATE([Cumulative Amount] , Dimheader[Category] = "Current Assets")
 		var CurrentLiabilities = CALCULATE([Cumulative Amount], Dimheader[Category] ="Current Liabilites")
 		return DIVIDE(CurrentAssets, CurrentLiabilities,0)

*Calculating Debt ratio 
		Debt Ratio = 
		 var TotalDebt = CALCULATE([Cumulative Amount], DimGLAccts[Subcategory] = "long-term debt")
 		var  TotalAssets = CALCULATE( [B/S subtotal] , Dimheader[Category] = "Total Assets")
 		return DIVIDE(TotalDebt ,TotalAssets,0)
 

```




		


### Findings
---

-- On the business over view we are able to dive in the performance by store , category and brand of bed that the company sells.
 
-- On the store  performance overview , we are able to see which day of the week generated the most sale , performance of each category on the business . 
   Also we can assess the performance of each manager responsible for the store at any given period
 

💻💻  


