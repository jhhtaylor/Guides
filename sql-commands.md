# SQL Server Concepts Demo - Employee/Department System

## 1. Database & Table Setup

    -- Create fresh database
    CREATE DATABASE Company;
    GO
    
    USE Company;
    GO
    
    -- Departments table with constraints
    CREATE TABLE Departments (
        DeptID INT PRIMARY KEY IDENTITY(1,1),
        DeptName VARCHAR(50) NOT NULL UNIQUE,
        Budget DECIMAL(15,2) CHECK (Budget > 0)
    );
    
    -- Employees table with relationships
    CREATE TABLE Employees (
        EmployeeID INT PRIMARY KEY IDENTITY(1,1),
        FirstName VARCHAR(50) NOT NULL,
        LastName VARCHAR(50) NOT NULL,
        Salary DECIMAL(10,2) CHECK (Salary > 30000),
        DeptID INT FOREIGN KEY REFERENCES Departments(DeptID)
    );
    GO

## 2. Data Population

    -- Insert departments
    INSERT INTO Departments (DeptName, Budget) VALUES
    ('IT', 150000),
    ('HR', 90000),
    ('Finance', 120000),
    ('Marketing', 95000); 
    
    -- Insert employees with various cases
    INSERT INTO Employees (FirstName, LastName, Salary, DeptID) VALUES
    ('Alice', 'Smith', 80000, 1),
    ('Bob', 'Johnson', 75000, 1),
    ('Carol', 'Williams', 82000, 1),
    ('David', 'Brown', 65000, 2),
    ('Eve', 'Davis', NULL, 3), -- Gotcha: NULL salary
    ('Frank', 'Miller', 85000, NULL); -- Gotcha: No department

## 3. Constraint Demonstrations

    -- Will fail: Duplicate department name
    INSERT INTO Departments (DeptName, Budget)
    VALUES ('IT', 160000);
    
    -- Will fail: Salary below minimum
    INSERT INTO Employees (FirstName, LastName, Salary)
    VALUES ('Grace', 'Wilson', 25000);

## 4. Query Concepts

### Basic SELECT
    SELECT * FROM Employees WHERE DeptID = 1;

### JOIN Operations
    -- INNER JOIN
    SELECT e.FirstName, d.DeptName
    FROM Employees e
    INNER JOIN Departments d ON e.DeptID = d.DeptID;
    
    -- LEFT JOIN with NULL handling
    SELECT e.FirstName, ISNULL(d.DeptName, 'No Department') AS Department
    FROM Employees e
    LEFT JOIN Departments d ON e.DeptID = d.DeptID;

### Aggregations
    -- Average salary per department
    SELECT d.DeptName, AVG(e.Salary) AS AvgSalary
    FROM Departments d
    LEFT JOIN Employees e ON d.DeptID = e.DeptID
    GROUP BY d.DeptName;

### Window Functions
    -- Salary ranking
    SELECT FirstName, LastName, Salary,
        RANK() OVER (ORDER BY Salary DESC) AS SalaryRank
    FROM Employees;

## 5. Programmable Objects

### Stored Procedure
    CREATE PROCEDURE GetDepartmentBudget
        @DeptName VARCHAR(50)
    AS
    BEGIN
        SELECT Budget 
        FROM Departments
        WHERE DeptName = @DeptName;
    END;
    
    -- Execute
    EXEC GetDepartmentBudget @DeptName = 'IT';

### View
    CREATE VIEW EmployeeSummary AS
    SELECT e.FirstName, e.LastName, d.DeptName
    FROM Employees e
    LEFT JOIN Departments d ON e.DeptID = d.DeptID;
    
    SELECT * FROM EmployeeSummary;

## 6. Advanced Features

### Transaction
    BEGIN TRY
        BEGIN TRANSACTION
            UPDATE Departments
            SET Budget = Budget * 1.1
            WHERE DeptID = 1;
            
            INSERT INTO Employees (FirstName, LastName, Salary, DeptID)
            VALUES ('Grace', 'Hopper', 95000, 1);
        COMMIT TRANSACTION
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION
        PRINT 'Transaction rolled back'
    END CATCH

### Indexing
    CREATE INDEX IX_Employees_Name
    ON Employees (LastName, FirstName);

## 7. Cleanup
    USE master;
    GO
    DROP DATABASE IF EXISTS Company;
    GO

# Key Learning Points
1. **Identity Columns**: Auto-increment even on failed inserts
2. **Constraint Types**: PRIMARY KEY, FOREIGN KEY, CHECK, UNIQUE
3. **Join Variations**: INNER vs LEFT vs RIGHT
4. **Transaction Safety**: ACID properties in action
5. **Performance**: Index usage and query plans

# Common Errors & Solutions
| Error | Solution |
|-------|----------|
| Foreign key violation | Check identity seed reset |
| NULL in aggregates | Use ISNULL/COALESCE |
| Duplicate key | Add unique constraint |
| Transaction deadlock | Optimize query order |

---

# SQL Server Comprehensive Demo: Every Major Concept  

## 1. Database Setup with Filegroups  

    CREATE DATABASE CompanyDemo  
    ON PRIMARY  
    (NAME = CompanyDemo_Data, FILENAME = '/var/opt/mssql/data/CompanyDemo.mdf'),  
    FILEGROUP ActiveData  
    (NAME = CompanyDemo_Active, FILENAME = '/var/opt/mssql/data/CompanyDemo_Active.ndf')  
    LOG ON  
    (NAME = CompanyDemo_Log, FILENAME = '/var/opt/mssql/data/CompanyDemo.ldf');  
    GO  

---

## 2. Table Creation with Constraints  

    CREATE TABLE Departments (  
        DeptID INT PRIMARY KEY,  
        DeptName VARCHAR(50) NOT NULL UNIQUE,  
        Budget DECIMAL(15,2)  
    );  

    CREATE TABLE Employees (  
        EmployeeID INT IDENTITY(1,1) PRIMARY KEY,  
        FirstName VARCHAR(50) NOT NULL CHECK (FirstName <> ''),  
        LastName VARCHAR(50) NOT NULL,  
        Salary DECIMAL(10,2) CHECK (Salary > 30000),  
        DeptID INT NULL FOREIGN KEY REFERENCES Departments(DeptID) -- Added DeptID  
    ) ON ActiveData;  

---

## 3. Data Population  

    INSERT INTO Departments (DeptID, DeptName, Budget) VALUES  
    (1, 'IT', 150000),  
    (2, 'HR', 90000);  

    INSERT INTO Employees (FirstName, LastName, Salary, DeptID) VALUES  
    ('Alice', 'Smith', 80000, 1),  
    ('Bob', 'Johnson', 75000, 1),  
    ('Carol', 'Williams', 82000, 1),  
    ('David', 'Brown', 65000, 2),  
    ('Eve', 'Davis', 70000, NULL);  

---

## 4. Complex Query Fix  

    WITH DeptSummary AS (  
        SELECT   
            d.DeptID,  
            AvgSalary = AVG(e.Salary),  
            TotalEmployees = COUNT(e.EmployeeID)  
        FROM Departments d  
        LEFT JOIN Employees e ON d.DeptID = e.DeptID  
        GROUP BY d.DeptID  
    )  
    SELECT  
        ds.DeptID,  
        d.DeptName,  
        ds.AvgSalary,  
        EmployeeRank = RANK() OVER (ORDER BY ds.AvgSalary DESC),  
        BudgetStatus = CASE   
            WHEN d.Budget > 100000 THEN 'High'   
            ELSE 'Normal'   
        END  
    FROM DeptSummary ds  
    JOIN Departments d ON ds.DeptID = d.DeptID  
    FOR JSON PATH;  

---

## 5. Key Changes from Previous Version  
1. Added `DeptID` column to Employees table  
2. Established proper foreign key relationship  
3. Updated data insertion with valid department IDs  
4. Fixed COUNT() to use explicit column  
5. Added NULL department example  

---

## 6. Expected JSON Output  

    [
        {
            "DeptID": 1,
            "DeptName": "IT",
            "AvgSalary": 79000.00,
            "EmployeeRank": 1,
            "BudgetStatus": "High"
        },
        {
            "DeptID": 2,
            "DeptName": "HR", 
            "AvgSalary": 65000.00,
            "EmployeeRank": 2,
            "BudgetStatus": "Normal"
        }
    ]

---

## 7. Common Errors & Solutions  
| Error | Solution |  
|-------|----------|  
| Invalid DeptID | Run DBCC CHECKIDENT on Departments |  
| NULL in joins | Use ISNULL(d.DeptName, 'Unassigned') |  
| Division errors | Add NULLIF(Budget, 0) |  

---

## 8. Quick Cleanup  

    USE master;  
    GO  
    DROP DATABASE IF EXISTS CompanyDemo;  
    GO  


## 9. Comprehensive Guaranteed Cleanup

    USE master;
    GO

    -- Force close connections and delete
    IF DB_ID('CompanyDemo') IS NOT NULL
    BEGIN
        ALTER DATABASE CompanyDemo SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
        DROP DATABASE CompanyDemo;
        PRINT 'CompanyDemo database destroyed successfully';
    END
    ELSE
    BEGIN
        PRINT 'CompanyDemo database does not exist';
    END