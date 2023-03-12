# What issues will you address by cleaning the data?

### 1. I will remove any unwanted or irrelavent observations.
### 2. I will ensure accurate datatypes.
### 3. I will address missing values.






# Queries:
Below, provide the SQL queries you used to clean your data.

## salesbysku
The following queries were used to find duplicate rows and remove them from the dataset. I then set `productsku` as the primary key for this table.
```SQL
-- Finding duplicate rows 
SELECT *, COUNT(*) AS count 
FROM salesbysku
GROUP BY productsku, totalordered
HAVING COUNT(*) > 1;

SELECT * FROM salesbysku;

CREATE TABLE salesbysku_clean (LIKE salesbysku);

SELECT * FROM salesbysku_clean;

-- Removing duplicate rows
INSERT INTO salesbysku_clean(productsku, totalordered)
SELECT DISTINCT ON (productsku) productsku, totalordered
FROM salesbysku;

SELECT * FROM salesbysku_clean;
```
This table did not have any missing values.
```SQL
-- Checking for missing values
SELECT * FROM salesbysku_clean
WHERE totalordered = NULL;
```

## salesreport
Looking at the salesreport table, I test first for `productsku` duplicates:
```SQL
-- Checking for duplicates
SELECT productsku, COUNT(*) AS count 
FROM salesreport
GROUP BY productsku 
HAVING COUNT(*) > 1;
```
There are none. I assign `productsku` as the primary key.

This table has 78 NULL values in the `ratio` column.
``` SQL
-- Checking for missing values
SELECT * FROM salesreport
WHERE totalordered IS NULL OR
	  productname IS NULL OR
	  stocklevel IS NULL OR
	  restockingleadtime IS NULL OR
	  sentimentscore IS NULL OR
	  sentimentmagnitude IS NULL OR
	  ratio IS NULL;
```

Next, I used the `COALESCE` function to replace all of the NULL values with 0
```SQL
-- Replacing NULL values with correct data
CREATE TABLE salesreport_clean (LIKE salesreport);

SELECT * FROM salesreport_clean;

INSERT INTO salesreport_clean(productsku, totalordered, productname, 
							  stocklevel, restockingleadtime, sentimentscore, 
							  sentimentmagnitude, ratio)
SELECT DISTINCT ON (productsku) productsku, 
				    totalordered, productname, stocklevel, 
					restockingleadtime, sentimentscore, 
					sentimentmagnitude, COALESCE(ratio, 0) AS ratio
FROM salesreport;
```

I checked the `totalordered` column to be sure that no values were below zero:
```SQL
-- Checking for outliers
SELECT * FROM salesreport
WHERE totalordered < 0;
```

Next, as this is a table of sales reports, I made the assumption that `totalordered` values of 0 were irrelevant data for this table.
```SQL
-- Removing irrelevant rows
DELETE FROM salesreport_clean
WHERE totalordered = 0;
```

## products
First, I test for `sku` duplicates:
```SQL
-- Checking for duplicates
SELECT sku, COUNT(*) AS count 
FROM products
GROUP BY sku
HAVING COUNT(*) > 1;
```
There are none. I set `sku` as the primary key.

Next, I checked for NULL values:
```SQL
-- Checking for missing values
SELECT * FROM products_clean
WHERE sku IS NULL OR
      name IS NULL OR
	  orderedquantity IS NULL OR
	  stocklevel IS NULL OR
	  restockingleadtime IS NULL OR
	  sentimentscore IS NULL OR
	  sentimentmagnitude IS NULL;
```
Only one row contained NULL values in both the `sentimentscore` and `sentimentmagnitude` columns. 

In this case, I am unsure if I should replace these NULL values. I can't replace them with 0 as that (assuming) would indicate something and I can't replace them with something like 'no value' as the column data type is double precision. But, if I change the column data type to VARCHAR, then I could replace the NULL values with 'no value'. 
Changing the column datatypes:
```SQL 
-- Changing column data type
ALTER TABLE products
ALTER COLUMN sentimentscore TYPE VARCHAR(200);

ALTER TABLE products
ALTER COLUMN sentimentmagnitude TYPE VARCHAR(200);
```
To deal with the NULL values:
```SQL
-- Replacing NULL values with useful information
SELECT COALESCE(sentimentscore, 'no value') AS sentimentscore,
	   COALESCE(sentimentmagnitude, 'no value') AS sentimentmagnitude
FROM products;
```

Creating the new cleaned table:
```SQL
-- Creating a clean table
CREATE TABLE products_clean (LIKE products);

SELECT * FROM products_clean;

INSERT INTO products_clean(sku, name, orderedquantity,
						  stocklevel, restockingleadtime,
						  sentimentscore, sentimentmagnitude)
SELECT DISTINCT ON (sku) sku, name, orderedquantity, stocklevel, restockingleadtime, 
	   COALESCE(sentimentscore, 'no value') AS sentimentscore,
	   COALESCE(sentimentmagnitude, 'no value') AS sentimentmagnitude
FROM products;
```

## sessions
There appear to be several columns with all NULL values, first I will look into this.
```SQL
-- Checking to see if said column is fully empty
SELECT * FROM sessions
WHERE totaltransactionrevenue IS NOT NULL;
```
I used the above query format for each relevant column.
From the above queries I discovered that the following columns are entirely empty: productrefundamount, itemquantity, itemrevenue, and searchkeyword. I then deleted these columns.
```SQL
-- Deleting fully empty columns
ALTER TABLE sessions
DROP COLUMN productrefundamount,
DROP COLUMN	itemquantity,
DROP COLUMN	itemrevenue,
DROP COLUMN searchkeyword;
```

I will check for duplicates in the `fullvisitorid` column:
```SQL
-- Checking for duplicates
SELECT fullvisitorid, COUNT(*) AS count 
FROM sessions
GROUP BY fullvisitorid 
HAVING COUNT(*) > 1;
```
After querying this, I am pretty sure that `fullvisitorid` is not the primary key of this table. A single visitor may have multiple sessions on the site. Next, I will check for duplicates in the `visitid` column instead:
```SQL
-- Checking for duplicates
SELECT visitid, COUNT(*) AS count 
FROM sessions
GROUP BY visitid 
HAVING COUNT(*) > 1
ORDER BY count DESC;
```
Again, there are duplicates but this time I am confident that visitid should be the primary key of this table and that I simply need to deal with the duplicates, which I will do when I deal with the NULL values.

Next, I checked for NULL values in every column using the following query format:
```SQL
-- Checking for missing values
SELECT * FROM sessions
WHERE fullvisitorid IS NULL;
```
And found that the following columns all have NULL values: totaltransactionrevenue, transactions, timeonsite, sessionqualitydim, productquantity, productrevenue, currencycode, transactionrevenue, transactionid, pagetitle, and ecommerceactionoption.

Now to deal with the NULL values.
```SQL
-- Replacing NULL values with relevant information
SELECT COALESCE(totaltransactionrevenue, 0) AS totaltransactionrevenue,
	   COALESCE(transactions, 0) AS transactions,
	   COALESCE(timeonsite, 0) AS timeonsite,
	   COALESCE(sessionqualitydim, 0) AS sessionqualitydim,
	   COALESCE(productquantity, 0) AS productquantity,
	   COALESCE(productrevenue, 0) AS productrevenue,
	   COALESCE(currencycode, 'Unavailable') AS currencycode,
	   COALESCE(transactionrevenue, 0) AS transactionrevenue,
	   COALESCE(transactionid, 'Unknown') AS transactionid,
	   COALESCE(pagetitle, 'Unknown') AS pagetitle,
	   COALESCE(ecommerceactionoption, 'Not Specified') AS ecommerceactionoption
FROM sessions;
```
Now to deal with duplicates and the NULL values:
```SQL
-- Creating a new table without the duplicates and NULL values
CREATE TABLE temp_table(LIKE sessions);

SELECT * FROM temp_table;

INSERT INTO temp_table(fullvisitorid, channelgrouping, time, 
					   country, city, totaltransactionrevenue, 
					   transactions, timeonsite, pageviews, sessionqualitydim, 
					   date, visitid, type, productquantity, productprice,
					  productrevenue, productsku, v2productname, v2productcategory,
					  productvariant, currencycode, transactionrevenue, 
					  transactionid, pagetitle, pagepathlevel1, ecommerceactiontype,
					  ecommerceactionstep, ecommerceactionoption)
SELECT DISTINCT ON (visitid) fullvisitorid, channelgrouping, time, 
					   country, city, 
					   COALESCE(totaltransactionrevenue, 0) AS totaltransactionrevenue, 
					   COALESCE(transactions, 0) AS transactions, 
					   COALESCE(timeonsite, 0) AS timeonsite, 
					   pageviews, 
					   COALESCE(sessionqualitydim, 0) AS sessionqualitydim, 
					   date, visitid, type, 
					   COALESCE(productquantity, 0) AS productquantity, 
					   productprice,
					  COALESCE(productrevenue, 0) AS productrevenue, 
					  productsku, v2productname, v2productcategory,
					  productvariant,  
					  COALESCE(currencycode, 'Unavailable') AS currencycode, 
					  COALESCE(transactionrevenue, 0) AS transactionrevenue,
					  COALESCE(transactionid, 'Unknown') AS transactionid, 
					  COALESCE(pagetitle, 'Unknown') AS pagetitle, 
					  pagepathlevel1, ecommerceactiontype,
					  ecommerceactionstep, 
					  COALESCE(ecommerceactionoption, 'Not Specified') AS ecommerceactionoption
FROM sessions;
```
I then set `visitid` as the primary key for this cleaned table, which I also renamed to `sessions_clean`.
```SQL
-- Renaming table
ALTER TABLE temp_table
RENAME TO sessions_clean;
```

I divided the `productprice` data by 1,000,000:
```SQL
-- productprice altered for readability
CREATE TABLE sessions_clean2(LIKE sessions_clean);

SELECT * FROM sessions_clean2;

INSERT INTO sessions_clean2(fullvisitorid, channelgrouping, time, 
					   country, city, totaltransactionrevenue, 
					   transactions, timeonsite, pageviews, sessionqualitydim, 
					   date, visitid, type, productquantity, productprice,
					  productrevenue, productsku, v2productname, v2productcategory,
					  productvariant, currencycode, transactionrevenue, 
					  transactionid, pagetitle, pagepathlevel1, ecommerceactiontype,
					  ecommerceactionstep, ecommerceactionoption)
SELECT DISTINCT ON (visitid) fullvisitorid, channelgrouping, time, 
					   country, city, 
					   totaltransactionrevenue, 
					   transactions, 
					   timeonsite, 
					   pageviews, 
					   sessionqualitydim, 
					   date, visitid, type, 
					   productquantity, 
					   (productprice / 1000000) AS productprice,
					  productrevenue, 
					  productsku, v2productname, v2productcategory,
					  productvariant,  
					  currencycode, 
					  transactionrevenue,
					  transactionid, 
					  pagetitle, 
					  pagepathlevel1, ecommerceactiontype,
					  ecommerceactionstep, 
					  ecommerceactionoption
FROM sessions_clean;
```

## analytics
First I looked for duplicate values.
```SQL
-- Checking for duplicate values
SELECT visitid, COUNT(*) AS count 
FROM analytics
GROUP BY visitid 
HAVING COUNT(*) > 1;
```
There are a lot of duplicate rows in this table!
Looking at the data closely, I'm not sure that there is a viable primary key?

I see that `visitstarttime` contains the same data as `visitid` and that `userid` is a fully empty column. I delete both of these.


Next, I check for NULL values.
```SQL
-- Checking for missing values
SELECT * FROM analytics
WHERE socialengagementtype IS NULL;
```
Then I deal with the found NULL values in columns unitssold, pageviews, timeonsite, bounces, and revenue. I also divide unitprice by 1,000,000
```SQL
-- Replace NULL values with relevant data, and divide unitprice by 1,000,000 for readability
CREATE TABLE analytics_clean(LIKE analytics);

SELECT * FROM analytics_clean;

INSERT INTO analytics_clean(visitnumber, visitid, date, fullvisitorid, 
							channelgrouping, socialengagementtype, unitssold, 
							pageviews, timeonsite, bounces, revenue, unitprice)
SELECT visitnumber, visitid, date, fullvisitorid, 
	   channelgrouping, socialengagementtype, COALESCE(unitssold, 0) AS unitssold, 
	   COALESCE(pageviews, 0) AS pageviews, 
	   COALESCE(timeonsite, 0) AS timeonsite, 
	   COALESCE(bounces, 0) AS bounces, 
	   COALESCE(revenue, 0) AS revenue, (unitprice / 1000000) AS unitprice
FROM analytics;
```

After working with the data a bit more, I'm back here again to add a new column to the analytics table. The current `revenue` column is nearly all 0's but that doesn't really make sense when looking at the `unitssold` column. So, I am going to create a `revenue2` column.
```SQL
-- Creating a revenue2 column based on unitssold*unitprice
CREATE TABLE analytics_clean3(LIKE analytics_clean2);

SELECT * FROM analytics_clean3;

INSERT INTO analytics_clean3 (visitnumber, visitid, date, fullvisitorid, 
							 channelgrouping, socialengagementtype, 
							 unitssold, pageviews, timeonsite, bounces,
							 revenue, unitprice, revenue2)
SELECT visitnumber, visitid, date, fullvisitorid, 
							 channelgrouping, socialengagementtype, 
							 unitssold, pageviews, timeonsite, bounces,
							 revenue, unitprice, 
							 (unitprice * unitssold) AS revenue2
FROM analytics_clean2;
```

