## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  

## Objectives

1. **Database Setup**: Populate the database with tables, then assign constraints to them.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **EDA (Exploratory Data Analysis)**: Dive into the data to address specific questions.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status.
- **Foreign Keys**: Assigned foreign keys where relevant and needed.
  
```sql
create database library_db;

create table branch
(
	branch_id varchar(10) primary key,
	manager_id varchar(10),
	branch_address varchar(50),
	contact_no varchar(10)
);

create table employees
(
	emp_id varchar(10) primary key,
	emp_name varchar(25),
	position varchar(15),
	salary int,
	branch_id varchar(25)
);

create table books
(
	isbn varchar(20) primary key,
	book_title varchar(75),
	category varchar(10),
	rental_price float,
	status varchar(15),
	author varchar(35),
	publisher varchar(55)
);

create table members
(
	member_id varchar(10) primary key,
	member_name varchar(25),
	member_address varchar(75),
	reg_date date
);

create table issued_status
(
    issued_id varchar(10) primary key,
    issued_member_id varchar(30),
    issued_book_name varchar(80),
    issued_date date,
    issued_book_isbn varchar(50),
    issued_emp_id varchar(10)
);

create table return_status
(
	return_id varchar(10) primary key,
	issued_id varchar(10),
	return_book_name varchar(75),
	return_date date,
	return_book_isbn varchar(20)
);

--

alter table issued_status
add constraint fk_members
foreign key (issued_member_id)
references members(member_id);

alter table issued_status
add constraint fk_books
foreign key (issued_book_isbn)
references books(isbn);

alter table issued_status
add constraint fk_employees
foreign key (issued_emp_id)
references employees(emp_id);

alter table return_status
add constraint fk_issued_status
foreign key (issued_id)
references issued_status(issued_id);

alter table employees
add constraint fk_branch
foreign key (branch_id)
references branch(branch_id);
```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `members` table.
- **Delete**: Removed records from the `issued_status` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
insert into books(isbn, book_title, category, rental_price, status, author, publisher)
values('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')
```

**Task 2. Update an Existing Member's Address.**

```sql
update members
set member_address = '125 Oak St.'
where member_id = 'C103';
```

**Task 3. Delete the record with issued_id = 'IS121' from the issued_status table.**

```sql
delete from issued_status
where issued_id = 'IS121';
```

**Task 4. Select all books issued by the employee with emp_id = 'E101'.**

```sql
select * 
from 
	issued_status 
where 
	issued_emp_id = 'E101';
```

**Task 5. Use GROUP BY to find members who have issued more than one book.**

```sql
select 
	issued_member_id, 
	count(*) as count_issued_books
from 
	issued_status
group by 1
having count(*) > 1;
```

### 3. EDA (Exploratory Data Analysis)

**Task 6. Find how many times each book has been issued.**

```sql
select 
	issued_book_isbn as isbn, 
	issued_book_name as book_title, 
	count(*) as cnt_issued_books
from 
	issued_status
group by 
	1, 2
order by 
	3 desc;
```

**Task 7. Find out all books in each category.**

```sql
select 
	category, book_title
from 
	books
order by 1;
```

**Task 8. Find total rental income by category.**

```sql
select 
	b.category, 
	sum(b.rental_price) as total_rental_income
from 
	issued_status as ist
join 
	books as b
    on 
	ist.issued_book_isbn = b.isbn
group by 1
order by 
	2 desc;
```

**Task 9. List members who registered in the last 2 years.**

```sql
select * 
from 
	members
where 
	reg_date >= current_date - interval '2 years';
```

**Task 10. List employees with their branch manager's name and their branch details.**

```sql
select  
		e1.emp_id,
	    e1.emp_name,
	    e1.position,
	    e1.salary,
	    b.*,
	    e2.emp_name as manager 
from 
	employees e1
join 
	branch b 
	on 
	e1.branch_id = b.branch_id
join 
	employees as e2
    on b.manager_id = e2.emp_id;
```

**Task 11. Create a table of books with rental price above a certain threshold.**

```sql
create table expensive_books as
(
select * 
from 
	books
where 
	rental_price  > 7.00
);
```

**Task 12. Create a list of books not returned yet.**

```sql
select * 
from 
	issued_status as ist
left join
	return_status as rs
	on rs.issued_id = ist.issued_id
where 
	rs.return_id is null;
```

## Advanced Operations

**Task 13. identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.**

```sql
with x_cte as
(
select date '2024-05-07' as example_date
)
select 
	member_id, 
	member_name, 
	issued_book_name as book_title, 
	issued_date, 
	example_date - issued_date as overdue_days
from 
	issued_status ist
join 
	members m
    on ist.issued_member_id = m.member_id
full join 
	return_status rs
	on ist.issued_id = rs.issued_id
cross join 
	x_cte x
where 
	return_date is null 
and
	(example_date - issued_date) > 30;
```

**Task 14. Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).**

```sql
create or replace procedure add_return_records(p_return_id VARCHAR(10), 
                                               p_issued_id VARCHAR(10))
											   
language plpgsql --mandatory 
as $$

declare
	v_isbn varchar(50);
	v_book_name varchar(80);

begin
	
	insert into return_status(return_id, issued_id, return_date)
	values(p_return_id, p_issued_id, current_date);

	select 
		issued_book_isbn, 
		issued_book_name
		into
		v_isbn,
		v_book_name
	from 
		issued_status
	where 
		issued_id = p_issued_id;
	
    update 
		books 
	set 
		status = 'yes'
	where 
		isbn = v_isbn;

    raise notice 'Thank you for returning the book: %', v_book_name;

end;
$$

call add_return_records('RS141', 'IS134');
```

**Task 15. For each branch_id, show the number of books issued, the number of books returned, and the total revenue generated from book rentals.**

```sql
select 
       branch_id, 
	   count(issued_id) as cnt_issued_books, 
       count(return_id) as cnt_returned_books,
	   sum(rental_price) as revenue_book_rental
from
(
	select 
		b.branch_id, 
		ist.issued_id, 
		rs.return_id, 
		bk.rental_price
	from 
		employees emp
	join 
		branch b
		on emp.branch_id = b.branch_id
	join 
		issued_status ist
		on emp.emp_id = ist.issued_emp_id
	join 
		books bk
		on ist.issued_book_isbn = bk.isbn
	left join 
		return_status rs
		on rs.issued_id = ist.issued_id
) as branch_info
group by 1
order by 1;
```

**Task 16. Create a new table 'active_members' containing  members who have issued at least one book in the last 420 days.**

```sql
create table active_members as
(
select * 
from 
	members
where 
	member_id 
	in 
	(
	select 
		distinct issued_member_id
     from 
	 	issued_status
	 where 
	 	issued_date >= current_date - interval '420 days'
	)
);
```


**Task 17. Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.**

```sql
select 
	emp_id, 
	emp_name, 
	count(issued_id) as cnt_processed_books, 
	branch_id
from 
	issued_status ist
full join 
	employees emp
	on ist.issued_emp_id = emp.emp_id
group by 
	1, 2
order by 
	3 desc
limit 3;
```

**Task 18. Create a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:**
- The stored procedure should take issued_id, issued_member_id, issued_book_isbn, issued_emp_id as an input parameters. 
- The procedure should first check if the book is available (status = 'yes'). 
- If the book is available, it should be issued, and the status in the books table should be updated to 'no'. 
- If the book is not available (status = 'no'), the procedure should return an error message 
indicating that the book is currently not available.

```sql

create or replace procedure issue_book(p_issued_id varchar(20), p_issued_member_id varchar(30), 
                                       p_issued_book_isbn varchar(50), p_issued_emp_id varchar(10))

language plpgsql
as $$

declare
	v_status varchar(10);

begin

    -- is the book available?
	select 
		status
		into 
		v_status
	from 
		books
	where 
		isbn = p_issued_book_isbn;
	

	if v_status = 'yes' then
	
		insert into issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
		values(p_issued_id, p_issued_member_id, current_date, p_issued_book_isbn, p_issued_emp_id);
	
		update 
			books
		set 
			status = 'no'
		where 
			isbn =  p_issued_book_isbn;
	
		raise notice 'Book records added successfully for book isbn : %', p_issued_book_isbn;

	
	else 
		raise notice 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    
	end if;
	
end;
$$


call issue_book('IS141', 'C105', '978-0-307-37840-1', 'E105');
```


## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
