# Day 1: SQL and Database Fundamentals

Today I studied how databases organize data, how SQL interacts with relational databases, and why database design matters for reliable analysis.

**Learning activity:** Tutorial study and conceptual revision. SQL examples in these notes illustrate the concepts and are not presented as completed hands-on exercises.

## Contents

1. Database, DBMS, and RDBMS
2. SQL command categories
3. Keys and relationships
4. Transactions and ACID
5. SQL and NoSQL databases
6. Data warehouses, lakes, and lakehouses
7. Logical processing order of SQL queries
8. Introduction to normalization
9. Relevance to data analysis
10. Revision and next steps

Related materials: [Normalization example](normalization-example.md) · [Interview questions and answers](interview-questions.md)

## 1. Database, DBMS, and RDBMS

### Database

A database is an organized collection of data. A retail business might store customer details, product information, and sales transactions in a database.

### Database Management System (DBMS)

A DBMS is software used to create, store, retrieve, and manage data in a database. Depending on the system, it also provides facilities such as access control, backup, and transaction management.

The database contains the data. The DBMS manages it.

### Relational Database Management System (RDBMS)

An RDBMS manages relational databases. Data is organized in tables with rows and columns, and keys connect related records.

| Term | Meaning | Retail example |
| --- | --- | --- |
| Table | A collection of records about a subject | Customers |
| Row | One record in a table | One customer |
| Column | An attribute of a record | CustomerCity |
| Schema | The defined structure of database objects | Tables, columns, data types, and constraints |

Examples from the lesson include MySQL, PostgreSQL, Oracle Database, and Microsoft SQL Server.

Relational systems support constraints, transactions, and complex queries. Indexes can improve retrieval performance, although their usefulness depends on the query and workload.

## 2. SQL command categories

**SQL means Structured Query Language.** It is used to define structures, retrieve and modify data, manage access, and control transactions.

SQL is a language. MySQL is a database management system that implements SQL.

| Category | Full name | Purpose | Examples |
| --- | --- | --- | --- |
| DDL | Data Definition Language | Define or change structures | CREATE, ALTER, DROP, TRUNCATE |
| DML | Data Manipulation Language | Add, change, or remove rows | INSERT, UPDATE, DELETE |
| DQL | Data Query Language | Retrieve data | SELECT |
| DCL | Data Control Language | Manage permissions | GRANT, REVOKE |
| TCL | Transaction Control Language | Manage transactions | COMMIT, ROLLBACK, SAVEPOINT |

These are common teaching categories. Some references include SELECT within DML instead of treating DQL as a separate category.

### Example: reading customer data

```sql
SELECT CustomerName, CustomerCity
FROM Customers
WHERE CustomerCity = 'Dhaka';
```

This illustrative query returns names and cities for customers whose city is Dhaka. It assumes the table already exists.

### DELETE, TRUNCATE, and DROP

- `DELETE` removes rows and can use a `WHERE` condition.
- `TRUNCATE` removes all rows while retaining the table definition. Its restrictions and transaction behavior depend on the database system.
- `DROP TABLE` removes the table itself, including its definition and data.

I should check the specific database's behavior before making claims about rollback, identity values, or performance.

## 3. Keys and relationships

This section expands the keys introduced in the lesson and used in the normalization example.

### Primary key

A primary key uniquely identifies each row. Primary-key values must be unique and cannot be null. A primary key can consist of one column or multiple columns.

For example, CustomerID identifies a customer. CustomerName is a poor identifier because different people can share a name, and names can change.

### Foreign key

A foreign key references a key in another table, or sometimes the same table. When enforced, it helps prevent references to records that do not exist.

For example, Sales.CustomerID references Customers.CustomerID.

### One-to-many relationship

One customer can have many sales records. Each sale in the lesson references one customer.

```text
Customers.CustomerID  1 ---- many  Sales.CustomerID
Products.ProductID   1 ---- many  Sales.ProductID
```

Before joining tables, I need to know how many matches each row can have. Unexpected multiple matches can duplicate amounts and inflate a report's totals.

## 4. Transactions and ACID

A transaction is a unit of work containing one or more database operations. ACID describes properties that help transactions behave reliably.

Consider transferring 50 from account A to account B:

| Account | Before | After a successful transfer |
| --- | ---: | ---: |
| A | 100 | 50 |
| B | 20 | 70 |
| Combined balance | 120 | 120 |

### Atomicity

The changes succeed together or are rolled back together. A failed transfer should not leave A debited without crediting B.

### Consistency

A successful transaction preserves the database's defined rules. In this example, a correctly implemented transfer preserves the combined balance, assuming no fees or other operations.

Consistency requires appropriate constraints and correct application logic. The database cannot infer every business rule automatically.

### Isolation

Isolation governs how concurrent transactions interact and what changes they can observe. Different isolation levels provide different guarantees.

For example, simultaneous withdrawals and deposits should behave according to the guarantees selected for the application.

### Durability

Once a transaction commits, its changes persist according to the database's durability guarantees, including recovery from supported failure scenarios.

### Transaction commands

| Command | Meaning |
| --- | --- |
| COMMIT | Complete the transaction and make its changes permanent |
| ROLLBACK | Cancel uncommitted changes |
| SAVEPOINT | Mark a point within a transaction that can be rolled back to |

**Clarification:** ACID concerns transaction reliability. It does not replace security controls such as authentication and authorization.

## 5. SQL and NoSQL databases

In comparisons, “SQL databases” usually refers to relational databases that use SQL. NoSQL covers several database families with different data models.

| Aspect | Relational databases | NoSQL databases |
| --- | --- | --- |
| Data organization | Related tables | Documents, key-value pairs, graphs, or wide-column models |
| Schema | Explicit table structures that can evolve | Flexibility varies by system |
| Query interface | SQL, with system-specific differences | Languages and APIs vary |
| Examples | MySQL, PostgreSQL, Oracle, SQL Server | MongoDB, Cassandra, CouchDB, Neo4j |

### NoSQL models introduced in the lesson

- **Key-value:** Retrieve a value through a key.
- **Document:** Store records as documents, potentially with nested fields.
- **Graph:** Represent entities and connections using nodes and edges.
- **Wide-column:** Organize data using a column-family model.

### Clarifications for revision

Relational databases are not limited to vertical scaling. NoSQL systems are not universally faster, and some support ACID transactions. Flexible schemas still require careful data design.

The wide-column NoSQL model is also distinct from the column-oriented storage used by many analytical databases.

The appropriate system depends on data relationships, access patterns, transaction needs, and operational requirements.

## 6. Data warehouses, lakes, and lakehouses

### Data warehouse

A warehouse brings together data for analysis and reporting. It typically contains curated data organized around business questions and consistent definitions.

**Example:** Combining branch sales to produce monthly revenue reports.

### Data lake

A lake stores data in varied formats, often including raw data. It can hold structured tables, semi-structured data such as JSON, and unstructured content such as images.

**Example:** Keeping sales extracts, application logs, and customer feedback for different future analyses.

### Data lakehouse

A lakehouse combines lake-style storage with management and analytical capabilities associated with warehouses, such as governed tables and transaction support.

### ETL

The lesson's architecture diagram includes **Extract, Transform, Load**:

1. Extract data from source systems.
2. Transform it through operations such as cleaning or standardizing values.
3. Load the prepared data into a destination.

These are introductory distinctions. Actual platforms can overlap in their capabilities.

## 7. Logical processing order of SQL queries

SQL's written order differs from the logical order used to understand its result.

**Typical written order:**

```text
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

**Simplified logical processing order:**

```text
FROM and JOIN conditions
WHERE
GROUP BY
HAVING
SELECT
DISTINCT, if present
ORDER BY
LIMIT or equivalent row limiting
```

### Example

```sql
SELECT CustomerID, SUM(TotalAmount) AS TotalSpent
FROM Sales
WHERE Quantity >= 2
GROUP BY CustomerID
HAVING SUM(TotalAmount) > 1000
ORDER BY TotalSpent DESC
LIMIT 5;
```

This illustrative query uses LIMIT syntax, supported by systems such as PostgreSQL and MySQL. Other systems can use different row-limiting syntax.

Reading it logically:

1. Start with Sales.
2. Keep rows where quantity is at least two.
3. Group the remaining rows by customer.
4. Keep groups with a total above 1,000.
5. Produce the customer ID and calculated total.
6. Sort totals from highest to lowest.
7. Return up to five rows.

Here, TotalSpent measures spending on qualifying sales rows, not necessarily the customer's complete purchase history.

### WHERE versus HAVING

WHERE filters individual rows before grouping. HAVING filters groups after grouping.

**Clarification:** This is logical processing order. The optimizer may choose a different physical execution plan while preserving the query's meaning.

## 8. Introduction to normalization

Normalization organizes relational data around dependencies to reduce unnecessary repetition and modification anomalies.

The lesson starts with a sales table containing customer details, product descriptions, and transaction values. The same customer's details appear in several rows.

If Rahim changes his current city or phone number, several rows need updating. Missing one could leave conflicting information.

The lesson separates the data into:

- DimCustomer for customer details.
- DimProduct for product details.
- FactSales for sales records containing customer and product references.

See the [worked normalization example](normalization-example.md) for the tables and reasoning.

### Anomalies

| Anomaly | Example |
| --- | --- |
| Update | Changing the same customer's current city in several sales rows |
| Insertion | Being unable to add a product independently because the only table requires a sale |
| Deletion | Losing the only stored customer details when deleting that customer's last sale |

**Clarifications:** Normalization organizes dependencies rather than eliminating all dependencies. A reduction in the number of table cells does not establish actual storage savings.

The supplied deck demonstrates decomposition through an example. Formal definitions of 1NF, 2NF, and 3NF are further-study topics rather than topics explicitly covered in that deck.

## 9. Relevance to data analysis

Before analyzing data, I need to understand its **grain**, meaning what one row represents. I also need to know its keys and relationships.

This helps me:

- Choose joins that match the business question.
- Avoid duplicate counting.
- Recognize conflicting customer information.
- Distinguish current attributes from historical transaction facts.
- Interpret aggregates correctly.

For example, using a product's current price to recalculate an older sale can misstate historical revenue. Database design influences whether the information needed for an accurate report is available.

## 10. Revision and next steps

### Self-check

- [ ] Explain database, DBMS, and RDBMS without reading these notes.
- [ ] Give an example from each SQL command category.
- [ ] Explain ACID through a bank transfer.
- [ ] Identify primary and foreign keys in the retail example.
- [ ] Explain WHERE versus HAVING.
- [ ] Describe the logical processing order of a query.
- [ ] Explain how table separation addresses an update anomaly.
- [ ] Explain warehouse, lake, and lakehouse in plain language.

### Further practice

- Implement the retail example in a chosen database system.
- Join the customer, product, and sales tables.
- Calculate sales by customer and compare against a manual calculation.
- Study functional dependencies and the first three normal forms.

## Sources and scope

Based on **Class 1: Introduction to SQL and Database Fundamentals** and **Normalization**. The introductory deck credits **Tanvir Taushif**.

These notes paraphrase the tutorials. Explanations about keys, anomalies, historical prices, and common simplifications expand on the lesson for revision. They do not imply that every detail appeared in the tutorials.

[Back to learning log](../README.md)
