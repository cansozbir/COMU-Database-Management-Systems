# COMU-Database-Management-Systems

## Exercise 5.5.a
##### For each department whose average employee salary is more than $30,000, retrieve the department name and the number of employees working for that department.

    select d.dname, count(fname)
    from department d, employee e
    where d.dnumber = e.dno and (select avg(salary) from employee where dno = d.dnumber ) > 30000
    group by d.dname

## Exercise 5.5.b
##### Suppose that we want the number of male employees in each department making more than $30,000, rather than all employees (as in Exercise 5.4a). Can we specify this query in SQL? Why or why not?

    select d.dname, count(*)
    from department d, employee e
    where d.dnumber = e.dno and e.sex="M" and e.salary > 30000
    group by d.dname



## Exercise 5.7.a
##### Retrieve the names of all employees who work in the department that has the employee with the highest salary among all employees.

    select e.fname
    from employee e
    where e.dno = (	select dno
                    from employee
                    order by salary desc
                    limit 1 )


## Exercise 5.7.b
##### Retrieve the names of all employees whose supervisor’s supervisor has ‘888665555’ for Ssn .

    select e.fname
    from employee e, employee e2
    where e.superssn= e2.ssn and e2.superssn = "888665555"


## Exercise 5.7.c
##### Retrieve the names of employees who make at least $10,000 more than the employee who is paid the least in the company.

    select *
    from employee e
    where e.salary > (select min(salary) from employee) + 10000


## Exercise 5.8.a
##### A view that has the department name, manager name, and manager salary for every department.

    create view dnameMgrnameMgrsalary(department_name, manager_name, manager_salary) as 
    select d.dname, e.fname, e.salary
    from department d, employee e
    where d.mgrssn = e.ssn


## Exercise 5.8.b
##### A view that has the employee name, supervisor name, and employee salary for each employee who works in the ‘Research’ department.

    create view EnameSnameEsalary (employee_name, supervisor_name, employee_salary) as
    select e1.fname, e2.fname, e.salary
    from employee e1, employee e2
    where e1.superssn = e2.ssn


## Exercise 5.8.c
##### A view that has the project name, controlling department name, number of employees, and total hours worked per week on the project for each project.

    create view PnameDnameNofemployeesTotalhourss(project_name, department_name, numOfEmployees, hoursWorkedPerWeek) as
    select p.pname, d.dname, count(*), avg(salary)
    from employee e, department d, works_on wo, project p
    where e.dno = d.dnumber and wo.pno=p.pnumber and wo.essn=e.ssn and p.dnum = d.dnumber
    group by p.pname


## Exercise 5.8.d
##### A view that has the project name, controlling department name, number of employees, and total hours worked per week on the project for each project with more than one employee working on it.

    create view PnameDnameNofemployeesTotalhoursss(project_name, department_name, numOfEmployees, hoursWorkedPerWeek) as
    select p.pname, d.dname, count(*), avg(salary)
    from employee e, department d, works_on wo, project p
    where e.dno = d.dnumber and wo.pno=p.pnumber and wo.essn=e.ssn and p.dnum = d.dnumber and ( select from employee, department where dno = dnumber
    group by p.pname


## Exercise 6.15. 
##### Show the result of each of the sample queries in Section 6.5 as it would apply to the database state in Figure 3.6. (sayfa 171 den itibaren)

###### Query 1. Retrieve the name and address of all employees who work for the ‘Research’ department.​
    select e.fname, e.address
    from employee e, department d
    where e.dno = d.dnumber and d.dname = "Research";


###### Query 2. For every project located in ‘Stafford’, list the project number, the controlling department number, and the department manager’s last name, address, and birth date.

    select p.pnumber, d.dnumber, e.lname
    from project p, department d, employee e
    where e.ssn = d.mgrssn and p.dnum = d.dnumber and p.plocation="Stafford";
    ​
​
###### Query 3. Find the names of employees who work on all the projects controlled by department number 5.

    select e.fname
    from employee e
    where not exists (	select p.pnumber
                        from project p
                        where p.dnum = 5
                        
                        except
                        
                        select wo.pno
                        from works_on wo
                        where wo.essn = e.ssn 
                    )

​

###### Query 4. Make a list of project numbers for projects that involve an employee whose last name is ‘Smith’, either as a worker or as a manager of the department that controls the project.

    select p.pnumber
    from project p
    -- where p.pnumber in (smith soyadinda calisani olan projelerin pnumberlari) or p.number in (smith soyadindaki bir mudurun departmaninin yuruttugu projelerin pnumberlari)
    where p.pnumber in (select wo.pno from works_on wo, employee e where e.ssn=wo.essn and e.lname="Smith") or 
          p.pnumber in (select p.pnumber from department d2, employee e2, project p2 where d2.mgrssn=e2.ssn and p2.dnum=d2.dnumber and e2.lname="Smith");
​
​ 
###### Query 5. List the names of all employees with two or more dependents.

    select e.fname, count(*) as yakinsayisi
    from employee e, dependent dep
    where e.ssn = dep.essn
    group by e.ssn
    having count(*) > 1 ;-- or e.fname


###### Query 6. Retrieve the names of employees who have no dependents.

    select fname
    from employee
    where ssn not in (	select essn
                        from dependent
                    );
​
​
###### Query 7. List the names of managers who have at least one dependent.
    select fname
    from employee
    where ssn in ( select mgrssn from department) and ssn in ( select essn from dependent)


## Exercise 6.16.a
##### Retrieve the names of all employees in department 5 who work more than 10 hours per week on the ProductX project.

    select distinct fname
    from employee e, works_on wo, project p
    where e.ssn = wo.essn and wo.pno = p.pnumber  and e.dno = 5 and p.pname="ProductX" and wo.hours>10


## Exercise 6.16.b
##### List the names of all employees who have a dependent with the same first name as themselves.

    select e.fname
    from employee e, dependent d
    where e.ssn = d.essn and e.fname = d.dependent_name


## Exercise 6.16.c
##### Find the names of all employees who are directly supervised by ‘Franklin Wong’.

    select distinct e.fname 
    from employee e, employee s
    where e.superssn = s.ssn and s.fname = "Franklin" and s.lname = "Wong";


## Exercise 6.16.c
##### Find the names of all employees who are directly supervised by ‘Franklin Wong’.

    select distinct e.fname 
    from employee e, employee s
    where e.superssn = s.ssn and s.fname = "Franklin" and s.lname = "Wong";
