# Day 1: Interview Revision Questions

These questions support revision of the Day 1 topics. Some answers include additional explanations beyond the slides. I can cover the answers and practice explaining each concept aloud.

## 1. What is the difference between a database and a DBMS?

A database is an organized collection of data. A DBMS is the software used to store, retrieve, and manage that data.

## 2. What is an RDBMS?

An RDBMS manages relational databases, where data is organized in tables. Keys connect records and constraints help enforce data integrity. Examples include PostgreSQL and MySQL.

## 3. What is the difference between SQL and MySQL?

SQL is a language for working with data. MySQL is a database management system that supports SQL. Other systems also implement SQL, with differences in syntax and features.

## 4. What are the SQL command categories?

The lesson uses DDL for structures, DML for row changes, DQL for queries, DCL for permissions, and TCL for transactions. Example commands are CREATE, INSERT, SELECT, GRANT, and COMMIT. Some references include queries within DML.

## 5. How do DELETE, TRUNCATE, and DROP differ?

DELETE removes rows and can filter them with WHERE. TRUNCATE removes all rows while keeping the table definition, subject to system-specific restrictions. DROP TABLE removes the table itself. Transaction and rollback behavior should be checked for the actual database system.

## 6. What is a primary key?

A primary key uniquely identifies a row and cannot contain null values. It may consist of one or several columns. CustomerID is usually a better identifier than CustomerName because names can repeat or change.

## 7. What is a foreign key?

A foreign key references a key in another table or the same table. It helps enforce referential integrity. Sales.CustomerID can reference Customers.CustomerID.

## 8. What is a transaction?

A transaction is a unit of work containing one or more operations. A bank transfer can group a debit and credit so that they succeed together or are rolled back together.

## 9. Explain ACID with a bank transfer.

Atomicity means the debit and credit succeed together. Consistency means the transaction preserves defined rules, such as the correct combined balance. Isolation governs interactions with simultaneous transactions. Durability means committed changes persist according to the database's guarantees.

## 10. How do COMMIT and ROLLBACK differ?

COMMIT completes a transaction and makes its changes permanent. ROLLBACK cancels uncommitted changes. A savepoint can mark a point for a partial rollback within a transaction.

## 11. How do relational and NoSQL databases differ?

Relational databases organize data in related tables. NoSQL includes document, key-value, graph, and wide-column systems. Their schemas, query interfaces, and transaction guarantees vary. Neither category is automatically faster or better for every workload.

## 12. How do a warehouse and a lake differ?

A warehouse typically contains curated data organized for analysis and reporting. A lake can hold data in varied formats, often including raw data. A lakehouse combines lake-style storage with warehouse-style management and analytical capabilities.

## 13. What is ETL?

ETL means Extract, Transform, Load. Data is collected from sources, prepared through steps such as cleaning or standardization, and loaded into a destination.

## 14. What is the logical processing order of a typical SELECT query?

A simplified order is FROM and joins, WHERE, GROUP BY, HAVING, SELECT, DISTINCT if present, ORDER BY, and row limiting. This describes the query's meaning rather than the optimizer's physical execution plan.

## 15. What is the difference between WHERE and HAVING?

WHERE filters rows before grouping. HAVING filters groups. WHERE Quantity >= 2 selects sales rows, while HAVING SUM(TotalAmount) > 1000 selects groups whose total exceeds 1,000.

## 16. What is normalization?

Normalization organizes relational data around dependencies to reduce unnecessary repetition and modification anomalies. In the lesson, separating customers from sales allows current customer details to be maintained in one place.

## 17. What are update, insertion, and deletion anomalies?

An update anomaly requires changing the same fact in multiple rows. An insertion anomaly makes it difficult to add a fact independently, such as a product without a sale. A deletion anomaly occurs when removing one fact also removes another, such as losing customer information when deleting the customer's only sale.

## 18. Why should an analyst understand table relationships?

Relationships determine how records match in joins. Unexpected multiple matches can duplicate amounts and inflate totals. I should understand the grain of each table and the uniqueness of its keys before joining.

## 19. Why preserve the price charged on a historical sale?

A product's current price can change. Historical sales should preserve the transaction's price or amount so that reports do not reinterpret old purchases using today's prices.

## 20. Does normalization always make queries faster?

No. It can reduce repeated data and improve consistency, but retrieving information may require additional joins. Performance depends on design, indexes, data volume, and workload. Analytical systems may deliberately use denormalized structures.

## Further-study questions

The supplied normalization deck introduces table separation but does not formally define normal forms. My next questions are:

- What is a functional dependency?
- What are 1NF, 2NF, and 3NF?
- How do partial and transitive dependencies differ?
- How does dimensional modeling relate to normalization?

[Back to Day 1](README.md)
