# Day 2: Database Design, DML & NULL Handling

Today I practiced PostgreSQL in pgAdmin using a customer dataset from my faculty's tutorial. I learned how to create a table, insert records, filter rows, handle missing values, split results into pages, and search text patterns.

I also explored database design through two business examples using fact and dimension tables.

## Day 2 files

| File | Contents |
| --- | --- |
| [Logistics ER project](logistics-er-diagram.md) | Purchasing, receiving, customer orders, shipments, and inventory |
| [E-commerce ER project](ecommerce-marketplace-er-diagram.md) | Orders, payments, deliveries, returns, refunds, and seller stock |

**Tools:** PostgreSQL is the database system. pgAdmin is the application I used to write and run SQL against it.

**Scope:** These notes follow the supplied tutorial code and NULL-handling slides. Extra examples are labeled. UPDATE, DELETE, and detailed HAVING practice are not presented as completed lesson exercises.

## 1. The practice dataset

The `customers` table has 12 records. Each row represents one customer.

| Column | Meaning |
| --- | --- |
| customer_id | Unique customer identifier |
| full_name | Customer's name |
| email | Email address, if recorded |
| country / city | Customer's location |
| age | Age, if known |
| total_orders | Recorded number of orders |
| total_spent | Recorded total spending |
| last_order_date | Last recorded order date |
| status | Active or inactive |
| referral_id | Identifier intended to describe a referring customer |

`referral_id` is an integer column in the supplied schema. It is **not an enforced foreign key**, because the CREATE TABLE statement does not declare a foreign-key constraint.

The tutorial includes deliberate missing values. For example, Maria and Nusrat have no recorded age. Ayesha, Fatima, David, and Priya have no recorded email.

## 2. Creating the table

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    full_name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    city VARCHAR(50),
    age INT,
    total_orders INT,
    total_spent DECIMAL(10,2),
    last_order_date DATE,
    status VARCHAR(20),
    referral_id INT
);
```

- `INT` stores whole numbers.
- `VARCHAR(100)` stores text up to the declared length.
- `DECIMAL(10,2)` allows up to ten digits in total, with two after the decimal point.
- `DATE` stores a calendar date.
- `PRIMARY KEY` makes customer_id unique and prevents it from being NULL.

`CREATE TABLE` is DDL because it defines a structure. `INSERT` is DML because it adds data to that structure.

## 3. Adding records with INSERT

The lesson inserts 12 customers. The examples below use those tutorial records.

Here is one row to understand the syntax:

```sql
INSERT INTO customers (
    customer_id, full_name, email, country, city, age,
    total_orders, total_spent, last_order_date, status, referral_id
)
VALUES (
    1, 'Tanvir Pial', 'tanvir@gmail.com', 'Bangladesh', 'Dhaka', 28,
    12, 8500.00, '2025-01-10', 'active', NULL
);
```

The column list tells PostgreSQL where each value belongs. Listing columns explicitly also makes the statement easier to read.

Text values use single quotes. SQL NULL is written without quotes. `'NULL'` would be a text value, not a missing value.

**Running the practice:** Use a practice database where `customers` does not already exist. Run the setup once. If the tutorial table already exists with the data, run only the query section. Inserting the same IDs again would cause a primary-key conflict.

## 4. Reading the records

```sql
SELECT *
FROM customers
ORDER BY customer_id;
```

`*` requests every column. To make a smaller report, name only the columns you need:

```sql
SELECT customer_id, full_name, city
FROM customers
ORDER BY customer_id;
```

The examples use customer_id ordering so their displayed results can be compared consistently.

## 5. Filtering rows with WHERE

WHERE keeps rows whose condition is true.

### Customers in Dhaka

```sql
SELECT full_name, email, city
FROM customers
WHERE city = 'Dhaka'
ORDER BY customer_id;
```

| full_name | email | city |
| --- | --- | --- |
| Tanvir Pial | tanvir@gmail.com | Dhaka |
| Sara Ahmed | sara@yahoo.com | Dhaka |

### Active customers

```sql
SELECT full_name, country
FROM customers
WHERE status = 'active'
ORDER BY customer_id;
```

Expected customer IDs: **1, 2, 4, 6, 7, 9, 10, 11**. That is eight rows.

### Not equal: <>

```sql
SELECT customer_id, full_name, status
FROM customers
WHERE status <> 'active'
ORDER BY customer_id;
```

Expected IDs: **3, 5, 8, 12**. In this dataset they are inactive. A NULL status would not satisfy this comparison.

## 6. Understanding NULL

NULL represents missing or unknown information. Its exact business meaning depends on the column and the data rules.

It is different from:

- `0`: a known numeric value.
- `''`: an empty text value in PostgreSQL.
- `'NULL'`: the four-letter text string.

Nusrat's total_orders is 0, which records zero orders. Her age is NULL, which does not mean that her age is zero.

Rahul has two recorded orders but no last_order_date. That is missing date information; it does not prove he never ordered.

### Find missing values

```sql
SELECT customer_id, full_name, email
FROM customers
WHERE email IS NULL
ORDER BY customer_id;
```

| customer_id | full_name |
| --- | --- |
| 2 | Ayesha Rahman |
| 6 | Fatima Khan |
| 9 | David Wilson |
| 12 | Priya Sharma |

### Find known values

```sql
SELECT customer_id, full_name, age
FROM customers
WHERE age IS NOT NULL
ORDER BY customer_id;
```

This returns ten rows. Maria Garcia and Nusrat Jahan are excluded because their ages are NULL.

### Why age <> NULL is wrong for this task

Comparisons such as `age = NULL` or `age <> NULL` normally return an unknown result, rather than true. WHERE only keeps true conditions.

Use `IS NULL` or `IS NOT NULL` to test for missing values. See the [PostgreSQL comparison documentation](https://www.postgresql.org/docs/current/functions-comparison.html).

## 7. COALESCE: choose the first available value

COALESCE returns the first argument that is not NULL, reading from left to right.

```sql
SELECT full_name,
       COALESCE(email, 'Not Provided') AS email_display
FROM customers
ORDER BY customer_id;
```

For Ayesha, email_display becomes `Not Provided`. For Tanvir, it stays `tanvir@gmail.com`.

This SELECT changes the displayed result. **It does not update the stored email column.** `AS email_display` gives the output column a readable name.

### Understanding the order

```sql
SELECT COALESCE(NULL, NULL, 5, 10) AS first_available;
-- Result: 5

SELECT COALESCE(NULL, 0, 10) AS first_available;
-- Result: 0
```

COALESCE does not choose the largest value and does not skip zero. If every argument is NULL, the result is NULL. Arguments must be compatible with a common data type.

Use fallback values carefully. Replacing unknown spending with zero could change the meaning of a report. [PostgreSQL conditional expressions](https://www.postgresql.org/docs/current/functions-conditional.html) describes COALESCE and CASE.

## 8. CASE WHEN: make a conditional choice

The slides also use CASE to explain a fallback. This email example applies the same idea:

```sql
SELECT full_name,
       CASE
           WHEN email IS NOT NULL THEN email
           ELSE 'Not Provided'
       END AS email_display
FROM customers
ORDER BY customer_id;
```

Read it as: if an email exists, show it; otherwise, show Not Provided.

CASE can handle more general conditions. COALESCE is shorter when the task is simply to choose the first non-NULL expression.

## 9. The logistics example from the slide

The slide shows confirmation, first-mile, and sorting milestones. Assume these are timestamps:

- `confirmed_at`: the shipment was confirmed.
- `first_mile_at`: the first-mile milestone was recorded.
- `sorted_at`: sorting was recorded.

The example defines processing time as the first-mile time minus confirmation, falling back to sorting minus confirmation if the first interval is missing.

```sql
COALESCE(
    first_mile_at - confirmed_at,
    sorted_at - confirmed_at
) AS processing_time
```

| Confirmation | First mile | Sorting | Result |
| --- | --- | --- | --- |
| 09:00 | 10:00 | 11:00 | 1 hour |
| 09:00 | NULL | 11:00 | 2 hours |
| 09:00 | NULL | NULL | NULL |

Times in this illustration are on the same date. Timestamp subtraction produces an interval in PostgreSQL.

The first row returns one hour because COALESCE takes the first available interval, not the latest milestone. If confirmation is missing, both differences are NULL.

An equivalent expression for this example is:

```sql
CASE
    WHEN first_mile_at IS NOT NULL
        THEN first_mile_at - confirmed_at
    ELSE sorted_at - confirmed_at
END AS processing_time
```

This is a lesson-specific definition of processing time, not a universal logistics rule. These illustrative timestamps are not columns in the customers table.

## 10. LIMIT and OFFSET: split results into pages

LIMIT controls how many rows are returned. OFFSET skips rows before returning the next page.

```sql
-- Page 1: customer IDs 1 to 5
SELECT * FROM customers
ORDER BY customer_id
LIMIT 5 OFFSET 0;

-- Page 2: customer IDs 6 to 10
SELECT * FROM customers
ORDER BY customer_id
LIMIT 5 OFFSET 5;

-- Page 3: customer IDs 11 and 12
SELECT * FROM customers
ORDER BY customer_id
LIMIT 5 OFFSET (5 * 2);
```

For page numbers starting at 1:

**Offset = (page number - 1) x page size.**

ORDER BY is essential for predictable pages. Using the unique customer_id defines a complete ordering for this static dataset. Without it, PostgreSQL does not promise a particular row order. See [LIMIT and OFFSET](https://www.postgresql.org/docs/current/queries-limit.html).

## 11. LIKE and ILIKE: search text patterns

These are pattern-matching operators. The `%` and `_` wildcards are not PostgreSQL's full regular-expression syntax.

| Pattern | Meaning |
| --- | --- |
| `'A%'` | Starts with A |
| `'%a'` | Ends with a |
| `'%gmail%'` | Contains gmail |
| `'A_'` | A followed by exactly one character |

`%` matches zero or more characters. `_` matches exactly one character.

### Starts with uppercase A

```sql
SELECT customer_id, full_name
FROM customers
WHERE full_name LIKE 'A%'
ORDER BY customer_id;
```

Expected matches: **Ayesha Rahman (2)** and **Alex Turner (11)**.

### Starts with lowercase a

```sql
SELECT customer_id, full_name
FROM customers
WHERE full_name LIKE 'a%'
ORDER BY customer_id;
```

Expected result: **no rows**, assuming the case-sensitive collation used for these examples. Collation settings can affect LIKE behavior.

### Ends with a, ignoring case

```sql
SELECT customer_id, full_name
FROM customers
WHERE full_name ILIKE '%a'
ORDER BY customer_id;
```

Expected matches: **Maria Garcia (4), Rahul Verma (5), Priya Sharma (12)**.

ILIKE performs case-insensitive matching according to the active locale. Notice that `%a` searches the end of the full name, not its beginning.

### Email contains gmail

```sql
SELECT customer_id, full_name, email
FROM customers
WHERE email LIKE '%gmail%'
ORDER BY customer_id;
```

Expected IDs: **1, 4, 7, 10, 11**. NULL emails do not match.

This checks for the substring gmail anywhere in the value. It does not validate an email address or guarantee an exact Gmail domain.

For the syntax and collation details, see [PostgreSQL pattern matching](https://www.postgresql.org/docs/current/functions-matching.html).

## 12. WHERE, HAVING, and CASE are different

The tutorial code mentions these together, but they serve different purposes:

| Feature | Purpose | Example idea |
| --- | --- | --- |
| WHERE | Select rows before grouping | Keep customers from Dhaka |
| HAVING | Select groups after grouping | Keep countries with more than three customers |
| CASE | Calculate a value using conditions | Show Not Provided for missing email |

The supplied practice demonstrates WHERE and the slide explains CASE. Detailed GROUP BY and HAVING practice is a later exercise.

## 13. Expected results checklist

These reference results are derived from the supplied 12-row dataset. They are not screenshots of a new PostgreSQL execution.

| Query | Expected result |
| --- | --- |
| All customers | 12 rows |
| City is Dhaka | IDs 1, 8 |
| Status is active | 8 rows |
| Status is not active | IDs 3, 5, 8, 12 |
| Age is NULL | IDs 4, 10 |
| Age is not NULL | 10 rows |
| Email is NULL | IDs 2, 6, 9, 12 |
| LIKE 'A%' | IDs 2, 11 |
| LIKE 'a%' | No rows under a case-sensitive collation |
| ILIKE '%a' | IDs 4, 5, 12 |
| Email LIKE '%gmail%' | IDs 1, 4, 7, 10, 11 |
| Page 3, five rows per page | IDs 11, 12 |

## 14. Quick interview revision

**What is NULL?** A marker for missing or unknown information. It is not the same as zero.

**How do I find missing emails?** Use `WHERE email IS NULL`.

**What does COALESCE do?** It returns the first non-NULL argument.

**Does COALESCE update the table?** Not when used in a SELECT like these examples. It changes the query result.

**Can COALESCE return zero?** Yes. Zero is a value, so it is not skipped.

**How do LIKE and ILIKE differ?** With the case-sensitive setup in this lesson, LIKE distinguishes uppercase and lowercase. ILIKE ignores case according to the locale.

**What do LIMIT and OFFSET do?** LIMIT sets the maximum returned rows. OFFSET skips a number of rows.

**Why use ORDER BY for pagination?** It defines which rows belong on each page for a fixed dataset.

## 15. My main takeaways

- Understand what a missing value means before replacing it.
- Use IS NULL and IS NOT NULL for missing-value checks.
- Read COALESCE arguments from left to right.
- Use a unique ordering column for stable pagination.
- Choose text patterns according to whether I need starts-with, ends-with, or contains.
- Separate database-design projects from actual SQL query practice.

## Learning sources

The customer records and lesson topics come from my faculty's tutorial. The supplied NULL-handling slides show Data360 Solution branding. These notes restate the lesson in my own learning format, with AI assistance for organization and technical clarification.

The ER projects use fictional business assumptions. Their database implementations are still planned.
