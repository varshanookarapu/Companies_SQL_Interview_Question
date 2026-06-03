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


