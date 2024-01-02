## Dremio Cloud SQL Function Examples

#### ST_FROMGEOHASH

```sql
-- Step 1: Creating a table to store geohash data
CREATE TABLE IF NOT EXISTS GeohashData (
    id INT,
    geohash VARCHAR
);

-- Step 2: Populating the table with 10 geohash records
INSERT INTO GeohashData (id, geohash) VALUES
(1, 'ezs42'),
(2, '9q9j8ue2v71y5zzy0s4q'),
(3, 'u4pruydqqvj'),
(4, '7zzzzzzzzzz'),
(5, 's0000000000'),
(6, 'dpz83d7p3dg'),
(7, 'xn774c06kdt'),
(8, 'gbsuv7ztqz0'),
(9, 'ezzzzzzzzzz'),
(10, 's9s0p38zmps');

-- Step 3: Running queries using the ST_FROMGEOHASH function

-- Example: Retrieving the latitude and longitude of the first geohash
SELECT id, ST_FROMGEOHASH(geohash) AS coordinates
FROM GeohashData
WHERE id = 1;
-- Result: { 'Latitude': 42.60498046875, 'Longitude': -5.60302734375 }

-- Example: Retrieving only the latitude of the second geohash
SELECT id, ST_FROMGEOHASH(geohash)['latitude'] AS latitude
FROM GeohashData
WHERE id = 2;
-- Result: 37.554162000000034

-- Example: Retrieving only the longitude of the third geohash
SELECT id, ST_FROMGEOHASH(geohash)['longitude'] AS longitude
FROM GeohashData
WHERE id = 3;
-- Result: [longitude value for 'u4pruydqqvj']

-- Story:
-- Suppose you have a dataset of geohash strings representing locations of various landmarks.
-- By using the ST_FROMGEOHASH function, you can convert these geohashes into readable latitude and longitude coordinates.
-- This is particularly useful for mapping applications or for performing spatial analysis on geolocation data.
```

#### CONVERT_TIMEZONE

```sql
-- Step 1: Creating a table to store events with timestamps
CREATE TABLE IF NOT EXISTS EventLog (
    id INT,
    eventName VARCHAR,
    eventTime TIMESTAMP
);

-- Step 2: Inserting records into the table
-- Imagine we are logging events that occurred in different time zones
INSERT INTO EventLog (id, eventName, eventTime) VALUES
(1, 'Conference Call', '2021-04-01 15:27:32'), -- This event was logged in Los Angeles time
(2, 'Webinar', '2021-04-01 09:00:00'),          -- This event was logged in New York time
(3, 'Team Meeting', '2021-04-01 14:00:00');     -- This event was logged in London time

-- Step 3: Using the CONVERT_TIMEZONE function to standardize event times to UTC
-- The goal is to have a consistent time zone (UTC) for all events in our system for easy comparison and analysis.

-- Convert Los Angeles time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('America/Los_Angeles', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 1;
-- Result: '2021-04-01 22:27:32' (UTC time for Conference Call)

-- Convert New York time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('America/New_York', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 2;
-- Result: '2021-04-01 14:00:00' (UTC time for Webinar)

-- Convert London time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('Europe/London', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 3;
-- Result: '2021-04-01 14:00:00' (UTC time for Team Meeting)

-- Story Context:
-- In a global company, events like conference calls, webinars, and meetings are scheduled in various time zones.
-- To streamline the process of scheduling and analyzing these events, it's helpful to convert all event times to a single standard time zone, such as UTC.
-- The CONVERT_TIMEZONE function in DremioSQL makes this process easy by allowing us to convert times from local time zones to UTC.
-- By storing all event times in UTC, we can more easily coordinate between teams in different time zones and perform time-based analysis on our events.
```

#### ARRAY_TO_STRING

```sql
-- Step 1: Creating a table with an array column
CREATE TABLE IF NOT EXISTS ProductInventory (
    id INT,
    productName VARCHAR,
    sizesAvailable ARRAY(VARCHAR)
);

-- Step 2: Inserting records into the table
-- Imagine we are a clothing store, and we want to list the sizes available for each product
INSERT INTO ProductInventory (id, productName, sizesAvailable) VALUES
(1, 'T-Shirt', ARRAY['S', 'M', 'L', 'XL']),
(2, 'Jeans', ARRAY['32', '34', '36']);

-- Step 3: Using the ARRAY_TO_STRING function to create a readable list of sizes
-- Our goal is to display the sizes in a human-readable format on our website.

-- Convert the array of sizes for T-Shirts into a comma-separated string
SELECT id, productName,
       ARRAY_TO_STRING(sizesAvailable, ',') AS sizesList
FROM ProductInventory
WHERE id = 1;
-- Result: 'S,M,L,XL'

-- Convert the array of sizes for Jeans into a comma-separated string
SELECT id, productName,
       ARRAY_TO_STRING(sizesAvailable, ', ') AS sizesList
FROM ProductInventory
WHERE id = 2;
-- Result: '32, 34, 36'

-- Story Context:
-- As an online clothing retailer, we maintain an inventory system where the available sizes for each product are stored as arrays.
-- To display this information on our website, we need to convert these arrays into a user-friendly, comma-separated string format.
-- The ARRAY_TO_STRING function in DremioSQL helps us transform these arrays into a format that's easy for customers to read and understand.
```

#### CURRENT_DATE_UTC & DATEDIFF

```sql
-- Step 1: Creating a table for events with timestamps and timezones
CREATE TABLE IF NOT EXISTS EventSchedule (
    id INT,
    eventName VARCHAR,
    eventTime TIMESTAMP,
    timezone VARCHAR
);

-- Step 2: Inserting records into the table
-- Imagine we are organizing events in different global locations
INSERT INTO EventSchedule (id, eventName, eventTime, timezone) VALUES
(1, 'Global Webinar', '2021-07-01 10:00:00', 'America/New_York'), -- Event in New York
(2, 'Team Meeting', '2021-07-01 16:00:00', 'Europe/London'),      -- Event in London
(3, 'Product Launch', '2021-07-01 20:00:00', 'Asia/Tokyo');       -- Event in Tokyo

-- Step 3: Running a query to convert event timestamps to UTC
-- Additionally, we include the current UTC date and the difference in days between the event date and the current UTC date.
SELECT id, eventName,
       CONVERT_TIMEZONE('America/New_York', 'UTC', eventTime) AS eventTimeUTC,
       CURRENT_DATE_UTC() AS currentDateUTC,
       DATEDIFF(currentDateUTC, eventTimeUTC) AS daysSinceEvent
FROM EventSchedule WHERE timezone = 'America/New_York';

-- In this query:
-- CONVERT_TIMEZONE is used to standardize event times to UTC.
-- CURRENT_DATE_UTC() gives us the current date in UTC.
-- DATEDIFF calculates the difference in days between the current UTC date and the event date in UTC.

-- Story Context:
-- In an international organization, coordinating events across multiple time zones is a common challenge.
-- By converting all event times to UTC, we can compare and manage these events more effectively.
-- The CURRENT_DATE_UTC function helps us understand the current date in a standard time zone, regardless of where our team members are located.
-- The daysUntilEvent column is particularly useful for planning and logistics, as it tells us how many days are left until each event.
```

#### REGEXP_COL_LIKE

```sql
-- Step 1: Creating a table for music albums with a field for genres
CREATE TABLE IF NOT EXISTS MusicAlbums (
    id INT,
    albumName VARCHAR,
    genres VARCHAR
);

-- Step 2: Inserting records into the table
-- Albums can have multiple genres, listed in a single string
INSERT INTO MusicAlbums (id, albumName, genres) VALUES
(1, 'The Dark Side of the Moon', 'Rock, Progressive Rock, Psychedelic Rock'),
(2, 'Thriller', 'Pop, Rock, R&B'),
(3, 'Back in Black', 'Hard Rock, Rock'),
(4, 'The Bodyguard', 'R&B, Soul, Pop'),
(5, 'Rumours', 'Rock, Pop');

-- Step 3: Using the REGEXP_COL_LIKE function to find albums in a specific genre
-- For example, we want to find all albums that are classified under 'Rock'

SELECT id, albumName, genres
FROM MusicAlbums
WHERE REGEXP_COL_LIKE(genres, 'Rock');

-- Story Context:
-- In a music database, albums can often fall under multiple genres, making it challenging to categorize them strictly.
-- By storing the genres as a single string and using REGEXP_COL_LIKE, we can flexibly search for albums that include a specific genre in their classification.
-- This approach is particularly useful for music enthusiasts and analysts who are interested in exploring albums across overlapping genres.
-- For example, finding all albums that include 'Rock' in their genres helps identify a wide range of albums that touch upon the Rock genre, regardless of their primary classification.
```


#### RANK & DENSE_RANK

```sql
-- Step 1: Creating a table to store sales records
CREATE TABLE SalesData (
    id INT,
    salesPerson VARCHAR,
    totalSales INT
);

-- Step 2: Inserting records into the table
INSERT INTO SalesData (id, salesPerson, totalSales) VALUES
(1, 'Alice', 300),
(2, 'Bob', 300),
(3, 'Charlie', 200),
(4, 'Diana', 400),
(5, 'Eve', 200),
(6, 'Frank', 500);

-- Step 3: Using the RANK function
-- The RANK function assigns a unique rank to each row within a partition of a result set, with gaps in rank values for ties
SELECT salesPerson, totalSales,
       RANK() OVER (ORDER BY totalSales DESC) AS salesRank
FROM SalesData;

-- In this query, sales persons are ranked based on their total sales.
-- If two sales persons have the same total sales, they will receive the same rank, and the next rank will be incremented by the number of tied ranks.

-- Step 4: Using the DENSE_RANK function
-- DENSE_RANK also assigns a unique rank to each row within a partition, but without gaps in rank values for ties
SELECT salesPerson, totalSales,
       DENSE_RANK() OVER (ORDER BY totalSales DESC) AS denseSalesRank
FROM SalesData;

-- Similar to RANK, sales persons are ranked based on their total sales.
-- However, with DENSE_RANK, if there are ties, the next rank after the tie will be the immediately following integer, without skipping any ranks.

-- Example Context:
-- In a company's sales department, we might use these functions to rank salespersons based on their performance.
-- The RANK function is useful when we want to emphasize the performance gap between salespersons.
-- On the other hand, DENSE_RANK is beneficial when we want to avoid gaps in ranking, making it more suitable for recognizing each individual's relative performance without penalizing for ties.
```

#### NTILE

```sql
-- Step 1: Creating a table to store student grades
CREATE TABLE IF NOT EXISTS StudentGrades (
    id INT,
    studentName VARCHAR,
    grade INT
);

-- Step 2: Inserting records into the table
-- Let's say we have a list of student grades for a particular exam
INSERT INTO StudentGrades (id, studentName, grade) VALUES
(1, 'Alice', 85),
(2, 'Bob', 92),
(3, 'Charlie', 78),
(4, 'Diana', 88),
(5, 'Eve', 95),
(6, 'Frank', 70),
(7, 'Grace', 82),
(8, 'Hank', 90),
(9, 'Ivy', 75),
(10, 'John', 80);

-- Step 3: Using the NTILE function to find percentile ranks
-- We want to divide students into 4 quartiles based on their grades
SELECT studentName, grade,
       NTILE(4) OVER (ORDER BY grade DESC) AS percentileRank
FROM StudentGrades;

-- In this query:
-- NTILE(4) divides the students into 4 groups (quartiles) based on their grades.
-- Each student is assigned a percentile rank from 1 to 4, where 1 represents the top 25%, 2 represents the 50-75% range, etc.
-- This ranking helps in understanding where each student stands in terms of academic performance compared to their peers.

-- Example Context:
-- In an educational setting, understanding student performance in percentile ranks can be useful for various purposes.
-- It can help teachers identify students who are excelling (top 25%) or those who may need additional support (bottom 25%).
-- Such an analysis is also beneficial for academic counselors when providing personalized advice and resources to students.
-- Percentile ranks provide a relative measure of performance, which can be more informative than just looking at raw scores or grades.
```

#### FIRST_VALUE

```sql
-- Step 1: Creating a table to store sales records for different departments
CREATE TABLE IF NOT EXISTS SalesRecords (
    id INT,
    salesPerson VARCHAR,
    department VARCHAR,
    totalSales INT
);

-- Step 2: Inserting records into the table
-- Imagine we are a company with salespeople in various departments
INSERT INTO SalesRecords (id, salesPerson, department, totalSales) VALUES
(1, 'Alice', 'Electronics', 12000),
(2, 'Bob', 'Electronics', 15000),
(3, 'Charlie', 'Home Appliances', 8000),
(4, 'Diana', 'Electronics', 20000),
(5, 'Eve', 'Home Appliances', 11000),
(6, 'Frank', 'Furniture', 9000),
(7, 'Grace', 'Furniture', 13000),
(8, 'Hank', 'Electronics', 17000);

-- Step 3: Using the FIRST_VALUE function
-- We will order salespeople by total sales within their department and compare each person's sales to the top performer in that department.
SELECT salesPerson, department, totalSales,
       FIRST_VALUE(totalSales) OVER (PARTITION BY department ORDER BY totalSales DESC) AS topSalesInDept,
       totalSales - topSalesInDept AS diffFromTopPerformer
FROM SalesRecords;

-- In this query:
-- The FIRST_VALUE function finds the highest sales figure (top performer) in each department.
-- 'topSalesInDept' shows the sales figure of the top performer in the same department.
-- 'diffFromTopPerformer' shows the difference in sales between the salesperson and the top performer in their department.

-- Example Context:
-- For a sales manager, this analysis is useful for identifying the gap between top performers and other team members in each department.
-- This can help in setting targets, understanding training needs, and recognizing top performers.
-- It also allows for tailored strategies for each department based on their specific performance dynamics.
```

#### LAG

```sql
-- Step 1: Creating a table to store monthly sales data
CREATE TABLE IF NOT EXISTS MonthlySales (
    id INT,
    salesMonth DATE,
    totalSales INT
);

-- Step 2: Inserting records into the table
-- These records represent the total sales for each month in a given year
INSERT INTO MonthlySales (id, salesMonth, totalSales) VALUES
(1, '2021-01-01', 10000),
(2, '2021-02-01', 12000),
(3, '2021-03-01', 11000),
(4, '2021-04-01', 13000),
(5, '2021-05-01', 12500),
(6, '2021-06-01', 14000),
(7, '2021-07-01', 13500),
(8, '2021-08-01', 15000),
(9, '2021-09-01', 15500),
(10, '2021-10-01', 16000),
(11, '2021-11-01', 15000),
(12, '2021-12-01', 17000);

-- Step 3: Using the LAG function to compare current month's sales with the previous month
SELECT salesMonth, totalSales,
       LAG(totalSales, 1) OVER (ORDER BY salesMonth) AS previousMonthSales,
       totalSales - previousMonthSales AS salesDifference
FROM MonthlySales;

-- In this query:
-- LAG(totalSales, 1) retrieves the sales figure from the previous month.
-- 'previousMonthSales' shows the sales figure of the previous month.
-- 'salesDifference' shows the difference in sales between the current month and the previous month.

-- Example Context:
-- In a business setting, analyzing month-over-month sales data is crucial for understanding sales trends and making informed decisions.
-- By comparing current month's sales with the previous month, the company can identify growth trends, seasonal variations, or any unexpected changes.
-- This analysis can inform marketing strategies, inventory management, and financial planning.
-- The LAG function provides an efficient way to perform this comparison without complex joins or subqueries.
```

#### COVAR_POP & COVAR_SAMP

```sql
-- Step 1: Creating a table to store monthly marketing spend and sales data
CREATE TABLE IF NOT EXISTS MarketingSalesData (
    id INT,
    sales_month DATE,
    marketingSpend INT, -- The amount spent on marketing in a given month
    salesRevenue INT    -- The total sales revenue generated in that month
);

-- Step 2: Inserting records into the table
-- These records represent the marketing spend and sales revenue for each month in a given year
INSERT INTO MarketingSalesData (id, sales_month, marketingSpend, salesRevenue) VALUES
(1, '2021-01-01', 20000, 80000),
(2, '2021-02-01', 22000, 85000),
(3, '2021-03-01', 25000, 90000),
(4, '2021-04-01', 24000, 95000),
(5, '2021-05-01', 30000, 100000),
(6, '2021-06-01', 28000, 110000),
(7, '2021-07-01', 32000, 115000),
(8, '2021-08-01', 31000, 120000),
(9, '2021-09-01', 33000, 125000),
(10, '2021-10-01', 34000, 130000),
(11, '2021-11-01', 35000, 135000),
(12, '2021-12-01', 36000, 140000);

-- Step 3: Using COVAR_POP to find the population covariance between marketing spend and sales revenue
SELECT COVAR_POP(marketingSpend, salesRevenue) AS covarPop
FROM MarketingSalesData;

-- Step 4: Using COVAR_SAMP to find the sample covariance between marketing spend and sales revenue
SELECT COVAR_SAMP(marketingSpend, salesRevenue) AS covarSamp
FROM MarketingSalesData;

-- Example Context:
-- In a business setting, understanding the relationship between marketing spend and sales revenue is crucial.
-- A positive covariance indicates that higher marketing spend is generally associated with higher sales revenue.
-- The COVAR_POP function calculates the population covariance, assuming the dataset represents the entire population.
-- The COVAR_SAMP function calculates the sample covariance, which is more appropriate if the dataset is a sample of a larger population.
-- This analysis can inform the effectiveness of marketing strategies and guide budget allocation decisions.
```

#### PERCENTILE_CONT & PERCENTILE_DISC

```sql
-- Step 1: Creating a table to store employee salary data
CREATE TABLE EmployeeSalaries (
    id INT,
    employeeName VARCHAR,
    salary INT
);

-- Step 2: Inserting records into the table
-- These records represent the salaries of various employees in the company
INSERT INTO EmployeeSalaries (id, employeeName, salary) VALUES
(1, 'Alice', 70000),
(2, 'Bob', 80000),
(3, 'Charlie', 55000),
(4, 'Diana', 75000),
(5, 'Eve', 90000),
(6, 'Frank', 60000),
(7, 'Grace', 85000),
(8, 'Hank', 65000),
(9, 'Ivy', 72000),
(10, 'John', 62000);

-- Step 3: Using PERCENTILE_CONT to find the continuous percentile of salaries
-- Let's find the 60th percentile of salaries in an ascending order
SELECT PERCENTILE_CONT(0.6) WITHIN GROUP (ORDER BY salary ASC) AS percentileCont
FROM EmployeeSalaries;

-- The PERCENTILE_CONT function computes the 60th percentile based on a continuous distribution.
-- The result is interpolated and represents the salary value at the 60th percentile of the salary distribution.

-- Step 4: Using PERCENTILE_DISC to find the discrete percentile of salaries
-- Now, let's find the 60th percentile of salaries in a descending order
SELECT PERCENTILE_DISC(0.6) WITHIN GROUP (ORDER BY salary DESC) AS percentileDisc
FROM EmployeeSalaries;

-- The PERCENTILE_DISC function computes the 60th percentile in a discrete manner.
-- The result is the actual salary value from the dataset that corresponds to the 60th percentile.

-- Example Context:
-- In HR or finance departments, understanding salary distributions is crucial for budgeting and compensation planning.
-- PERCENTILE_CONT provides a way to estimate where a certain percentage of employees fall in terms of salary.
-- PERCENTILE_DISC, on the other hand, gives the actual salary value that separates the top 40% of earners from the rest.
-- These insights can be used to evaluate compensation fairness, plan salary increments, or compare salary distributions across different departments.
```

#### XOR

```sql
-- Step 1: Creating a table to store user feature usage data
CREATE TABLE IF NOT EXISTS FeatureUsage (
    id INT,
    userID INT,
    featuresEnabled INT -- Each bit in this integer represents whether a certain feature is enabled (1) or not (0)
);

-- Step 2: Inserting records into the table
-- These records represent different combinations of enabled features for users
INSERT INTO FeatureUsage (id, userID, featuresEnabled) VALUES
(1, 101, 5),  -- Binary 0101: Features 1 and 3 are enabled
(2, 102, 3),  -- Binary 0011: Features 1 and 2 are enabled
(3, 103, 6),  -- Binary 0110: Features 2 and 3 are enabled
(4, 104, 4);  -- Binary 0100: Only Feature 3 is enabled

-- Step 3: Using XOR to compare feature configurations between two users
-- XOR will return a binary number where bits are set if the features are differently enabled between the two users
SELECT userID, 
       XOR(featuresEnabled, (SELECT featuresEnabled FROM FeatureUsage WHERE userID = 101)) AS diffFeatures
FROM FeatureUsage
WHERE userID != 101;

-- This query compares each user's feature configuration with that of user 101
-- A '1' in the result indicates a feature that is differently enabled between user 101 and the other user.


-- Example Context:
-- In a software company, understanding how different features are used across the user base is crucial for development and support.
-- The XOR function can be used to compare feature usage between users, identifying differences in configuration.
-- These insights are valuable for product managers and developers to prioritize features and understand user preferences.
```

#### LOG

```sql
-- Step 1: Creating a table to store financial data
CREATE TABLE IF NOT EXISTS InvestmentData (
    id INT,
    "year" INT,
    investmentValue FLOAT
);

-- Step 2: Inserting records into the table
-- These records represent the value of an investment over a period of years
INSERT INTO InvestmentData (id, "year", investmentValue) VALUES
(1, 2010, 10000.0),
(2, 2011, 10500.0),
(3, 2012, 11025.0),
(4, 2013, 11576.25),
(5, 2014, 12155.06),
(6, 2015, 12762.82);

-- Step 3: Using the LOG function to analyze the rate of growth
-- Calculating the natural logarithm of the investment value to understand its growth rate
SELECT "year", investmentValue,
       LOG(investmentValue) AS lnInvestmentValue
FROM InvestmentData;

-- In this query, LOG(investmentValue) computes the natural logarithm (base e) of the investment value.
-- The natural logarithm helps in understanding the exponential growth rate of the investment.

-- Step 4: Using LOG with a specified base
-- Calculating the logarithm of the investment value with base 2 to compare the doubling rate
SELECT "year", investmentValue,
       LOG(2, investmentValue) AS log2InvestmentValue
FROM InvestmentData;

-- LOG(2, investmentValue) computes the logarithm of the investment value with base 2.
-- This can be useful for understanding how many times the investment value has doubled since the initial year.

-- Example Context:
-- In finance, logarithmic calculations are often used to understand growth patterns.
-- Natural logarithms (base e) are particularly useful for continuous growth rates like compound interest.
-- Logarithms with base 2 can be insightful for quickly assessing doubling periods in investments.
-- These insights are valuable for financial analysts, investors, and portfolio managers for making informed investment decisions.
```

#### CASE

```sql
-- Step 1: Creating a table to store customer data
CREATE TABLE IF NOT EXISTS CustomerData (
    id INT,
    customerName VARCHAR,
    purchaseAmount FLOAT,
    preferredCategory VARCHAR
);

-- Step 2: Inserting records into the table
-- These records represent various customers and their purchasing preferences
INSERT INTO CustomerData (id, customerName, purchaseAmount, preferredCategory) VALUES
(1, 'Alice', 300.0, 'Electronics'),
(2, 'Bob', 150.0, 'Clothing'),
(3, 'Charlie', 500.0, 'Groceries'),
(4, 'Diana', 250.0, 'Books'),
(5, 'Eve', 800.0, 'Electronics');

-- Step 3: Using the CASE statement to categorize customers based on their preferred category
SELECT customerName, preferredCategory,
       CASE preferredCategory
           WHEN 'Electronics' THEN 'Tech Enthusiast'
           WHEN 'Clothing' THEN 'Fashionista'
           WHEN 'Groceries' THEN 'Home Maker'
           WHEN 'Books' THEN 'Bookworm'
           ELSE 'General Shopper'
       END AS customerType
FROM CustomerData;

-- In this query:
-- The CASE statement categorizes each customer based on their preferred shopping category.
-- 'Tech Enthusiast', 'Fashionista', 'Home Maker', and 'Bookworm' are categories derived from the preferredCategory field.
-- Customers whose preferred category doesn't match any of the specified ones are categorized as 'General Shopper'.

-- Example Context:
-- In retail, understanding customer preferences is crucial for personalized marketing and product recommendations.
-- The CASE statement allows for a dynamic classification of customers, which can be used in targeted marketing campaigns.
-- It also helps in understanding the customer base and tailoring the store's inventory and promotions accordingly.
```

#### DATE_ADD & DATE_SUB

```sql
-- Step 1: Creating a table to store project deadlines
CREATE TABLE ProjectDeadlines (
    id INT,
    projectName VARCHAR,
    startDate DATE,
    durationDays INT -- Duration of the project in days
);

-- Step 2: Inserting records into the table
-- These records represent different projects and their start dates and durations
INSERT INTO ProjectDeadlines (id, projectName, startDate, durationDays) VALUES
(1, 'Alpha', '2022-01-01', 30),
(2, 'Beta', '2022-02-15', 45),
(3, 'Gamma', '2022-03-10', 60);

-- Step 3: Using DATE_ADD to calculate the end dates of the projects
SELECT id, projectName, startDate, durationDays,
       DATE_ADD(startDate, durationDays) AS endDate
FROM ProjectDeadlines;

-- This query calculates the end date of each project by adding the duration (in days) to the start date.

-- Step 4: Using DATE_SUB to calculate the buffer period before the start dates
-- Assuming a 5-day buffer period before each project starts for preparation
SELECT id, projectName, startDate,
       DATE_SUB(startDate, 5) AS bufferStartDate
FROM ProjectDeadlines;

-- This query calculates a buffer start date for each project by subtracting 5 days from the actual start date.

-- Example Context:
-- In project management, it's crucial to have clear timelines for start and end dates, as well as buffer periods for preparation.
-- The DATE_ADD function is used to determine project completion dates based on their durations.
-- The DATE_SUB function helps to calculate buffer periods, ensuring that teams have enough time to prepare before the actual start of the project.
-- These functions facilitate effective planning and help in setting realistic expectations for project timelines.
```

#### DATE_ADD & DATE_SUB

```sql
-- Step 1: Creating a table to store sales data
CREATE TABLE IF NOT EXISTS SalesData (
    id INT,
    salesPerson VARCHAR,
    totalSales INT
);

-- Step 2: Inserting records into the table
-- These records represent the total sales made by different salespersons
INSERT INTO SalesData (id, salesPerson, totalSales) VALUES
(1, 'Alice', 5000),
(2, 'Bob', 7000),
(3, 'Charlie', 5500),
(4, 'Diana', 8000),
(5, 'Eve', 6000);

-- Step 3: Using STDDEV to calculate the standard deviation of total sales
SELECT STDDEV(totalSales) AS stddevSales
FROM SalesData;

-- This query calculates the standard deviation of total sales among the salespersons.
-- A higher standard deviation indicates a larger variation in sales performance.

-- Step 4: Using STDDEV_POP to calculate the population standard deviation of total sales
SELECT STDDEV_POP(totalSales) AS stddevPopSales
FROM SalesData;

-- STDDEV_POP calculates the population standard deviation, which is used when your data set represents the entire population.

-- Step 5: Using STDDEV_SAMP to calculate the sample standard deviation of total sales
SELECT STDDEV_SAMP(totalSales) AS stddevSampSales
FROM SalesData;

-- STDDEV_SAMP calculates the sample standard deviation, which is used when your data set is a sample of a larger population.

-- Example Context:
-- In a sales department, understanding the variation in sales performance among salespersons is crucial for identifying training needs and setting targets.
-- The standard deviation helps in understanding how much sales figures vary from the average.
-- A lower standard deviation indicates that the sales figures are closer to the average, suggesting consistency in sales performance.
-- A higher standard deviation suggests more variability, indicating that some salespersons might be performing significantly differently than others.
```

#### IS_SUBSTR

```sql
-- Step 1: Creating a table to store customer data
CREATE TABLE IF NOT EXISTS CustomerInterestsData (
    id INT,
    customerName VARCHAR,
    interests VARCHAR
);

-- Step 2: Inserting records into the table
-- These records represent different customers and their interests
INSERT INTO CustomerInterestsData (id, customerName, interests) VALUES
(1, 'Alice', 'Hiking, Photography, Travel'),
(2, 'Bob', 'Gaming, Programming, Technology'),
(3, 'Charlie', 'Cooking, Baking, Gardening'),
(4, 'Diana', 'Yoga, Meditation, Health'),
(5, 'Eve', 'Technology, Science, Robotics');

-- Step 3: Using IS_SUBSTR to find customers interested in 'Technology'
SELECT customerName, interests,
       IS_SUBSTR(interests, 'Technology') AS isInterestedInTechnology
FROM CustomerInterestsData;

-- This query checks each customer's interests for the keyword 'Technology'.
-- The IS_SUBSTR function returns true if 'Technology' is found in the interests string.

-- Example Context:
-- In a company that sells technology products, it's useful to identify customers with an interest in technology for targeted marketing.
-- The IS_SUBSTR function allows for a simple and efficient way to filter customers based on their interests.
-- Customers who have 'Technology' as part of their interests can be targeted for promotions and product recommendations related to technology.

-- Additional Use Case:
-- This function can also be used in text analysis for content categorization, customer feedback analysis, or even in search functionalities within an application.
```

#### GEO_NEARBY

```sql
-- Step 1: Creating a table to store warehouse and destination coordinates
CREATE TABLE IF NOT EXISTS DeliveryLocations (
    id INT,
    locationName VARCHAR,
    latitude FLOAT,
    longitude FLOAT
);

-- Step 2: Inserting records into the table
-- These records represent the coordinates (latitude and longitude) of the warehouse and several delivery destinations
INSERT INTO DeliveryLocations (id, locationName, latitude, longitude) VALUES
(1, 'Warehouse', 40.7128, -74.0060), -- Coordinates for the warehouse (e.g., New York City)
(2, 'Destination A', 40.730610, -73.935242), -- A nearby location (e.g., within New York City)
(3, 'Destination B', 41.8781, -87.6298), -- A far location (e.g., Chicago)
(4, 'Destination C', 40.748817, -73.985428); -- Another nearby location (e.g., within New York City)

-- Step 3: Using GEO_NEARBY to determine if destinations are within 30,000 meters (30 km) of the warehouse
SELECT locationName,
       GEO_NEARBY(
           (SELECT latitude FROM DeliveryLocations WHERE locationName = 'Warehouse'), 
           (SELECT longitude FROM DeliveryLocations WHERE locationName = 'Warehouse'), 
           latitude, 
           longitude, 
           30000.0
       ) AS isWithinDeliveryRadius
FROM DeliveryLocations
WHERE locationName != 'Warehouse';

-- This query checks if each destination is within a 30 km radius of the warehouse.
-- The GEO_NEARBY function returns true if the distance between the warehouse and a destination is less than or equal to 30,000 meters.

-- Example Context:
-- For a delivery service company, it's important to determine which destinations fall within their standard delivery radius.
-- The GEO_NEARBY function allows for a quick and efficient way to filter destinations based on their distance from a central location, like a warehouse.
-- This functionality is essential for route planning, delivery charge calculations, and service area determination.
```

#### GEO_DISTANCE

```sql
-- Step 1: Creating a table to store coordinates of tourist attractions
CREATE TABLE IF NOT EXISTS TouristAttractions (
    id INT,
    attractionName VARCHAR,
    latitude FLOAT,
    longitude FLOAT
);

-- Step 2: Inserting records into the table
-- These records represent the coordinates (latitude and longitude) of various tourist attractions
INSERT INTO TouristAttractions (id, attractionName, latitude, longitude) VALUES
(1, 'Statue of Liberty', 40.6892, -74.0445), -- New York
(2, 'Eiffel Tower', 48.8584, 2.2945), -- Paris
(3, 'Colosseum', 41.8902, 12.4922), -- Rome
(4, 'Taj Mahal', 27.1751, 78.0421); -- India

-- Step 3: Using GEO_DISTANCE to calculate the distance between two attractions
-- For example, calculate the distance between the Statue of Liberty and the Eiffel Tower
SELECT a.attractionName AS fromAttraction,
       b.attractionName AS toAttraction,
       GEO_DISTANCE(a.latitude, a.longitude, b.latitude, b.longitude) AS distanceInMeters
FROM TouristAttractions a, TouristAttractions b
WHERE a.attractionName = 'Statue of Liberty' AND b.attractionName = 'Eiffel Tower';

-- This query calculates the distance in meters between the Statue of Liberty and the Eiffel Tower.

-- Example Context:
-- For a travel company, calculating distances between major tourist attractions is crucial for planning tours and estimating travel times.
-- The GEO_DISTANCE function provides an accurate and efficient way to measure distances between geographical points.
-- This information is essential for creating tour itineraries, pricing travel packages, and providing tourists with relevant information.

-- Additional Use Case:
-- This function can also be used in logistics for route optimization, in real estate for proximity analysis, or in any industry where geographical distance calculations are necessary.
```

#### LISTAGG

```sql
-- Creating the Books table to store book information
CREATE TABLE IF NOT EXISTS Books (
    id INT,
    title VARCHAR,
    genre VARCHAR
);

-- Inserting sample book records into the Books table
INSERT INTO Books (id, title, genre) VALUES
(1, 'To Kill a Mockingbird', 'Classic'),
(2, '1984', 'Classic'),
(3, 'The Great Gatsby', 'Classic'),
(4, 'Neuromancer', 'Science Fiction'),
(5, 'Dune', 'Science Fiction'),
(6, 'Brave New World', 'Science Fiction'),
(7, 'The Catcher in the Rye', 'Classic');

-- Using LISTAGG to concatenate book titles by genre
-- This query groups books by their genre and concatenates their titles
SELECT genre, LISTAGG(title, '; ') AS titles
FROM Books
GROUP BY genre;

-- Using LISTAGG with DISTINCT to avoid duplicate titles in each genre list
-- This is useful if there are duplicate book entries in the Books table
SELECT genre, LISTAGG(DISTINCT title, '| ') AS distinct_titles
FROM Books
GROUP BY genre;

-- Using LISTAGG with ORDER BY to sort book titles alphabetically within each genre
-- This orders the titles alphabetically before concatenating them
SELECT genre, LISTAGG(title, ', ') WITHIN GROUP (ORDER BY title ASC) AS ordered_titles
FROM Books
GROUP BY genre;

-- The resulting output from these queries will show a string of book titles for each genre
-- 'titles' will have all titles concatenated with '; '
-- 'distinct_titles' will have distinct titles concatenated with '| '
-- 'ordered_titles' will have titles alphabetically ordered and concatenated with ', '
```

#### CORR

```sql
-- Creating a SalesData table to store information about sales and advertising expenses
CREATE TABLE IF NOT EXISTS SalesDataCORR (
    id INT,
    "month" VARCHAR,
    advertisingSpend FLOAT, -- Amount spent on advertising in a given month
    salesRevenue FLOAT       -- Total sales revenue generated in the same month
);

-- Inserting sample records into the SalesData table
-- These records represent the advertising spend and sales revenue for each month
INSERT INTO SalesDataCORR (id, "month", advertisingSpend, salesRevenue) VALUES
(1, 'January', 2000, 5000),
(2, 'February', 3000, 7000),
(3, 'March', 4000, 9000),
(4, 'April', 5000, 12000),
(5, 'May', 6000, 15000);

-- Using the CORR function to calculate the Pearson correlation coefficient
-- This coefficient measures the linear correlation between two variables: advertisingSpend and salesRevenue
SELECT 
    "CORR"(advertisingSpend, salesRevenue) AS correlationCoefficient
FROM 
    SalesDataCORR;

-- Explanation:
-- The CORR function is used here to understand the relationship between advertising spend and sales revenue.
-- The Pearson correlation coefficient (result of CORR) ranges from -1 to 1.
-- A value close to 1 indicates a strong positive correlation (as advertising spend increases, sales revenue tends to increase).
-- A value close to -1 indicates a strong negative correlation (as advertising spend increases, sales revenue tends to decrease).
-- A value close to 0 implies little or no linear relationship between the expenditures on advertising and the sales revenue.
-- This analysis is crucial for the marketing department to understand the effectiveness of their advertising strategies.
```