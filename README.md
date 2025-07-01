use odin_School;

SELECT * FROM odin_school.amazon;

-- Data Wrangling --

-- change the column names and datatypes of each column

alter table amazon rename column Tax_5 to Tax;

alter table amazon modify Tax decimal(10,6) not null;

-- Checking Rows

SELECT COUNT(*) as Total_Rows FROM amazon;

-- Checking Columns

SELECT count(*) as No_of_Column FROM information_schema.columns 
WHERE table_name ='amazon' and table_schema='odin_school';

-- Checking Null Values

SELECT COUNT(*) AS Null_values FROM amazon WHERE NULL;

-- Feature Engineering 

-- Step 1: Update values with correct DATE format

UPDATE amazon
SET D_ate = STR_TO_DATE(D_ate, '%d-%m-%Y');

-- Step 2: Now safely convert the column type

ALTER TABLE amazon MODIFY COLUMN D_ate DATE;

-- Addin a new column named 'month_name' that captures the month of the year

alter table amazon add column month_name varchar(50);
update amazon set month_name = monthname(D_ate);

-- Adding a new column named `dayname` that captures the day of the week on which each transaction occurred.

ALTER TABLE AMAZON ADD COLUMN Day_Name varchar(50);
UPDATE amazon SET Day_Name = DAYNAME(D_ate);

-- Adding new column that is Time_of_day that captures the time of the day (Morning , AfterNoon, Evening) 

alter table amazon add column Time_of_day varchar(50);

UPDATE amazon 

SET Time_of_day = 

CASE

WHEN hour(T_ime) BETWEEN 06 and 11 THEN "Morning"

WHEN hour(T_ime) BETWEEN 12 and 17 THEN 'Afternoon'

ELSE 'Evening'

END;

 -- disabling safe mode
 
 set SQL_SAFE_UPDATES = 0;

-- Solve the Business Problems --

-- 1. What is the count of distinct cities in the dataset?

 select city, count(distinct(city)) as count from amazon group by city;

-- 2. For each branch, what is the corresponding city?

 select distinct city,branch from amazon order by branch;
 
-- 3. What is the count of distinct product lines in the dataset?
 select count(distinct(product_line)) as distinct_product_lines from amazon;

-- 4. Which payment method occurs most frequently? (Ewallet - 345)
 select distinct(payment), count(*) as cnt from amazon group by payment order by cnt desc; 

-- 5. Which product line has the highest sales? (Food and Beverages - 56144.84)
 select distinct(product_line), sum(total) as cnt from amazon group by Product_line order by cnt desc limit 1; 

-- 6. How much revenue is generated each month? (January is highest revenue generated)
 select month_name, round(sum(total),2)as total_revenue from amazon group by month_name 
 order by total_revenue desc;

-- 7. In which month did the cost of goods sold reach its peak?(January)
select month_name,round(sum(cogs),1) as cost_of_goods from amazon group by month_name 
order by cost_of_goods desc limit 1;

-- 8. Which product line generated the highest revenue?
select Product_line, round(sum(total),2) as total_revenue from amazon group by Product_line 
order by total_revenue desc limit 1;

-- 9. In which city was the highest revenue recorded?
 select city, round(sum(total),2)as revenue from amazon group by city order by revenue desc;

-- 10. Which product line incurred the highest Value Added Tax?
 select Product_line, max(Tax)as Max_Tax from amazon group by Product_line;
 
-- 11. For each product line, add a column indicating "Good" 
--      if its sales are above average, otherwise "Bad."
select Product_line, round(sum(total),0) as Total_val, round(avg(total),0) as average_val, 
case
when sum(total) > avg(total)  then "Good"
else "Bad" 
end as Performance
from amazon group by Product_line;

-- 12. Identify the branch that exceeded the average number of products sold.
 select Branch, count(Quantity) as cnt, avg(Quantity) as Averg from amazon 
 group by Branch having count(Quantity) > avg(Quantity);

-- 13. Which product line is most frequently associated with each gender?(Male: Health & Beverages = 88
-- 																		  Female: Fashion accessories = 96 )

 select Gender, Product_line, count(Product_line) as product_line_count from amazon group by Gender,Product_line order by Product_line;
 
-- 14. Calculate the average rating for each product line.(Food and Beverages 7.11
-- 														   Fashion accessories 7.03)
 select Product_line, round(avg(rating),2) as rating_ from amazon group by Product_line order by rating_ desc;
 
-- 15. Count the sales occurrences for each time of day on every weekday.(Tuesday, Afternoon)
 select Day_name,Time_of_day,count(Quantity)as sales_cnt from amazon 
 where day_name in ('Monday','Tuesday','Wednesday','Thursday','Friday') 
 group by day_name,Time_of_day order by field(day_name, 'Monday','Tuesday','Wednesday','Thursday','Friday'),
 field(time_of_day, 'Morning','Afternoon','Evening');
 
 -- 16. Identify the customer type contributing the highest revenue.(Member type generates high revnue)
  select Customer_type, round(sum(Total),2) as revenue from amazon group by Customer_type order by revenue desc;
 
 -- 17. Determine the city with the highest VAT(Value-Add Tax) percentage.(Naypyitaw city is highest VAT)
  select city,round(sum(Quantity * Unit_price),2) as before_tax,
  round(sum(total),2) as after_tax,
  round(sum(Tax),2) as VAT_amount,
  round(((sum(tax) / sum(total)) * 100),2) as vat_percatage
  from amazon group by city order by after_tax desc;
 
-- 18. Identify the customer type with the highest VAT payments.(Member - 7810.53)
 select Customer_type, round(sum(Tax),2) as Tax from amazon group by Customer_type order by Tax desc;
 
-- 19. What is the count of distinct customer types in the dataset? (Normal , Member)
 select count(distinct(customer_type)) as cnt from amazon;
 
-- 20. What is the count of distinct payment methods in the dataset?(Cash, Credit Card, Ewallet)
 select  count(distinct(Payment)) as payment_methods from amazon;
 
-- 21. Which customer type occurs most frequently? (1.Member, 2.Normal)
 select Customer_type, count(Customer_type) as cnt from amazon group by Customer_type order by cnt desc;

-- 22. Identify the customer type with the highest purchase frequency. (Member)
select Customer_type,Sum(Quantity) as purchase_quantity, round(sum(total),2) as revenue_generated from amazon 
group by Customer_type order by purchase_quantity desc;

-- 23. Determine the predominant gender among customers.
 select Gender,
 sum(case when Gender = 'Female' then 1 end) as Female,
 sum(case when Gender = 'Male' then 1  end) as Male
 from amazon group by Gender;
 
 select gender, count(Quantity) as cnt from amazon group by gender; 
 
-- 24. Examine the distribution of genders within each branch.
 select Branch, gender,count(Gender) as Gender_base from amazon group by Branch,Gender order by Branch;

-- 25. Identify the time of day when customers provide the most ratings.(Afternoon having most ratings)
 select time_of_day,count(rating) as rating from amazon group by time_of_day order by rating desc; 
 
-- 26. Determine the time of day with the highest customer ratings for each branch.(Branch A,B,C most ratings on Afternoon)
select Branch,time_of_day,count(Rating) as cnt from amazon 
group by Branch,time_of_day order by field(Branch, 'A','B','C'), 
field(time_of_day, 'Morning','Afternoon','Evening');

SELECT Branch, Time_of_day, Most_Ratings
FROM (
    SELECT Branch, Time_of_day, COUNT(Rating) AS Most_Ratings,
           ROW_NUMBER() OVER (PARTITION BY Branch ORDER BY COUNT(Rating) DESC) AS Rating
    FROM amazon
    GROUP BY Branch, Time_of_day
) AS ranked
WHERE Rating = 1;

-- 27. Identify the day of the week with the highest average ratings. (Monday - 7.15)
 select day_name, round(avg(rating),2) as avg_rating from amazon group by day_name order by avg_rating desc;

-- 28. Determine the day of the week with the highest average ratings for each branch. (Branch C - 7.073)
 select branch, round(avg(rating),3) as avg_rating from amazon group by branch order by avg_rating desc;


 SELECT Day_Name, Branch, Avg_Rating FROM
 (SELECT Branch, Day_Name, ROUND(AVG(Rating), 3) AS Avg_Rating,
 ROW_NUMBER() OVER(PARTITION BY Branch ORDER BY AVG(Rating) DESC) AS High_Rating
 FROM amazon
 GROUP BY Branch, Day_Name 
 ORDER BY Avg_Rating DESC) AS bb
 WHERE High_Rating = 1;

 select * from amazon;
