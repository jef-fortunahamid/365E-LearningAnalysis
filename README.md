# Analysing Student Conversion and Engagement Metrics on the 365 E-Learning Platform

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

We have to look at the Venn Diagram provided. The shaded area (which is part of student_info) represents students who have both registered and watched a lecture. Some of these students have also made a purchase, but not all. With this information, it will tell us what type of JOIN we need to create the dataset we need for the next task.

The area of interest is the whole student_engagement table, so we need to do an INNER JOIN. For the next JOIN, we have to look at the requirements of the 'first_date_purchased' column. This shows the first-time purchase and is NULL if they have not made a purchase. This means we will do a LEFT JOIN to make sure we include all students who have registered and watched a lecture, whether or not they have made a purchase. This is important because we want to include students who have and haven't made a purchase in our analysis (this is part of our business task).

For the columns `first_date_watched` and `first_date_purchased`, we need the date of first engagement and first-time purchase respectively. So we will be using the MIN() function.
 
The last two columns for our dataset we need the date difference between `date_registered` and `first_date_watched` for `date_diff_reg_watch` column, and between `first_date_watched` and `first_date_purchased` for column `date_diff_watch_purch`. For this columns we need tro use MySQL DATEDIFF() function.
```sql
DATEDIFF(date1, date2)
```
The function will return `date1 - date2`, expressed as a value in days.

From the task we are asked to:
*The resulting set you retrieve should include the student IDs of students entering the diagram’s shaded region. Additionally, our objective is to determine the conversion rate of students *who have already watched a lecture*. Therefore, filter your result dataset so that the date of first-time engagement comes before (or is equal to) the date of first-time purchase.*

This tells us that first_date_watched is less than or equal to first_date_purchased. Also, first_date_purchased is NULL if they have not made a purchase. With this information, we need to include this in our WHERE clause. However, since we are getting the first date using the MIN() function, this can't be used in the WHERE clause. So, we will use the HAVING clause for this.

So our final query statement is:

```sql
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
```
If we count the number of rows we've got:
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
SELECT COUNT(*) AS number_of_records
FROM student_metrics;
```
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/df40fe8e-7073-44de-b239-af0b99d3dc8b)

It gives us 20,255 rows (the same number of records from our task 1).

## Task 2
In this task, you should use the query you’ve created and retrieve the following three metrics.

**Free-to-Paid Conversion Rate:**

This metric measures the proportion of engaged students who choose to benefit from full course access on the 365 platform by purchasing a subscription after watching a lecture. It is calculated as the ratio between:

- The number of students who watched a lecture and purchased a subscription.The total number of students who have watched a lecture. Convert the result to percentages and call the field `conversion_rate`.

**Average Duration Between Registration and First-Time Engagement:**

This metric measures the average duration between the date of registration and the date of first-time engagement. This will tell us how long it takes, on average, for a student to watch a lecture after registration. The metric is calculated by finding the ratio between:

- The sum of all such durations.The number of these durations. Call the field `av_reg_watch`.

**Average Duration Between First-Time Engagement and First-Time Purchase:**

This metric measures the average time it takes individuals to subscribe to the platform after viewing a lecture. It is calculated by dividing:

- The total sum of all these durationsThe total number of such durations. Call the field `av_watch_purch`.

This is our final query:

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
SELECT 
   ROUND(COUNT(*)/(SELECT COUNT(*) FROM student_metrics) * 100, 2) AS conversion_rate
 , ROUND(AVG(date_diff_reg_watch), 2) AS av_reg_watch
 , ROUND(AVG(date_diff_watch_pur), 2) AS av_watch_purch
FROM student_metrics
WHERE
	first_date_purchased IS NOT NULL;
```
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/5426bc41-b20b-4ef0-b186-9bbae58fd30a)

## Task 3
First, consider the conversion rate and compare this metric to industry benchmarks or historical data. Second, examine the duration between the registration date and date of first-time engagement. A short duration—watching on the same or the next day—could indicate that the registration process and initial platform experience are user-friendly. At the same time, a longer duration may suggest that users are hesitant or facing challenges. Third, regarding the time it takes students to convert to paid subscriptions after their first lecture, a shorter span would suggest compelling content or effective up-sell strategies.

Next, using a tool different from SQL (e.g., Excel, Python), calculate the median and mode values of the date difference between registering and watching a lecture. Do the same for the date difference between watching a lecture and purchasing a subscription. Compare the two metrics of each set to their respective mean values. To interpret the results even better, create a distribution graph and try to understand the relationship between these metrics (mean, median, and mode). Focus on the following key points.

Distribution Symmetry
The distribution is likely symmetrical when the mean, median, and mode are equal or very close, forming a bell curve. If they differ, the data might be skewed to the left—indicated by a long tail on the left side—or to the right with a long tail on the right side.

Outliers
If the mean is much higher or lower than the median, it suggests that there are outliers. For instance, if the average time to purchase a subscription is significantly higher than the median, it may imply that a few students took an exceptionally long time to decide.

Common Patterns
If a specific value or set of values has a high frequency—corresponding to the mode of the dataset—it can spotlight common behaviors. For example, if the mode of the time between registration and watching a lecture is one day, it indicates that many students start watching the day after registration.

Answer:
For date_diff_reg_watch:

Distribution Symmetry: The distribution is right-skewed, indicated by a long tail on the right side. The mean is significantly higher than the median and the mode, both of which are at zero.

Outliers: The long tail suggests that there are outliers—students who take an exceptionally long time to start watching lectures after registration. These outliers have inflated the mean.

Common Patterns: The mode being at zero indicates that a high number of students start watching lectures right after registration.
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/a453bfc6-dd78-4ea7-8e1d-21b1e1cd6595)

For date_diff_watch_pur:

Distribution Symmetry: Similar to date_diff_reg_watch, the distribution for date_diff_watch_pur is also right-skewed, suggesting that most students make a purchase shortly after their first engagement, but there are outliers who take much longer.

Outliers: The mean is significantly higher than the median and the mode, indicating the presence of outliers. These outliers are students who took an exceptionally long time to make a purchase.

Common Patterns: The mode being at zero suggests that many students make a purchase immediately after watching their first lecture.
![image](https://github.com/jef-fortunahamid/365E-LearningAnalysis/assets/125134025/de3ce60b-b9bd-4850-ad09-95a14fa351b0)


