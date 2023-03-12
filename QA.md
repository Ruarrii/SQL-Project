What are your risk areas? Identify and describe them.  
Risk areas: 
- Readability 
- NULL values 
- Duplicates


QA Process:
Describe your QA process and include the SQL queries used to execute it.

1. Are my queries readable? 

-- Reading over my work in the relevant files, I am confident that my queries are readable as I  

    - Used uppercase for keywords
    - Used aliases for readability
    - Used indentation and whitespace 
    - Broke long queries into subqueries  

The only issue I have found here is that perhaps I should have commented my queries. So, as part of my QA process, I will go through my work and comment where I deem necessary.   


2. Checking field values.  
-- There should be no NULL values within the tables:
- salesbysku_clean
```SQL
SELECT * FROM salesbysku_clean
WHERE totalordered IS NULL OR
	  productsku IS NULL;
```
- salesreport_clean
```SQL
SELECT * FROM salesreport_clean
WHERE productsku IS NULL OR
      totalordered IS NULL OR
	  productname IS NULL OR
	  stocklevel IS NULL OR
	  restockingleadtime IS NULL OR
	  sentimentscore IS NULL OR
	  sentimentmagnitude IS NULL OR
	  ratio IS NULL;
```
- products_clean
```SQL
SELECT * FROM products_clean
WHERE sku IS NULL OR
      name IS NULL OR
	  orderedquantity IS NULL OR
	  stocklevel IS NULL OR
	  restockingleadtime IS NULL OR
	  sentimentscore IS NULL OR
	  sentimentmagnitude IS NULL;
```
- analytics_clean3
```SQL
SELECT * FROM analytics_clean3
WHERE visitnumber IS NULL OR
      visitid IS NULL OR
	  date IS NULL OR
	  fullvisitorid IS NULL OR
	  channelgrouping IS NULL OR
	  socialengagementtype IS NULL OR
	  unitssold IS NULL OR
	  pageviews IS NULL OR
	  timeonsite IS NULL OR
	  bounces IS NULL OR
	  revenue IS NULL OR
	  unitprice IS NULL OR
	  revenue2 IS NULL;
```
- sessions_clean2
```SQL
SELECT * FROM sessions_clean2
WHERE fullvisitorid IS NULL OR
      channelgrouping IS NULL OR
	  time IS NULL OR
	  country IS NULL OR
	  city IS NULL OR
	  totaltransactionrevenue IS NULL OR
	  transactions IS NULL OR
	  timeonsite IS NULL OR
	  pageviews IS NULL OR
	  sessionqualitydim IS NULL OR
	  date IS NULL OR
	  visitid IS NULL OR
	  type IS NULL OR
	  productquantity IS NULL OR
	  productprice IS NULL OR
	  productrevenue IS NULL OR
	  productsku IS NULL OR
	  v2productname IS NULL OR
	  v2productcategory IS NULL OR
	  productvariant IS NULL OR
	  currencycode IS NULL OR
	  transactionrevenue IS NULL OR
	  transactionid IS NULL OR
	  pagetitle IS NULL OR
	  pagepathlevel1 IS NULL OR
	  ecommerceactiontype IS NULL OR
	  ecommerceactionstep IS NULL OR
	  ecommerceactionoption IS NULL;
```
-- There have been no NULL values created after the initial data cleaning!  

3. Checking for unique fields  
-- Ensure that primary keys are unique
- salesbysku_clean
```SQL
SELECT *, COUNT(*) AS count 
FROM salesbysku_clean
GROUP BY productsku, totalordered
HAVING COUNT(*) > 1;
```
- salesreport_clean
```SQL
SELECT *, COUNT(*) AS count 
FROM salesreport_clean
GROUP BY productsku 
HAVING COUNT(*) > 1;
```
- products_clean
```SQL
SELECT *, COUNT(*) AS count 
FROM products_clean
GROUP BY sku
HAVING COUNT(*) > 1;
```
- sessions_clean2
```SQL
SELECT *, COUNT(*) AS count 
FROM sessions_clean2
GROUP BY visitid 
HAVING COUNT(*) > 1;
```
- analytics_clean3
```SQL
SELECT fullvisitorid, COUNT(*) AS count 
FROM analytics_clean3
GROUP BY fullvisitorid 
HAVING COUNT(*) > 1;
```
-- I am still confused by the analytics table, I'm not sure if I'm missing something but there are so many duplicates! I'm at a loss for how to deal with them.  

4. Combining multiple assertions into a single query  
- salesbysku_clean
```SQL
SELECT productsku, 'missing_sku' AS reason
FROM salesbysku_clean
WHERE productsku IS NULL
UNION ALL
SELECT productsku, 'duplicate_id' AS reason
FROM (
	  SELECT productsku, SUM(1) AS count
	  FROM salesbysku_clean
	  GROUP BY 1
) subq
WHERE count > 1
```
The above query can easily be modified to test other columns/tables.  

5. Unit Testing  

-- I wanted to do unit testing but even after extensive reading, I'm totally lost; if it wasn't Sunday, I would talk to a mentor.

6. Integration Testing  

I'm not clear on how to query a query, or how to test if a query gets the expected results - how does one determine the expected results?

