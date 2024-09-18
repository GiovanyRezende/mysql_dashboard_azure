# MySQL Dashboard
*This is a challenge of Data Engineering NTT DATA Bootcamp in [DIO](https://web.dio.me/).* It consists in creating a dashboard for Data Analysis in Power BI, using a MySQL database.

## MySQL server creation
It was used [Microsoft Azure](https://portal.azure.com/) to create the MySQL server.

## [SQL code](https://github.com/julianazanelatto/power_bi_analyst/tree/main/M%C3%B3dulo%203)

```
create schema if not exists azure_company;
use azure_company;

select * from information_schema.table_constraints
	where constraint_schema = 'azure_company';

-- restrição atribuida a um domínio
-- create domain D_num as int check(D_num> 0 and D_num< 21);

CREATE TABLE employee(
	Fname varchar(15) not null,
    Minit char,
    Lname varchar(15) not null,
    Ssn char(9) not null, 
    Bdate date,
    Address varchar(30),
    Sex char,
    Salary decimal(10,2),
    Super_ssn char(9),
    Dno int not null,
    constraint chk_salary_employee check (Salary> 2000.0),
    constraint pk_employee primary key (Ssn)
);

alter table employee 
	add constraint fk_employee 
	foreign key(Super_ssn) references employee(Ssn)
    on delete set null
    on update cascade;

alter table employee modify Dno int not null default 1;

desc employee;

create table departament(
	Dname varchar(15) not null,
    Dnumber int not null,
    Mgr_ssn char(9) not null,
    Mgr_start_date date, 
    Dept_create_date date,
    constraint chk_date_dept check (Dept_create_date < Mgr_start_date),
    constraint pk_dept primary key (Dnumber),
    constraint unique_name_dept unique(Dname),
    foreign key (Mgr_ssn) references employee(Ssn)
);

-- 'def', 'company_constraints', 'departament_ibfk_1', 'company_constraints', 'departament', 'FOREIGN KEY', 'YES'
-- modificar uma constraint: drop e add
alter table departament drop  departament_ibfk_1;
alter table departament 
		add constraint fk_dept foreign key(Mgr_ssn) references employee(Ssn)
        on update cascade;

desc departament;

create table dept_locations(
	Dnumber int not null,
	Dlocation varchar(15) not null,
    constraint pk_dept_locations primary key (Dnumber, Dlocation),
    constraint fk_dept_locations foreign key (Dnumber) references departament (Dnumber)
);

alter table dept_locations drop fk_dept_locations;

alter table dept_locations 
	add constraint fk_dept_locations foreign key (Dnumber) references departament(Dnumber)
	on delete cascade
    on update cascade;

create table project(
	Pname varchar(15) not null,
	Pnumber int not null,
    Plocation varchar(15),
    Dnum int not null,
    primary key (Pnumber),
    constraint unique_project unique (Pname),
    constraint fk_project foreign key (Dnum) references departament(Dnumber)
);


create table works_on(
	Essn char(9) not null,
    Pno int not null,
    Hours decimal(3,1) not null,
    primary key (Essn, Pno),
    constraint fk_employee_works_on foreign key (Essn) references employee(Ssn),
    constraint fk_project_works_on foreign key (Pno) references project(Pnumber)
);

drop table dependent;
create table dependent(
	Essn char(9) not null,
    Dependent_name varchar(15) not null,
    Sex char,
    Bdate date,
    Relationship varchar(8),
    primary key (Essn, Dependent_name),
    constraint fk_dependent foreign key (Essn) references employee(Ssn)
);

show tables;
desc dependent;
```

```
insert into employee values ('John', 'B', 'Smith', 123456789, '1965-01-09', '731-Fondren-Houston-TX', 'M', 30000, 333445555, 5),
							('Franklin', 'T', 'Wong', 333445555, '1955-12-08', '638-Voss-Houston-TX', 'M', 40000, 888665555, 5),
                            ('Alicia', 'J', 'Zelaya', 999887777, '1968-01-19', '3321-Castle-Spring-TX', 'F', 25000, 987654321, 4),
                            ('Jennifer', 'S', 'Wallace', 987654321, '1941-06-20', '291-Berry-Bellaire-TX', 'F', 43000, 888665555, 4),
                            ('Ramesh', 'K', 'Narayan', 666884444, '1962-09-15', '975-Fire-Oak-Humble-TX', 'M', 38000, 333445555, 5),
                            ('Joyce', 'A', 'English', 453453453, '1972-07-31', '5631-Rice-Houston-TX', 'F', 25000, 333445555, 5),
                            ('Ahmad', 'V', 'Jabbar', 987987987, '1969-03-29', '980-Dallas-Houston-TX', 'M', 25000, 987654321, 4),
                            ('James', 'E', 'Borg', 888665555, '1937-11-10', '450-Stone-Houston-TX', 'M', 55000, NULL, 1);

insert into dependent values (333445555, 'Alice', 'F', '1986-04-05', 'Daughter'),
							 (333445555, 'Theodore', 'M', '1983-10-25', 'Son'),
                             (333445555, 'Joy', 'F', '1958-05-03', 'Spouse'),
                             (987654321, 'Abner', 'M', '1942-02-28', 'Spouse'),
                             (123456789, 'Michael', 'M', '1988-01-04', 'Son'),
                             (123456789, 'Alice', 'F', '1988-12-30', 'Daughter'),
                             (123456789, 'Elizabeth', 'F', '1967-05-05', 'Spouse');

insert into departament values ('Research', 5, 333445555, '1988-05-22','1986-05-22'),
							   ('Administration', 4, 987654321, '1995-01-01','1994-01-01'),
                               ('Headquarters', 1, 888665555,'1981-06-19','1980-06-19');

insert into dept_locations values (1, 'Houston'),
								 (4, 'Stafford'),
                                 (5, 'Bellaire'),
                                 (5, 'Sugarland'),
                                 (5, 'Houston');

insert into project values ('ProductX', 1, 'Bellaire', 5),
						   ('ProductY', 2, 'Sugarland', 5),
						   ('ProductZ', 3, 'Houston', 5),
                           ('Computerization', 10, 'Stafford', 4),
                           ('Reorganization', 20, 'Houston', 1),
                           ('Newbenefits', 30, 'Stafford', 4)
;

insert into works_on values (123456789, 1, 32.5),
							(123456789, 2, 7.5),
                            (666884444, 3, 40.0),
                            (453453453, 1, 20.0),
                            (453453453, 2, 20.0),
                            (333445555, 2, 10.0),
                            (333445555, 3, 10.0),
                            (333445555, 10, 10.0),
                            (333445555, 20, 10.0),
                            (999887777, 30, 30.0),
                            (999887777, 10, 10.0),
                            (987987987, 10, 35.0),
                            (987987987, 30, 5.0),
                            (987654321, 30, 20.0),
                            (987654321, 20, 15.0),
                            (888665555, 20, 0.0);
```
⚠*Warning*: don't insert data in employee table after creating the constraint. It will generate an error 1452.

## Project rules

```
1. Check headers and data types;
2. Modify currency values ​​to double type;
3. Check for nulls and analyze removal;
4. Employees with nulls in Super_ssn can be managers. Check if there are any employees without a manager;
5. Check if there are any departments without a manager;
6. If there is a department without a manager, assume you have the data and fill in the gaps;
7. Check the number of hours for projects;
8. Separate complex columns;
9. Merge employee and department queries to create an employee table with the name of the departments associated with the employees. The merge will be based on the employee table. Be careful, this information influences the type of occurrence;
10. In this process, eliminate unnecessary loads;
11. Join employees and the names of managers. This can be done with a SQL query or by merging tables with Power BI. If you use SQL, specify in the README the query used in the process;
12. Merge the First Name and Last Name columns to have just one column defining the names of collaborators;
13. Merge department names and location. This will make each department-location combination unique. This will help with creating the star model in a future module;
14. Explain why, in this case mentioned above, we can only use merge and not cover it;
15. Group the data to find out how many employees there are per manager;
16. Eliminate unnecessary columns, which will not be used in the report, from each table
```
## Data Cleaning

- UPPER case to string columns (except employees names);
- Convert data types to correct types;
- Convert the NULL value in ```employee``` to the own Ssn;
- Remove unnecessary columns

## Rule 11
Done in Power BI. However, this would be the SQL query:

```
SELECT 
    CONCAT(e1.Fname,' ',e1.Lname) AS Employee,
    CONCAT(e2.Fname,' ',e2.Lname) AS Supervisor
FROM  employee e1
LEFT JOIN employee e2 ON e1.Super_ssn = e2.Ssn;
```

## Rule 14: why merge?
The merge in Power BI is a SQL JOIN's counterpart. In SQL, a JOIN unifies the required columns in a query of the tables which are being joined. The Rule 11 example explains it. The combine in Power BI is a SQL UNION's counterpart. It means that the rows of the right table would appear in the query of the left table. It's not joining columns, it's an adding of rows. That's why merge is the correct option.

## The dashboard
![image](https://github.com/user-attachments/assets/d14c7b21-aba3-4541-9ac3-ba4d8eeb0b50)

