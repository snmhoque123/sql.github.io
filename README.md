## Overview
The primary goal of this project is to help our retail client to use data for better decision-making. This project includes the tasks of combining, refining, and organizing data, developing analytical models, and showcasing the outcomes in a user-friendly manner using data visualizations and dashboards.

## Contributions
- A robust Operational Data Store (ODS) and Data Warehouse were developed to meet the business needs.
- The developed system has facilitated the management to monitor the flow of supplies, the quantity of purchases, and the payments made to each supplier over a period of time.
- It also helped the management to identify the suppliers who could consistently deliver products on time in line with customer demand.
  
## Software requirements:
- MS SQL Server, SSMS, SSIS, SSAS, and Visual Studio
- Windows OS

## Data Management and Processes
Data Management and Processes involved with
- Data Governance
- Data Quality and Validation
- Data Retention Policy
- Master Data Management

## System Development Life Cycle (SDLC)
- Agile Kanban Methodology

## High-Level Model Diagram
- Created a high-level model diagram, including a conceptual model and a bubble chart, to effectively communicate complex data models to non-technical stakeholders, improving understanding and decision-making.
  
   ![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/Conceptual%20model.png)

Figure 1: A high-level graphical (conceptual model) representation of OLTP Operational Data Store (ODS) using Bill Inmon’s Relational Modeling Techniques (Normalized to 3NF).

  ![Purchase Analysis](https://github.com/sshahidul29/Supply-Chain-Data-Modernization/blob/main/Figures/Bubble%20Chart.PNG)  

Figure 2: A high-level graphical (bubble chart) representation of a business process of Enterprise Data Warehouse using Ralph Kimball’s Dimensional Modelling Approach.
## SQL Code
```SQL
/*

Sample Database transfer script from OLTP to Staging environment where entities are 
Purchasetransaction, SalesTransaction, Sore, City, State, Customer, Department, Employee, MaritalStatus, Product, Promotion, PromotionType and Vendor 
*/

use cansoft_oltp

Select  * from PurchaseTransaction

----join SalesTransaction with Promotion, and PromotionType tables for staging
	select * from SalesTransaction s  inner join Promotion  p  on p.PromotionID=s.PromotionID
	inner join PromotionType t  on t.PromotionTypeID=p.PromotionTypeID

--create schema for staging

use cansoft_staging

create schema oltp

---create schema for edw

use cansoft_edw
create schema edw

---- inner join the product table with department
use cansoft_oltp

select p.ProductID, p.Product, p.ProductNumber, p.UnitPrice, d.Department,getdate() as LoadDate  from Product p
inner join Department d on p.DepartmentID=d.DepartmentID

---Create and Load Product on Staging environment on each day and set null 

use cansoft_staging

select OBJECT_ID('oltp.Product')
IF OBJECT_ID('oltp.Product') is not null
	truncate table oltp.Product 

create table oltp.Product
( 
   productID int, 
   product nvarchar(50),
   ProductNumber nvarchar(50),
   Unitprice float,
   Department nvarchar(50),
   LoadDate datetime default getdate(),
   constraint oltp_product_pk  primary key(productid)
)

-----check Product data
select *  from oltp.Product

---Create and Load Product on OLTP environment/ Enterprise database Warehouse

use cansoft_edw

create table edw.dimProduct
( 
  productsk int identity(1,1),
  productID int, 
  product nvarchar(50),
  ProductNumber nvarchar(50),
  Unitprice float,
  Department nvarchar(50),
  effectiveStartdate datetime,
  effectiveEnddate datetime,  
   constraint edw_dimproduct_sk  primary key(productsk)
)

--- store -------
use tescaoltp
select s.StoreID,s.StoreName AS Store,s.StreetAddress, c.CityName as City,st.State, getdate() as LoadDate  from Store s
inner join City c on s.CityID=c.CityID
inner join State st on st.StateID=c.StateID


select count(*) as OltpCount  from Store s
inner join City c on s.CityID=c.CityID
inner join State st on st.StateID=c.StateID


------- Create Time dimension ------
alter procedure edw.spTime AS
 BEGIN
 set nocount on
 declare @starthour int=0 
 IF (select count(*) from edw.dimTime) >0 
    TRUNCATE Table edw.dimTime

-----create day
while @starthour<=23
 BEGIN 
 insert into edw.dimTime(hour,dayperiod,effectiveStart)
 select @starthour as [hour], case
     when @starthour>=0 and @starthour<=3 then 'MidNight'
	 when @starthour>=4 and @starthour<=11 then 'Morning'
	 when @starthour=12 then 'Noon'
	 when @starthour>=13 and @starthour<=16 then 'Afternoon'
	 when @starthour>=17 and @starthour<=20 then 'Evening'
	 when @starthour>=21 and @starthour<=23 then 'Night'
	 end Dayperiod, GETDATE() as effectivestartdate

 select @starthour =@starthour+1
 END
 END

```

## Enterprise Data Warehouse was built in MS SQL Server using SSMS
- Completed database lifecycle management including installation, upgrade, troubleshooting, migration, and security.
- Work closely with Managers, Project Managers, Technical Product Managers, clients, and subject-matter experts to obtain requirements, objectives, and business rules for projects.
- Conducted stakeholder analysis and requirements gathering sessions, aligning data with business needs.
- Designed Conceptual and Logical data models for the OLTP Operational Data Store (ODS) and implemented Physical Data models using Bill Inmon’s Relational Modeling Techniques in the MS SQL Server using SSMS.
- Created the opportunity/stakeholder Matrix (it helps identify which business groups should be invited to the collaborative design sessions for each process-centric row).
- Created Bus Matrix (composition of Business process, Granularity, Facts, Fact Tables, and Dimensions).
- Designed and implemented Enterprise Data Warehouse (EDW) using Ralph Kimball’s Dimensional Modelling approach.
- Created and configured Staging, EDW, and Control Framework databases on MS SQL Server. 

  ![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/SCODS.png)

Figure 3: OLTP Operational Data Store (ODS) using Bill Inmon’s Relational Modeling Techniques (Normalized to 3NF).

  ![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/SCEDW.png)

Figure 4: Enterprise Data Warehouse using Ralph Kimball’s Dimensional Modelling Approach.

## ETL Pipeline was built in Visual Studio using SSIS

- The project aimed to create an ETL (Extract, Transform, and Load) pipeline for data extraction, transformation, and loading into SQL Server Databases from the CSV and OLE DB data sources.
- Designed SQL Server Integration Service (SSIS) packages and wrote T-SQL scripts for extraction, transformation, and loading (ETL) of data from different data sources to ODS, ODS to a staging area, and eventually to the 
  data warehouse.
- Design and Implement Ralph Kimball slowly changing dimension (SCD) Type 1 and 2 using SQL Server Integration Services (SSIS) data flow transformation.
- Created a metric table for an audit of Source Count, Pre-Count, Destination Count, and Post Count for ODS, Source Count, and Destination Count Staging database, and Pre, Current, Post, Type1, and Type2 Counts for EDW using the Control framework database.
- Troubleshooting and root cause analysis activities to fix bugs in the data integration process. 
- Implemented server agent for automated data loading and scheduling.
  
![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/ODSETL1.png)

 Figure 5: Control-flow diagram for ETL Pipeline from flat file source to ODS

  ![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/ODSETL2.png)

 Figure 6: Data-flow diagram of ETL Pipeline from flat file source to ODS Product table

  ![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/ODSETL1.png)

Figure 7: Data-flow diagram for Incremental load of ETL Pipeline from flat file source to ODS Purchase transactions table

![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/SCETL4.PNG) 

 Figure 8: Control-flow diagram for ETL Pipeline for EDW

  ![Purchase Analysis](https://github.com/snmhoque123/sql.github.io/blob/main/Figures/SCETL5.png) 

 Figure 9: Data-flow diagram of ETL Pipeline for EDW Dimension table

  ![Purchase Analysis](https://github.com/snmhoque123/sql.github.io/blob/main/Figures/SCETL6.png) 

Figure 10: Data-flow diagram for Incremental load of ETL Pipeline EDW Fact table

![Sales Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/Control.png)

Figure 11: Control-flow diagram for ETL Pipeline to automate the system through SQL Server Agent

## Data Mart Cubes were built using SSAS for Business Users

- Data Mart Cubes were built using SQL Server Analysis Services (SSAS) for multi-dimensional and Tabular analysis for business users. These cubes supported interactive dashboards and data visualizations for informed decision-making.

![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/SCM.png)  

Figure 12: Purchase Cube for Multidimensional Analysis

![Purchase Analysis](https://github.com/snmhoque123/SQL_Scripts/blob/main/Figures/SCT.png)  

Figure 13: Purchase Cube for Tabular Analysis
