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