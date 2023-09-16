# Analyzing Student Conversion and Engagement Metrics on the 365 E-Learning Platform

*(This project is from 365 Data Science. A Database Schema Script was provided to create the database)*

## Business Task

The primary objective was to analyze student behavior on the 365 e-learning platform, focusing on:
1. Free-to-Paid Conversion Rate: The rate at which students who have engaged by watching a lecture go on to make a purchase.
2. Average Duration Between Registration and First-Time Engagement: The average number of days it takes for a student to engage with a lecture after registering.
3. Average Duration Between First-Time Engagement and First-Time Purchase: The average number of days it takes for a student to make a purchase after first engaging with a lecture.

The goal was to derive these metrics from a database containing tables for student information, student engagement, and student purchases.

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









