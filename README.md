# Library-Management-System

## PROJECT TASKS ##

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure
- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
DROP TABLE IF EXISTS branch;
CREATE TABLE branch(
	branch_id VARCHAR(10) PRIMARY KEY,
	manager_id VARCHAR(10),
	branch_address VARCHAR(55),
	contact_no VARCHAR(20)
	
);

DROP TABLE IF EXISTS employee;
CREATE TABLE employee(
	emp_id VARCHAR(10) PRIMARY KEY,
	emp_name VARCHAR(30),
	positn VARCHAR(20),
	salary	INT,
	branch_id VARCHAR(10)

);

DROP TABLE IF EXISTS books;
CREATE TABLE books(
	isbn VARCHAR(20) PRIMARY KEY,
	book_title	VARCHAR(55),
	category VARCHAR(20),
	rental_price FLOAT,
	status	VARCHAR(10),
	author	VARCHAR(30),
	publisher VARCHAR(35)

);

DROP TABLE IF EXISTS members;
CREATE TABLE members(
	member_id VARCHAR(15) PRIMARY KEY,
	member_name	VARCHAR(25),
	member_address VARCHAR(35),
	reg_date date
);

DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status(
	issued_id VARCHAR(10) PRIMARY KEY,
	issued_member_id VARCHAR(15),
	issued_book_name VARCHAR(55),
	issued_date DATE,
	issued_book_isbn VARCHAR(20),
	issued_emp_id VARCHAR(10)
)

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status(
    return_id VARCHAR(10) PRIMARY KEY,
	issued_id VARCHAR(10),
	return_book_name VARCHAR(55),
	return_date DATE,
	return_book_isbn VARCHAR(20)
)

-- FOREIGN KEY --
	
ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY(issued_id)
REFERENCES issued_status(issued_id)

ALTER TABLE return_status
ADD CONSTRAINT fk_books
FOREIGN KEY(return_book_isbn)
REFERENCES books(isbn)

ALTER TABLE issued_status
ADD CONSTRAINT fk_employee
FOREIGN KEY(issued_emp_id)
REFERENCES employee(emp_id)

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY(issued_book_isbn)
REFERENCES books(isbn)

ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY(issued_member_id)
REFERENCES members(member_id)

ALTER TABLE employee
ADD CONSTRAINT fk_branch
FOREIGN KEY(branch_id)
REFERENCES branch(branch_id)

ALTER TABLE branch
ALTER COLUMN branch_id TYPE VARCHAR(20)

ALTER TABLE branch
ALTER COLUMN manager_id TYPE VARCHAR(20)

ALTER TABLE branch
ALTER COLUMN contact_no TYPE VARCHAR(20)

select * from branch
```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', '6.00', 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

SELECT * FROM books
WHERE isbn = '978-1-60129-456-2';
```


**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';

SELECT member_address FROM members
WHERE member_id = 'C103';
```

**Task 3: Delete a Record from the Issued Status Table.**
--Objective: Delete the record with issued_id = 'IS121' from the issued_status table. 

```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';

SELECT * FROM issued_status
WHERE issued_id = 'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee.**
--Objective: Select all books issued by the employee with emp_id = 'E101'. 

```sql
SELECT issued_book_name FROM issued_status
WHERE issued_emp_id = 'E101';
```

**Task 5: List Members Who Have Issued More Than One Book**
--Objective: Use GROUP BY to find members who have issued more than one book. 

```sql
SELECT issued_emp_id, COUNT(issued_emp_id) FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_emp_id) >= '2';
```

### 3. CTAS (Create Table As Select)

**Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt** 

```sql
CREATE TABLE book_issued_count AS 
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issued_count
FROM books AS b
JOIN issued_status AS ist
ON b.isbn = ist.issued_book_isbn
GROUP BY b.isbn, b.book_title;

SELECT * FROM book_issued_count;
```

**Task 7. Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

**Task 8: Find Total Rental Income by Category**:

```sql
SELECT category, SUM(rental_price) AS rental_income
FROM books
GROUP BY category;
```

**TASK 9: List Members Who Registered in the Last 350 Days**:

```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '350 days';
```

**TASK 10: Employees with Their Branch Manager's Name and their branch details:**

```sql
SELECT e1.emp_id, b.*, e2.emp_name AS manager
FROM employee AS e1
JOIN branch AS b
ON e1.branch_id = b.branch_id
JOIN employee AS e2
ON e2.emp_id = b.manager_id
```

**Task 11. Create a Table of Books with Rental Price above a Certain Threshold: **

```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;

SELECT * FROM expensive_books
```

**Task 12: Retrieve the List of Books Not Yet Returned**

```sql
SELECT DISTINCT ist.issued_book_name FROM issued_status AS ist
LEFT JOIN return_status AS rs
ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL;
```

##Advanced Sql Operations 

**Task 13: Identify Members with Overdue Books**
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT ist.issued_member_id, m.member_name, b.book_title,
ist.issued_date, CURRENT_DATE - ist.issued_date as overdue_days
FROM issued_status as ist
JOIN members as m
ON m.member_id = ist.issued_member_id
JOIN books as b
ON b.isbn = ist.issued_book_isbn
LEFT JOIN 
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL AND
(CURRENT_DATE - ist.issued_date) > 30
ORDER BY ist.issued_member_id

```

**Task 14: Update Book Status on Return**
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).

```sql
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
v_isbn VARCHAR(50);
v_book_name VARCHAR(80);
    
BEGIN
-- All your logic and code
-- Inserting into returns based on users input
INSERT INTO return_status(return_id, issued_id, return_date)
VALUES
(p_return_id, p_issued_id, CURRENT_DATE);

SELECT issued_book_isbn, issued_book_name
INTO v_isbn, v_book_name
FROM issued_status
WHERE issued_id = p_issued_id;

UPDATE books
SET status = 'yes'
WHERE isbn = v_isbn;

RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
    
END;
$$

-- Calling function 
CALL add_return_records('RS138', 'IS135');

-- calling function 
CALL add_return_records('RS148', 'IS140');
```

**Task 15: Branch Performance Report:
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports AS
SELECT br.branch_id,COUNT(i.issued_id) AS total_books_issued,COUNT(r.return_id) AS total_books_returned,SUM(b.rental_price) AS total_revenue
FROM issued_status AS i
JOIN employee AS e
ON i.issued_emp_id = e.emp_id
JOIN branch AS br
ON e.branch_id = br.branch_id
JOIN books AS b
ON b.isbn = i.issued_book_isbn
LEFT JOIN return_status AS r
ON r.issued_id = i.issued_id
GROUP BY br.branch_id
ORDER BY br.branch_id;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 11 months.

```sql
DROP TABLE IF EXISTS active_members;
CREATE TABLE active_members AS 
SELECT * FROM members
WHERE member_id IN (
SELECT issued_member_id 
FROM issued_status
WHERE issued_date > CURRENT_DATE - INTERVAL '11 month');

SELECT * FROM active_members;
```

**Task 17: Find Employees with the Most Book Issues Processed.**
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch. 

```sql
SELECT e.emp_name, br.*, COUNT(ist.issued_id) AS no_book_issued
FROM issued_status AS ist
JOIN employee as e
ON e.emp_id = ist.issued_emp_id
JOIN branch as br
ON e.branch_id = br.branch_id
GROUP BY e.emp_name, br.branch_id
```

**Task 18: Stored Procedure Objective: 
Create a stored procedure to manage the status of books in a library system.

Description: 
Write a stored procedure that updates the status of a book in the library based on its issuance. 
The procedure should function as follows: The stored procedure should take the book_id as an input parameter. 
The procedure should first check if the book is available (status = 'yes'). 
If the book is available, it should be issued, and the status in the books table should be updated to 'no'. 
If the book is not available (status = 'no'), theprocedure should return an error message indicating that the book is currently not available.

```sql
CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
v_status VARCHAR(10);

BEGIN
SELECT status 
INTO v_status
FROM books
WHERE isbn = p_issued_book_isbn;

IF v_status = 'yes' THEN
INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
VALUES (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

UPDATE books
SET status = 'no'
WHERE isbn = p_issued_book_isbn;

RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;

ELSE RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
END IF;
END;
$$

SELECT * FROM books;
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'
```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

  ## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.

2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.



