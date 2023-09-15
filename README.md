# Project: Investigate a Relational Database

## Table of Contents 

- [Project Overview](project-overview)
- [Data Source](data-source)
- [Tools](tools)
- [Qestion 1](question-1)
- [Question 2](question-2)
- [Question 3](question-3)
- [Question 4](question-4)

## Project Overview
In this project, I queried a database called the Sakila DVD Rental database. The Sakila Database holds information about a company that rents movie DVDs. For this project, I queried the database to gain an understanding of the customer base, such as what the patterns in movie watching are across different customer groups, how they compare on payment earnings, and how the stores compare in their performance.

### Data Source
[](http://www.postgresqltutorial.com/postgresql-sample-database/)http://www.postgresqltutorial.com/postgresql-sample-database/

![ERD](/assets/img/dvd-rental-erd-2.png)

### Tools
- SQL
- Excel
- Power BI

## Questions, Data Analysis and Visualization

### Question 1

We want to understand more about the movies that families are watching. The following categories are considered family movies: Animation, Children, Classics, Comedy, Family and Music.

Create a query that lists each movie, the film category it is classified in, and the number of times it has been rented out.

### Analysis 1

```sql
SELECT t1.film_title, 
       t1.category_name, 
       COUNT(t1.film_title) AS rental_count
FROM(SELECT f.title AS film_title, 
            c.name AS category_name
     FROM category c
     JOIN film_category fc
     ON c.category_id = fc.category_id
     JOIN film f
     ON fc.film_id = f.film_id
     JOIN inventory i
     ON f.film_id = i.film_id
     JOIN rental r
     ON i.inventory_id = r.inventory_id
     WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')) t1
GROUP BY 1, 2
ORDER BY 2, 1;
```
### Visualization 1a

![](/assets/img/viz1.jpg)

### Visualization 1b

![](/assets/img/viz2.jpg)


### Question 2

We want to find out how the two stores compare in their count of rental orders during every month for all the years we have data for. Write a query that returns the store ID for the store, the year and month and the number of rental orders each store has fulfilled for that month. 

Your table should include a column for each of the following: year, month, store ID and count of rental orders fulfilled during that month.

### Analysis 2

```sql
WITH t1 AS (SELECT *, 
	           DATE_PART('year', rental_date) AS year,
                   DATE_PART('month', rental_date) AS month
            FROM rental)
SELECT t1.month AS Rental_month, 
	   t1.year AS Rental_year, 
	   sto.store_id AS store_id, 
	   COUNT(t1.rental_id) AS count_rentals
FROM t1
JOIN staff sta
ON t1.staff_id = sta.staff_id
JOIN store sto
ON sto.store_id = sta.store_id
GROUP BY 1, 2, 3
ORDER BY 4 DESC;
```

### Visualization 2

![](/assets/img/viz3.jpg)

### Question 3

Now we need to know how the length of rental duration of these family-friendly movies compares to the duration that all movies are rented for. Can you provide a table with the movie titles and divide them into 4 levels (first_quarter, second_quarter, third_quarter, and final_quarter) based on the quartiles (25%, 50%, 75%) of the average rental duration(in the number of days) for movies across all categories?

Finally, provide a table with the family-friendly film category, each of the quartiles, and the corresponding count of movies within each combination of film category for each corresponding rental duration category. The resulting table should have three columns:

- Category
- Rental length category
- Count

### Analysis 3

```sql
WITH t1 AS (SELECT f.title AS film_title, 
		   c.name AS category_name, 
		   f.rental_duration
  	    FROM category c
	    JOIN film_category fc
	    ON c.category_id = fc.category_id
	    JOIN film f
	    ON fc.film_id = f.film_id
            WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')),
			
t2 AS (SELECT t1.film_title AS film_title, 
              t1.category_name AS category_name, 
	      t1.rental_duration AS rental_duration,
	      NTILE(4) OVER (ORDER BY t1.rental_duration) AS standard_quartile 	
       FROM t1)

SELECT category_name AS name, 
       standard_quartile, 
       COUNT(film_title) AS count
FROM t2
GROUP BY 1, 2
ORDER BY 1, 2, 3 DESC;
```

### Visualization 3

![](/assets/img/viz4.jpg)

### Question 4

Finally, for each of these top 10 paying customers, I would like to find out the difference across their monthly payments during 2007. Please go ahead and write a query to compare the payment amounts in each successive month. Repeat this for each of these 10 paying customers. Also, it will be tremendously helpful if you can identify the customer name who paid the most difference in terms of payments.

### Analysis 4

```sql
WITH t1 AS (SELECT c.customer_id, 
                   CONCAT(first_name, ' ', last_name) AS full_name, 
		   SUM(p.amount)
            FROM customer c
            JOIN payment p
            ON c.customer_id = p.customer_id
            GROUP BY 1, 2
            ORDER BY 3 DESC
            LIMIT 10 ),
			
t2 AS (SELECT customer_id, 
	      DATE_TRUNC('month', payment_date) AS pay_mon, 
	      amount, 
	      payment_id
       FROM payment)
	   
SELECT t2.pay_mon, 
       t1.full_name, 
       SUM(t2.amount) AS pay_amount, 
       LAG(SUM(t2.amount)) OVER (ORDER BY t2.pay_mon) AS lag,
       LAG(SUM(t2.amount)) OVER (ORDER BY t2.pay_mon) - SUM(t2.amount) AS lag_difference
FROM t1
JOIN t2
ON t1.customer_id = t2.customer_id  
WHERE t1.full_name = 'Eleanor Hunt'      
GROUP BY 1, 2
ORDER BY 1;
```

### Visualization 4a

![](/assets/img/viz5.jpg)

### Visualization 4b

![](/assets/img/viz6.jpg)
