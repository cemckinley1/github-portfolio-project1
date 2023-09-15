# Project: Investigate a Relational Database

## Project Overview
In this project, I queried a database called the Sakila DVD Rental database. The Sakila Database holds information about a company that rents movie DVDs. For this project, I queried the database to gain an understanding of the customer base, such as what the patterns in movie watching are across different customer groups, how they compare on payment earnings, and how the stores compare in their performance.

### Data Source
[](http://www.postgresqltutorial.com/postgresql-sample-database/)http://www.postgresqltutorial.com/postgresql-sample-database/

![ERD](/assets/img/dvd-rental-erd-2.png)

### Tools
- PostgreSQL
- Excel
- Power BI

## Questions, Data Analysis and Visualization

Question 1

We want to understand more about the movies that families are watching. The following categories are considered family movies: Animation, Children, Classics, Comedy, Family and Music.

Create a query that lists each movie, the film category it is classified in, and the number of times it has been rented out.

Analysis 1

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
![](/assets/img/Screenshot 2023-09-15 111155.jpg)

Question 2
