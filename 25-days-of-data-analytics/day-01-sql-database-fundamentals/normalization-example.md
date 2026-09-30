# Worked Example: Organizing Retail Sales Data

This walkthrough is based on the **Dream Retail Shop** example in the normalization lesson. It is a conceptual exercise showing how repeated customer and product details can be separated from sales records.

## 1. Original design

The lesson contains ten sales rows. This excerpt shows three purchases by the same customer. Phone values are omitted here because the repetition is already visible in the city column.

| OrderID | CustomerName | CustomerCity | ProductName | Price | Quantity | TotalAmount |
| --- | --- | --- | --- | ---: | ---: | ---: |
| 1 | Rahim Uddin | Dhaka | Laptop | 500 | 2 | 1000 |
| 3 | Rahim Uddin | Dhaka | Tablet | 200 | 3 | 600 |
| 8 | Rahim Uddin | Dhaka | Laptop | 500 | 1 | 500 |

Amounts come from the lesson. No currency is specified.

## 2. The problem

Suppose Rahim changes his city from Dhaka to Chittagong. If CustomerCity means his current city, every row containing his details needs updating.

Updating only Order 1 leaves conflicting current cities for the same customer. This is an **update anomaly**.

Other possible problems:

- A customer or product cannot easily be added independently if every row must represent a sale.
- Deleting a customer's only sale could remove the only stored customer details.
- Descriptive information repeats across transactions.

## 3. Separate customers, products, and sales

### Customers

| CustomerID | CustomerName | CustomerCity |
| --- | --- | --- |
| 101 | Rahim Uddin | Dhaka |
| 102 | Sultana Begum | Chittagong |
| 103 | Ayesha Sultana | Khulna |
| 104 | Mahmudul Hasan | Rajshahi |

CustomerID is the primary key. The full lesson also includes each customer's phone number.

### Products

| ProductID | ProductName | Price |
| --- | --- | ---: |
| 1 | Laptop | 500 |
| 2 | Mobile | 300 |
| 3 | Tablet | 200 |

ProductID is the primary key.

### Sales

| OrderID | CustomerID | ProductID | Quantity | TotalAmount |
| --- | --- | --- | ---: | ---: |
| 1 | 101 | 1 | 2 | 1000 |
| 2 | 102 | 2 | 1 | 300 |
| 3 | 101 | 3 | 3 | 600 |
| 4 | 103 | 1 | 1 | 500 |
| 5 | 102 | 3 | 2 | 400 |
| 6 | 104 | 2 | 4 | 1200 |
| 7 | 103 | 2 | 1 | 300 |
| 8 | 101 | 1 | 1 | 500 |
| 9 | 104 | 3 | 2 | 400 |
| 10 | 102 | 1 | 3 | 1500 |

In this simplified model, OrderID identifies one sales row. CustomerID and ProductID reference the corresponding customer and product records.

The deck calls these tables DimCustomer, DimProduct, and FactSales. This walkthrough uses shorter names. Fact and dimension naming also appears in dimensional modeling, which is related to, but distinct from, formal normalization.

## 4. What improves?

Rahim's current city now appears in one customer row. Updating that row changes the information retrieved through the relationship, while sales continue to reference CustomerID 101.

Customer identity remains stable even when descriptive attributes change.

**Important assumption:** City represents the customer's current city. A shipping destination for an old order is historical information and may need to be stored separately.

## 5. Reconstructing a report

This illustrative query assumes the tables already exist:

```sql
SELECT
    s.OrderID,
    c.CustomerName,
    c.CustomerCity,
    p.ProductName,
    s.Quantity,
    s.TotalAmount
FROM Sales AS s
JOIN Customers AS c
    ON s.CustomerID = c.CustomerID
JOIN Products AS p
    ON s.ProductID = p.ProductID
ORDER BY s.OrderID;
```

Each sale connects to one customer and one product. With unique referenced keys and valid references, these joins preserve one output row per sale.

The query shows how descriptions can be retrieved without repeating them in the sales table. This is a study example rather than a claim of completed SQL practice.

## 6. Reference totals for future practice

The lesson's ten sales rows contain **20 units** and a total sales amount of **6,700**.

| Customer | Sales total |
| --- | ---: |
| Rahim Uddin | 2100 |
| Sultana Begum | 2200 |
| Ayesha Sultana | 800 |
| Mahmudul Hasan | 1600 |
| **Total** | **6700** |

These manual totals provide a reference for checking a future SQL implementation.

## 7. Further design questions

### What happens when a product's price changes?

The product table may store its current price. A sale should preserve the amount charged at the time of purchase.

A possible extension is UnitPriceAtSale on each sales line. In a simplified example without tax or discounts:

```text
LineAmount = Quantity × UnitPriceAtSale
```

This prevents a new catalog price from changing the interpretation of historical sales. A fuller model may need discounts, taxes, returns, and rounding rules.

### What if an order contains several products?

The lesson assumes one product per order. A more general model could use:

```text
Orders(OrderID, CustomerID, OrderDate)
OrderItems(OrderID, LineNumber, ProductID, Quantity, UnitPriceAtSale)
```

The pair (OrderID, LineNumber) identifies an order line. One order can then contain multiple products.

### Does normalization always save storage?

The deck compares 80 cells before separation with 75 cells afterward. Cell counts do not measure bytes. Storage also depends on data types, value sizes, indexes, and database overhead.

The main demonstrated benefit is that customer and product details can be maintained independently, reducing repeated information and inconsistent updates.

## Takeaway

Before analyzing a table, I should identify what each row represents, which columns identify records, and how the table connects to others. These decisions affect both data quality and analytical accuracy.

[Back to Day 1](README.md)

