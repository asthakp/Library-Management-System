# Library Management System using SQL 

## Project Overview

**Project Title**: Library Management System  

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branch, employee, member, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

- **Database Creation**: Created a database named `library`.
- **Table Creation**: Created tables for branch, employee, member, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(20) PRIMARY KEY,
            manager_id VARCHAR(20),
            branch_address VARCHAR(55),
            contact_no VARCHAR(20)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employee;
CREATE TABLE employee
(
            emp_id VARCHAR(20) PRIMARY KEY,
            emp_name VARCHAR(25),
            position VARCHAR(20),
            salary DECIMAL(10,2),
            branch_id VARCHAR(25),
            
);


-- Create table "Member"
DROP TABLE IF EXISTS member;
CREATE TABLE member
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
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
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
         
);

```
### Foreign key constraints added
```sql
ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES member(member_id)

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books(isbn)

ALTER TABLE issued_status
ADD CONSTRAINT fk_employee
FOREIGN KEY (issued_emp_id)
REFERENCES employee(emp_id

ALTER TABLE employee
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id)

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id)
```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employee` table.
- **Delete**: Removed records from the `member` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

**Task 2: Update an Existing Member's Address**

```sql
UPDATE member
SET member_address="125 Main St"
WHERE member_id="C101"

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE issued_id ='IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT m.member_name, count(i.issued_member_id) as book_count
FROM member m
JOIN issued_status i
ON m.member_id=i.issued_member_id
GROUP BY m.member_name
HAVING book_count>1
```


**Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_issued_count AS
SELECT 
    b.isbn, 
    b.book_title, 
    COUNT(i.issued_id) AS issued_count
FROM 
    books b
JOIN 
    issued_status i 
ON 
    b.isbn = i.issued_book_isbn
GROUP BY 
    b.isbn, 
    b.book_title;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

**Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
    b.category,
    SUM(b.rental_price),
    COUNT(*)
FROM 
issued_status as ist
JOIN
books as b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT * FROM member
WHERE reg_date >= current_date - interval 180 day
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
FROM employee as e1
JOIN 
branch as b
ON e1.branch_id = b.branch_id    
JOIN
employee as e2
ON e2.emp_id = b.manager_id
```


Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
       m.member_id as member_id,
       m.member_name as member_name,
b.book_title as book_title,
       i.issued_date as issued_date,
       datediff(current_date, issued_date)as days_overdue
FROM issued_status i
JOIN member m 
ON m.member_id=i.issued_member_id
JOIN books b 
ON b.isbn=i.issued_book_isbn
LEFT JOIN return_status r
ON i.issued_id=r.issued_id
WHERE r.return_id IS NULL and datediff(current_date, issued_date)>30
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql
DROP PROCEDURE IF EXISTS update_book_status;  -- Drop the procedure if it exists

DELIMITER $$

CREATE PROCEDURE update_book_status (p_return_id VARCHAR(20), p_issued_id VARCHAR(30))
BEGIN
  -- Declare variables for ISBN and book name
  DECLARE v_book_isbn VARCHAR(20);
  DECLARE v_book_name VARCHAR(100);

  -- Insert into the return_status table
  INSERT INTO return_status(return_id, issued_id, return_date)
  VALUES (p_return_id, p_issued_id, CURRENT_DATE);
  
    -- Select book information from issued_status table
  SELECT issued_book_isbn, issued_book_name
  INTO v_book_isbn, v_book_name
  FROM issued_status
  WHERE issued_id = p_issued_id;
  
    -- Update the books table to set the status to 'yes' for the returned book
  UPDATE books
  SET status = 'yes'
  WHERE isbn = v_book_isbn;
  
  select concat('Thank you , for returning the book ', v_book_name) as message;

END $$

DELIMITER ;

call update_book_status('RS119', 'IS135');

```

**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
SELECT 
       b.branch_id , 
       COUNT(i.issued_id)as books_issued , 
       COUNT(r.return_id) as books_returned , 
       SUM(bks.rental_price) as total_rental_price 
FROM branch b
JOIN employee e 
ON b.branch_id=e.branch_id
JOIN issued_status i
ON e.emp_id=i.issued_emp_id
LEFT JOIN return_status r
ON i.issued_id=r.issued_id
JOIN books bks 
ON i.issued_book_isbn=bks.isbn
GROUP BY b.branch_id;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 6 months.

```sql

CREATE TABLE active_member AS
SELECT 
       m.member_id, 
m.member_name, 
COUNT(i.issued_id) AS book_count
FROM member m
JOIN issued_status i
ON m.member_id= i.issued_member_id
WHERE timestampdiff(month,current_date, i.issued_date)<=6
GROUP BY m.member_id, m.member_name
HAVING COUNT(i.issued_id)>=1; 

SELECT * FROM active_member;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist
JOIN
employee as e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2
ORDER BY no_book_issued DESC
LIMIT 3
```


**Task 18: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

DROP PROCEDURE IF EXISTS book_status;

DELIMITER $$ 

CREATE PROCEDURE book_status(
    p_issued_id VARCHAR(20), 
    p_issued_member_id VARCHAR(20),
    p_issued_book_isbn VARCHAR(20), 
    p_issued_emp_id VARCHAR(20)
)
BEGIN  

    -- Declare variables 
    DECLARE v_book_status VARCHAR(20);
    DECLARE v_book_title VARCHAR(75);

    -- Check the status of the book (if available 'yes')
    SELECT status, book_title INTO v_book_status, v_book_title 
    FROM books
    WHERE isbn = p_issued_book_isbn;  

    -- If the book is available, issue the book
    IF v_book_status = 'yes' THEN
        INSERT INTO issued_status (issued_id, issued_member_id, issued_book_name, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (p_issued_id, p_issued_member_id, v_book_title, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        -- Update the book's status to 'no' after issuing
        UPDATE books
        SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        -- Return a message confirming the book was issued
        SELECT CONCAT(v_book_title, ' is issued.') AS message;

    -- If the book is not available, send a message
    ELSE 
        SELECT CONCAT(v_book_title, ' is not available at the moment.') AS message;
    END IF;

END $$ 

DELIMITER ; 

-- test the procedure --
-- isbn : 978-0-375-41398-8 'no'
-- isbn : 978-0-06-025492-6 'yes' 

call book_status('IS141', 'C111', '978-0-375-41398-8', 'E120');

call book_status('IS141', 'C104', '978-0-06-025492-6', 'E110')
```

** Task 19: Create Table As Select (CTAS)**

Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.  
Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
The total fines, with each day's fine calculated at $0.50.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines 

```sql

SELECT 
    m.member_id,
    SUM(CASE WHEN DATEDIFF(CURRENT_DATE, i.issued_date) > 30 THEN 1 ELSE 0 END) AS overdue_books,
    SUM(CASE WHEN DATEDIFF(CURRENT_DATE, i.issued_date) > 30 THEN DATEDIFF(CURRENT_DATE, i.issued_date) * 0.50 ELSE 0 END) AS total_fine
FROM 
    issued_status i
JOIN 
    member m ON i.issued_member_id = m.member_id
GROUP BY 
    m.member_id
ORDER BY 
    m.member_id;
```


## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.


