# DBS302 Practical 3 Report

## Title
Design and Implement an E-Commerce Platform Schema in MongoDB, Use Aggregation Framework, and Optimize Query Performance

## Aim
To design and implement an e-commerce platform schema using MongoDB, write advanced aggregation queries, and apply indexing plus query plan analysis to optimize performance.

## Objectives Achieved
1. Modeled a realistic e-commerce domain with users, categories, products, and orders.
2. Implemented the schema as MongoDB collections and inserted representative sample data.
3. Built non-trivial aggregation pipelines for business analytics.
4. Created indexes (compound and text) aligned to query patterns.
5. Used explain("executionStats") to validate optimization impact.

## Brief Design Rationale
1. Embedding vs Referencing
Order items are embedded in orders because they are read together and bounded per order.
Users, products, and categories are referenced by ObjectId to avoid duplication of shared entities.

2. Attribute Pattern for Catalog Flexibility
The products collection uses an attributes object for variable product properties (brand, color, switchType, etc.), which supports heterogeneous product categories.

3. Query-First Design
Indexes were created based on frequent read patterns such as status/date filtering, user order history, category listing, and product text search.

## Part 1: Database and Collection Setup
### 1.1 Create Database
```javascript
use ecommerce
```

### 1.2 Create Collections
Created collections: users, categories, products, orders.

<img src="screenshots/Create collections.png" alt="Create collections command" width="60%"/>

This screenshot shows the commands used to create the required MongoDB collections for the e-commerce schema.

![alt text](<screenshots/ecommerce database.png>)

This output confirms that the `ecommerce` database is active and ready for data insertion.

## Part 2: Insert Sample Data
### 2.1 Insert Users
Inserted two users: Tashi Dorji and Sonam Choden.

<img src="screenshots/Insert Users.png" alt="Insert users command" width="60%"/>

This screenshot shows the insertMany command used to add user documents with profile and address details.

![alt text](<screenshots/inserted user data.png>)

This output verifies that both user records were successfully inserted into the users collection.

### 2.2 Insert Categories
Inserted Electronics and Accessories categories using reusable ObjectId variables.

![alt text](<screenshots/Insert Categories.png>)

This screenshot shows category insertion with predefined ObjectId values for later product references.

![alt text](<screenshots/inserted categories.png>)

This confirms that Electronics and Accessories categories were inserted correctly.

### 2.3 Insert Products
Inserted three products with variable attributes via the Attribute Pattern.

<img src="screenshots/Insert Products.png" alt="Insert products command" width="60%"/>

This screenshot captures product insertion using flexible `attributes` fields for different product properties.

![alt text](<screenshots/inserted product.png>)

This output confirms all sample products were added with category links and pricing details.

### 2.4 Insert Orders
Inserted two orders with embedded order items and historical pricing stored in each item.


<img src="screenshots/Insert Orders.png" alt="Insert orders command" width="60%"/>

This screenshot shows order creation with embedded items, quantities, line totals, and payment details.

![alt text](<screenshots/inserted orders.png>)

This output verifies order documents were inserted successfully for both users.

## Part 3: Basic Retrieval Queries
### 3.1 Retrieve All Users
<img src="screenshots/Retrieve All Users.png" alt="Retrieve all users" width="60%"/>

This query output displays all documents in the users collection for data verification.

### 3.2 Retrieve All Products
<img src="screenshots/RetrieveAllProducts.png" alt="Retrieve all products" width="60%"/>

This output lists all products and confirms catalog data was inserted as expected.

### 3.3 Retrieve Specific User Order
<img src="screenshots/RetrieveUserOrder.png" alt="Retrieve user order" width="60%"/>

This screenshot shows how orders are filtered for a specific user to validate order linkage.

## Part 4: Aggregation Queries for Analytics
### 4.1 Daily Sales Totals
Uses $match, $group, $project, and $sort to compute daily revenue and order counts.

<img src="screenshots/Query 1_Daily Sales Totals.png" alt="Daily sales totals aggregation" width="60%"/>

This result summarizes total revenue and order count per day from paid orders.

### 4.2 Top Products by Revenue
Uses $unwind + $group + $sort + $limit to identify top-performing products.

<img src="screenshots/Query 2_Top Products by Revenue.png" alt="Top products aggregation" width="60%"/>

This output ranks products by total revenue and total quantity sold.

### 4.3 Average Order Value per Customer
Uses $group + $lookup + $project for user-level spending analytics.

<img src="screenshots/Query 3_Average Order.png" alt="Average order value aggregation" width="60%"/>

This result computes each customer's total spent, minimum, maximum, and average order value.

### 4.4 Product Catalog View with Category Name
Uses $lookup and $project to enrich products with category names.

<img src="screenshots/Query 4_Product Catalog.png" alt="Product catalog with category aggregation" width="60%"/>

This output joins products with categories to show enriched catalog information.

## Part 5: Indexing for Performance Optimization
### 5.1 Orders by User and Date
Index created for user order history sorted by recent date.

<img src="screenshots/Index 1_Orders by User and Date.png" alt="Index on orders by user and date" width="60%"/>

This screenshot confirms creation of a compound index for fast user order history queries.

### 5.2 Orders by Status and Date (ESR-aligned)
Compound index created to support status filter and createdAt sorting.

<img src="screenshots/Index 2.png" alt="Compound index for status and date" width="60%"/>

This index supports status filtering and date sorting, aligned with frequent operational queries.

### 5.3 Products by Category and Price
Compound index created for category filtering and price sort.

<img src="screenshots/Index 3.png" alt="Index on products category and price" width="60%"/>

This screenshot shows the index used to optimize product listing by category and price.

### 5.4 Text Index for Product Search
Weighted text index created on product name and tags.

<img src="screenshots/Index 4.png" alt="Text index creation" width="60%"/>

This confirms creation of a weighted text index for product keyword search.

### 5.5 Text Search Execution
Text search query tested using relevance score sorting.

<img src="screenshots/Query 5_Text Search.png" alt="Text search query output" width="60%"/>

This output demonstrates text search results ranked by text relevance score.

## Part 6: Query Plan Analysis with explain("executionStats")
### 6.1 Before Index (COLLSCAN)
Query initially performed collection scan.

<img src="screenshots/Part A_1 winningPlan.png" alt="Winning plan before index" width="60%"/>

This plan shows the pre-optimization execution path, indicating slower scan behavior.

<img src="screenshots/Part A_2 executionStats.png" alt="Execution stats before index" width="60%"/>

These statistics provide baseline metrics before index optimization.

### 6.2 Create Supporting Index
Created status + createdAt index.

<img src="screenshots/Part C Recreate the index.png" alt="Create status-date index" width="60%"/>

This screenshot shows the index creation step used to optimize status and date-based queries.

### 6.3 After Index (IXSCAN)
Query plan changed to index scan.

<img src="screenshots/Part D executionStats.png" alt="Execution stats after index" width="60%"/>

These post-index metrics show improved efficiency and reduced scanning cost.

<img src="screenshots/Part D winningPlan section.png" alt="Winning plan after index" width="60%"/>

This winning plan confirms index usage after optimization.

### 6.4 Before vs After Summary
| Metric | Before | After |
|---|---|---|
| Winning stage | COLLSCAN | IXSCAN |
| totalDocsExamined | High / full scan behavior | Reduced |
| totalKeysExamined | 0 | > 0 (index keys examined) |
| executionTimeMillis | Higher | Lower |
| indexName | None | idx_orders_status_createdAt |

This confirms that the index reduced scan cost and improved query efficiency.

### 6.5 Verify Available Indexes
<img src="screenshots/Part E View All Indexes.png" alt="View all indexes" width="60%"/>

This output verifies all active indexes on the collection after tuning.

### 6.6 Validate Text Search
<img src="screenshots/Part F Test Text Search.png" alt="Text search validation" width="60%"/>

This screenshot validates that the text index works correctly with practical search terms.

## Reflection
1. The document model fit e-commerce workloads well by combining embedded order items with referenced master data.
2. Aggregation pipelines provided business insights directly from operational collections.
3. Performance validation using explain("executionStats") was essential to prove index impact instead of assuming optimization.

## Conclusion
In this practical, we designed a MongoDB database for an e-commerce system using a query-first approach. We created four main collections: users, categories, products, and orders. Some data was embedded (like order items inside orders) because it is always used together, while other data (like products) was referenced to avoid duplication.

We also built aggregation queries to get useful insights, such as total daily sales, top-selling products, and average order value. These queries used stages like filtering, grouping, and joining data from different collections.

A key part of this practical was improving performance using indexes. We created different indexes to speed up common queries, and used explain **("executionStats")** to compare performance before and after. This showed a clear improvement, changing from full collection scans to faster index scans.

Finally, we avoided common mistakes like storing too much data in one place or using indexes incorrectly. Overall, this practical showed how to design an efficient and scalable MongoDB database for real-world use.