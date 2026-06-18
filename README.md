# SQL Interview Questions From Various Companies

## KPMG Interview Question

Problem Statement:

The Delivery table contains four columns: Brand_1, Brand_2, Brand_3 and Winner.

We need to write an SQL query to calculate:

• Total number of rides
• Number of rides won
• Number of rides lost


```sql
--Dataset - Mysqlv5.7
CREATE TABLE Delivery_Partner (
    Brand_1 VARCHAR(512),
    Brand_2 VARCHAR(512),
    Brand_3 VARCHAR(512),
    Winner VARCHAR(512)
);

INSERT INTO Delivery_Partner (Brand_1, Brand_2, Brand_3, Winner) VALUES
 ('A', 'B', 'C', 'B'),
 ('B', 'C', 'E', 'E'),
 ('C', 'A', 'D', 'D'),
 ('D', 'E', 'A', 'A'),
 ('F', 'B', 'C', 'F');
```

```sql
WITH brands AS
(
SELECT Brand_1 FROM Delivery_Partner
UNION ALL
SELECT Brand_2 FROM Delivery_Partner
UNION ALL
SELECT Brand_3 FROM Delivery_Partner
),

winner AS 
(
SELECT winner , COUNT(winner) as wincount FROM Delivery_Partner
GROUP BY winner ,brand_1
ORDER BY winner
),


race_info AS
(
SELECT b.brand_1 as brand, COUNT(b.brand_1) as total_rides, CASE WHEN wincount IS NULL THEN 0 ELSE wincount END as total_rides_won 
FROM brands b
LEFT JOIN winner w ON
b.brand_1 = w.winner
GROUP BY b.brand_1,wincount
ORDER BY b.brand_1
)

SELECT brand,total_rides,total_rides_won , total_rides - total_rides_won as total_rides_lost FROM race_info
ORDER BY brand
```
<img width="1792" height="391" alt="image" src="https://github.com/user-attachments/assets/27bcad5a-da70-48f6-a2b6-695a5cba1556" />
---

**Question 2 :**  Write an SQL query that produces a comma-separated list of passengers who can be accommodated in each lift without exceeding its capacity.
The passengers should be listed in increasing order of their weight.

```sql
CREATE TABLE  Lift_Passengers(
Passenger_Name Varchar(20),
Weight_Kg int,
Lift_Id int);

CREATE TABLE Lift
( Id int,
Capacity_Kg Bigint);

INSERT INTO Lift_Passengers  (Passenger_Name,Weight_Kg,Lift_Id) VALUES
('Mark',85,1),
('Antony',73,1),
('David',95,1),
('Mary',80,1),
('John',83,2),
('Robert',77,2),
('Maria',73,2),
('Susan',85,2);


Insert into Lift (Id,Capacity_Kg) Values
(1,300),
(2,350);

```
```sql
--SOLUTION 1 : Various combinations of passengers that can enter the lift without exceeding the lift capacity 
WITH lift_stats AS
(
SELECT lp1.lift_id,lp1.passenger_name as p1 ,lp2.passenger_name as p2 ,lp3.passenger_name as p3,lp1.weight_kg,lp2.weight_kg,lp3.weight_kg , lp1.weight_kg+lp2.weight_kg+lp3.weight_kg as combined_weight
FROM Lift_Passengers lp1 
JOIN Lift_Passengers lp2 ON 
lp1.Lift_Id = lp2.Lift_Id
JOIN Lift_Passengers lp3 ON
lp2.Lift_Id =lp3.Lift_Id
WHERE lp1.passenger_name <> lp2.passenger_name AND lp1.passenger_name != lp3.passenger_name AND lp2.passenger_name != lp3.passenger_name
AND lp1.weight_kg < lp2.weight_kg  AND lp1.weight_kg < lp3.weight_kg AND lp2.weight_kg < lp3.weight_kg
)


SELECT lift_id , p1|| ' ,' || p2 || ' ,' || p3 as passengers_combinations ,combined_weight
  FROM lift_stats

```
<img width="1538" height="439" alt="image" src="https://github.com/user-attachments/assets/c91873c9-a71e-4399-af1d-69abb1543bda" />

---
```sql
SOLUTION 2 : Using Cumulative Totals
WITH lift_stats AS(

SELECT * , SUM(weight_kg) OVER (PARTITION BY lift_id ORDER BY weight_kg) as cumulative_weight
FROM Lift_Passengers lp 
JOIN lift l ON 
lp.lift_id = l.id
)


SELECT lift_id, STRING_AGG(passenger_name, ' , ' ORDER BY weight_kg) as passenger_combinations 
FROM lift_stats
WHERE cumulative_weight < capacity_kg
GROUP BY lift_id

```
<img width="873" height="211" alt="image" src="https://github.com/user-attachments/assets/5f6996f4-28d0-4e01-a5da-13c39aa862bf" />

---
**Question 3 :** User Retention (Cohort Analysis) 
Calculate retention for users whose first purchase was in Jan 2024 and  returned in Feb 2024
```sql
CREATE TABLE user_purchases(
user_id INT,
purchase_date DATE );

INSERT INTO user_purchases VALUES
(1,'2024-01-10'),(1,'2024-02-15'), -- retained
(2,'2024-01-20'), -- not retained
(3,'2024-02-01'), -- not  in JAN cohort
(4,'2024-01-05'),(1,'2024-02-20') -- retained

```
```sql
WITH first_purchase AS
(
  SELECT user_id , MIN(purchase_date) as first_purchase FROM user_purchases
  GROUP BY user_id
),
jan_users_cohort AS
(
  SELECT user_id FROM first_purchase WHERE
  first_purchase >= '2024-01-01' AND first_purchase <'2024-02-01'
 ),
 
 users_retained_feb AS
 (
   SELECT up.user_id  FROM user_purchases up
   INNER JOIN jan_users_cohort jc ON
   up.user_id = jc.user_id
   WHERE purchase_date >= '2024-02-01' AND purchase_date < '2024-03-01'
  )
  
  SELECT 
  
ROUND(  100 * (SELECT COUNT(DISTINCT user_id) FROM users_retained_feb )::NUMERIC/ (SELECT COUNT(DISTINCT user_id) FROM jan_users_cohort) ,2)
  
  AS retention_date

```
<img width="305" height="143" alt="image" src="https://github.com/user-attachments/assets/fc6bea7b-3c10-4d52-a5cd-5e3d2cef38e4" />

---

**Question 4 :**  Identifying "Repeated" Transactions (10-Min Rule)
Identify potential duplicate transactions. Find instances where the same user_id spent the same amount at the same merchant_id within 10 minutes of a previous transaction.
Sample Data:
```sql
CREATE TABLE transactions (
    transaction_id INT,
    user_id INT,
    merchant_id INT,
    amount DECIMAL(10,2),
    created_at DATETIME
);

INSERT INTO transactions VALUES 
(101, 1, 10, 50.00, '2024-03-01 10:00:00'),
(102, 1, 10, 50.00, '2024-03-01 10:05:00'), -- Duplicate (within 5 mins)
(103, 1, 10, 50.00, '2024-03-01 10:20:00'); -- Not duplicate (>10 mins)
 ```

```sql
WITH transactions_cte AS
(
SELECT *, LAG(created_at) OVER(PARTITION BY user_id,merchant_id,amount ORDER BY created_at) as previous_transaction_created_at
FROM transactions
),

transaction_times AS
(
SELECT * , 
(created_at  - previous_transaction_created_at )  AS transaction_time_difference
FROM transactions_cte
)

SELECT transaction_id as duplicate_transaction_id FROM transaction_times WHERE

previous_transaction_created_at IS NOT NULL AND 
transaction_time_difference < INTERVAL '10 minutes'
```
<img width="1599" height="239" alt="image" src="https://github.com/user-attachments/assets/d2a6363d-f2c9-46e0-8320-e4fb7a7914e9" />

<img width="399" height="155" alt="image" src="https://github.com/user-attachments/assets/410386ca-976b-4f1d-8417-f01f1f33e050" />

---
**Question 5 :** Calculating Median Without Built-in Functions
 Calculate the median salary for each department. If a department has an even number of employees, the median is the average of the two middle values.

```sql 
Sample Data:

CREATE TABLE employees (
    emp_id INT,
    dept_id INT,
    salary INT
);
INSERT INTO employees VALUES 
(1, 1, 5000), (2, 1, 6000), (3, 1, 7000),    -- Median is 6000
(4, 2, 4000), (5, 2, 8000);                 -- Median is 6000 (avg of 4k & 8k) 
```
```sql
WITH employee_rank AS
(
SELECT * , ROW_NUMBER() OVER(PARTITION BY dept_id ORDER BY salary) as rank FROM employees
) ,

employee_count AS
(
 SELECT dept_id, COUNT(*) as employee_count FROM employees GROUP BY dept_id 
)

SELECT er.dept_id , AVG(salary) :: INT as median_salary
FROM 
employee_rank er JOIN 
employee_count ec
ON er.dept_id = ec.dept_id
WHERE (employee_count % 2 = 1 AND rank = (employee_count+1)/2  )
OR (employee_count % 2 = 0 AND rank IN ( employee_count/2 , (employee_count/2)+1))
GROUP BY er.dept_id
ORDER BY er.dept_id

```
<img width="946" height="204" alt="image" src="https://github.com/user-attachments/assets/f1a60b65-1258-4939-af23-d4fc09c32f3f" />

---

**Question 6 :** Continuous Winning Streaks
For each player, find the length of their longest continuous winning streak (consecutive 'Win' results).

```sql
Sample Data:

CREATE TABLE match_results (
    player_id INT,
    match_date DATE,
    result VARCHAR(10) -- 'Win', 'Loss', 'Draw'
);
INSERT INTO match_results VALUES 
(1, '2024-01-01', 'Win'), (1, '2024-01-02', 'Win'), (1, '2024-01-03', 'Loss'), (1, '2024-01-04', 'Win'),
(2, '2024-01-01', 'Win'), (2, '2024-01-02', 'Loss');
```

```sql
WITH previous_result_cte AS
(
SELECT *, LAG(result) OVER(PARTITION BY player_id ORDER BY match_date) as previous_result FROM match_results
),

--this is an islands and gaps problem, since we are trying to identify consecutive streaks we consider those groups as one island and when the streak breaks its called a gap

-- Now we need to identify the streak boundary , for that I will create flags 

flag_cte AS
(
SELECT *, CASE WHEN result='Win' AND previous_result='Win' THEN 0 ELSE 1 END as flag
FROM previous_result_cte
),

-- Here when flag is 0 it indicates that streak is continuning and whenever its a new group/ boundary it assigns flag 1

-- Now we assign group ids
group_cte AS
(
SELECT * ,SUM(flag) OVER(PARTITION BY player_id ORDER BY match_date) as group_id FROM flag_cte
),

-- Now we got the group ids from here we need to segregate the Win streaks
win_streak_cte AS
(
SELECT player_id,group_id,COUNT(*) as win_streak FROM group_cte
GROUP BY player_id,group_id
)

SELECT player_id, MAX(win_streak) as highest_win_streak FROM win_streak_cte 
GROUP BY player_id
ORDER BY player_id
```
<img width="1034" height="213" alt="image" src="https://github.com/user-attachments/assets/46bc588b-96c6-4795-a8d9-59740f6c99b2" />

---

**Question 7 :**  Sessionizing User Events (30-Min Gap)
Question: Group clicks into unique sessions. A new session starts if a user has been inactive for more than 30 minutes. Assign a session_id to each click.

```sql
Sample Data:

CREATE TABLE web_logs (
    user_id INT,
    event_time datetime
);
INSERT INTO web_logs VALUES 
(1, '2024-04-01 12:00:00'),
(1, '2024-04-01 12:15:00'), -- Session 1
(1, '2024-04-01 13:00:00'), -- Session 2 (Gap > 30m)
(1, '2024-04-01 13:10:00'); -- Session 2
```

```sql
WITH cte AS(
SELECT *, LAG(event_time) OVER(ORDER BY event_time) as previous_event_time FROM web_logs
),

cte2 AS
(

SELECT user_id, event_time,previous_event_time , (event_time-previous_event_time) as time_difference FROM cte
),

boundary_cte3 AS(

SELECT *, CASE 
  WHEN previous_event_time IS NULL THEN 1
  WHEN time_difference >= INTERVAL '30 minutes' THEN 1 ELSE 0 END as flag FROM cte2
)

SELECT  user_id,event_time , SUM(flag) OVER(ORDER BY event_time) AS session_id FROM boundary_cte3

```
<img width="1609" height="353" alt="image" src="https://github.com/user-attachments/assets/436a2586-5c1e-4f1b-a012-a7beeab408e7" />

---

## FAANG Interview Question 

**Problem Statement:**
You’re given a table named `Club` with three columns:
Club_ID  
Member_ID  
EDU  

A member can belong to multiple clubs, and the `EDU` column represents different rewards with the following point values:
MM – 0.5  
CI – 0.5  
CO – 0.5  
CD – 1  
CL – 1  
CM – 1  

**Your task:**
Write an SQL query to calculate the total points scored by each club.

```sql
Create Table Club (
Club_Id int,
Member_Id int,
EDU varchar(30)) ;

Insert into Club Values (1001,210,Null),
 (1001,211,'MM:CI'),
 (1002,215,'CD:CI:CM'),
 (1002,216,'CL:CM'),
 (1002,217,'MM:CM'),
 (1003,255,Null),
 (1001,216,'CO:CD:CL:MM'),
 (1002,210,Null)
```

```sql
WITH club_cte AS(
SELECT club_id,member_id , 
   UNNEST (STRING_TO_ARRAY(EDU, ':'))  as club_name FROM club 

),

club_cte2 AS
(
SELECT club_id, member_id, club_name FROM  club_cte 
UNION 
SELECT club_id,member_id,EDU FROM club WHERE EDU IS NULL 
ORDER BY club_id
),

club_points AS
(
SELECT * , CASE WHEN club_name IS NULL THEN 0 
                WHEN club_name = 'MM' THEN 0.5 
                WHEN club_name = 'CI' THEN 0.5
                WHEN club_name = 'CO' THEN 0.5
                WHEN club_name = 'CD' THEN 1
                WHEN club_name = 'CL' THEN 1
                WHEN club_name = 'CM' THEN 1
                END AS points
FROM club_cte2
)

SELECT club_id, SUM(points) :: INT as total_points_scored FROM club_points
GROUP BY club_id
ORDER BY club_id
```
<img width="1023" height="278" alt="image" src="https://github.com/user-attachments/assets/947a0860-5c63-4f99-857d-804a79e10232" />

---
## SQL Server Interview Question 

**Problem Statement:** Find the video(s) that received the most user flags and determine how many of these were reviewed by YouTube.

We are given two tables:
```sql
CREATE TABLE User_Flag (
    flag_id VARCHAR(50),
    user_firstname VARCHAR(50),
    user_lastname VARCHAR(50),
    video_id VARCHAR(50)
);
 

INSERT INTO User_Flag (user_firstname, user_lastname, video_id, flag_id) VALUES
('Richard', 'Hasson', 'y6120QOlsfU', '0cazx3'),
('Mark', 'May', 'Ct6BUPvE2sM', '1cn76u'),
('Gina', 'Korman', 'dQw4w9WgXcQ', '1i43zk'),
('Mark', 'May', 'Ct6BUPvE2sM', '1n0vef'),
('Mark', 'May', 'jNQXAC9IVRw', '1sv6ib'),
('Gina', 'Korman', 'dQw4w9WgXcQ', '20xekb'),
('Mark', 'May', '5qap5aO4i9A', '4cvwuv'),
('Daniel', 'Bell', '5qap5aO4i9A', '4sd6dv'),
('Richard', 'Hasson', 'y6120QOlsfU', '6jjkvn'),
('Pauline', 'Wilks', 'jNQXAC9IVRw', '7ks264'),
('Courtney', NULL, 'dQw4w9WgXcQ', NULL),
('Helen', 'Hearn', 'dQw4w9WgXcQ', '8946nx'),
('Mark', 'Johnson', 'y6120QOlsfU', '8wwg0l'),
('Richard', 'Hasson', 'dQw4w9WgXcQ', 'arydfd'),
('Gina', 'Korman', NULL, NULL),
('Mark', 'Johnson', 'y6120QOlsfU', 'bl40qw'),
('Richard', 'Hasson', 'dQw4w9WgXcQ', 'ehn1pt'),
('Lopez', NULL, 'dQw4w9WgXcQ', 'hucyzx'),
('Greg', NULL, '5qap5aO4i9A', NULL),
('Pauline', 'Wilks', 'jNQXAC9IVRw', 'i2l3oo'),
('Richard', 'Hasson', 'jNQXAC9IVRw', 'i6336w'),
('Johnson', NULL, 'y6120QOlsfU', 'iey5vi'),
('William', 'Kwan', 'y6120QOlsfU', 'kktiwe'),
(NULL, NULL, 'Ct6BUPvE2sM', NULL),
('Loretta', 'Crutcher', 'y6120QOlsfU', 'nkjgku'),
('Pauline', 'Wilks', 'jNQXAC9IVRw', 'ov5gd8'),
('Mary', 'Thompson', 'Ct6BUPvE2sM', 'qa16ua'),
('Daniel', 'Bell', '5qap5aO4i9A', 'xciyse'),
('Evelyn', 'Johnson', 'dQw4w9WgXcQ', 'xvhk6d');



CREATE TABLE Flag_Review (
    flag_id VARCHAR(50),
    reviewed_by_yt VARCHAR(10),
    reviewed_date DATE,
    reviewed_outcome VARCHAR(50)
);


INSERT INTO Flag_Review (flag_id, reviewed_by_yt, reviewed_date, reviewed_outcome) VALUES
('0cazx3', 'FALSE', NULL, NULL),
('1cn76u', 'TRUE', '2022-03-15', 'REMOVED'),
('1i43zk', 'TRUE', '2022-03-15', 'REMOVED'),
('1n0vef', 'TRUE', '2022-03-15', 'REMOVED'),
('1sv6ib', 'TRUE', '2022-03-15', 'APPROVED'),
('20xekb', 'TRUE', '2022-03-17', 'REMOVED'),
('4cvwuv', 'TRUE', '2022-03-15', 'APPROVED'),
('4l1tk7', 'FALSE', NULL, NULL),
('4sd6dv', 'TRUE', '2022-03-14', 'REMOVED'),
('6jjkvn', 'TRUE', '2022-03-16', 'APPROVED'),
('7ks264', 'TRUE', '2022-03-15', 'APPROVED'),
('8946nx', 'FALSE', NULL, NULL),
('8wwg0l', 'FALSE', NULL, NULL),
('arydfd', 'TRUE', '2022-03-15', 'APPROVED'),
('bl40qw', 'TRUE', '2022-03-16', 'REMOVED'),
('ehn1pt', 'TRUE', '2022-03-18', 'APPROVED'),
('hucyzx', 'FALSE', NULL, NULL),
('i2l3oo', 'TRUE', '2022-03-17', 'REMOVED'),
('i6336w', 'FALSE', NULL, NULL),
('iey5vi', 'FALSE', NULL, NULL),
('kc41jd', 'TRUE', '2022-03-14', 'REMOVED'),
('kktiwe', 'TRUE', '2022-03-14', 'APPROVED'),
('nkjgku', 'FALSE', NULL, NULL),
('ov5gd8', 'TRUE', '2022-03-17', 'APPROVED'),
('qa16ua', 'FALSE', NULL, NULL),
('xciyse', 'TRUE', '2022-03-16', 'APPROVED'),
('xvhk6d', 'TRUE', '2022-03-17', 'APPROVED');

```
