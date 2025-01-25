# Gross-pay
Analysis of employee gross pay for year 2024, State of Missouri, USA. 
I sourced for the dataset from Kaggle.com/datasets/ishajangir/2024-state-employee-pay. 
Thanks to Ishajangir for sharing. 

This dataset containns salary information for state employees of the State of Missouri, USA. 
It includes key features such as the calender year, agency name, employee position, employee name and salary. 
The dataset offers a comprehensive view of public sector compensation witthin the state. 
The dataset contains 72,869 rows of data. 

I explored the data set for null values and blanks within Excel and proceeded to create a new schema in Mysql named missouri_public_employees. 
I then created a table named pay_data and uploaded the .csv dataset file. 
i.e missouri_public_employees.pay_data. 
To gain insights into the dataset i came up with the following analytical questions and solutions using SQL and Excel: 

1. what is the total number of public sector employees paid in year 2024
2. what is the average salary of public employees accross the entire state
3. what is the maximum salary paid in year 2024
4. what is the minimum salary paid in year 2024
5. who was paid the highest salary and in what agency was the person employed/job title
6. who was paid the minimum salary and in what agency was the person employed/job title
7. what is the average salary of missori public sector workers
8. what proportion of employees earn between above 100k annually and below 100k annually
9. which job postions earned the highest and lowest on average.
10. which agencies paid the highest and lowest on average

	## To view the dataset:
SELECT *
FROM missori_public_employees.pay_data;

	## QUESTION 1:
SELECT COUNT(*) 
FROM missori_public_employees.pay_data;
	## Total count: 74170

	## QUESTION 2:
SELECT AVG(gross_pay)
FROM missori_public_employees.pay_data;
	## Average gross pay: 37737.81219765355
    
    ## Rounding gross pay to 2 decimal places:
SELECT round(avg(gross_pay),2)
FROM missori_public_employees.pay_data;
	## Average gross pay: 37737.81

	## QUESTION 3:
SELECT ROUND(MAX(gross_pay),2)
FROM missori_public_employees.pay_data;
	## Maximum gross pay rounded up to 2 decimal places: 1537077.17

	## QUESTION 4:
SELECT ROUND(MIN(gross_pay),2)
FROM missori_public_employees.pay_data;
	## Minimum gross pay rounded up to 2 decimal places: -1834.88
    
	## QUESTION 5:
SELECT employee_first_name, employee_last_name, position_title, gross_pay
FROM missori_public_employees.pay_data
WHERE gross_pay >= 1537077.17;
	## This provides us with an anomaly: first name:workers, last name: client/patient, positon client/patient worker.
    
    ## I therefore search through the dataset for the second and third highest earners accross the state
SELECT employee_first_name, employee_last_name, position_title, gross_pay
FROM missori_public_employees.pay_data
ORDER BY gross_pay DESC
LIMIT 3;
	## This provides the top three highest earners in the state and shows that the first entry is an anomaly as it earns 
    ## close to 1 million more than the second highest. 
    
    ## To confirm this i search employees with client/patient worker job title
SELECT employee_first_name, employee_last_name, position_title, gross_pay
FROM missori_public_employees.pay_data
WHERE position_title = 'client/patient worker';
	## The result: 1 row
    
    ## Referencing the source csv file, using conditional formating and filters it becomes obvious that the information is erroneous
    ## and should be removed
DELETE FROM missori_public_employees.pay_data
WHERE position_title = 'client/patient worker';

	## Find new maximum gross pay to 2 decimal places
SELECT ROUND(MAX(gross_pay),2)
FROM missori_public_employees.pay_data;
	## Maximum gross pay = 509932.87
    
    ## The data has negative gross pay values. to determine the number of negative gross pay:
SELECT COUNT(*)
FROM missori_public_employees.pay_data
WHERE gross_pay <= 0;
	## Total number of employees earning less than equal to 0: 63
    
    ## Remove all 63 rows containing gross pay values equal to less than 0 i.e negative gross pay
DELETE FROM missori_public_employees.pay_data
WHERE gross_pay <= 0;

	## To make the data more usable, gross pay less than 1000 were then removed
DELETE FROM missori_public_employees.pay_data
WHERE gross_pay < 1000;
    ## Total number of employees earning less than 1000 removed: 3114
    
	## calculate new average and minimum gross pay rounded to 2 decimal places
SELECT ROUND(Min(gross_pay),2)
FROM missori_public_employees.pay_data;
	## new minimum gross pay: 1000

SELECT ROUND(AVG(gross_pay),2)
FROM missori_public_employees.pay_data;
    ## new average gross pay: 39385.42
    
	## QUESTION 6
SELECT *
FROM missori_public_employees.pay_data
WHERE gross_pay = 1000;
	## Employees with lowest pay: 10 employees earn 1000 - 6 in agriculture, 3 in commerce and insurance and one in mental health

	## QUESTION 7 AND QUESTION 10
SELECT ROUND(AVG(gross_pay),2), agency_name
FROM missori_public_employees.pay_data 
GROUP BY agency_name
ORDER BY ROUND(AVG(gross_pay),2) DESC;
	## This provides the average gross pay per agency. it shows that the office of governor earns the highest on average: 59558.24 
    ## and agriculture earns the lowest on average: 20197.4
    
    ## QUESTION 8
SELECT *, SUM(gross_pay) OVER (PARTITION BY agency_name) AS agency_pay_total,
	gross_pay / SUM(gross_pay) OVER (PARTITION BY agency_name) AS gross_pay_percentage
FROM missori_public_employees.pay_data;

	## QUESTION 9
SELECT AVG(gross_pay), position_title
FROM missori_public_employees.pay_data
GROUP BY position_title
ORDER BY AVG(gross_pay) DESC;
	## The highest average gross pay position is medical administrator followed by psychiatris
    ## The lowest average gross pay positon is accounting specialist ii followed by accounting specialist

I hope to delve deeper into the dataset in the near future.
