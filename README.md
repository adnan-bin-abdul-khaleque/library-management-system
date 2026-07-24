# 📚 Library Management System Using SQL

## 📌 Project Overview

This project demonstrates the implementation of a **Library Management System** using **MySQL**. It covers database design, table creation, data manipulation (CRUD operations), business reporting, data analysis, table creation from queries, and a stored procedure for issuing books.

The project simulates a real-world library system where books can be issued and returned while allowing administrators to monitor branch performance, member activities, employee productivity, and book availability through SQL queries.

---

# 🗂 Database

**Database Name:** `library_db`

---

# 🛠 Technologies Used

* MySQL
* SQL
* MySQL Workbench

---

# 📂 Database Schema

The database consists of six relational tables:

| Table           | Description                            |
| --------------- | -------------------------------------- |
| `branch`        | Stores branch information              |
| `employees`     | Employee details and assigned branches |
| `members`       | Library member information             |
| `books`         | Book catalog                           |
| `issued_status` | Records of issued books                |
| `return_status` | Records of returned books              |

---

# Database Setup

## Create Database

```sql
CREATE DATABASE library_db;
```

## Create Tables

* Branch
* Employees
* Members
* Books
* Issued Status
* Return Status

Each table is created using primary keys and foreign key constraints to maintain referential integrity.

---

# SQL Tasks

## Task 1. Add a New Book Record

Insert a new book into the library catalog.

```sql
INSERT INTO books
(isbn, book_title, category, rental_price, status, author, publisher)

VALUES
('978-1-60129-456-2',
'To Kill a Mockingbird',
'Classic',
6.00,
'yes',
'Harper Lee',
'J.B. Lippincott & Co.');
```

---

## Task 2. Update a Member's Address

Modify the address of an existing member.

```sql
UPDATE members
SET member_address='420 Elm St'
WHERE member_id='C102';
```

---

## Task 3. Delete an Issued Book Record

Remove a record from the issued books table.

```sql
DELETE FROM issued_status
WHERE issued_id='IS121';
```

---

## Task 4. Find Books Issued by a Specific Employee

```sql
SELECT *
FROM issued_status
WHERE issued_emp_id='E101';
```

---

## Task 5. Find Members Who Borrowed More Than One Book

```sql
SELECT
    issued_member_id,
    member_name,
    COUNT(issued_member_id) AS total_books
FROM issued_status i
LEFT JOIN members m
ON i.issued_member_id=m.member_id
GROUP BY issued_member_id
HAVING COUNT(*)>1;
```

---

## Task 6. Create a Book Issue Summary Table

Create a new table showing how many times each book has been issued.

```sql
CREATE TABLE book_issued_cnt AS

SELECT
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS issue_count

FROM issued_status ist
JOIN books b
ON ist.issued_book_isbn=b.isbn

GROUP BY
b.isbn,
b.book_title;
```

---

# Data Analysis & Business Questions

## Task 7. Find All Books in a Specific Category

```sql
SELECT *
FROM books
WHERE category='Horror';
```

---

## Task 8. Calculate Total Rental Income by Category

```sql
SELECT
    b.category,
    SUM(b.rental_price) AS total_rental_price,
    COUNT(*) AS total_books

FROM issued_status ist
JOIN books b
ON b.isbn=ist.issued_book_isbn

GROUP BY b.category;
```

---

## Task 9. Display the Latest Registered Members

```sql
SELECT *
FROM members
ORDER BY reg_date DESC
LIMIT 10;
```

---

## Task 10. Employee List with Branch Manager

```sql
SELECT
    e.emp_id,
    e.emp_name,
    e.position,
    e.salary,
    b.branch_id,
    b.branch_address,
    m.emp_name AS manager_name

FROM branch b

JOIN employees e
ON b.branch_id=e.branch_id

JOIN employees m
ON b.manager_id=m.emp_id;
```

---

## Task 11. Find Expensive Books

Display books with rental prices greater than or equal to **7.00**.

```sql
SELECT *
FROM books
WHERE rental_price>=7.00;
```

---

## Task 12. Find Books That Have Not Been Returned

```sql
SELECT *

FROM issued_status ist

LEFT JOIN return_status rs
ON rs.issued_id=ist.issued_id

WHERE rs.return_id IS NULL;
```

---

## Task 13. Display Members with Outstanding Books

Show the member name, book title, and issue date for books that have not yet been returned.

```sql
SELECT
    m.member_name,
    bk.book_title,
    ist.issued_date,
    rs.return_date

FROM issued_status ist

JOIN members m
ON ist.issued_member_id=m.member_id

JOIN books bk
ON ist.issued_book_isbn=bk.isbn

LEFT JOIN return_status rs
ON ist.issued_id=rs.issued_id

WHERE rs.return_date IS NULL

ORDER BY ist.issued_date;
```

---

## Task 14. Update Book Status After Return

Update the availability status of all returned books.

```sql
UPDATE books b

JOIN issued_status i
ON b.isbn=i.issued_book_isbn

JOIN return_status r
ON i.issued_id=r.issued_id

SET b.status='Yes';
```

---

## Task 15. Branch Performance Report

Generate a report containing:

* Books Issued
* Books Returned
* Total Revenue

```sql
SELECT
    b.branch_id,
    COUNT(i.issued_id) AS books_issued,
    COUNT(r.return_id) AS books_returned,
    SUM(bk.rental_price) AS total_revenue

FROM branch b

JOIN employees e
ON b.branch_id=e.branch_id

JOIN issued_status i
ON e.emp_id=i.issued_emp_id

LEFT JOIN return_status r
ON i.issued_id=r.issued_id

JOIN books bk
ON i.issued_book_isbn=bk.isbn

GROUP BY b.branch_id;
```

---

## Task 16. Create a Table of Active Members

Create a table containing members who have borrowed at least one book.

```sql
CREATE TABLE active_members AS

SELECT DISTINCT
    m.member_id,
    m.member_name

FROM members m

JOIN issued_status i
ON m.member_id=i.issued_member_id;
```

---

## Task 17. Find Employees Who Processed the Most Book Issues

```sql
SELECT
    e.emp_id,
    e.emp_name,
    COUNT(i.issued_id) AS total_books_issued

FROM employees e

JOIN issued_status i
ON e.emp_id=i.issued_emp_id

GROUP BY
e.emp_id,
e.emp_name

HAVING COUNT(i.issued_id)=

(
SELECT MAX(book_count)

FROM
(
SELECT COUNT(*) AS book_count
FROM issued_status
GROUP BY issued_emp_id
) counts

);
```

---

## Task 18. Create a Stored Procedure to Issue a Book

The stored procedure checks whether a book is available.

* If available → updates the status to **No** (Issued)
* Otherwise → returns a message indicating the book is unavailable.

```sql
DELIMITER $$

CREATE PROCEDURE issue_book(IN p_isbn VARCHAR(30))

BEGIN

DECLARE book_status VARCHAR(10);

SELECT status
INTO book_status
FROM books
WHERE isbn=p_isbn;

IF book_status='Yes' THEN

UPDATE books
SET status='No'
WHERE isbn=p_isbn;

SELECT 'Book issued successfully.' AS Message;

ELSE

SELECT 'Book is not available.' AS Message;

END IF;

END $$

DELIMITER ;
```

Execute the procedure:

```sql
CALL issue_book('978-0-553-29698-2');
```

---

# SQL Concepts Covered

* Database Creation
* Table Creation
* Primary Keys
* Foreign Keys
* INSERT
* UPDATE
* DELETE
* SELECT
* WHERE
* ORDER BY
* GROUP BY
* HAVING
* Aggregate Functions (`COUNT`, `SUM`)
* INNER JOIN
* LEFT JOIN
* Subqueries
* CREATE TABLE AS SELECT (CTAS)
* Stored Procedures
* Referential Integrity
* Data Analysis Queries

---

# Key Business Insights

Using SQL, this project answers practical library management questions, including:

* Book availability
* Frequently borrowed books
* Active library members
* Employee performance
* Branch performance
* Rental revenue by category
* Outstanding book returns
* Member registration trends
* High-value books
* Automated book issuing process

---

# Project Structure

```
Library-Management-SQL/
│
├── library_management.sql
├── README.md
└── dataset/
    ├── branch.csv
    ├── employees.csv
    ├── members.csv
    ├── books.csv
    ├── issued_status.csv
    └── return_status.csv
```

---

# Learning Outcomes

This project helped strengthen practical SQL skills in:

* Relational Database Design
* Data Manipulation (CRUD)
* SQL Joins
* Aggregate Analysis
* Business Reporting
* Table Creation from Queries
* Stored Procedures
* Library Management Database Design
* Real-world SQL Problem Solving

---

## Author

**Adnan Bin Abdul Khaleque**

Aspiring Data Analyst with a passion for SQL, data analytics, Excel, Tableau, and Python. This project showcases practical SQL skills through a real-world Library Management System.

---

⭐ If you found this project useful, consider giving it a **Star** on GitHub!
