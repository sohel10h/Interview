# SQL Common Coding Interview Questions with Answers

These are practical SQL coding problems frequently asked in interviews. Each problem includes a description, table schema, and solution.

---

## Sample Tables (used across many examples)

```sql
-- Employee Table
CREATE TABLE Employee (
    EmpID INT PRIMARY KEY,
    Name VARCHAR(50),
    Salary DECIMAL(10,2),
    DeptID INT,
    ManagerID INT,
    HireDate DATE
);

-- Department Table
CREATE TABLE Department (
    DeptID INT PRIMARY KEY,
    DeptName VARCHAR(50)
);
```

---

## Q1. Find the Second Highest Salary

**Question:** Write a query to find the second-highest salary from the Employee table.

**Solution 1 (using LIMIT):**
```sql
SELECT MAX(Salary) AS SecondHighest
FROM Employee
WHERE Salary < (SELECT MAX(Salary) FROM Employee);
```

**Solution 2 (using LIMIT/OFFSET):**
```sql
SELECT DISTINCT Salary
FROM Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1;
```

**Solution 3 (using DENSE_RANK):**
```sql
SELECT Salary FROM (
    SELECT Salary, DENSE_RANK() OVER (ORDER BY Salary DESC) AS rnk
    FROM Employee
) t WHERE rnk = 2;
```

---

## Q2. Find the Nth Highest Salary

**Question:** Find the Nth highest salary.

**Solution:**
```sql
SELECT Salary FROM (
    SELECT Salary, DENSE_RANK() OVER (ORDER BY Salary DESC) AS rnk
    FROM Employee
) t WHERE rnk = N;
```

---

## Q3. Find Duplicate Records

**Question:** Find all duplicate emails in the Employee table.

**Solution:**
```sql
SELECT Email, COUNT(*) AS CountOfEmail
FROM Employee
GROUP BY Email
HAVING COUNT(*) > 1;
```

---

## Q4. Delete Duplicate Rows but Keep One

**Question:** Delete duplicate rows but keep one occurrence.

**Solution:**
```sql
WITH CTE AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY Name, Email ORDER BY EmpID) AS rn
    FROM Employee
)
DELETE FROM CTE WHERE rn > 1;
```

---

## Q5. Find Employees Who Earn More Than Their Manager

**Question:** Display employees whose salary is more than their manager's.

**Solution:**
```sql
SELECT e.Name AS Employee, e.Salary, m.Name AS Manager, m.Salary AS ManagerSalary
FROM Employee e
JOIN Employee m ON e.ManagerID = m.EmpID
WHERE e.Salary > m.Salary;
```

---

## Q6. Find Departments With Highest Average Salary

**Question:** Find the department with the highest average salary.

**Solution:**
```sql
SELECT d.DeptName, AVG(e.Salary) AS AvgSalary
FROM Employee e
JOIN Department d ON e.DeptID = d.DeptID
GROUP BY d.DeptName
ORDER BY AvgSalary DESC
LIMIT 1;
```

---

## Q7. Find Top N Highest Paid Employees per Department

**Question:** Find top 3 highest paid employees in each department.

**Solution:**
```sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (PARTITION BY DeptID ORDER BY Salary DESC) AS rnk
    FROM Employee e
) t
WHERE rnk <= 3;
```

---

## Q8. Employees Who Joined in a Specific Year

**Question:** Find employees who joined in 2023.

**Solution:**
```sql
SELECT * FROM Employee
WHERE YEAR(HireDate) = 2023;
```

---

## Q9. Count Employees in Each Department

**Question:** Count number of employees in each department.

**Solution:**
```sql
SELECT d.DeptName, COUNT(e.EmpID) AS EmpCount
FROM Department d
LEFT JOIN Employee e ON d.DeptID = e.DeptID
GROUP BY d.DeptName;
```

---

## Q10. Find Departments With No Employees

**Question:** List departments that have no employees.

**Solution:**
```sql
SELECT d.DeptName
FROM Department d
LEFT JOIN Employee e ON d.DeptID = e.DeptID
WHERE e.EmpID IS NULL;
```

---

## Q11. Find Employees With the Same Salary

**Question:** Find pairs of employees having the same salary.

**Solution:**
```sql
SELECT a.Name, b.Name, a.Salary
FROM Employee a
JOIN Employee b ON a.Salary = b.Salary AND a.EmpID < b.EmpID;
```

---

## Q12. Cumulative Sum of Salaries

**Question:** Find a running total of salaries ordered by EmpID.

**Solution:**
```sql
SELECT EmpID, Name, Salary,
SUM(Salary) OVER (ORDER BY EmpID) AS RunningTotal
FROM Employee;
```

---

## Q13. Difference Between Current and Previous Salary (LAG)

**Question:** Show employee salaries along with the previous employee's salary.

**Solution:**
```sql
SELECT Name, Salary,
LAG(Salary, 1, 0) OVER (ORDER BY EmpID) AS PrevSalary,
Salary - LAG(Salary, 1, 0) OVER (ORDER BY EmpID) AS Diff
FROM Employee;
```

---

## Q14. Find the Median Salary

**Question:** Find the median salary of all employees.

**Solution (MySQL 8+ / PostgreSQL):**
```sql
SELECT AVG(Salary) AS Median FROM (
    SELECT Salary,
    ROW_NUMBER() OVER (ORDER BY Salary) AS rn,
    COUNT(*) OVER () AS cnt
    FROM Employee
) t
WHERE rn IN (FLOOR((cnt+1)/2), CEIL((cnt+1)/2));
```

---

## Q15. Swap Values in Two Columns (Gender Swap Problem)

**Question:** Swap 'M' and 'F' values in a gender column without using a temp column.

**Solution:**
```sql
UPDATE Employee
SET Gender = CASE
    WHEN Gender = 'M' THEN 'F'
    WHEN Gender = 'F' THEN 'M'
END;
```

---

## Q16. Find Customers Who Never Placed an Order

**Question:** Given `Customers` and `Orders` tables, find customers who never ordered.

**Solution:**
```sql
SELECT c.Name
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderID IS NULL;
```

---

## Q17. Department-Wise Highest Salary Employee

**Question:** Find the employee with the highest salary in each department.

**Solution:**
```sql
SELECT DeptID, Name, Salary
FROM (
    SELECT *, RANK() OVER (PARTITION BY DeptID ORDER BY Salary DESC) AS rnk
    FROM Employee
) t WHERE rnk = 1;
```

---

## Q18. Consecutive Numbers Problem

**Question:** From a `Logs(Id, Num)` table, find numbers that appear at least 3 times consecutively.

**Solution:**
```sql
SELECT DISTINCT l1.Num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.Id = l2.Id - 1
JOIN Logs l3 ON l1.Id = l3.Id - 2
WHERE l1.Num = l2.Num AND l2.Num = l3.Num;
```

---

## Q19. Find the Manager of Each Employee

**Question:** Show employee name with manager name (self join).

**Solution:**
```sql
SELECT e.Name AS Employee, m.Name AS Manager
FROM Employee e
LEFT JOIN Employee m ON e.ManagerID = m.EmpID;
```

---

## Q20. Employees Hired in Last 30 Days

**Question:** Find employees hired in the last 30 days.

**Solution:**
```sql
SELECT * FROM Employee
WHERE HireDate >= CURRENT_DATE - INTERVAL '30 days';
```

---

## Q21. Pivot Data (Department-wise Salary Sum)

**Question:** Display total salary per department as columns.

**Solution:**
```sql
SELECT
    SUM(CASE WHEN DeptID = 1 THEN Salary ELSE 0 END) AS HR,
    SUM(CASE WHEN DeptID = 2 THEN Salary ELSE 0 END) AS IT,
    SUM(CASE WHEN DeptID = 3 THEN Salary ELSE 0 END) AS Finance
FROM Employee;
```

---

## Q22. Find Employees Whose Name Starts With 'A'

**Solution:**
```sql
SELECT * FROM Employee WHERE Name LIKE 'A%';
```

---

## Q23. Find the Longest Name in a Table

**Solution:**
```sql
SELECT Name FROM Employee
ORDER BY LENGTH(Name) DESC
LIMIT 1;
```

---

## Q24. Find Employees Who Have the Same Name

**Solution:**
```sql
SELECT Name, COUNT(*) AS Cnt
FROM Employee
GROUP BY Name
HAVING COUNT(*) > 1;
```

---

## Q25. Display Even and Odd Numbered Rows

**Even rows:**
```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY EmpID) AS rn
    FROM Employee
) t WHERE rn % 2 = 0;
```

**Odd rows:**
```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY EmpID) AS rn
    FROM Employee
) t WHERE rn % 2 = 1;
```

---

## Q26. Find Maximum Salary Without Using MAX()

**Solution:**
```sql
SELECT Salary FROM Employee
ORDER BY Salary DESC
LIMIT 1;
```

---

## Q27. Find the Department With Lowest Average Salary

**Solution:**
```sql
SELECT d.DeptName, AVG(e.Salary) AS AvgSal
FROM Department d
JOIN Employee e ON d.DeptID = e.DeptID
GROUP BY d.DeptName
ORDER BY AvgSal ASC
LIMIT 1;
```

---

## Q28. Find the Date Difference Between Two Dates

**Solution:**
```sql
SELECT DATEDIFF('2026-12-31', '2026-01-01') AS Days;
```

---

## Q29. Get the First and Last Record From a Table

**First record:**
```sql
SELECT * FROM Employee ORDER BY EmpID ASC LIMIT 1;
```

**Last record:**
```sql
SELECT * FROM Employee ORDER BY EmpID DESC LIMIT 1;
```

---

## Q30. Find Employees With Salary Between Two Values

**Solution:**
```sql
SELECT * FROM Employee
WHERE Salary BETWEEN 30000 AND 60000;
```

---

## Q31. Find Employees Whose Name Contains 'ohn'

**Solution:**
```sql
SELECT * FROM Employee WHERE Name LIKE '%ohn%';
```

---

## Q32. Get Total Number of Departments

**Solution:**
```sql
SELECT COUNT(DISTINCT DeptID) FROM Employee;
```

---

## Q33. Update Salary of All Employees in a Department

**Solution:**
```sql
UPDATE Employee
SET Salary = Salary * 1.10
WHERE DeptID = 2;
```

---

## Q34. Find Employees Who Don't Have a Manager

**Solution:**
```sql
SELECT * FROM Employee WHERE ManagerID IS NULL;
```

---

## Q35. Find Total Salary Paid Per Year

**Solution:**
```sql
SELECT YEAR(HireDate) AS HireYear, SUM(Salary) AS TotalSalary
FROM Employee
GROUP BY YEAR(HireDate);
```

---

## Q36. Find Department Where Number of Employees > 5

**Solution:**
```sql
SELECT DeptID, COUNT(*) AS EmpCount
FROM Employee
GROUP BY DeptID
HAVING COUNT(*) > 5;
```

---

## Q37. Top 3 Earning Departments

**Solution:**
```sql
SELECT DeptID, SUM(Salary) AS TotalSalary
FROM Employee
GROUP BY DeptID
ORDER BY TotalSalary DESC
LIMIT 3;
```

---

## Q38. Get Employees With Their Rank by Salary

**Solution:**
```sql
SELECT Name, Salary,
RANK() OVER (ORDER BY Salary DESC) AS SalaryRank
FROM Employee;
```

---

## Q39. Customer Order Count (LEFT JOIN with Aggregation)

**Question:** Show each customer with the total number of orders placed.

**Solution:**
```sql
SELECT c.CustomerID, c.Name, COUNT(o.OrderID) AS TotalOrders
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.Name;
```

---

## Q40. Department with Maximum Employees

**Solution:**
```sql
SELECT DeptID, COUNT(*) AS EmpCount
FROM Employee
GROUP BY DeptID
ORDER BY EmpCount DESC
LIMIT 1;
```

---

## Q41. Find Employees Whose Salary is Greater Than Average

**Solution:**
```sql
SELECT * FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

---

## Q42. Find Salary Difference Between Highest and Lowest

**Solution:**
```sql
SELECT MAX(Salary) - MIN(Salary) AS SalaryGap FROM Employee;
```

---

## Q43. Display Names in Uppercase

**Solution:**
```sql
SELECT UPPER(Name) FROM Employee;
```

---

## Q44. Concatenate First and Last Name

**Solution:**
```sql
SELECT CONCAT(FirstName, ' ', LastName) AS FullName FROM Employee;
```

---

## Q45. Find Overlapping Date Ranges

**Question:** From a `Booking(StartDate, EndDate)` table, find overlapping bookings.

**Solution:**
```sql
SELECT b1.*, b2.*
FROM Booking b1
JOIN Booking b2 ON b1.BookingID < b2.BookingID
WHERE b1.StartDate <= b2.EndDate AND b2.StartDate <= b1.EndDate;
```

---

## Q46. Find Gaps in Sequential IDs

**Question:** Find missing IDs in a sequential series.

**Solution:**
```sql
SELECT (t1.ID + 1) AS MissingID
FROM Employee t1
LEFT JOIN Employee t2 ON t1.ID + 1 = t2.ID
WHERE t2.ID IS NULL
AND t1.ID < (SELECT MAX(ID) FROM Employee);
```

---

## Q47. Get the Day of the Week from a Date

**Solution:**
```sql
SELECT DAYNAME(HireDate) AS Day FROM Employee;
```

---

## Q48. Find Employees Who Joined on Weekends

**Solution:**
```sql
SELECT * FROM Employee
WHERE DAYOFWEEK(HireDate) IN (1, 7);  -- 1=Sunday, 7=Saturday
```

---

## Q49. Group Concatenation of Employee Names per Department

**Solution (MySQL):**
```sql
SELECT DeptID, GROUP_CONCAT(Name SEPARATOR ', ') AS Employees
FROM Employee
GROUP BY DeptID;
```

**Solution (PostgreSQL):**
```sql
SELECT DeptID, STRING_AGG(Name, ', ') AS Employees
FROM Employee
GROUP BY DeptID;
```

---

## Q50. Find Employees Hired After Their Manager

**Solution:**
```sql
SELECT e.Name AS Employee, e.HireDate, m.Name AS Manager, m.HireDate AS ManagerHireDate
FROM Employee e
JOIN Employee m ON e.ManagerID = m.EmpID
WHERE e.HireDate > m.HireDate;
```

---

## Tips for SQL Coding Interviews

1. **Always clarify schemas and edge cases** before writing the query.
2. **Use CTEs** to break complex problems into smaller readable steps.
3. **Prefer window functions** (RANK, ROW_NUMBER, LAG, LEAD) for ranking/comparison problems.
4. **Watch for NULL handling** — joins, aggregates, and comparisons often behave unexpectedly with NULL.
5. **Use indexes wisely** — filter early with WHERE, avoid functions on indexed columns.
6. **Know your database dialect** — MySQL, PostgreSQL, SQL Server, Oracle have slight differences (e.g., `LIMIT` vs `TOP` vs `ROWNUM`).
7. **Test with edge cases**: empty tables, duplicates, NULLs, single-row tables.

---
