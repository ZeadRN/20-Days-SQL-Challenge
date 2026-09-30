# Day 1: Interview Questions and Simple Answers

These questions help me revise SQL and database fundamentals. My goal is to understand each answer and explain it in my own words.

## 1. What is the difference between a database and a DBMS?

A **database** is an organized collection of data.

A **DBMS** is software that helps us store, find, and change that data.

**Example:** A shop's customer records are data in a database. MySQL is software that can manage the database.

## 2. What is an RDBMS?

An RDBMS is a type of DBMS that stores data in related tables.

Each table has rows and columns. Keys help connect the tables.

**Example:** A Customers table and an Orders table can be connected using CustomerID.

MySQL and PostgreSQL are examples of an RDBMS.

## 3. What is the difference between SQL and MySQL?

**SQL** is a language used to work with data in databases.

**MySQL** is database software that understands SQL.

We write SQL commands and run them in a system such as MySQL.

## 4. What are the main SQL command categories?

| Category | Full Name | What It Does | Examples |
|---|---|---|---|
| DDL | Data Definition Language | Creates or changes database structures | CREATE, ALTER, DROP |
| DQL | Data Query Language | Reads data | SELECT |
| DML | Data Manipulation Language | Adds, changes, or removes records | INSERT, UPDATE, DELETE |
| DCL | Data Control Language | Controls access | GRANT, REVOKE |
| TCL | Transaction Control Language | Controls transactions | COMMIT, ROLLBACK, SAVEPOINT |

**Note:** Some references include SELECT under DML instead of using a separate DQL category.

## 5. What is the difference between DELETE, TRUNCATE, and DROP?

- **DELETE:** Removes rows. We can use WHERE to choose which rows to remove.
- **TRUNCATE:** Removes all rows but keeps the table structure.
- **DROP TABLE:** Removes the entire table, including its structure and data.

**Example:** Think of a table as a notebook:

- DELETE removes selected entries.
- TRUNCATE clears all entries but keeps the notebook.
- DROP removes the notebook itself.

Whether these actions can be rolled back depends on the database system and transaction context.

## 6. What is a primary key?

A primary key identifies each row in a table.

Its values must be unique and cannot be NULL. NULL means a missing or unknown value.

**Example:** Each customer can have a different CustomerID.

Names are not a good choice because two customers can have the same name.

A primary key can also use more than one column together.

## 7. What is a foreign key?

A foreign key connects records by referring to a key in another table. It can also refer to the same table.

**Example:** CustomerID in the Orders table refers to CustomerID in the Customers table.

When this rule is enforced, an order cannot refer to a customer ID that does not exist.

## 8. What is a transaction?

A transaction is one or more database operations treated as a single unit of work.

**Example:** Transferring money requires two changes:

1. Subtract money from one account.
2. Add money to another account.

These changes should succeed together. If the transfer fails, its changes should be undone.

## 9. What are ACID properties?

ACID describes four properties that help database transactions work reliably.

### Atomicity

All changes in a transaction succeed together, or they are undone together.

**Example:** A bank transfer should not subtract money from one account without adding it to the other.

### Consistency

A transaction must follow the database's defined rules.

**Example:** A transfer without fees should keep the combined balance of the two accounts unchanged.

The transaction must be written correctly, and the necessary rules must be defined.

### Isolation

Isolation controls how transactions affect each other when they run at the same time.

**Example:** If two people withdraw money from the same account, the database needs rules for handling those overlapping operations.

Different isolation levels provide different protections.

### Durability

Once a transaction is committed, its changes should remain saved, including after a system restart or crash covered by the database's guarantees.

**Example:** A completed transfer should still be recorded after the database restarts.

## 10. What is the difference between COMMIT, ROLLBACK, and SAVEPOINT?

- **COMMIT:** Finishes a transaction and saves its changes permanently.
- **ROLLBACK:** Undoes changes that have not been committed.
- **SAVEPOINT:** Marks a point inside a transaction that we can return to.

**Example:** We can create a savepoint before changing a salary. If we undo that salary change, earlier changes in the transaction can still remain.

## 11. What is the difference between relational and NoSQL databases?

Relational databases organize data in related tables.

NoSQL databases use models such as documents, key-value pairs, graphs, or wide-column structures.

**Examples:**

- Relational: MySQL and PostgreSQL.
- NoSQL: MongoDB and Neo4j.

The better choice depends on the data and how it will be used. NoSQL is not automatically faster, and some NoSQL systems support ACID transactions.

## 12. What is the difference between a data warehouse, a data lake, and a lakehouse?

A **data warehouse** usually stores cleaned and organized data for reports and analysis.

A **data lake** can store many types of data, often in their original form.

A **lakehouse** combines lake-style storage with features that help manage data and run reliable analysis.

**Example:**

- Warehouse: Monthly sales data prepared for reporting.
- Lake: Sales files, website logs, images, and customer feedback.
- Lakehouse: Lake storage with managed tables for reporting and other uses.

## 13. What is ETL?

ETL stands for **Extract, Transform, Load**.

1. **Extract:** Collect data from its sources.
2. **Transform:** Clean or change the data into the needed format.
3. **Load:** Store the prepared data in the destination.

**Example:** Collect sales files from several branches, standardize their date formats, and load them into a warehouse.

## 14. What is the logical processing order of a SELECT query?

A simplified order is:

1. FROM and JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. DISTINCT, if used
7. ORDER BY
8. LIMIT or another row limit

This order helps explain how the query's result is formed.

The database may use a different internal execution plan to produce that result.

## 15. What is the difference between WHERE and HAVING?

**WHERE** filters individual rows before grouping.

**HAVING** filters groups after grouping.

**Example:**

- WHERE keeps sales where Quantity is at least 2.
- HAVING keeps customer groups whose total sales are above 1,000.

## 16. What is normalization?

Normalization is a way of organizing database tables to reduce unnecessary repeated data and prevent problems when data changes.

**Example:** Instead of writing a customer's current address in every sales row, we store it once in a Customers table.

Sales records then connect to that customer using CustomerID.

## 17. What are update, insertion, and deletion anomalies?

An anomaly is a problem that can happen because of how data is organized.

### Update anomaly

The same information needs to be changed in several rows.

**Example:** A customer's phone number appears in five sales rows. Updating only four leaves conflicting phone numbers.

### Insertion anomaly

We cannot easily add one piece of information without adding unrelated information.

**Example:** A table requires sale details, so we cannot record a new product until someone buys it.

### Deletion anomaly

Deleting one record also removes other information we still need.

**Example:** Deleting a customer's only sale removes the only stored copy of their contact details.

## 18. Why should a data analyst understand table relationships?

Relationships tell us how records match when we join tables.

If a row matches more records than expected, an amount may appear multiple times and make the total too high.

Before joining tables, I should check:

- What does one row represent?
- Which columns identify a record?
- How many matches should each row have?

## 19. Why should we keep the price charged at the time of a sale?

Product prices can change.

A sales report should use the price the customer actually paid.

**Example:** A laptop sold for 500 last month. Its price is now 550. Last month's sale should still be reported using 500.

## 20. Does normalization always make queries faster?

No.

Normalization can reduce repeated information and make updates easier. But reading the data may require joining more tables.

Query speed also depends on factors such as indexes, data size, and how the query is written.

## Topics to Study Next

The normalization lesson introduced splitting data into related tables. My next topics are:

- Functional dependencies
- First Normal Form: 1NF
- Second Normal Form: 2NF
- Third Normal Form: 3NF
- Normalization versus dimensional modeling

## How I Will Practice

1. Read a question.
2. Cover the answer.
3. Explain it aloud in a few sentences.
4. Give one example.
5. Check the answer and correct anything I missed.

[Back to Day 1](README.md)
