# Top 40 SQL Interview Questions with Answers

---

## 1. What is SQL?

**Answer:** SQL (Structured Query Language) is a standard language used to communicate with relational databases. It is used to perform tasks such as querying data, inserting records, updating records, deleting records, and creating and modifying database schema.

---

## 2. What are the different types of SQL commands?

**Answer:**
- **DDL (Data Definition Language):** `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
- **DML (Data Manipulation Language):** `INSERT`, `UPDATE`, `DELETE`
- **DQL (Data Query Language):** `SELECT`
- **DCL (Data Control Language):** `GRANT`, `REVOKE`
- **TCL (Transaction Control Language):** `COMMIT`, `ROLLBACK`, `SAVEPOINT`

---

## 3. What is the difference between DELETE, TRUNCATE, and DROP?

**Answer:**
| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| Removes | Rows (with WHERE) | All rows | Table structure + data |
| Rollback | Yes | No (in most DBs) | No |
| Speed | Slower | Faster | Fastest |
| Triggers | Fires | Does not fire | Does not fire |

---

## 4. What is a Primary Key?

**Answer:** A Primary Key is a column (or combination of columns) that uniquely identifies each row in a table. It cannot contain NULL values and must be unique.

```sql
CREATE TABLE Employee (
    EmpID INT PRIMARY KEY,
    Name VARCHAR(50)
);
```

---

## 5. What is a Foreign Key?

**Answer:** A Foreign Key is a column in one table that refers to the Primary Key in another table, enforcing referential integrity.

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    EmpID INT,
    FOREIGN KEY (EmpID) REFERENCES Employee(EmpID)
);
```

---

## 6. What is the difference between Primary Key and Unique Key?

**Answer:**
- **Primary Key:** Only one per table, does NOT allow NULL, creates a clustered index by default.
- **Unique Key:** Can have multiple per table, allows ONE NULL value, creates a non-clustered index by default.

---

## 7. What are JOINS in SQL?

**Answer:** JOINs combine rows from two or more tables based on related columns.

- **INNER JOIN:** Returns matching rows in both tables.
- **LEFT JOIN:** Returns all rows from left table and matched from right.
- **RIGHT JOIN:** Returns all rows from right table and matched from left.
- **FULL OUTER JOIN:** Returns all rows when there is a match in either table.
- **CROSS JOIN:** Cartesian product of both tables.
- **SELF JOIN:** Joining a table with itself.

---

## 8. What is the difference between WHERE and HAVING?

**Answer:**
- **WHERE:** Filters rows BEFORE grouping. Cannot use with aggregate functions.
- **HAVING:** Filters rows AFTER grouping. Used with aggregate functions.

```sql
SELECT Department, COUNT(*)
FROM Employee
WHERE Salary > 50000
GROUP BY Department
HAVING COUNT(*) > 5;
```

---

## 9. What is the difference between UNION and UNION ALL?

**Answer:**
- **UNION:** Combines result sets and removes duplicates.
- **UNION ALL:** Combines result sets and keeps duplicates (faster).

---

## 10. What are Aggregate Functions?

**Answer:** Functions that operate on a set of rows and return a single value.
- `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`

```sql
SELECT AVG(Salary) FROM Employee;
```

---

## 11. What are Scalar Functions?

**Answer:** Functions that operate on a single value and return a single value.
- `UPPER()`, `LOWER()`, `LEN()`, `ROUND()`, `NOW()`

---

## 12. What is a View?

**Answer:** A View is a virtual table based on the result set of a SQL query. It does not store data physically.

```sql
CREATE VIEW HighSalaryEmployees AS
SELECT Name, Salary FROM Employee WHERE Salary > 100000;
```

---

## 13. What is an Index?

**Answer:** An index is a database object that speeds up data retrieval. Types include:
- **Clustered Index:** Sorts and stores data rows physically.
- **Non-Clustered Index:** Logical ordering with pointers to actual data.
- **Unique Index:** Ensures no duplicate values.
- **Composite Index:** On multiple columns.

---

## 14. What is Normalization?

**Answer:** The process of organizing data to reduce redundancy and improve data integrity.
- **1NF:** Atomic values, no repeating groups.
- **2NF:** 1NF + no partial dependencies.
- **3NF:** 2NF + no transitive dependencies.
- **BCNF:** Stricter version of 3NF.

---

## 15. What is Denormalization?

**Answer:** The process of intentionally adding redundancy to improve read performance, often used in data warehouses.

---

## 16. What is a Stored Procedure?

**Answer:** A precompiled set of SQL statements stored in the database that can be executed when needed.

```sql
CREATE PROCEDURE GetEmployee(@id INT)
AS
BEGIN
    SELECT * FROM Employee WHERE EmpID = @id;
END;
```

---

## 17. What is a Trigger?

**Answer:** A trigger is a stored procedure that automatically executes when an event (INSERT, UPDATE, DELETE) occurs on a table.

```sql
CREATE TRIGGER trg_AfterInsert
ON Employee
AFTER INSERT
AS
BEGIN
    INSERT INTO AuditLog VALUES ('New employee added');
END;
```

---

## 18. What is the difference between CHAR and VARCHAR?

**Answer:**
- **CHAR(n):** Fixed-length, pads with spaces.
- **VARCHAR(n):** Variable-length, uses only required space.

---

## 19. What is a Subquery?

**Answer:** A query nested inside another query.

```sql
SELECT Name FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

Types: Single-row, Multi-row, Correlated, Nested.

---

## 20. What is a Correlated Subquery?

**Answer:** A subquery that depends on the outer query for its values.

```sql
SELECT e1.Name FROM Employee e1
WHERE Salary > (
    SELECT AVG(Salary) FROM Employee e2
    WHERE e1.DeptID = e2.DeptID
);
```

---

## 21. What is the difference between IN and EXISTS?

**Answer:**
- **IN:** Compares a value with a list/subquery result.
- **EXISTS:** Returns true if subquery returns any rows; usually faster for large datasets.

---

## 22. What is ACID property?

**Answer:**
- **Atomicity:** All or nothing.
- **Consistency:** Database remains in a valid state.
- **Isolation:** Transactions don't interfere.
- **Durability:** Committed data persists.

---

## 23. What is a Transaction?

**Answer:** A logical unit of work that contains one or more SQL statements executed as a single unit.

```sql
BEGIN TRANSACTION;
UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;
UPDATE Account SET Balance = Balance + 100 WHERE ID = 2;
COMMIT;
```

---

## 24. What are the different types of Constraints?

**Answer:**
- `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`

---

## 25. What is the difference between Clustered and Non-Clustered Index?

**Answer:**
| Clustered | Non-Clustered |
|-----------|---------------|
| Only one per table | Multiple per table |
| Sorts data physically | Logical ordering |
| Faster reads | Slightly slower |

---

## 26. What is the difference between RANK, DENSE_RANK, and ROW_NUMBER?

**Answer:**
- **ROW_NUMBER():** Unique sequential number.
- **RANK():** Same rank for ties, leaves gaps.
- **DENSE_RANK():** Same rank for ties, no gaps.

```sql
SELECT Name, Salary,
ROW_NUMBER() OVER(ORDER BY Salary DESC) AS RowNum,
RANK() OVER(ORDER BY Salary DESC) AS Rnk,
DENSE_RANK() OVER(ORDER BY Salary DESC) AS DenseRnk
FROM Employee;
```

---

## 27. What are Window Functions?

**Answer:** Functions that perform calculations across a set of rows related to the current row.
Examples: `ROW_NUMBER()`, `RANK()`, `LEAD()`, `LAG()`, `SUM() OVER()`.

---

## 28. What is a CTE (Common Table Expression)?

**Answer:** A temporary named result set used within a SELECT, INSERT, UPDATE, or DELETE statement.

```sql
WITH HighEarners AS (
    SELECT * FROM Employee WHERE Salary > 50000
)
SELECT * FROM HighEarners;
```

---

## 29. What is a Recursive CTE?

**Answer:** A CTE that references itself, used for hierarchical data.

```sql
WITH EmpHierarchy AS (
    SELECT EmpID, ManagerID, Name FROM Employee WHERE ManagerID IS NULL
    UNION ALL
    SELECT e.EmpID, e.ManagerID, e.Name FROM Employee e
    JOIN EmpHierarchy eh ON e.ManagerID = eh.EmpID
)
SELECT * FROM EmpHierarchy;
```

---

## 30. What is the difference between GROUP BY and ORDER BY?

**Answer:**
- **GROUP BY:** Groups rows that have the same values.
- **ORDER BY:** Sorts the result set.

---

## 31. What is a Self Join?

**Answer:** Joining a table with itself, useful for hierarchical data.

```sql
SELECT e1.Name AS Employee, e2.Name AS Manager
FROM Employee e1
JOIN Employee e2 ON e1.ManagerID = e2.EmpID;
```

---

## 32. What are Indexes' disadvantages?

**Answer:**
- Slow down INSERT, UPDATE, DELETE.
- Take additional storage.
- Need maintenance.

---

## 33. What is COALESCE?

**Answer:** Returns the first non-NULL value in a list.

```sql
SELECT COALESCE(NULL, NULL, 'Hello', 'World'); -- Returns 'Hello'
```

---

## 34. What is the difference between NULL and 0?

**Answer:** `NULL` represents an unknown/missing value, while `0` is a numeric value. NULL is not equal to anything (including itself).

---

## 35. What is the use of CASE statement?

**Answer:** Adds if-then-else logic to SQL queries.

```sql
SELECT Name,
CASE
    WHEN Salary > 100000 THEN 'High'
    WHEN Salary > 50000 THEN 'Medium'
    ELSE 'Low'
END AS SalaryGrade
FROM Employee;
```

---

## 36. What is a Composite Key?

**Answer:** A primary key composed of two or more columns.

```sql
CREATE TABLE OrderDetails (
    OrderID INT,
    ProductID INT,
    PRIMARY KEY (OrderID, ProductID)
);
```

---

## 37. What is the difference between INNER JOIN and OUTER JOIN?

**Answer:**
- **INNER JOIN:** Returns only matching rows.
- **OUTER JOIN (LEFT/RIGHT/FULL):** Returns matching rows + unmatched rows from one or both tables (with NULLs).

---

## 38. How do you find duplicate records?

**Answer:**
```sql
SELECT Name, COUNT(*)
FROM Employee
GROUP BY Name
HAVING COUNT(*) > 1;
```

---

## 39. How do you delete duplicate records but keep one?

**Answer:**
```sql
WITH CTE AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY Name, Email ORDER BY ID) AS rn
    FROM Employee
)
DELETE FROM CTE WHERE rn > 1;
```

---

## 40. What is the order of execution of SQL queries?

**Answer:**
1. `FROM`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `DISTINCT`
7. `ORDER BY`
8. `LIMIT / OFFSET`

---
