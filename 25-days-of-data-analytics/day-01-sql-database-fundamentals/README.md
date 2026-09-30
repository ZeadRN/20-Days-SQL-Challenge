# Day 1: SQL and Database Fundamentals

Day 1 of my 25-day data analytics learning series focused on understanding databases, SQL commands, transactions, and the basics of database design.

These notes summarize what I studied through tutorials, with examples for revision and technical interview preparation.

> Learning status: Conceptual study. The SQL examples illustrate the concepts; they are not a record of completed practical exercises.

## Topics Covered

- Database, DBMS, and RDBMS
- Tables, keys, and constraints
- SQL command categories: DDL, DQL, DML, DCL, and TCL
- Transactions and ACID properties
- SQL versus NoSQL
- Data warehouses, data lakes, and lakehouses
- Logical processing order of SQL queries
- Data redundancy and normalization

---

## 1. Database Fundamentals

### What is a database?

A database is an organized collection of data.

For example, a retail business might maintain information about its customers, products, orders, and payments.

### What is a DBMS?

A **Database Management System (DBMS)** is software used to create, access, and manage databases.

The database holds the data. The DBMS provides tools for working with it.

### What is an RDBMS?

A **Relational Database Management System (RDBMS)** manages data organized into related tables.

Tables contain rows and columns, while keys connect records across tables.

Examples:

- MySQL
- PostgreSQL
- Microsoft SQL Server
- Oracle Database

### Basic terminology

| Term | Meaning | Example |
|---|---|---|
| Table | A collection of related records | `Customers` |
| Row / Record | One entry in a table | One customer's details |
| Column / Field | An attribute of a record | `CustomerName` |
| Schema | The defined structure of database objects | Tables, columns, types, and constraints |
| Query | A request to retrieve or work with data | Retrieve customers from Dhaka |
| Constraint | A rule enforced on data | A customer ID must be unique |

---

## 2. Keys and Constraints

### Primary key

A primary key uniquely identifies each row in a table.

- Its values must be unique.
- It cannot contain `NULL`.
- It can consist of one column or multiple columns.

For example, `CustomerID` can identify each customer.

A name is usually a poor primary key because different people can share the same name.

### Foreign key

A foreign key references a key in another table or, in some cases, the same table.

For example:

```text
Customers.CustomerID ← Sales.CustomerID
```

An enforced foreign key helps prevent a sale from referencing a customer who does not exist.

### Common constraints

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each row |
| `FOREIGN KEY` | Enforces a reference to another record |
| `NOT NULL` | Requires a value |
| `UNIQUE` | Prevents duplicate values or combinations, subject to the database's NULL rules |
| `CHECK` | Enforces a condition, such as a nonnegative price |

### Why this matters for analysis

Keys and relationships determine how tables connect.

If I join tables without understanding their relationships, I may duplicate rows and overstate totals.

---

## 3. What Is SQL?

**SQL stands for Structured Query Language.**

It is used to:

- Define database structures.
- Retrieve records.
- Insert, update, and delete data.
- Manage access permissions.
- Control transactions.

### SQL versus MySQL

SQL is a language.

MySQL is a database management system that supports SQL. PostgreSQL, SQL Server, and Oracle also support SQL, with differences in syntax and features.

---

## 4. SQL Command Categories

SQL commands are commonly taught in five categories:

| Category | Full Name | Main Purpose | Examples |
|---|---|---|---|
| DDL | Data Definition Language | Define database structures | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DQL | Data Query Language | Retrieve data | `SELECT` |
| DML | Data Manipulation Language | Change stored data | `INSERT`, `UPDATE`, `DELETE` |
| DCL | Data Control Language | Manage access | `GRANT`, `REVOKE` |
| TCL | Transaction Control Language | Manage transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

> These are useful teaching categories. Some references classify `SELECT` under DML instead of using a separate DQL category.

### About the examples

The examples below use PostgreSQL-style syntax and an `employees` table.

They are individual demonstrations, not one script to execute from beginning to end. Examples involving permissions assume the referenced role already exists.

---

## 5. DDL: Data Definition Language

DDL changes the structure of database objects.

### CREATE

Creates an object, such as a table.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    department VARCHAR(50),
    salary NUMERIC(12, 2) CHECK (salary >= 0),
    hire_date DATE
);
```

This defines the columns, data types, and constraints of the table.

### ALTER

Changes an existing table's structure.

```sql
ALTER TABLE employees
ADD COLUMN email VARCHAR(100);
```

This adds an email column.

### DROP

Removes a database object.

```sql
DROP TABLE employees;
```

This removes the table, including its definition and data.

### TRUNCATE

Removes all rows while keeping the table definition.

```sql
TRUNCATE TABLE employees;
```

Restrictions, storage behavior, and transaction behavior depend on the database system.

### COMMENT

Adds descriptive metadata where supported.

```sql
COMMENT ON TABLE employees
IS 'Employee details for SQL learning examples';
```

### Rename a table

Renaming syntax differs across database systems. In PostgreSQL:

```sql
ALTER TABLE employees
RENAME TO staff;
```

### Main takeaway

DDL describes or changes the structures that hold data.

---

## 6. DQL: Data Query Language

DQL retrieves data using `SELECT`.

### SELECT

```sql
SELECT first_name, last_name, department
FROM employees;
```

This retrieves selected columns from the table.

### SELECT and its related clauses

`SELECT` is the statement. Terms such as `FROM`, `WHERE`, and `ORDER BY` are parts of that statement, not separate commands.

| Clause or Keyword | Purpose |
|---|---|
| `FROM` | Identifies the source tables |
| `WHERE` | Filters individual rows |
| `GROUP BY` | Forms groups of rows |
| `HAVING` | Filters groups |
| `DISTINCT` | Removes duplicate output rows |
| `ORDER BY` | Sorts the result |
| `LIMIT` | Restricts the number of returned rows in supported systems |

### Filtering with WHERE

```sql
SELECT first_name, department
FROM employees
WHERE department = 'Sales';
```

Returns employees in the Sales department.

### Removing duplicates with DISTINCT

```sql
SELECT DISTINCT department
FROM employees;
```

Returns distinct department values.

When multiple columns are selected, `DISTINCT` applies to the complete combination of selected values.

### Sorting with ORDER BY

```sql
SELECT first_name, salary
FROM employees
ORDER BY salary DESC, employee_id;
```

Returns employees from highest to lowest salary, using employee ID to break ties.

### Grouping with GROUP BY

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Counts employees in each department.

### Filtering groups with HAVING

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

Returns departments with more than five employees.

### Limiting output

```sql
SELECT employee_id, first_name, salary
FROM employees
ORDER BY salary DESC, employee_id
LIMIT 5;
```

Returns up to five employees with the highest salaries.

`LIMIT` is supported by PostgreSQL and MySQL. Other systems may use different syntax.

### WHERE versus HAVING

| WHERE | HAVING |
|---|---|
| Filters individual rows | Filters groups |
| Applied before grouping | Applied after grouping |
| Example: employees in Sales | Example: departments with more than five employees |

---

## 7. DML: Data Manipulation Language

DML changes the records stored in tables.

### INSERT

Adds new rows.

```sql
INSERT INTO employees (
    employee_id,
    first_name,
    last_name,
    department,
    salary,
    hire_date
)
VALUES (
    1,
    'Amina',
    'Rahman',
    'Sales',
    45000.00,
    '2026-01-15'
);
```

### UPDATE

Changes existing rows.

```sql
UPDATE employees
SET department = 'Marketing'
WHERE employee_id = 1;
```

This changes one employee's department.

Without a `WHERE` condition, the update applies to all rows.

### DELETE

Removes rows.

```sql
DELETE FROM employees
WHERE employee_id = 1;
```

Without a `WHERE` condition, `DELETE` removes all rows from the table.

### DELETE versus TRUNCATE versus DROP

| Command | Effect | Supports WHERE? | Keeps Table Definition? |
|---|---|---|---|
| `DELETE` | Removes matching rows | Yes | Yes |
| `TRUNCATE` | Removes all rows | No | Yes |
| `DROP TABLE` | Removes the table itself | No | No |

Rollback behavior and other details depend on the database system and transaction context.

---

## 8. DCL: Data Control Language

DCL manages privileges on database objects.

### GRANT

Gives privileges to a user or role.

```sql
GRANT SELECT ON employees TO analyst_role;
```

This grants the role permission to read the table.

### REVOKE

Removes a grant.

```sql
REVOKE SELECT ON employees FROM analyst_role;
```

Other grants or role memberships may still provide access.

### Why this matters for analysts

An analyst may need permission to read data without permission to modify it. Access can be assigned according to the work the role needs to perform.

**Interview example:** Which SQL category manages access permissions?

**Answer:** DCL.

---

## 9. TCL: Transaction Control Language

A transaction groups operations into one unit of work.

### Main commands

| Command | Purpose |
|---|---|
| `BEGIN` | Starts a transaction |
| `COMMIT` | Completes the transaction and makes changes permanent |
| `ROLLBACK` | Cancels uncommitted changes |
| `SAVEPOINT` | Marks a point within a transaction |
| `ROLLBACK TO SAVEPOINT` | Reverses changes made after that point |

### Example with a savepoint

```sql
BEGIN;

UPDATE employees
SET department = 'Marketing'
WHERE employee_id = 1;

SAVEPOINT before_salary_change;

UPDATE employees
SET salary = 50000.00
WHERE employee_id = 1;

ROLLBACK TO SAVEPOINT before_salary_change;

COMMIT;
```

Assuming the employee exists:

1. The department changes to Marketing.
2. A savepoint is created.
3. The salary changes.
4. Rolling back to the savepoint reverses the salary change.
5. Committing saves the department change.

### Main takeaway

Rolling back to a savepoint can undo part of a transaction. A full rollback cancels the transaction's uncommitted changes.

---

## 10. ACID Properties

ACID describes properties that help transactions behave reliably.

Consider transferring 50 from account A to account B:

| Account | Before | After |
|---|---:|---:|
| A | 100 | 50 |
| B | 20 | 70 |
| Combined balance | 120 | 120 |

### Atomicity

The transaction's changes succeed together or are rolled back together.

The transfer should not leave A debited without crediting B.

### Consistency

A successful transaction preserves the database's defined rules.

In this example, a correctly implemented transfer preserves the combined balance, assuming no fees or other operations.

Correct constraints and application logic are still necessary.

### Isolation

Isolation governs interactions between concurrent transactions.

Different isolation levels provide different guarantees about what transactions can observe.

### Durability

Committed changes persist according to the database's durability guarantees, including recovery from supported failure scenarios.

### Quick revision

| Property | Meaning |
|---|---|
| Atomicity | Changes succeed or roll back together |
| Consistency | Defined rules remain satisfied |
| Isolation | Concurrent interactions follow selected guarantees |
| Durability | Committed changes persist |

ACID concerns transaction reliability. Authentication and authorization address separate security needs.

---

## 11. SQL versus NoSQL

In this comparison, “SQL databases” usually means relational databases that use SQL.

NoSQL includes several database families.

| Aspect | Relational Databases | NoSQL Databases |
|---|---|---|
| Typical model | Related tables | Documents, key-value pairs, graphs, or wide-column models |
| Schema | Explicit structures that can evolve | Flexibility varies by system |
| Query interface | SQL with system-specific differences | Languages and APIs vary |
| Examples | PostgreSQL, MySQL, SQL Server | MongoDB, Cassandra, Neo4j |

### Common NoSQL models

- **Key-value:** Values are accessed through keys.
- **Document:** Records can contain nested document structures.
- **Graph:** Data represents entities and their relationships.
- **Wide-column:** Data uses a column-family model.

### Important distinctions

- Relational databases are not limited to vertical scaling.
- NoSQL systems are not automatically faster.
- Some NoSQL systems support ACID transactions.
- Flexible schemas still require careful design.

The choice depends on the data model, access patterns, transaction requirements, and workload.

---

## 12. Data Warehouse, Data Lake, and Lakehouse

### Data warehouse

A warehouse typically contains curated data organized for reporting and analysis.

**Example:** Combining branch sales to produce monthly business reports.

### Data lake

A lake stores data in varied formats, often including raw data.

**Example:** Storing sales files, JSON logs, images, and customer feedback for different uses.

### Data lakehouse

A lakehouse combines lake-style storage with management and analytical capabilities associated with warehouses.

### ETL

**ETL means Extract, Transform, Load.**

1. Extract data from sources.
2. Transform it through cleaning or standardization.
3. Load it into a destination.

Actual platforms can overlap in their capabilities.

---

## 13. Logical Processing Order of SQL

The order in which SQL is written differs from its logical processing order.

### Written order

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
WHERE salary >= 30000
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY employee_count DESC, department
LIMIT 5;
```

### Simplified logical order

1. `FROM` and joins identify the source rows.
2. `WHERE` filters rows.
3. `GROUP BY` forms groups.
4. `HAVING` filters groups.
5. `SELECT` produces output expressions.
6. `DISTINCT`, when present, removes duplicate output rows.
7. `ORDER BY` sorts the result.
8. `LIMIT` or equivalent syntax restricts the output.

In this example, the counts include only employees whose salary meets the `WHERE` condition.

> Logical processing order explains the query's meaning. The optimizer may choose a different physical execution plan.

---

## 14. Introduction to Normalization

Normalization organizes relational data around dependencies to reduce unnecessary repetition and modification anomalies.

### Original sales table

The lesson's retail example repeats customer and product details across sales.

| OrderID | CustomerName | CustomerCity | ProductName | Price | Quantity | TotalAmount |
|---|---|---|---|---:|---:|---:|
| 1 | Rahim Uddin | Dhaka | Laptop | 500 | 2 | 1000 |
| 3 | Rahim Uddin | Dhaka | Tablet | 200 | 3 | 600 |
| 8 | Rahim Uddin | Dhaka | Laptop | 500 | 1 | 500 |

Amounts are illustrative; the lesson does not specify a currency.

### The problem

If Rahim changes his current city, several records need updating.

Updating only one row could leave conflicting details.

### Common anomalies

| Anomaly | Example |
|---|---|
| Update | The same customer's city must be changed in several rows |
| Insertion | Adding a product is difficult if every row must include a sale |
| Deletion | Removing a customer's only sale also removes their only stored details |

### Separating the data

```text
Customers
- CustomerID
- CustomerName
- CustomerCity
- PhoneNo

Products
- ProductID
- ProductName
- Price

Sales
- OrderID
- CustomerID
- ProductID
- Quantity
- TotalAmount
```

Customer details can now be maintained in one customer record, while sales reference the customer through CustomerID.

### Additional design considerations

- A customer's current city is different from an old order's shipping destination.
- Historical sales should preserve the price or amount charged at the time.
- An order containing multiple products usually needs separate order-line records.
- Fewer table cells do not automatically prove lower storage usage.
- Normalization organizes dependencies rather than removing all dependencies.

The supplied lesson introduces decomposition through an example. Formal definitions of 1NF, 2NF, and 3NF are topics for further study.

---

## 15. Why This Matters for Data Analysis

Understanding database fundamentals helps me reason about the data before calculating results.

Before writing an analysis, I should ask:

1. What does one row represent?
2. Which columns identify records?
3. How do the tables relate?
4. Can a join produce multiple matches?
5. Does an attribute describe the current state or a historical event?
6. Does my query measure the business question correctly?

These questions help prevent duplicate counting, misleading totals, and incorrect interpretations.

---

## 16. Interview Revision

### What is the difference between a database and a DBMS?

A database contains organized data. A DBMS is the software used to manage it.

### What is the difference between SQL and MySQL?

SQL is a language. MySQL is a database management system that supports it.

### What is the difference between DDL and DML?

DDL changes database structures. DML changes stored records.

### Are WHERE and ORDER BY separate SQL commands?

They are clauses used within statements such as SELECT.

### What is the difference between WHERE and HAVING?

WHERE filters rows before grouping. HAVING filters groups.

### Which command category manages permissions?

DCL, including GRANT and REVOKE.

### What does ACID stand for?

Atomicity, Consistency, Isolation, and Durability.

### What is the difference between a primary key and a foreign key?

A primary key identifies a row. A foreign key references a key in another table or the same table.

### Why is normalization useful?

It reduces unnecessary repetition and helps avoid update, insertion, and deletion anomalies.

### Does normalization always make queries faster?

No. Performance depends on the design, indexes, data volume, and workload. Retrieving data may require additional joins.

---

## 17. Next Steps

- [ ] Create a practice database and tables.
- [ ] Practice SELECT, WHERE, DISTINCT, and ORDER BY.
- [ ] Practice INSERT, UPDATE, and DELETE.
- [ ] Test COMMIT, ROLLBACK, and SAVEPOINT.
- [ ] Implement the retail normalization example.
- [ ] Study functional dependencies, 1NF, 2NF, and 3NF.
- [ ] Add executed SQL scripts and verified results to this repository.

---

## Resources and Acknowledgments

- **Class 1: Introduction to SQL and Database Fundamentals** — tutorial slides identifying the instructor as Tanvir Taushif.
- **Normalization** — supplied tutorial slides.
- [GeeksforGeeks: SQL Commands — DDL, DQL, DML, DCL and TCL](https://www.geeksforgeeks.org/sql/sql-ddl-dql-dml-dcl-tcl-commands/)

These notes paraphrase the learning materials and include additional explanations for revision. The examples use a consistent schema and identify database-specific syntax where relevant.
