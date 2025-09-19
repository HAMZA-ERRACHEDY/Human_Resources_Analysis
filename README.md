
CREATE TABLE human_resources_copy
LIKE human_resources;
INSERT INTO human_resources_copy
SELECT * FROM human_resources;

SELECT * FROM human_resources;

ALTER TABLE human_resources
CHANGE COLUMN ï»¿id emp_id VARCHAR(20) ;

-- Disable Safe Updates Mode
SET sql_safe_updates=0;

UPDATE human_resources 
SET birthdate = CASE 
	WHEN birthdate LIKE '%/%' THEN DATE_FORMAT(STR_TO_DATE(birthdate,'%m/%d/%Y'),'%Y-%m-%d')
	WHEN birthdate LIKE '%-%' THEN DATE_FORMAT(STR_TO_DATE(birthdate,'%m-%d-%Y'),'%Y-%m-%d')
	ELSE NULL
END;

-- Store birthdate as a true DATE, not just a String
ALTER TABLE human_resources
MODIFY COLUMN birthdate DATE;

DESCRIBE human_resources;

UPDATE human_resources
SET hire_date = CASE 
	WHEN hire_date LIKE '%/%' THEN STR_TO_DATE(hire_date,'%m/%d/%Y') 
	WHEN hire_date LIKE '%-%' THEN STR_TO_DATE(hire_date,'%m-%d-%Y') 
	ELSE NULL
END;

ALTER TABLE human_resources
MODIFY COLUMN hire_date DATE;

SELECT termdate FROM human_resources ;

UPDATE human_resources 
SET termdate = DATE(STR_TO_DATE(termdate,'%Y-%m-%d %H:%i:%s UTC'))
WHERE termdate IS NOT NULL AND termdate != '';


UPDATE human_resources
SET termdate = '0000-00-00'
WHERE termdate = '';

UPDATE human_resources
SET termdate = NULL
WHERE termdate = '0000-00-00';

ALTER TABLE human_resources
MODIFY COLUMN termdate DATE;

ALTER TABLE human_resources
ADD COLUMN age INT
AFTER birthdate;

UPDATE human_resources
SET age = TIMESTAMPDIFF(YEAR, birthdate, CURDATE()) ;


-- QUESTIONS

-- 1. What is the gender breakdown of employees in the company?

SELECT  gender,
COUNT(*) AS count
FROM human_resources
WHERE age >= 18 and termdate IS NULL
GROUP BY gender ;

-- 2. What is the race/ethnicity breakdown of employees in the company?

SELECT race,
COUNT(*) AS count
FROM human_resources
WHERE age >= 18 and termdate IS NULL
GROUP BY race 
ORDER BY count DESC;

-- 3. What is the age distribution of employees in the company?

SELECT  MIN(age) AS Yongest , 
		MAX(age) AS Oldest
FROM human_resources;

SELECT
	CASE
		WHEN age >= 20 AND age <= 29 THEN '20-29'
		WHEN age >= 30 AND age <= 39 THEN '30-39'
		WHEN age >= 40 AND age <= 49 THEN '40-49'
		ELSE '50-59'
	END AS age_group ,
	COUNT(*) AS count
FROM human_resources
WHERE age >= 18 and termdate IS NULL
GROUP BY age_group
ORDER BY age_group ;


SELECT
	CASE
		WHEN age >= 20 AND age <= 29 THEN '20-29'
		WHEN age >= 30 AND age <= 39 THEN '30-39'
		WHEN age >= 40 AND age <= 49 THEN '40-49'
		ELSE '50-59'
	END AS age_group , gender,
	COUNT(*) AS count
FROM human_resources
WHERE age >= 18 and termdate IS NULL
GROUP BY age_group , gender
ORDER BY age_group , gender;

-- 4. How many employees work at headquarters versus remote locations?

SELECT location, COUNT(emp_id) count
FROM human_resources
WHERE age >= 18 and termdate IS NULL
GROUP BY location;

-- 5. What is the average length of employment for employees who have been terminated?

SELECT
ROUND(AVG(TIMESTAMPDIFF(YEAR, hire_date, termdate))) AS avg_leng 
FROM human_resources
WHERE termdate <= CURDATE() and age >= 18 and termdate IS NOT NULL ;

-- 6. How does the gender distribution vary across departments?

SELECT department, gender, COUNT(*) AS count
FROM human_resources 
WHERE age >= 18 and termdate IS NOT NULL 
GROUP BY department, gender
ORDER BY department ;


-- 7. What is the distribution of job titles across the company?

SELECT jobtitle, COUNT(*) AS count
FROM human_resources 
WHERE age >= 18 and termdate IS NOT NULL 
GROUP BY jobtitle 
ORDER BY jobtitle;

-- 8. Which department has the highest turnover rate?

SELECT 
	department,
	total_count,
	terminated_count ,
	terminated_count/total_count AS terminated_rate
FROM (
	SELECT 
		department,
		COUNT(*) AS total_count,
		SUM(CASE WHEN termdate IS NOT NULL AND termdate <= CURDATE()  THEN 1 ELSE 0 END) AS terminated_count
	FROM human_resources 
	WHERE age >= 18 
	GROUP BY department
	) AS subquery
ORDER BY terminated_rate DESC;

**9. What is the distribution of employees across locations by state?**
```sql
SELECT
	location_state ,
	COUNT(emp_id) as count 
FROM human_resources
WHERE age >= 18 and termdate IS NOT NULL 
GROUP BY location_state 
ORDER BY count DESC;
```
**10. How has the company's employee count changed over time based on hire and term dates?**
```sql
SELECT
	year,
    hires,
	terminations,
	hires - terminations AS net_change,
    ROUND((hires - terminations)/hires * 100 ,2) AS net_change_percent
FROM (
		SELECT 
			YEAR(hire_date) AS year,
			COUNT(*) AS hires,
			SUM(CASE WHEN termdate IS NOT NULL AND termdate <= CURDATE() THEN 1 ELSE 0 END) AS terminations 
		FROM human_resources 
		WHERE age >= 18 
		GROUP BY YEAR(hire_date)
	)AS subquery
ORDER BY year;
```
**11. What is the tenure distribution for each department?**
```sql
SELECT 
	department,
	ROUND(AVG(DATEDIFF(termdate, hire_date)/365)) as avg_tenure 
FROM human_resources 
WHERE termdate <= CURDATE() AND age >= 18 AND termdate IS NOT NULL 
GROUP BY department;
```

 
