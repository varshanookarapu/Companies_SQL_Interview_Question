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
