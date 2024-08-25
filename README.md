# Library Management System - SQL Project (P2)

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the development of a Library Management System using SQL. It encompasses creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The purpose of this project is to highlight database design, manipulation, and querying skills.

## Project Goals

- **Database Setup**: Design and populate the `library_db` with tables for branches, employees, members, books, and tracking the issued and returned status of books.
- **CRUD Operations**: Implement Create, Read, Update, and Delete operations on the records.
- **CTAS (Create Table As Select)**: Generate new tables from query results using CTAS.
- **Advanced SQL Queries**: Execute complex queries to extract and analyze specific data.

## Project Structure

### 1. Database Setup

- **Database Creation**: Initialize the database and define its structure.

```sql
CREATE DATABASE library_db;

-- Create table "Branch"
DROP TABLE IF EXISTS branch;
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Create table "Books"
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

-- Create table "IssueStatus"
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

-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
2. CRUD Operations
Create: Insert new records into the tables.
Read: Fetch data from the tables.
Update: Modify existing records.
Delete: Remove records from the tables.
Examples:
Task 1: Insert a new book record into the books table.
sql
Copy code
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
Task 2: Update an existing member's address.
sql
Copy code
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
Task 3: Delete a record from the issued_status table.
sql
Copy code
DELETE FROM issued_status
WHERE issued_id = 'IS121';
Task 4: Retrieve all books issued by a specific employee.
sql
Copy code
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
Task 5: List members who have issued more than one book.
sql
Copy code
SELECT
    issued_emp_id,
    COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1;
3. CTAS (Create Table As Select)
Task 6: Create summary tables using CTAS to count issued books per title.
sql
Copy code
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist
JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
4. Data Analysis & Findings
Task 7: Retrieve all books in a specific category.
sql
Copy code
SELECT * FROM books
WHERE category = 'Classic';
Task 8: Calculate total rental income by category.
sql
Copy code
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
Task 9: List members who registered within the last 180 days.
sql
Copy code
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
Task 10: Display employees with their branch manager's name and branch details.
sql
Copy code
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
Task 11: Create a table of books with rental price above a certain threshold.
sql
Copy code
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
Task 12: Retrieve the list of books not yet returned.
sql
Copy code
SELECT * FROM issued_status as ist
LEFT JOIN
    return_status as rs
ON 
    rs.issued_id = ist.issued_id
WHERE 
    rs.return_id IS NULL;
5. Advanced SQL Operations
Task 13: Identify members with overdue books.
sql
Copy code
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
Task 14: Update book status on return.
sql
Copy code
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
Task 15: Generate a branch performance report.
sql
Copy code
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
Task 16: Create a table of active members using CTAS.
sql
Copy code
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (SELECT DISTINCT issued_member_id FROM issued_status
                    WHERE issued_date >= CURRENT_DATE - INTERVAL '2 month');
Task 17: Find employees with the most book issues processed.
sql
Copy code
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
Task 18: Identify members frequently issuing damaged books.
sql
Copy code
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
    ist.issued_id = rs.issued_id
WHERE 
    rs.book_quality = 'damaged'
GROUP BY 1, 2
HAVING COUNT(*) >= 2;
Task 19: Write a stored procedure to manage book issuance.
sql
Copy code
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
Task 20: Generate a table of overdue books with fines.
sql
Copy code
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
Reports
Database Schema: Documentation of table structures and relationships.
Data Analysis: Insight into various aspects, such as book categories, employee details, and membership trends.
Summary Reports: Aggregated data on popular books and employee performance.
Conclusion
This project is a comprehensive demonstration of SQL capabilities in building and managing a library management system. It covers database setup, CRUD operations, advanced SQL queries, and data analysis, providing a strong foundation for database management and querying skills.
