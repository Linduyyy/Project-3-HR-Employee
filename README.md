# HR Employee Analysis SQL and Power BI Project

## Project Overview

**Project Title**: HR Employee Analysis
**Level**: Beginner

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a HR Employee Analysis database, performing exploratory data analysis (EDA), and answering specific analysis questions through SQL queries.

## Objectives
1. **Set up a hr employee analysis database**: Create and populate a hr analysis database with the provided hr data.
2. **Data Cleaning**: Standardize inconsistent date formats and ensure column data types are accurate.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4.  **HR Analysis**: Use SQL and Power BI to clean, explore, and visualize employee data to uncover trends and workforce insights.


## Project Structure

### 1. Database Setup
- **Database Creation**: Created a database named `project`.
- **Table import**: Imported HR employee data into a table named `hr`.

```sql
create database project;
```

### 2. CRUD Operations
**Task 1. Standardized Format: Converted inconsistent date formats in the birthdate, hire_date, and termdate columns to a uniform YYYY-MM-DD format using SQL, then updated the column type to DATE.**

**birthdate column**

```sql
update hr
set birthdate = case
    when birthdate like '%/%' then date_format(str_to_date(birthdate, '%m/%d/%Y'), '%Y-%m-%d')
    when birthdate like '%-%' then date_format(str_to_date(birthdate, '%m-%d-%Y'), '%Y-%m-%d')
    else null
    end;

alter table hr
modify column birthdate date;  
```

**hire_date column**

```sql
update hr 
set hire_date = case
    when hire_date like '%/%' then date_format(str_to_date(hire_date, '%m/%d/%Y'), '%Y-%m-%d')
    when hire_date like '%-%' then date_format(str_to_date(hire_date, '%m-%d-%Y'), '%Y-%m-%d')
    else null
    end;
    
alter table hr
modify column hire_date date;
```

**termdate column**

```sql
UPDATE hr
SET termdate = CASE
  WHEN TRIM(termdate) = '' OR termdate IS NULL THEN '0000-00-00'
  ELSE DATE(STR_TO_DATE(termdate, '%Y-%m-%d %H:%i:%s UTC'))
  END;

SET sql_mode = 'ALLOW_INVALID_DATES';

alter table hr
modify column termdate date;
```

**Add column age**
```sql
alter table hr
add column age int;

update hr
set age = timestampdiff(year, birthdate, curdate());
```

### 3. Data Analysis & Findings
The following SQL Queries were developed to answer specific analysis questions:

1. **what is the gender breakdown of employees in the company?**
  ```sql
select gender, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by gender;
 ```

2. **what is the race/ethnicity breakdown of employees in the company?**
```sql
select race, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by race
order by 2 desc;
```

3. **what is the age distribution of employees in the company?**
```sql
select 
	min(age) as youngest,
    max(age) as oldest
from hr
where age >= 18 and termdate = '000-00-00';

select 
case
	  when age >= 18 and age <= 24 then '18-24'
    when age >= 25 and age <= 34 then '25-34'
    when age >= 35 and age <= 44 then '35-44'
    when age >= 45 and age <= 54 then '45-54'
    when age >= 55 and age <= 64 then '55-64'
    else '65+'
    end as age_group, gender,
    count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by age_group, gender
order by age_group, gender;
```

4. **How many employees work at headquarters vs remote location?**
 ```sql
select location, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by location;
```

5. **what is the average length of employment for employees who have been terminated?**
```sql
select round(avg(datediff(termdate, hire_date))/365, 0) as avg_length_employment
from hr
where termdate <= curdate() and termdate <> '0000-00-00' and age >= 18;
```

6. **how does the gender distribution vary across departments and job title?**
```sql
select department, gender, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by department, gender
order by department;
```

7. **what is distribution of the job titles across the company?**
```sql
select jobtitle, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by jobtitle
order by jobtitle asc;
```

8. **which department has the highest turnover rate?**
```sql
select
	  department,
    total_count,
    terminated_count,
    terminated_count/total_count as termination_rate
from (
	select department,
    count(*) as total_count,
    sum(case when termdate <> '0000-00-00' and termdate <= curdate() then 1 else 0 end) as terminated_count
    from hr
    where age>= 18
    group by department
    ) as subquery
order by termination_rate desc;
```

9. **what is the distribution of employees across location by city and state?**
```sql
select * from hr;
select location_state, count(*) as count
from hr
where age >= 18 and termdate = '000-00-00'
group by location_state
order by 2 desc;
```

10. **how has the company employee count changed overtime based on hire and term date?**
```sql
select 
	  years,
    hires,
    terminations, 
	  hires - terminations as net_change,
    round((hires-terminations)/hires*100, 2)as net_chage_percent
from (
	select year(hire_date) as years,
    count(*) as hires,
    sum(case when termdate <> '0000-00-00' and termdate <= curdate() then 1 else 0 end) as terminations
  from hr
  where age >= 18
  group by year(hire_date)
  ) as subquery
order by years asc;
```
    
11. **what is the tenure distribution for each department?**
```sql
select department, round(avg(datediff(termdate, hire_date)/365),0) as avg_tenure
from hr
where termdate <= curdate() and termdate <> '0000-00-00' and age >= 18
group by department;
```

## Key Insights

- The majority of employees are between 25 and 34 years old.
- The turnover rate is highest in the Sales department.
- Remote work employees show slightly different age distributions compared to those at headquarters.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analyst, converting database set up, data cleaning, exploratory data analysis.

## How to Use :

1. **Clone the Repository**: Clone this project repository from GitHub.
2. **Setup the Database**: Run the SQL scripts provided in the `database_setup.sql` file to create and populate the database.
3. **Import file**: Import `hr_employee.csv` into MySQL
4. **Run the Queries**: Use the SQL queries provided in the `database projects - data cleaning & analysis.sql` file to perform your analysis.
5. **Explore and Modify**: Feel free to modify the queries to explore different aspects of the dataset or answer additional business questions.
6. **Visualization**: Use the clean data in Power BI for visualization

## Power BI Dashboard

Here is a snapshot of the Power BI dashboard created from the cleaned dataset:

### Page 1:
![page 1](![Screenshot 2025-06-15 143736](https://github.com/user-attachments/assets/dc2663d7-2725-40a3-b28d-32fc770246ac)


