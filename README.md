# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.



## Project Structure

### 1. Database Setup
![ERD](https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png)


- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(25)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
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
            category VARCHAR(20),
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
            issued_book_isbn VARCHAR(25),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
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
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO BOOKS(ISBN,BOOK_TITLE,CATEGORY,RENTAL_PRICE,STATUS,AUTHOR,PUBLISHER)
VALUES('978-1-60129-456-2','To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')

```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE MEMBERS
SET MEMBER_ADDRESS= '125 MAIN STREET'
WHERE MEMBER_ID='C101'
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM ISSUED_STATUS
WHERE ISSUED_ID = 'IS121'
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
SELECT ISSUED_MEMBER_ID, COUNT(*) FROM  ISSUED_STATUS
GROUP BY 1
HAVING COUNT(*) > 1

```

### CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE BOOK_ISSUED_COUNT AS
SELECT B.ISBN, B.BOOK_TITLE,COUNT(IST.ISSUED_ID) AS ISSUE_COUNT  FROM ISSUED_STATUS AS IST
JOIN BOOKS AS B
ON IST.ISSUED_BOOK_ISBN = B.ISBN
GROUP BY 1,2

SELECT * FROM BOOK_ISSUED_COUNT
```

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select b.Category,sum(b.rental_price) AS RENTAL_INCOME, count(*) from issued_status as ist
join books as b 
on ist.issued_book_isbn = b.isbn 
GROUP BY 1
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT * FROM MEMBERS
WHERE REG_DATE >= CURRENT_DATE - INTERVAL '180 DAYS'
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT E.*,BR.MANAGER_ID,E2.EMP_NAME AS MANAGER FROM EMPLOYEE AS E
JOIN BRANCH AS BR
ON E.BRANCH_ID=BR.BRANCH_ID
JOIN EMPLOYEE AS E2
ON BR.MANAGER_ID=E2.EMP_ID
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE EXPENSIVE_BOOKS AS
SELECT * FROM BOOKS 
WHERE RENTAL_PRICE > 7.00

SELECT * FROM EXPENSIVE_BOOKS

```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT DISTINCT IST.ISSUED_BOOK_NAME FROM ISSUED_STATUS AS IST
LEFT JOIN RETURN_STATUS AS RS
ON IST.ISSUED_ID = RS.ISSUED_ID
WHERE RS.RETURN_ID IS NULL
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
ist.issued_member_id,
m.member_name,
b.book_title,
ist.issued_date,
r.return_date,
CURRENT_DATE - ist.issued_date as days_overdue
from issued_status as ist
join 
members as m
on ist.issued_member_id = m.member_id
join
books as b
on ist.issued_book_isbn=b.isbn
left join
return_status as r
on ist.issued_id = r.issued_id
WHERE r.return_date is null
and
(CURRENT_DATE - ist.issued_date)>30
order by 1
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT 
        issued_book_isbn,
        issued_book_name
        INTO
        v_isbn,
        v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
    
END;
$$


-- Testing FUNCTION add_return_records

issued_id = IS135
ISBN = WHERE isbn = '978-0-307-58837-1'

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM books
WHERE isbn = '978-0-7432-7357-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-7432-7357-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');

-- calling function 
CALL add_return_records('RS148', 'IS140', 'Good');

-- calling function 
CALL add_return_records('RS149', 'IS136', 'Good');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
select 
	b.branch_id,
	b.manager_id,
	count(ist.issued_id) as no_of_books_issued,
	count(r.return_id) as no_of_books_returned,
	sum(rental_price) as total_revenue
from issued_status as ist
join employee as e
on ist.issued_emp_id=e.emp_id
join branch as b
on b.branch_id= e.branch_id
left join return_status as r
on r.issued_id=ist.issued_id
join books as bk
on bk.isbn = ist.issued_book_isbn
group by 1,2;

-- or 

SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(DISTINCT ist.issued_id) AS no_of_books_issued,
    COUNT(DISTINCT r.return_id)  AS no_of_books_returned,
    COALESCE(SUM(bk.rental_price),0) AS total_revenue
FROM branch b
LEFT JOIN employee e
    ON e.branch_id = b.branch_id
LEFT JOIN issued_status ist
    ON ist.issued_emp_id = e.emp_id
LEFT JOIN return_status r
    ON r.issued_id = ist.issued_id
LEFT JOIN books bk
    ON bk.isbn = ist.issued_book_isbn
GROUP BY b.branch_id, b.manager_id
ORDER BY b.branch_id;

```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

CREATE TABLE active_memberss
AS
SELECT * FROM members
WHERE member_id IN (SELECT 
                        DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE 
                        issued_date >= (select max(issued_date) from issued_status)- INTERVAL '2 month'
                    )
;
SELECT * FROM active_memberss;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
select 
            e.emp_name,
	b.branch_id,
	b.manager_id,
	b.branch_address,
	count(issued_id) as no_books_issued
	
from issued_status as ist
join  employee as e
on ist.issued_emp_id=e.emp_id
join branch as b
ON e.branch_id = b.branch_id
group by  1,2
order by no_books_issued desc
limit 3

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

create or replace procedure issued_books(p_issued_id varchar(10),p_issued_member_id varchar(10),p_issued_book_isbn varchar(25),p_issued_emp_id varchar(10))
language plpgsql
as $$
DECLARE
	v_status varchar(10);
BEGIN
	 select status 
	 into v_status
	 from books
	 where isbn=p_issued_book_isbn;

	 if v_status = 'yes' then
	 	insert into issued_status(issued_id,issued_member_id,issued_date,issued_book_isbn,issued_emp_id)
		 values
		 (p_issued_id,p_issued_member_id,current_date,p_issued_book_isbn,p_issued_emp_id);
		 
	 update books 
	 set status = 'no'
	 where isbn=p_issued_book_isbn;

	 raise notice 'Book records added successfully for book isbn : %',p_issued_book_isbn;
	 
	 else 
	 RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;

	 end if;
end;
$$
-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;


SELECT * FROM books
where isbn= '978-0-553-29698-2';

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'

CALL issued_books('IS157', 'C108', '978-0-553-29698-2', 'E104');
CALL issued_books('IS158', 'C108', '978-0-375-41398-8', 'E104');


```



