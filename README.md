# Experiment on Neo4j Northwind example

## Step 1: Create a database
```
create database northwind
:use northwind
```

## Step 2: Load product table
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/products.csv" AS row
CREATE (n:Product)
SET n = row,
n.unitPrice = toFloat(row.unitPrice),
n.unitsInStock = toInteger(row.unitsInStock), n.unitsOnOrder = toInteger(row.unitsOnOrder),
n.reorderLevel = toInteger(row.reorderLevel), n.discontinued = (row.discontinued <> "0")
```

## Step 3: Load category table
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/categories.csv" AS row
CREATE (n:Category)
SET n = row
```

## Step 4: Load supplier table
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/suppliers.csv" AS row
CREATE (n:Supplier)
SET n = row
```

## Step 5: Create indexes
```
CREATE INDEX FOR (p:Product) ON (p.productID)
CREATE INDEX FOR (p:Product) ON (p.productName)
CREATE INDEX FOR (c:Category) ON (c.categoryID)
CREATE INDEX FOR (s:Supplier) ON (s.supplierID)
```

## Step 6: Create relationships
```
MATCH (p:Product),(c:Category)
WHERE p.categoryID = c.categoryID
CREATE (p)-[:PART_OF]->(c)
```
```
MATCH (p:Product),(s:Supplier)
WHERE p.supplierID = s.supplierID
CREATE (s)-[:SUPPLIES]->(p)
```

## Step 7: Query
- What categories of food does each supplier supply?
```
MATCH (s:Supplier)-->(:Product)-->(c:Category)
RETURN s.companyName as Company, collect(distinct c.categoryName) as Categories
```
- How to find the produce suppliers?
```
MATCH (c:Category {categoryName:"Produce"})<--(:Product)<--(s:Supplier)
RETURN DISTINCT s.companyName as ProduceSuppliers
```

## Step 8: Load customer data
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/customers.csv" AS row
CREATE (n:Customer)
SET n = row
```

## Step 9: Load order data
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/orders.csv" AS row
CREATE (n:Order)
SET n = row
```

## Step 10: Create indexes for customer and order
```
CREATE INDEX FOR (n:Customer) ON (n.customerID)
```
```
CREATE INDEX FOR (o:Order) ON (o.orderID)
```

## Step 11: Create relationship between customer and order
```
MATCH (n:Customer),(o:Order)
WHERE n.customerID = o.customerID
CREATE (n)-[:PURCHASED]->(o)
```

## Step 12: Promote OrderDetail record into a relationship
```
LOAD CSV WITH HEADERS FROM "https://data.neo4j.com/northwind/order-details.csv" AS row
MATCH (p:Product), (o:Order)
WHERE p.productID = row.productID AND o.orderID = row.orderID
CREATE (o)-[details:ORDERS]->(p)
SET details = row,
details.quantity = toInteger(row.quantity)
```

## Step 13: Try some more queries
Sample query: how many products did each customer purchase?
```
MATCH (cust:Customer)-[:PURCHASED]->(:Order)-[o:ORDERS]->(p:Product),
  (p)-[:PART_OF]->(c:Category {categoryName:"Produce"})
RETURN DISTINCT cust.contactName as CustomerName, SUM(o.quantity) AS TotalProductsPurchased
```
