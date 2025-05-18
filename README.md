## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  

## Objectives

1. **Set up the Library Management System Database**: Populate the database with tables, assigning constraints
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Simple and Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status.
- **Foreign Keys**: Assign foreign keys where relevant and needed.

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


### 3. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
insert into books(isbn, book_title, category, rental_price, status, author, publisher)
values('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')

**Task 2: Update an Existing Member's Address**

```sql
update members
set member_address = '125 Oak St.'
where member_id = 'C103';

