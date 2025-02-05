# fetch-bbh

# Which is the leading brand in the Dips & Salsa category?

[https://console.cloud.google.com/bigquery?sq=1084805749063:bacb27c54e06466aa3b0397dfaba861e](https://console.cloud.google.com/bigquery?sq=1084805749063:bacb27c54e06466aa3b0397dfaba861e)

`WITH brand_metrics AS (`  
 `SELECT`  
   `p.BRAND,`  
   `COUNT(DISTINCT t.RECEIPT_ID) as receipt_count,`  
   `SUM(SAFE_CAST(NULLIF(TRIM(t.FINAL_SALE), '') AS NUMERIC)) as total_sales,`  
   `ROUND(AVG(SAFE_CAST(NULLIF(TRIM(t.FINAL_SALE), '') AS NUMERIC)),2) as avg_sale_amount`  
 ``FROM `Fetch.TRANSACTION` t``  
 ``JOIN `Fetch.PRODUCTS` p``  
   `ON t.BARCODE = p.BARCODE`  
 `WHERE`  
   `(p.CATEGORY_1 = 'Dips & Salsa' OR`  
    `p.CATEGORY_2 = 'Dips & Salsa' OR`  
    `p.CATEGORY_3 = 'Dips & Salsa')`  
   `AND p.BRAND IS NOT NULL`  
 `GROUP BY 1`  
`)`  
`SELECT`  
 `*,`  
 `RANK() OVER (ORDER BY total_sales DESC) as sales_rank,`  
 `RANK() OVER (ORDER BY receipt_count DESC) as receipt_rank,`  
`FROM brand_metrics`  
`ORDER BY total_sales DESC`  
`LIMIT 5;`

| BRAND | receipt\_count | total\_sales | avg\_sale\_amount | sales\_rank | receipt\_rank |
| :---- | ----- | ----- | ----- | ----- | ----- |
| TOSTITOS | 36 | 260.99 | 5.02 | 1 | 1 |
| GOOD FOODS | 9 | 118.89 | 10.81 | 2 | 10 |
| PACE | 24 | 118.58 | 3.49 | 3 | 2 |
| MARKETSIDE | 16 | 103.29 | 4.13 | 4 | 5 |
| FRITOS | 19 | 91.73 | 3.53 | 5 | 3 |

# 

# What are the top 5 brands by receipts scanned among users 21 and over?

[https://console.cloud.google.com/bigquery?sq=1084805749063:3437f4e278fe4b9696966314cf94cd57](https://console.cloud.google.com/bigquery?sq=1084805749063:3437f4e278fe4b9696966314cf94cd57)

`WITH user_age AS (`  
 `SELECT`  
   `ID,`  
   `DATE_DIFF(DATE('2024-09-08'), DATE(BIRTH_DATE), YEAR) as age`  
 `` FROM `Fetch.USER` ``  
 `WHERE BIRTH_DATE IS NOT NULL`  
`)`  
`SELECT`  
 `p.BRAND,`  
 `COUNT(DISTINCT t.RECEIPT_ID) as receipt_count,`  
 `COUNT(DISTINCT t.USER_ID) as unique_users,`  
 `COUNT(*) as total_transactions`  
``FROM `Fetch.TRANSACTION` t``  
``JOIN `Fetch.PRODUCTS` p``  
 `ON t.BARCODE = p.BARCODE`  
`JOIN user_age u`  
 `ON t.USER_ID = u.ID`  
`WHERE`  
 `u.age >= 21`  
 `AND p.BRAND IS NOT NULL`  
`GROUP BY p.BRAND`  
`ORDER BY receipt_count DESC`  
`LIMIT 5;`

| BRAND | total\_sales | receipt\_count | unique\_users |
| :---- | ----- | ----- | ----- |
| CVS | 72 | 1 | 1 |
| TRIDENT | 46.72 | 2 | 1 |
| DOVE | 42.88 | 3 | 3 |
| COORS LIGHT | 34.96 | 1 | 1 |
| AXE | 15.98 | 1 | 1 |

# What are the top 5 brands by sales among users that have had their account for at least six months?

[https://console.cloud.google.com/bigquery?sq=1084805749063:e88b059d54644ce0aacd407701dfae20](https://console.cloud.google.com/bigquery?sq=1084805749063:e88b059d54644ce0aacd407701dfae20) 

`WITH eligible_users AS (`  
 `SELECT ID`  
 `` FROM `Fetch.USER` ``  
 `WHERE DATE(CREATED_DATE) <= DATE_SUB(DATE('2024-09-08'), INTERVAL 6 MONTH)`  
`)`  
`SELECT`  
 `p.BRAND,`  
 `SUM(SAFE_CAST(NULLIF(TRIM(t.FINAL_SALE), '') AS NUMERIC)) as total_sales,`  
 `COUNT(DISTINCT t.RECEIPT_ID) as receipt_count,`  
 `COUNT(DISTINCT t.USER_ID) as unique_users`  
``FROM `Fetch.TRANSACTION` t``  
``JOIN `Fetch.PRODUCTS` p``  
 `ON t.BARCODE = p.BARCODE`  
`JOIN eligible_users u`  
 `ON t.USER_ID = u.ID`  
`WHERE p.BRAND IS NOT NULL`  
`GROUP BY p.BRAND`  
`ORDER BY total_sales DESC`  
`LIMIT 5;`

| BRAND | receipt\_count | unique\_users | total\_transactions |
| :---- | ----- | ----- | ----- |
| NERDS CANDY | 3 | 1 | 6 |
| DOVE | 3 | 3 | 6 |
| TRIDENT | 2 | 1 | 4 |
| MEIJER | 2 | 2 | 4 |
| COCA-COLA | 2 | 2 | 4 |

# Communication with the stakeholders:

Subject: Data Review: Key Issues & Trends in Our User Data

Hi Firstname Lastname,

Thank you for sending the data over, we have performed a thorough review and would like some clarifications on some data issues:

1. 0.48% of products are missing the BARCODE unique identifiers, which is a required field.  
2. 26.78% of products are missing the brand name and manufacturer which can help identify product performance.  
3. Please clarify the use case of having 4 categories as 92% of the data provided is missing a CATEGORY\_4 field.  
4. Duplicate RECEIPT\_IDs where FINAL\_QUANTITY='zero' \- are these returns or data errors?  
5. FINAL\_SALE has many empty spaces instead of proper values, please clarify if these are data entry issues and provide the missing values.  
6. The dataset provided are purchases from 2024-06-12 til 2024-09-08 but there are accounts with creation date 2024-09-11. Can we verify if these are guest checkouts with a later account creation date?  
7. Birth dates show users from ages 2 to 124 years old. Please verify if implementing age restrictions and validation to comply with data privacy regulations will be necessary.

Would you be available this week to discuss these data quality issues and find a solution? We can review each point in detail and create a plan for data standardization. 

On a more sparkling note, here's what's brewing in the Coca-Cola vs Pepsi rivalry:

Coca-Cola leads in total sales ($3,949 vs Pepsi's $3,266) with more transactions and receipts. However, Pepsi shows higher average sales per transaction ($6.15 vs $4.94) and per receipt ($9.28 vs $7.67). This suggests Pepsi buyers purchase larger quantities or higher-priced items, while Coca-Cola has more often but smaller purchases or Coca-Cola does more promotions during the Summer to lower their prices.

Best,  
BBH
