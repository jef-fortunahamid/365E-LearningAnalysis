# Analyzing Student Conversion and Engagement Metrics on the 365 E-Learning Platform

*(This project is from 365 Data Science. A Database Schema Script was provided to create the database.)*

## Business Task

The primary objective was to analyze student behavior on the 365 e-learning platform, focusing on:
1. Free-to-Paid Conversion Rate: The rate at which students who have engaged by watching a lecture go on to make a purchase.
2. Average Duration Between Registration and First-Time Engagement: The average number of days it takes for a student to engage with a lecture after registering.
3. Average Duration Between First-Time Engagement and First-Time Purchase: The average number of days it takes for a student to make a purchase after first engaging with a lecture.

The goal was to derive these metrics from a database containing tables for student information, student engagement, and student purchases.

## Techniques Used
**SQL Joins**:
- Used INNER JOIN between student_info and student_engagement to focus on students who have both registered and watched a lecture.
- Used LEFT JOIN with student_purchases to also include students who have watched but not purchased.
  
**Aggregation Functions**:
- Used MIN() to find the first date of engagement and purchase for each student.
- Used COUNT() for counting the number of records that meet certain criteria.
- Used AVG() to find the average duration between various events.

**Date Functions**:
- Used DATEDIFF() to calculate the number of days between two dates.

**Filtering Techniques**:
- Used GROUP BY to aggregate data at the student level.
- Used HAVING clause to filter aggregated data based on conditions related to first engagement and purchase dates.

**Common Table Expression (CTE)**:
- Utilized a CTE to encapsulate the complex query logic, improving code reusability and readability.

## Entity-Relationship Diagram
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/0798bc4b-a51b-4f91-bef2-12d063271bf1)

The ERD was created using the DB Diagram tool on `dbgiagram.io` with the following text command:
```sql
Table student_info {
  student_id integer [primary key]
  date_registered date
}

TABLE student_engagement {
  student_id integer
  date_watched date 
}

TABLE student_purchases {
  purchase_id integer [primary key]
  student_id integer
  date_opurchase date
}

Ref: "student_info"."student_id" < "student_engagement"."student_id"

Ref: "student_info"."student_id" < "student_purchases"."student_id"
```

## Data Exploration
### student_info Table
```sql
SELECT *
FROM student_info
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/adc58d13-3db3-418a-a346-27f833c4c3aa)

### student_engagement Table
```sql
SELECT *
FROM student_engagement
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/a490cd49-019e-4b90-bb8b-4f2e2e711437)

### student_purchases Table
```sql
SELECT *
FROM student_purchases
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/3989b5c4-2865-4c0f-9a5b-b746694e33d5)

The schema provided a clean database hence, no further cleaning has been done.

## Task 1
Appropriately join and aggregate the tables, create a new result dataset comprising the following columns:

- `student_id` – (`int`) the unique identification of a student
- `date_registered` – (`date`) the date on which the student registered on the 365 platform
- `first_date_watched` – (`date`) the date of the first engagement
- `first_date_purchased` – (`date`) the date of first-time purchase (`NULL` if they have no purchases)
- `date_diff_reg_watch` – (`int`) the difference in days between the registration date and the date of first-time engagement
- `date_diff_watch_purch` – (`int`) the difference in days between the date of first-time engagement and the date of first-time purchase (`NULL` if they have no purchases)

**Hint:** *Research the `DATEDIFF` function in MySQL.*

Note the Venn diagram below.
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/cac1bf57-cd11-4940-b2a6-e518cec48ff2)

The resulting set you retrieve should include the student IDs of students entering the diagram’s shaded region. Additionally, our objective is to determine the conversion rate of students *who have already watched a lecture*. Therefore, filter your result dataset so that the date of first-time engagement comes before (or is equal to) the date of first-time purchase.

**Note:** *To ensure an engagement occurs before the purchase, we need the timestamp of both events. But this information is not in the dataset we’re working with.*

**Sanity check:** *The number of records in the resulting set should be 20,255.*

Steps Taken:




```sql
WITH student_metrics AS (
SELECT
	  si.student_id
	, si.date_registered
    , MIN(se.date_watched) AS first_date_watched
    , MIN(sp.date_purchased) AS first_date_purchased
    , DATEDIFF(MIN(se.date_watched), si.date_registered) AS date_diff_reg_watch
    , DATEDIFF(MIN(sp.date_purchased), MIN(se.date_watched)) AS date_diff_watch_pur
FROM student_info si
JOIN student_engagement se
	ON si.student_id = se.student_id
LEFT JOIN student_purchases sp
	ON si.student_id = sp.student_id
GROUP BY 
	  si.student_id
	, si.date_registered
HAVING 
    (first_date_watched <= first_date_purchased OR first_date_purchased IS NULL)
)
SELECT COUNT(*)
FROM date_difference_student;
```
