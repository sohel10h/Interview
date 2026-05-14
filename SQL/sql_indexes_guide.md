# SQL Indexes — Complete Guide with Scenarios & Examples

## What is an Index?

An **index** is a data structure (usually a B-Tree or Hash table) that the database maintains to speed up data retrieval — similar to the index at the back of a book. Without an index, the database performs a **full table scan** (reads every row). With an index, it can jump straight to matching rows.

**Trade-off:** Indexes speed up `SELECT` queries but slow down `INSERT`, `UPDATE`, `DELETE` (because the index must also be updated). They also use disk space.

---

## Sample Table (used throughout)

```sql
CREATE TABLE Employee (
    EmpID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    Phone VARCHAR(15),
    DeptID INT,
    Salary DECIMAL(10,2),
    HireDate DATE,
    Status VARCHAR(10),  -- 'Active', 'Inactive'
    City VARCHAR(50)
);
```

---

# Types of Indexes

## 1. Clustered Index

**What it is:** Determines the **physical order** of data in the table. The table rows are sorted and stored on disk in the order of the clustered index key.

**Key Rules:**
- Only **ONE clustered index per table** (because data can be physically sorted only one way).
- Automatically created on the `PRIMARY KEY` (in most databases).
- The leaf nodes of the B-tree are the actual data rows.

**Syntax:**
```sql
CREATE CLUSTERED INDEX idx_emp_id ON Employee(EmpID);
```

**When to Use:**
- On columns that are frequently used in **range queries** (`BETWEEN`, `>`, `<`).
- On columns used for `ORDER BY` and `GROUP BY`.
- On the **primary key** (default behavior).

### Scenario: Range Search on EmpID
```sql
-- Fast because data is physically sorted by EmpID
SELECT * FROM Employee WHERE EmpID BETWEEN 100 AND 200;
```

### Scenario: ORDER BY
```sql
-- Already sorted physically — no extra sort needed
SELECT * FROM Employee ORDER BY EmpID;
```

---

## 2. Non-Clustered Index

**What it is:** A separate structure from the table data. The index stores the key + a pointer (RowID/ClusteredKey) to the actual row. The table itself is NOT reordered.

**Key Rules:**
- A table can have **many non-clustered indexes** (typically up to 999 in SQL Server).
- Slower than clustered for range scans but excellent for **point lookups**.

**Syntax:**
```sql
CREATE NONCLUSTERED INDEX idx_emp_lastname ON Employee(LastName);
-- Or simply:
CREATE INDEX idx_emp_lastname ON Employee(LastName);
```

**When to Use:**
- On columns frequently used in `WHERE` clauses.
- On foreign key columns (helps joins).
- On columns used in `JOIN` conditions.

### Scenario: Lookup by Last Name
```sql
-- Index lets DB jump straight to matching rows
SELECT * FROM Employee WHERE LastName = 'Smith';
```

### Scenario: Join Performance
```sql
CREATE INDEX idx_emp_deptid ON Employee(DeptID);

-- Faster join: DeptID lookup uses the index
SELECT e.FirstName, d.DeptName
FROM Employee e
JOIN Department d ON e.DeptID = d.DeptID;
```

---

## 3. Unique Index

**What it is:** A non-clustered index that **enforces uniqueness** of values in the column(s). Acts like a `UNIQUE` constraint.

**Syntax:**
```sql
CREATE UNIQUE INDEX idx_emp_email ON Employee(Email);
```

**When to Use:**
- On columns that should never have duplicates (e.g., Email, SSN, Username, Phone).
- When you want both **uniqueness enforcement** AND **fast lookups**.

### Scenario: Prevent Duplicate Emails
```sql
INSERT INTO Employee (EmpID, Email) VALUES (1, 'a@x.com');  -- OK
INSERT INTO Employee (EmpID, Email) VALUES (2, 'a@x.com');  -- ERROR: duplicate key
```

### Scenario: Quick Login Check
```sql
-- Lightning fast — uniqueness + index lookup
SELECT * FROM Employee WHERE Email = 'john@company.com';
```

---

## 4. Composite Index (Multi-Column Index)

**What it is:** An index on **two or more columns**. The order of columns matters greatly.

**Syntax:**
```sql
CREATE INDEX idx_dept_salary ON Employee(DeptID, Salary);
```

**When to Use:**
- When queries filter or sort on multiple columns together.
- When you want to avoid maintaining several single-column indexes.

### Leftmost Prefix Rule (CRITICAL):
A composite index on `(DeptID, Salary)` will be used for:
- `WHERE DeptID = 10`                          ✅ Uses index
- `WHERE DeptID = 10 AND Salary > 50000`       ✅ Uses index
- `WHERE Salary > 50000`                       ❌ Does NOT use index (skips leftmost)

### Scenario: Department + Salary Filter
```sql
-- Optimized by idx_dept_salary
SELECT * FROM Employee
WHERE DeptID = 10 AND Salary > 60000;
```

### Scenario: Wrong Column Order Hurts
```sql
-- This query CANNOT use idx_dept_salary because DeptID is the leftmost column
SELECT * FROM Employee WHERE Salary > 60000;

-- Either reorder the index OR create a separate one on Salary
CREATE INDEX idx_salary ON Employee(Salary);
```

---

## 5. Covering Index

**What it is:** An index that contains **all the columns** needed by a query, so the database can answer the query using only the index without touching the table (called an "index-only scan").

**Syntax:**
```sql
-- Method 1: Composite covering index
CREATE INDEX idx_emp_covering ON Employee(DeptID, FirstName, LastName);

-- Method 2: INCLUDE clause (SQL Server, PostgreSQL)
CREATE INDEX idx_emp_covering ON Employee(DeptID) INCLUDE (FirstName, LastName);
```

**When to Use:**
- For frequently run reports/queries that select the same few columns.
- When you want to eliminate "key lookup" / "bookmark lookup" overhead.

### Scenario: Report Query
```sql
-- All required columns (DeptID, FirstName, LastName) are in the index
-- DB returns results without touching the actual table
SELECT FirstName, LastName FROM Employee WHERE DeptID = 5;
```

---

## 6. Filtered / Partial Index

**What it is:** An index built only on a **subset of rows** matching a `WHERE` predicate. Smaller, cheaper to maintain.

**Syntax:**
```sql
-- SQL Server
CREATE INDEX idx_active_emp ON Employee(LastName) WHERE Status = 'Active';

-- PostgreSQL
CREATE INDEX idx_active_emp ON Employee(LastName) WHERE Status = 'Active';
```

**When to Use:**
- When queries always filter on a specific condition (e.g., active users, recent orders).
- When most rows wouldn't be queried (saves space).

### Scenario: Only Active Employees Queried
```sql
-- If 90% of queries look only at active employees:
CREATE INDEX idx_active_lastname ON Employee(LastName) WHERE Status = 'Active';

-- This query uses the smaller, faster filtered index
SELECT * FROM Employee WHERE Status = 'Active' AND LastName = 'Smith';
```

---

## 7. Bitmap Index

**What it is:** Uses bitmaps (bit arrays) instead of B-trees. Each distinct value gets a bitmap showing which rows have that value.

**When to Use:**
- On columns with **low cardinality** (few distinct values), e.g., Gender, Status, Country.
- Mostly used in **data warehouses / OLAP** systems (Oracle supports it).
- Bad for OLTP systems with frequent updates (locking issues).

### Scenario: Gender / Status Column
```sql
-- Oracle syntax
CREATE BITMAP INDEX idx_emp_status ON Employee(Status);

-- Fast for analytical queries
SELECT COUNT(*) FROM Employee WHERE Status = 'Active' AND Gender = 'F';
```

---

## 8. Hash Index

**What it is:** Uses a hash table. Excellent for **equality lookups** (`=`), but cannot support range queries (`>`, `<`, `BETWEEN`).

**Syntax (PostgreSQL):**
```sql
CREATE INDEX idx_emp_email_hash ON Employee USING HASH (Email);
```

**When to Use:**
- For exact-match lookups only.
- In-memory databases (MySQL MEMORY engine uses hash by default).

### Scenario: Token / UUID Lookup
```sql
-- Hash index is great for this exact-match lookup
SELECT * FROM Sessions WHERE SessionToken = 'abc123xyz';

-- Hash index would NOT help here (range query):
SELECT * FROM Employee WHERE EmpID BETWEEN 100 AND 200;  -- Use B-tree instead
```

---

## 9. Full-Text Index

**What it is:** Special index for searching text content (words, phrases) within string columns. Supports linguistic features (stemming, stop-words).

**Syntax:**
```sql
-- SQL Server
CREATE FULLTEXT INDEX ON Articles(Content) KEY INDEX PK_Articles;

-- MySQL
CREATE FULLTEXT INDEX idx_content ON Articles(Content);

-- PostgreSQL (uses GIN on tsvector)
CREATE INDEX idx_content ON Articles USING GIN(to_tsvector('english', Content));
```

**When to Use:**
- Searching text inside large `TEXT` / `VARCHAR(MAX)` columns.
- Building search features (blog search, product description search).

### Scenario: Article Search
```sql
-- MySQL
SELECT * FROM Articles
WHERE MATCH(Content) AGAINST('database optimization' IN NATURAL LANGUAGE MODE);

-- Don't use LIKE '%word%' on large text — full-text is dramatically faster
```

---

## 10. Spatial Index

**What it is:** Designed for geographic/geometric data types (points, polygons).

**When to Use:**
- Maps, ride-sharing, location-based search ("find restaurants within 5 km").

### Scenario: Find Nearby Stores
```sql
CREATE SPATIAL INDEX idx_store_location ON Stores(Location);

SELECT * FROM Stores
WHERE ST_Distance(Location, POINT(40.7128, -74.0060)) < 5000;
```

---

# Decision Guide: Which Index to Use?

| Scenario | Recommended Index |
|----------|-------------------|
| Primary key / unique row identifier | **Clustered Index** (default on PK) |
| Frequently searched single column (`WHERE col = ?`) | **Non-Clustered Index** |
| Column must be unique (Email, Username) | **Unique Index** |
| Filter on multiple columns together | **Composite Index** (correct column order!) |
| Same few columns selected in many queries | **Covering Index** |
| Queries always filter by a fixed condition (e.g., `Status='Active'`) | **Filtered/Partial Index** |
| Low-cardinality column in data warehouse | **Bitmap Index** |
| Exact equality lookups only | **Hash Index** |
| Search text/phrases inside content | **Full-Text Index** |
| Geographic/coordinate searches | **Spatial Index** |

---

# Real-World Scenarios

## Scenario A: E-Commerce Order Lookup

**Tables:** `Orders(OrderID, CustomerID, OrderDate, Status, TotalAmount)`

**Common Queries:**
1. Look up order by OrderID — Primary Key = clustered ✅
2. List all orders by a customer — `WHERE CustomerID = ?`
3. Find recent pending orders — `WHERE Status = 'Pending' AND OrderDate > ?`

**Recommended Indexes:**
```sql
-- 1. Clustered (auto from PK)
-- 2. Non-clustered on CustomerID for customer lookup
CREATE INDEX idx_orders_customer ON Orders(CustomerID);

-- 3. Composite covering index for the dashboard query
CREATE INDEX idx_orders_status_date
    ON Orders(Status, OrderDate)
    INCLUDE (TotalAmount, CustomerID);
```

---

## Scenario B: User Authentication System

**Table:** `Users(UserID, Email, Username, PasswordHash, LastLogin)`

**Common Queries:**
1. Login by email — must be unique and fast.
2. Login by username — must be unique and fast.

**Recommended Indexes:**
```sql
CREATE UNIQUE INDEX idx_users_email ON Users(Email);
CREATE UNIQUE INDEX idx_users_username ON Users(Username);
```

Why unique? Enforces no duplicates + accelerates login lookups.

---

## Scenario C: Reporting on Active Customers Only

**Table:** `Customers(CustomerID, Name, Status, Region, JoinDate)` — 10M rows, only 1M are active.

**Common Queries:** All reports filter `WHERE Status = 'Active'`.

**Recommended Index:**
```sql
CREATE INDEX idx_active_region
    ON Customers(Region)
    WHERE Status = 'Active';
```

Why filtered? Index size is 10x smaller, faster to scan and cheaper to maintain.

---

## Scenario D: Multi-Column Search with Sort

**Table:** `Employee(EmpID, DeptID, Salary, HireDate)`

**Query:**
```sql
SELECT * FROM Employee
WHERE DeptID = 5
ORDER BY Salary DESC;
```

**Recommended Index:**
```sql
CREATE INDEX idx_dept_salary ON Employee(DeptID, Salary DESC);
```

Both the filter AND the sort are satisfied by the index → no separate sort step needed.

---

# When NOT to Use an Index

Don't index every column blindly. Skip indexes when:

1. **Small tables** — full scan is faster than index lookup.
2. **Columns with very few distinct values** (e.g., `IsActive` boolean) — unless used in filtered/bitmap context.
3. **Tables with heavy write traffic** — every INSERT/UPDATE pays an index-maintenance cost.
4. **Columns rarely queried** — wasted disk and memory.
5. **Columns frequently updated** — index reshuffling cost adds up.

---

# Common Pitfalls

### 1. Functions on indexed columns disable the index
```sql
-- ❌ Index on HireDate NOT used
SELECT * FROM Employee WHERE YEAR(HireDate) = 2024;

-- ✅ Rewrite to allow index use
SELECT * FROM Employee
WHERE HireDate >= '2024-01-01' AND HireDate < '2025-01-01';
```

### 2. Leading wildcard in LIKE disables index
```sql
-- ❌ Index NOT used
SELECT * FROM Employee WHERE LastName LIKE '%son';

-- ✅ Index used (prefix match)
SELECT * FROM Employee WHERE LastName LIKE 'Jo%';
```

### 3. Implicit type conversion disables index
```sql
-- ❌ EmpID is INT, comparing to a string forces conversion
SELECT * FROM Employee WHERE EmpID = '100';

-- ✅
SELECT * FROM Employee WHERE EmpID = 100;
```

### 4. OR conditions can prevent index use
```sql
-- ❌ Often slow
SELECT * FROM Employee WHERE LastName = 'Smith' OR FirstName = 'John';

-- ✅ UNION can be faster when each column is indexed
SELECT * FROM Employee WHERE LastName = 'Smith'
UNION
SELECT * FROM Employee WHERE FirstName = 'John';
```

---

# Tools to Verify Index Usage

```sql
-- SQL Server
SET STATISTICS IO ON;
SELECT * FROM Employee WHERE LastName = 'Smith';

-- MySQL / PostgreSQL
EXPLAIN SELECT * FROM Employee WHERE LastName = 'Smith';
EXPLAIN ANALYZE SELECT * FROM Employee WHERE LastName = 'Smith';
```

Look for **"Index Seek"** (good) vs **"Index Scan"** or **"Table Scan"** (potentially bad).

---

# Summary Cheat Sheet

| Index Type | Best For | Avoid For |
|------------|----------|-----------|
| Clustered | Range queries, PK lookups, ORDER BY | Frequently updated keys |
| Non-Clustered | WHERE filters, joins | Small tables |
| Unique | Email, Username, SSN | Non-unique data |
| Composite | Multi-column WHERE | Wrong column order |
| Covering | Same columns in many queries | Wide tables (large index) |
| Filtered | Skewed data (90% one value) | Evenly distributed data |
| Bitmap | Low cardinality, OLAP | OLTP with frequent writes |
| Hash | Exact equality lookups | Range queries |
| Full-Text | Searching text content | Short strings |
| Spatial | Geographic queries | Non-spatial data |

**Golden Rule:** *Index the columns in your `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY` clauses — but only ones that get queried often, and measure with `EXPLAIN` before/after.*
