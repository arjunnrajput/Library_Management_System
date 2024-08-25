# Library Management System - SQL Project (P2)

## Overview

The Library Management System project is designed to demonstrate the implementation of a database system that manages a library's operations. This project covers database creation, CRUD operations, advanced SQL queries, and data analysis, providing a comprehensive understanding of SQL in a real-world scenario.

## Project Goals

1. **Database Setup**: Design and populate the `library_db` with relevant tables.
2. **CRUD Operations**: Implement Create, Read, Update, and Delete operations.
3. **CTAS (Create Table As Select)**: Generate new tables based on existing data.
4. **Advanced SQL Queries**: Execute complex queries to retrieve and analyze data.

## Database Setup

The following SQL scripts create the database and tables required for the Library Management System.

```sql
-- Create the library_db database
CREATE DATABASE library_db;

-- Create Branch table
DROP TABLE IF EXISTS branch;
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Create Employee table
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

-- Create Members table
DROP TABLE IF EXISTS members;
CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Create Books table
DROP TABLE IF EXISTS books;
CREATE TABLE books (
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

-- Create IssueStatus table
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status (
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(30),
    issued_book_name VARCHAR(80),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
    FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn)
);

-- Create ReturnStatus table
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

## CRUD Operations

This section covers basic CRUD (Create, Read, Update, Delete) operations to manage the records in the Library Management System.

### Insert a New Book

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```

### Update a Member's Address

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

### Delete a Record from Issued Status

```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```

### Retrieve Books Issued by a Specific Employee

```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

### List Members Who Have Issued More Than One Book

```sql
SELECT
    issued_emp_id,
    COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1;
```

## CTAS (Create Table As Select)

The CTAS operation allows for the creation of new tables based on existing query results.

### Create a Summary Table of Issued Books

```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist
JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```

## Data Analysis & Queries

This section includes advanced SQL queries to analyze the library's data.

### Retrieve Books in a Specific Category

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

### Calculate Total Rental Income by Category

```sql
SELECT 
    b.category,
    SUM(b.rental_price),
    COUNT(*)
FROM 
    issued_status as ist
JOIN
    books as b
ON 
    b.isbn = ist.issued_book_isbn
GROUP BY 1;
```

### List Members Registered in the Last 180 Days

```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

### Display Employees with Branch Manager and Branch Details

```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
FROM employees as e1
JOIN 
    branch as b
ON 
    e1.branch_id = b.branch_id    
JOIN
    employees as e2
ON 
    e2.emp_id = b.manager_id;
```

### Create a Table of Expensive Books

```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

### Retrieve List of Books Not Yet Returned

```sql
SELECT * FROM issued_status as ist
LEFT JOIN
    return_status as rs
ON 
    rs.issued_id = ist.issued_id
WHERE 
    rs.return_id IS NULL;
```

## Advanced SQL Operations

Advanced SQL operations include more complex procedures, reports, and analysis.

### Identify Members with Overdue Books

```sql
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date as over_due_days
FROM 
    issued_status as ist
JOIN 
    members as m
ON 
    m.member_id = ist.issued_member_id
JOIN 
    books as bk
ON 
    bk.isbn = ist.issued_book_isbn
LEFT JOIN 
    return_status as rs
ON 
    rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND
    (CURRENT_DATE - ist.issued_date) > 30
ORDER BY 1;
```

### Update Book Status on Return

```sql
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$
DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
BEGIN
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
END;
$$;
```

### Generate a Branch Performance Report

```sql
CREATE TABLE branch_reports
AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) as number_book_issued,
    COUNT(rs.return_id) as number_of_book_return,
    SUM(bk.rental_price) as total_revenue
FROM 
    issued_status as ist
JOIN 
    employees as e
ON 
    e.emp_id = ist.issued_emp_id
JOIN
    branch as b
ON 
    e.branch_id = b.branch_id
LEFT JOIN
    return_status as rs
ON 
    rs.issued_id = ist.issued_id
JOIN 
    books as bk
ON 
    ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;
```

### Create a Table of Active Members

```sql
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (SELECT DISTINCT issued_member_id FROM issued_status
                    WHERE issued_date >= CURRENT_DATE - INTERVAL '2 month');
```

### Find Employees with the Most Book Issues Processed

```sql
SELECT 
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) as total_book_issued
FROM 
    issued_status as ist
JOIN
    employees as e
ON 
    e.emp_id = ist.issued_emp_id
JOIN 
    branch as b
ON 
    b.branch_id = e.branch_id
GROUP BY 1, 2, 3, 4, 5, 6, 7, 8, 9
ORDER BY 6 DESC;
```

### Identify Members Frequently Issuing Damaged Books

```sql
CREATE TABLE frequent_damaged_book_issue AS
SELECT 
    rs.return_book_name,
    ist.issued_member_id,
    COUNT(*) AS issued_count
FROM 
    issued_status as ist
JOIN 
    return_status as rs
ON 
    ist.issued_id

 = rs.issued_id
WHERE 
    rs.book_quality = 'damaged'
GROUP BY 1, 2
HAVING COUNT(*) >= 2;
```

### Write a Stored Procedure to Manage Book Issuance

```sql
CREATE OR REPLACE PROCEDURE add_issue_records(p_issued_id VARCHAR(10), p_emp_id VARCHAR(10), p_member_id VARCHAR(10), p_isbn VARCHAR(50))
LANGUAGE plpgsql
AS $$
DECLARE
    v_book_title VARCHAR(80);
BEGIN
    INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_emp_id, issued_book_isbn)
    VALUES (p_issued_id, p_member_id, CURRENT_DATE, p_emp_id, p_isbn);

    UPDATE books
    SET status = 'no'
    WHERE isbn = p_isbn
    RETURNING book_title INTO v_book_title;

    RAISE NOTICE 'Book issued successfully. Book: %', v_book_title;
END;
$$;
```

### Generate a Table of Overdue Books with Fines

```sql
CREATE TABLE overdue_books_with_fines
AS
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    CURRENT_DATE - ist.issued_date as over_due_days,
    (CURRENT_DATE - ist.issued_date) * 0.20 as fine
FROM 
    issued_status as ist
JOIN 
    members as m
ON 
    m.member_id = ist.issued_member_id
JOIN 
    books as bk
ON 
    bk.isbn = ist.issued_book_isbn
LEFT JOIN 
    return_status as rs
ON 
    rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND
    (CURRENT_DATE - ist.issued_date) > 30;
```

## Reports

- **Database Schema**: Documentation of table structures and relationships.
- **Data Analysis**: Insight into various aspects, such as book categories, employee details, and membership trends.
- **Summary Reports**: Aggregated data on popular books and employee performance.

## Conclusion

This project showcases the application of SQL in managing and analyzing a library's database. It provides a comprehensive guide to setting up, maintaining, and extracting valuable insights from the system.

---
