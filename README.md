
import sqlite3

# Connect to SQLite Database
conn = sqlite3.connect("Ecommerce_SQL_Database.db")
cur = conn.cursor()

print("================================")
print("   SQL FOR DATA ANALYSIS")
print("================================")

# 1. Create Tables
cur.execute("""
CREATE TABLE IF NOT EXISTS Customers(
    customer_id INTEGER PRIMARY KEY,
    customer_name TEXT,
    city TEXT
)
""")

cur.execute("""
CREATE TABLE IF NOT EXISTS Products(
    product_id INTEGER PRIMARY KEY,
    product_name TEXT,
    price REAL
)
""")

cur.execute("""
CREATE TABLE IF NOT EXISTS Orders(
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_date TEXT,
    FOREIGN KEY(customer_id)
    REFERENCES Customers(customer_id)
)
""")

cur.execute("""
CREATE TABLE IF NOT EXISTS Order_Items(
    item_id INTEGER PRIMARY KEY,
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    FOREIGN KEY(order_id) REFERENCES Orders(order_id),
    FOREIGN KEY(product_id) REFERENCES Products(product_id)
)
""")

# 2. Insert Sample Data
cur.execute("DELETE FROM Order_Items")
cur.execute("DELETE FROM Orders")
cur.execute("DELETE FROM Products")
cur.execute("DELETE FROM Customers")

customers = [
    (1, "Ravi", "Hyderabad"),
    (2, "Sita", "Chennai"),
    (3, "Kiran", "Bangalore"),
    (4, "Priya", "Hyderabad")
]

products = [
    (1, "Laptop", 50000),
    (2, "Mobile", 20000),
    (3, "Headphones", 2000),
    (4, "Keyboard", 1500)
]

orders = [
    (101, 1, "2026-01-10"),
    (102, 2, "2026-01-12"),
    (103, 1, "2026-01-15"),
    (104, 3, "2026-01-18"),
    (105, 4, "2026-01-20")
]

items = [
    (1, 101, 1, 1),
    (2, 101, 3, 2),
    (3, 102, 2, 1),
    (4, 103, 4, 2),
    (5, 104, 1, 1),
    (6, 105, 2, 2),
    (7, 105, 3, 1)
]

cur.executemany("INSERT INTO Customers VALUES(?,?,?)", customers)
cur.executemany("INSERT INTO Products VALUES(?,?,?)", products)
cur.executemany("INSERT INTO Orders VALUES(?,?,?)", orders)
cur.executemany("INSERT INTO Order_Items VALUES(?,?,?,?)", items)

conn.commit()

# Function to display results
def show(title, query):
    print("\n" + "=" * 40)
    print(title)
    print("=" * 40)

    cur.execute(query)

    for row in cur.fetchall():
        print(row)


# 3. SELECT and WHERE
show("1. SELECT AND WHERE",
     "SELECT * FROM Customers WHERE city='Hyderabad'")

# 4. ORDER BY
show("2. ORDER BY",
     "SELECT * FROM Products ORDER BY price DESC")

# 5. GROUP BY and SUM
show("3. GROUP BY AND SUM",
     """
     SELECT customer_id, SUM(quantity)
     FROM Orders
     JOIN Order_Items USING(order_id)
     GROUP BY customer_id
     """)

# 6. INNER JOIN
show("4. INNER JOIN",
     """
     SELECT Customers.customer_name, Orders.order_id
     FROM Customers
     INNER JOIN Orders
     ON Customers.customer_id=Orders.customer_id
     """)

# 7. LEFT JOIN
show("5. LEFT JOIN",
     """
     SELECT Customers.customer_name, Orders.order_id
     FROM Customers
     LEFT JOIN Orders
     ON Customers.customer_id=Orders.customer_id
     """)

# 8. RIGHT JOIN equivalent (works on older SQLite)
show("6. RIGHT JOIN",
     """
     SELECT Customers.customer_name, Orders.order_id
     FROM Orders
     LEFT JOIN Customers
     ON Orders.customer_id=Customers.customer_id
     """)

# 9. Aggregate Functions
show("7. SUM AND AVG",
     """
     SELECT
     SUM(Products.price * Order_Items.quantity),
     AVG(Products.price)
     FROM Products
     JOIN Order_Items
     ON Products.product_id=Order_Items.product_id
     """)

# 10. Average Revenue Per User
show("8. AVERAGE REVENUE PER USER",
     """
     SELECT AVG(total_revenue)
     FROM (
         SELECT Orders.customer_id,
         SUM(Products.price * Order_Items.quantity)
         AS total_revenue
         FROM Orders
         JOIN Order_Items USING(order_id)
         JOIN Products USING(product_id)
         GROUP BY Orders.customer_id
     )
     """)

# 11. Subquery
show("9. SUBQUERY",
     """
     SELECT product_name, price
     FROM Products
     WHERE price > (
         SELECT AVG(price) FROM Products
     )
     """)

# 12. Create View
cur.execute("DROP VIEW IF EXISTS Sales_View")

cur.execute("""
CREATE VIEW Sales_View AS
SELECT Orders.order_id,
Customers.customer_name,
Products.product_name,
Order_Items.quantity,
Products.price * Order_Items.quantity AS revenue
FROM Orders
JOIN Customers USING(customer_id)
JOIN Order_Items USING(order_id)
JOIN Products USING(product_id)
""")

show("10. SQL VIEW",
     "SELECT * FROM Sales_View")

# 13. Create Index
cur.execute("""
CREATE INDEX IF NOT EXISTS
idx_customer_city
ON Customers(city)
""")

print("\n11. INDEX CREATED SUCCESSFULLY")

# 14. Handle NULL Values
show("12. NULL VALUE HANDLING",
     """
     SELECT customer_name,
     COALESCE(city, 'Unknown')
     FROM Customers
     """)

# 15. Total Sales
show("13. TOTAL SALES",
     """
     SELECT SUM(revenue)
     FROM Sales_View
     """)

print("\n================================")
print(" SQL DATA ANALYSIS COMPLETED")
print("================================")

conn.close()
# Data-analyst-Task-3