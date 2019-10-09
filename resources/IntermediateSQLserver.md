
# T - SQL (MySQL) Tutorial with DataCamp
Alicja Wilk. 

#### Tutorial Based on Data Camp Course - Intermediate SQL server - by Ginger Grant


- Data: https://data.world/timothyrenner/ufo-sightings 
- Data: Shipments - in DataCamp, Orders - in DataCamp

https://www.datacamp.com/courses/intermediate-t-sql

## Table of Contents

1. [Basic Aggregation Operatiors](#bao)
2. [Missign Values](#mv)
3. [Case statement](#cs)
4. [Datepart](#dp)
5. [Rounding and Truncating](#rat)
6. [Creating and using variables](#cauv)
7. [Creating a WHILE loop](#cawl)
8. [Derived Tables](#dt)
9. [Common Table Expressions](#cte)
10. [Window Functions](#wf)
11. [Common Window Functions](#cwf)

### Basic Aggregation Operatiors <a class="anchor" id="bao"></a>
#Calculate the average, minimum and maximum

SELECT 
AVG(DurationSeconds) AS Average,
MIN(DurationSeconds) AS Minimum,       
MAX(DurationSeconds) AS Maximum      
FROM Incidents#Calculate the aggregations by Shape

SELECT Shape,
       AVG(DurationSeconds) AS Average, 
       MIN(DurationSeconds) AS Minimum, 
       MAX(DurationSeconds) AS Maximum
FROM Incidents
GROUP BY Shape#Calculate the aggregations and group by shape
#Return records where minimum of DurationSeconds is greater than 1

SELECT Shape,
       AVG(DurationSeconds) AS Average, 
       MIN(DurationSeconds) AS Minimum, 
       MAX(DurationSeconds) AS Maximum
FROM Incidents
GROUP BY Shape
HAVING MIN(DurationSeconds) > 1
### Missing values <a class="anchor" id="mv"></a>


Blank is not null value. The best way to look for blank value is to look for len() >0 

We use INNULL function to replace missing values with something. 

COALESCE returns the first non missing value
#Exclude all the missing values from IncidentState 

SELECT IncidentDateTime, IncidentState
From Incidents
WHERE IncidentState IS NOT NULL#Check the IncidentState column for missing values and replace them with the City column
#Filter to only return missing values from IncidentState

SELECT IncidentState, ISNULL(IncidentState, City) AS Location
FROM Incidents
WHERE IncidentState IS NULL# Replace missing values

SELECT Country, COALESCE(Country, IncidentState, City) AS Location
FROM Incidents
WHERE Country IS NULL
### Case statement <a class="anchor" id="cs"></a>

Used to evaluate conditions in a query

# adding a new column 'SourceCountry' based whether a country is usa or international or not

SELECT Country, 
       CASE WHEN Country = 'us'  THEN 'USA'
       ELSE 'International'
       END AS SourceCountry
FROM Incidents# create new groups using the case statement
#cutting the duration into different cases

SELECT DurationSeconds, 
      CASE WHEN (DurationSeconds <= 120) THEN 1      
       WHEN (DurationSeconds > 120 AND DurationSeconds <= 600) THEN 2        
       WHEN (DurationSeconds > 601 AND DurationSeconds <= 1200) THEN 3            
       WHEN (DurationSeconds > 1201 AND DurationSeconds <= 5000) THEN 4    
       ELSE 5 
       END AS SecondGroup   
FROM Incidents# Write a T-SQL query which will return the sum of the Quantity column as Total for each type of MixDesc.

SELECT SUM(Quantity) as Total, MixDesc
FROM shipments
Group by MixDesc#Create a query that returns the number of rows for each type of MixDesc.
#Count the number of rows by MixDesc

SELECT MixDesc, COUNT(*) 
FROM Shipments
GROUP BY MixDesc
### DATEPART <a class="anchor" id="dp"></a>

Used when you want to specify which type od date we want to calculate. 

DD for day, MM for month, YY for year, HH for hour

DATEADD(): Add or subtract datetime values Always returns a date

DATEDIFF(): Obtain the difference between two datetime values Always returns a number

#Return the difference in OrderDate and ShipDate

SELECT OrderDate, ShipDate, 
       DATEDIFF(DD, OrderDate, ShipDate) AS Duration
FROM Shipments#Return the DeliveryDate as 5 days after the ShipDate

SELECT OrderDate, 
       DATEADD(DD, 5, ShipDate) AS DeliveryDate
FROM Shipments
### Rounding and truncating <a class="anchor" id="rat"></a>
 

The round() function can be used as trucate function if we specify the third argument
#Round Cost to the nearest dollar

SELECT Cost, 
       ROUND(Cost, 0) AS RoundedCost
FROM Shipments#Truncate cost to whole number

SELECT Cost, 
       ROUND(Cost, 0,1) AS TruncateCost
FROM Shipments#Return the absolute value of DeliveryWeight

SELECT DeliveryWeight,
       ABS(DeliveryWeight) AS AbsoluteValue
FROM Shipments#Return the square and square root of WeightValue

SELECT WeightValue, 
       SQUARE(WeightValue) AS WeightSquare, 
       SQRT(WeightValue) AS WeightSqrt
FROM Shipments
### Creating and using variables <a class="anchor" id="cauv"></a>

In T-SQL, to create a variable you use the DECLARE statement. 

The variables must have an at sign (@) as their first character. 

Like most things in T-SQL, variables are not case sensitive. 

Can either use the keyword SET or a SELECT statement, then the variable name followed by an equal sign and a value.
#Declare the variable (a SQL Command, the var name, the datatype)
#Set the counter to 20
#Select and increment the counter by one 

DECLARE @counter INT 
    SET @counter = 20
    SELECT @counter = @counter +1

SELECT @counter
### Creating a WHILE loop <a class="anchor" id="cawl"></a>


WHILE some_condition 

BEGIN 
    -- Perform some operation here
    
END
#Write a WHILE loop that increments counter by 1 until counter is less than 30.

DECLARE @counter INT 
SET @counter = 20
WHILE @counter < 30
BEGIN
	SELECT @counter = @counter + 1
END

SELECT @counter
### Derived tables <a class="anchor" id="dt"></a>

Complex query into smaller steps.

Specified in the FROM clause
#Return MaxGlucose from the derived table.
#Join the derived table to the main query on Age.

SELECT a.RecordId, a.Age, a.BloodGlucoseRandom,   
       b.MaxGlucose
FROM Kidney a
JOIN(SELECT Age, MAX(BloodGlucoseRandom) AS MaxGlucose FROM Kidney GROUP BY Age) b
ON a.Age = b.Age#Create a derived table to return all patient records with the highest BloodPressure at their Age level.

SELECT *
FROM Kidney a
JOIN (SELECT Age, MAX(BloodPressure) AS MaxBloodPressure FROM Kidney GROUP BY Age) b
ON a.BloodPressure = b.MaxBloodPressure
AND a.Age = b.Age
### Common Table Expressions - CTE - <a class="anchor" id="cte"></a>

Can be used multiple times in a query and are defined as a table. 

Creating CTEs (I)

A Common table expression or CTE is used to create a table that can later be used with a query. 

To create a CTE: 
- WITH keyword followed by the CTE name and the name of the columns the CTE contains. 
- The CTE will also include the definition of the table enclosed within the AS().


#Specify the keyowrds to create the CTE
#Join the CTE on blood glucose equal to max blood glucose

WITH BloodGlucoseRandom (MaxGlucose) 
AS (SELECT MAX(BloodGlucoseRandom) AS MaxGlucose FROM Kidney)

SELECT a.Age, b.MaxGlucose
FROM Kidney a
JOIN BloodGlucoseRandom b
ON a.BloodGlucoseRandom = b.MaxGlucose#Create a CTE BloodPressure that returns one column (MaxBloodPressure) which contains the maximum BloodPressure in the table.
#Join this CTE (using an alias b) to the main table (Kidney) to return information about patients with the maximum BloodPressure.


WITH BloodPressure (MaxBloodPressure) 
AS (SELECT MAX(BloodPressure) AS MaxBloodPressure FROM Kidney)

SELECT *
FROM Kidney a 
JOIN BloodPressure b
ON a.BloodPressure = b.MaxBloodPressure
### Window Functions <a class="anchor" id="wf"></a>

Window functions with aggregations (I)

Recall that using OVER(), you can create a window for the entire table. To create partitions using a specific column, you need to use OVER() along with PARTITION BY.
#Write a T-SQL query that returns the sum of OrderPrice by creating partitions for each TerritoryName.

SELECT OrderID, TerritoryName, 
       SUM(OrderPrice)
       OVER(PARTITION BY TerritoryName) AS TotalPrice
FROM Orders
Window functions with aggregations (II)

#Calculate the number of orders in each territory.

SELECT OrderID, TerritoryName, 
       COUNT(*)
       OVER(PARTITION BY TerritoryName) AS TotalOrders
FROM Orders
### Common Window Functions <a class="anchor" id="cwf"></a>

Used expressily with Window Functions. 

The window functions LEAD(), LAG(), FIRST_VALUE(), and LAST_VALUE() require ORDER BY in the OVER() clause.


**First value in a window**

First, create partitions for each territory
Then, order by OrderDate
Finally, use the FIRST_VALUE() and/or LAST_VALUE() functions as per your requirement
#Write a T-SQL query that returns the first OrderDate by creating partitions for each TerritoryName.

SELECT TerritoryName, OrderDate, 
       FIRST_VALUE(OrderDate) 
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS FirstOrder
FROM Orders
**Previous and next values**

First, create partitions
Then, order by a certain column
Finally, use the LEAD() and/or LAG() functions as per your requirement
#Write a T-SQL query that for each territory:
#Shifts the values in OrderDate one row down. Call this column PreviousOrder.
#Shifts the values in OrderDate one row up. Call this column NextOrder. You will need to PARTITION BY the territory


SELECT TerritoryName, OrderDate, 
       LAG(OrderDate) 
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS PreviousOrder,
       LEAD(OrderDate) 
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS NextOrder
FROM Orders
**Creating running totals**

You usually don't have to use ORDER BY when using aggregations, but if you want to create running totals, you should arrange your rows!
# Create the window, partition by TerritoryName and order by OrderDate to calculate a running total of OrderPrice.

SELECT TerritoryName, OrderDate, 
       -- Create a running total
       SUM(OrderPrice) 
       -- Create the partitions and arrange the rows
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS TerritoryTotal	  
FROM Orders
**Assigning row numbers**

Records in T-SQL are inherently unordered. Although in certain situations.

Sometimes may want to assign row numbers for reference.
#Write a T-SQL query that assigns row numbers to all records partitioned by TerritoryName and ordered by OrderDate.

SELECT TerritoryName, OrderDate, 
       ROW_NUMBER() 
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS OrderCount
FROM Orders
**Calculating standard deviation**

Calculating the standard deviation is quite common when dealing with numeric columns.
# Create the window, partition by TerritoryName and order by OrderDate to calculate a running standard deviation of OrderPrice.

SELECT OrderDate, TerritoryName, 
	   STDEV(OrderPrice)
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS StdDevPrice	  
FROM Orders
**Calculating mode (I)**

First, create a CTE containing an ordered count of values using ROW_NUMBER()

Write a query using the CTE to pick the value with the highest row number

#Create a CTE ModePrice that returns two columns (OrderPrice and UnitPriceFrequency).
#Write a query that returns all rows in this CTE.

WITH ModePrice (OrderPrice, UnitPriceFrequency)
AS
(
	SELECT OrderPrice, 
	ROW_NUMBER() 
	OVER(PARTITION BY OrderPrice ORDER BY OrderPrice) AS UnitPriceFrequency
	FROM Orders 
)


SELECT * FROM ModePrice
**Calculating mode (II)**

All you need to do now is to find the OrderPrice with the highest row number
#Use the CTE ModePrice to return the value of OrderPrice with the highest row number.

WITH ModePrice (OrderPrice, UnitPriceFrequency)
AS
(
	SELECT OrderPrice,
	ROW_NUMBER() 
    OVER (PARTITION BY OrderPrice ORDER BY OrderPrice) AS UnitPriceFrequency
	FROM Orders
)

#Select the order price from the CTE

SELECT OrderPrice AS ModeOrderPrice
FROM ModePrice
WHERE UnitPriceFrequency IN (SELECT MAX(UnitPriceFrequency) From ModePrice)