## Dremio Cloud Quality & Validations Examples

Below you'll find many working examples you can run illustrating techniques for validating and maintaining code quality.

#### Branch-Audit-Merge

1. Create a branch
2. Integrate new data on the branch
3. Validate the data on the branch
4. Merge Branch when complete

```sql
-- Creating the main sales data table in the default branch
CREATE TABLE DACSalesData (
    id INT,
    productId INT,
    saleAmount FLOAT,
    saleDate DATE
);

-- Creating a staging table for incoming sales data in the default branch
CREATE TABLE DACStagingSalesData (
    id INT,
    productId INT,
    saleAmount FLOAT,
    saleDate DATE
);

-- Inserting sample sales data into the staging table
INSERT INTO DACStagingSalesData (id, productId, saleAmount, saleDate) VALUES
(1, 101, 150.0, '2022-01-01'),
(2, 102, -50.0, '2022-01-02'),  -- Negative amount (problematic)
(3, 103, 200.0, '2023-01-03');  -- Future date (problematic)

-- Creating a new branch for data integration
CREATE BRANCH dataIntegration_010224;

-- Switching to the dataIntegration branch
USE BRANCH dataIntegration_010224;

-- Merging staging data into the SalesData table on the dataIntegration branch
MERGE INTO DACSalesData AS target
USING DACStagingSalesData AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET productId = source.productId, saleAmount = source.saleAmount, saleDate = source.saleDate
WHEN NOT MATCHED THEN
    INSERT (id, productId, saleAmount, saleDate) VALUES (source.id, source.productId, source.saleAmount, source.saleDate);

-- Performing data quality checks on the dataIntegration branch
-- Check for non-negative sale amounts
SELECT COUNT(*) AS InvalidAmountCount
FROM DACSalesData
WHERE saleAmount < 0;

-- Check for valid sale dates (not in the future)
SELECT COUNT(*) AS InvalidDateCount
FROM DACSalesData
WHERE saleDate > CURRENT_DATE;

-- QUERY MAIN BRANCH
SELECT * FROM DACSalesData AT BRANCH main;

-- QUERY INGESTION BRANCH
SELECT * FROM DACSalesData AT BRANCH dataIntegration_010224;

-- Assuming checks have passed, switch back to the main branch and merge changes from dataIntegration
-- This step should be executed after confirming that the checks have passed
USE BRANCH main;
MERGE BRANCH dataIntegration_010224 INTO main;

-- QUERY MAIN BRANCH
SELECT * FROM DACSalesData AT BRANCH main;

-- QUERY INGESTION BRANCH
SELECT * FROM DACSalesData AT BRANCH dataIntegration_010224;

-- The checks for data quality (negative amounts and future dates) are simplified for this example.
-- In a real-world scenario, more sophisticated validation logic and error handling would be required.
```

#### Constraints Table

Using a table to track universal business rules with which you can validate data against.

```sql
-- Creating a table for product data
CREATE TABLE ProductData (
    productId INT,
    productName VARCHAR,
    price FLOAT,
    dateAdded DATE,
    onSale BOOLEAN
);

-- Creating a table for business constraints
CREATE TABLE BusinessConstraints (
    key VARCHAR,
    value_1 INT,
    value_2 INT,
    note VARCHAR
);

-- Inserting sample product data
INSERT INTO ProductData (productId, productName, price, dateAdded, onSale) VALUES
(1, 'Product A', 25.0, '2022-01-01', FALSE),
(2, 'Product B', 55.0, '2022-03-15', TRUE),
(3, 'Product C', 105.0, '2022-04-01', FALSE);  -- Price potentially out of range

-- Inserting business constraints
-- Constraint for price range
INSERT INTO BusinessConstraints (key, value_1, value_2, note) VALUES
('PriceRange', 10, 100, 'Valid price range for products');

-- Constraint for days on shelf before going on sale
INSERT INTO BusinessConstraints (key, value_1, value_2, note) VALUES
('DaysBeforeSale', 90, NULL, 'Days on shelf before a product goes on sale');

-- Validating Data Against Constraints

-- Check for products priced outside the known range
SELECT p.productId, p.productName, p.price
FROM ProductData p
JOIN BusinessConstraints bc ON bc.key = 'PriceRange'
WHERE p.price < bc.value_1 OR p.price > bc.value_2;

-- Check for products that should be on sale but aren't
SELECT p.productId, p.productName, p.dateAdded, p.onSale
FROM ProductData p
JOIN BusinessConstraints bc ON bc.key = 'DaysBeforeSale'
WHERE DATEDIFF(CURRENT_DATE, p.dateAdded) > bc.value_1 AND p.onSale = FALSE;

-- Note: The results from these queries highlight products that do not meet the business constraints.
-- They can be used to flag products for review or automatically update product statuses.
```

### Checking Nulls

```sql
-- Creating a table for customer data
-- The table includes fields for customer ID, name, email, address, and phone number
CREATE TABLE CustomerDataToValidate (
    customerId INT,
    customerName VARCHAR,
    email VARCHAR,
    address VARCHAR,
    phoneNumber VARCHAR
);

-- Inserting sample customer data into the table
-- Some records intentionally include NULL values to simulate missing data
INSERT INTO CustomerDataToValidate (customerId, customerName, email, address, phoneNumber) VALUES
(1, 'John Doe', 'john.doe@example.com', '123 Main St', NULL),  -- Missing phone number
(2, 'Jane Smith', NULL, '456 Elm St', '555-0123'),             -- Missing email
(3, 'Alice Johnson', 'alice.j@example.com', NULL, '555-0456'), -- Missing address
(4, 'Bob Brown', 'bob.b@example.com', '789 Oak St', '555-0789');

-- Checking for customers with missing critical information
-- This helps in identifying data quality issues

-- Identifying customers with missing email addresses
SELECT customerId, customerName
FROM CustomerDataToValidate
WHERE email IS NULL;

-- Identifying customers with missing phone numbers
SELECT customerId, customerName
FROM CustomerDataToValidate
WHERE phoneNumber IS NULL;

-- Identifying customers with missing addresses
SELECT customerId, customerName
FROM CustomerDataToValidate
WHERE address IS NULL;

-- Remediation of Null Values
-- Setting default values for missing critical information
-- This is a temporary measure to ensure data integrity until actual data can be collected

-- Setting a default email for customers with missing emails
UPDATE CustomerDataToValidate
SET email = 'default@example.com'
WHERE email IS NULL;

-- Setting a default phone number for customers with missing phone numbers
UPDATE CustomerDataToValidate
SET phoneNumber = '000-0000'
WHERE phoneNumber IS NULL;

-- Setting a default address for customers with missing addresses
UPDATE CustomerDataToValidate
SET address = 'Unknown Address'
WHERE address IS NULL;

-- QUERY UPDATED TABLE
SELECT * FROM CustomerDataToValidate;

-- Note: These updates provide a temporary solution for missing data.
-- In a real-world application, you would likely follow up to collect the actual information from the customers.
```

#### Decision Tables

```sql
-- Creating a table to hold product performance data
CREATE TABLE ProductPerformance (
    ProductID INT,
    SalesVolume VARCHAR, -- 'High', 'Medium', 'Low', 'Very Low'
    CustomerFeedback INT, -- Percentage of positive feedback
    Trending VARCHAR, -- 'Yes' or 'No'
    LifecycleStage VARCHAR -- 'Introduction', 'Growth', 'Maturity', 'Decline'
);

-- Inserting sample data into the ProductPerformance table
INSERT INTO ProductPerformance (ProductID, SalesVolume, CustomerFeedback, Trending, LifecycleStage) VALUES
(1, 'High', 90, 'Yes', 'Growth'),
(2, 'Low', 40, 'No', 'Decline'),
(3, 'Medium', 60, 'No', 'Maturity'),
(4, 'Very Low', 85, 'Yes', 'Introduction'),
(5, 'Low', 45, 'No', 'Decline');

-- Query to determine which products should be discontinued
-- The decision is based on a combination of sales volume, customer feedback, market trend, and product lifecycle stage
SELECT 
    ProductID,
    CASE 
        WHEN SalesVolume = 'Low' AND CustomerFeedback < 50 AND Trending = 'No' AND LifecycleStage = 'Decline' THEN 'Yes'
        WHEN SalesVolume = 'Very Low' THEN 'Yes'
        ELSE 'No'
    END AS Discontinue
FROM 
    ProductPerformance;

-- Explanation:
-- The query analyzes each product's performance based on sales volume, customer feedback, current market trends, and its lifecycle stage.
-- Products with low sales volume, low customer feedback, not trending in the market, and in the decline stage are marked for discontinuation.
-- Additionally, products with very low sales volume are also considered for discontinuation regardless of other factors.
-- The output provides a list of product IDs along with a decision on whether they should be discontinued ('Yes') or not ('No').
```

#### SCD2

```sql
-- Creating a table for customer data with SCD Type 2 implementation
CREATE TABLE IF NOT EXISTS CustomerSCD2 (
    customerId INT,       -- Surrogate key
    customerBusinessId INT,           -- Business key
    customerName VARCHAR,
    email VARCHAR,
    effectiveDate DATE,               -- Date when the record becomes effective
    expirationDate DATE,              -- Date when the record expires
    isCurrent BOOLEAN                 -- Flag to indicate if the record is current
);

-- Inserting initial customer data
INSERT INTO CustomerSCD2 (customerId, customerBusinessId, customerName, email, effectiveDate, expirationDate, isCurrent)
VALUES (1, 1, 'John Doe', 'john.doe@example.com', '2021-01-01', '9999-12-31', TRUE);

-- Simulating an update in customer's email
-- Instead of updating the record directly, insert a new record and update the old record's expiration date and isCurrent flag

-- Step 1: Update existing record - Set expirationDate to yesterday and isCurrent to false
UPDATE CustomerSCD2
SET expirationDate = CURRENT_DATE - INTERVAL '1' DAY, isCurrent = FALSE
WHERE customerBusinessId = 1 AND isCurrent = TRUE;

-- Step 2: Insert new record with updated information (e.g., changed email)
INSERT INTO CustomerSCD2 (customerId, customerBusinessId, customerName, email, effectiveDate, expirationDate, isCurrent)
VALUES (2, 1, 'John Doe', 'john.newemail@example.com', CURRENT_DATE, '9999-12-31', TRUE);

-- Query to See Result

SELECT * FROM CustomerSCD2 WHERE customerBusinessId = 1;

-- Explanation:
-- The CustomerSCD2 table includes a surrogate key (customerId), a natural/business key (customerBusinessId), and other customer attributes.
-- SCD Type 2 is implemented by inserting a new record with updated information while keeping the historical record with updated expiration date and isCurrent flag.
-- This allows for maintaining a full history of changes for each customer.
```

#### SCD3

```sql
-- Creating a table for product data with SCD Type 3 implementation
-- This table includes a product ID, current price, previous price, and the date when the current price was effective
CREATE TABLE IF NOT EXISTS ProductSCD3 (
    productId INT,
    productName VARCHAR,
    currentPrice FLOAT,
    previousPrice FLOAT,      -- Field to store the previous price
    priceEffectiveDate DATE   -- Date when the current price became effective
);

-- Inserting initial product data
INSERT INTO ProductSCD3 (productId, productName, currentPrice, previousPrice, priceEffectiveDate)
VALUES 
(1, 'Gadget', 29.99, NULL, '2022-01-01'); -- Initial product entry with no previous price

-- Updating the product price and preserving the old price in previousPrice
-- Simulating a price change scenario
-- Assume the price of 'Gadget' is updated from 29.99 to 34.99 on '2022-06-01'

-- Step 1: Update the record with the new price and move the old price to previousPrice
UPDATE ProductSCD3
SET 
    previousPrice = currentPrice,         -- Move the current price to previousPrice
    currentPrice = 34.99,                 -- Set the new current price
    priceEffectiveDate = '2022-06-01'     -- Update the effective date of the new price
WHERE 
    productId = 1;

-- Query the Result

SELECT * FROM ProductSCD3;

-- Explanation:
-- The ProductSCD3 table captures the product's current and previous prices.
-- When a product's price changes, the current price is moved to the previousPrice column, and the new price is recorded in currentPrice.
-- The priceEffectiveDate is updated to reflect the date when the new price became effective.
-- This approach allows tracking the history of one attribute (price in this case) while keeping the rest of the record unchanged.

-- Note:
-- SCD Type 3 is not typically used for tracking multiple historical changes as it only retains the current and one previous state.
-- For a more comprehensive historical track, SCD Type 2 is usually preferred.
```

#### SCD4

```sql
-- Creating a main table for current product data
CREATE TABLE ProductDataSCD4 (
    productId INT,
    productName VARCHAR,
    price FLOAT
);

-- Creating a history table for tracking changes in product data
CREATE TABLE ProductDataHistory (
    historyId INT,
    productId INT,
    productName VARCHAR,
    price FLOAT,
    changeDate DATE
);

-- Inserting initial data into the ProductData table
INSERT INTO ProductDataSCD4 (productId, productName, price) VALUES
(1, 'Gadget', 29.99),
(2, 'Widget', 15.99);

-- Simulating an update: the price of 'Gadget' changes from 29.99 to 34.99

-- Step 1: Insert the current state into the history table before updating
-- This captures the historical state of the product before the price change
INSERT INTO ProductDataHistory (historyId, productId, productName, price, changeDate)
SELECT 1, productId, productName, price, CURRENT_DATE
FROM ProductDataSCD4
WHERE productId = 1;

-- Step 2: Update the main ProductData table with the new price
-- This reflects the current state of the product after the price change
UPDATE ProductDataSCD4
SET price = 34.99
WHERE productId = 1;

-- See Results
SELECT * FROM ProductDataHistory;
SELECT * FROM ProductDataSCD4;

-- Explanation:
-- The ProductData table stores the current state of product information.
-- The ProductDataHistory table keeps a record of all historical changes to the products.
-- When a product's information (like price) changes, the old state is first saved into the ProductDataHistory table.
-- Then, the ProductData table is updated with the new information.
-- This approach allows for tracking the entire history of changes for each product while maintaining current data separately.
```

## Semantic Layer Example

```sql
-- Creating bronze tables for local tax data
-- Bronze Table 1: Individual Tax Records
CREATE TABLE arctic."Tax Collections".bronze.individual_tax (
    taxpayer_id INT,
    full_name VARCHAR,
    income FLOAT,
    tax_paid FLOAT
);

-- Bronze Table 2: Business Tax Records
CREATE TABLE arctic."Tax Collections".bronze.business_tax (
    business_id INT,
    business_name VARCHAR,
    revenue FLOAT,
    tax_paid FLOAT
);

-- Inserting flawed data into bronze tables
-- Inserting data into Individual Tax Records
INSERT INTO arctic."Tax Collections".bronze.individual_tax (taxpayer_id, full_name, income, tax_paid) VALUES
(1, 'John Doe', 50000, 5000),
(2, 'Jane Smith', NULL, 4500), -- Missing income
(3, 'Alice Johnson', 70000, -700); -- Negative tax paid (flawed)

-- Inserting data into Business Tax Records
INSERT INTO arctic."Tax Collections".bronze.business_tax (business_id, business_name, revenue, tax_paid) VALUES
(101, 'ABC Corp', 200000, 20000),
(102, 'XYZ Inc', NULL, 18000), -- Missing revenue
(103, 'Acme LLC', 150000, -1500); -- Negative tax paid (flawed)

-- Creating silver views to clean up the data
-- Silver View 1: Cleaned Individual Tax Records
CREATE VIEW arctic."Tax Collections".silver.individual_tax AS
SELECT
    taxpayer_id,
    full_name,
    COALESCE(income, 0) AS income, -- Replacing NULL income with 0
    GREATEST(tax_paid, 0) AS tax_paid -- Correcting negative tax_paid
FROM arctic."Tax Collections".bronze.individual_tax;

-- Silver View 2: Cleaned Business Tax Records
CREATE VIEW arctic."Tax Collections".silver.business_tax AS
SELECT
    business_id,
    business_name,
    COALESCE(revenue, 0) AS revenue, -- Replacing NULL revenue with 0
    GREATEST(tax_paid, 0) AS tax_paid -- Correcting negative tax_paid
FROM arctic."Tax Collections".bronze.business_tax;

-- Creating a gold view: Consolidated Tax Records
CREATE VIEW arctic."Tax Collections".gold.tax_records AS
SELECT
    'Individual' AS taxpayer_type,
    taxpayer_id AS id,
    full_name AS name,
    income,
    tax_paid
FROM arctic."Tax Collections".silver.individual_tax
UNION ALL
SELECT
    'Business' AS taxpayer_type,
    business_id AS id,
    business_name AS name,
    revenue AS income,
    tax_paid
FROM arctic."Tax Collections".silver.business_tax;
```
