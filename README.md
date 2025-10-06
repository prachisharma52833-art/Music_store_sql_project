# Music Store data analysis using SQL
![Music_Logo](https://github.com/prachisharma52833-art/Music_store_sql_project/blob/main/Music_logo.png)  
![Logo3_png]()

## Overview

This project is based on Music Store dataset which has 11 different csv files such as Employee,invoice,customer,track,invoive_line,sales_per_country.I made this Project on PostgreSQL by importing Music_Store_Database.Using SQL,I performed data analysis by joining multiple tables,using subqueries,group by,order by,limit,where clause,aggregate functions,and filtering to find useful business insights such as Senior most employee,best customers,top 10 rock bands,Top Genres.

## Objective

The main objective of this Project is to analyze the music Store database and extract meaningful information using SQL.It involves:

* Understanding relationships between tables.
* Writing Complex queries using JOINs and sybqueries.
* Performing aggregation and filtering to derive insights.

 ## Business problem and Solutions

 
## Q1:Who is the senior most employee based on job title?

```sql
select *from employee
ORDER BY levels desc
limit 1
```

## Q2:Which countries have the most invoices?

```sql
select COUNT(*)as c,billing_country
from invoice
group by billing_country
order by c desc
```

## Q3:What are top 3 values of total invoice?

```sql
select total from invoice
order by total desc
limit 3
```

## Q4:Which city has the best customers?We would like to throw a promotional Music Festival in the city we made the most money.Write a query that returns one city that has the highest sum of invoice totals.Return both the city name & sum of all invoice totals?

```sql
select SUM(total) as invoice_total,billing_city
from invoice
group by billing_city
order by invoice_total desc
```

## Q5:Who is the best customer?The customer who has spent the most money will be declared the best customer.Write a query that returns the person who has spent the most money?

```sql
select customer.customer_id,customer.first_name,customer.last_name,SUM(invoice.total) as total
from customer
JOIN invoice ON customer.customer_id=invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total DESC
limit 1
```

## Q6:Write query to return the email,first name,last name, & Genre of all Rock Music listeners.Return your list ordered alphabetically by email starting with A?

```sql
SELECT DISTINCT email,first_name,last_name
FROM customer
JOIN invoice ON customer.customer_id=invoice.customer_id
JOIN invoice_line ON invoice.invoice_id=invoice_line.invoice_id
WHERE track_id IN(
        SELECT track_id FROM track
		JOIN genre ON track.genre_id=genre.genre_id
		WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```

## Q7:Let's invite the artists who have written the most rock music in our dataset.Write a query returns the Artist name and total track count of the top 10 rock bands?

```sql
SELECT artist.artist_id,artist.name,COUNT(artist.artist_id)AS number_of_songs
FROM track
JOIN album ON album.album_id=track.album_id
JOIN artist ON artist.artist_id=album.artist_id
JOIN genre ON genre.genre_id=track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

## Q8:Return all the track names that have a song length longer than the average song length.Return the Name and milliseconds for each track.Order by the song length with the longest songs listed first?

```sql
SELECT name,milliseconds
FROM track
WHERE milliseconds>(
      SELECT AVG(milliseconds) AS avr_track_length
	  FROM track)
ORDER BY milliseconds DESC;
```

## Q9:Find how much amount spent by each customer on artists? Write a query to return customer name,artist name and total spent?

```sql
WITH best_selling_artist AS(
     SELECT artist.artist_id AS artist_id,artist.name AS artist_name,
	 SUM(invoice_line.unit_price*invoice_line.quantity)AS total_sales
	 FROM invoice_line
	 JOIN track ON track.track_id=invoice_line.track_id
	 JOIN album ON album.album_id=track.album_id
	 JOIN artist ON artist.artist_id=album.artist_id
	 GROUP BY 1
	 ORDER BY 3 DESC
	 LIMIT 1
)
SELECT c.customer_id,c.first_name,c.last_name,bsa.artist_name,SUM(il.unit_price*il.quantity)AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id=i.customer_id
JOIN invoice_line il ON il.invoice_id=i.invoice_id
JOIN track t ON t.track_id=il.track_id
JOIN album alb ON alb.album_id=t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id=alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;
```

## Q10:We want to find out the most popular music Genre for each country.We determine the mostpopular genre as the genre with the highest amount of purchases.Write a query that returns each country along with the top Genre.For countries where the maximum number of purchases is shared return all Genres?

```sql
WITH RECURSIVE
       sales_per_country AS(
          SELECT COUNT(*)AS purchases_per_genre,customer.country,genre.name,genre.genre_id
		  FROM invoice_line
		  JOIN invoice ON invoice.invoice_id=invoice_line.invoice_id
		  JOIN customer ON customer.customer_id=invoice.customer_id
		  JOIN track ON track.track_id=invoice_line.track_id
		  JOIN genre ON genre.genre_id=track.genre_id
		  GROUP BY 2,3,4
		  ORDER BY 2
	   ),
	   max_genre_per_country AS (SELECT MAX(purchases_per_genre)AS max_genre_number,country
	      FROM sales_per_country
		  GROUP BY 2
		  ORDER BY 2)
SELECT sales_per_country.*
FROM sales_per_country
JOIN max_genre_per_country ON sales_per_country.country=max_genre_per_country.country
WHERE sales_per_country.purchases_per_genre=max_genre_per_country.max_genre_number
```


## Q11:Write a query that determines the customer that has spent the most on music for each country.Write a query that returns the country along with the top customer and how much theyspent is shared,provide all customers who spent this amount?

```sql
WITH RECURSIVE
    customer_with_country AS(
       SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total)AS total_spending
	   FROM invoice
	   JOIN customer ON customer.customer_id=invoice.customer_id
	   GROUP BY 1,2,3,4
	   ORDER BY 1,5 desc),
	 country_max_spending AS(
        SELECT billing_country,MAX(total_spending)AS max_spending
		FROM customer_with_country
		GROUP BY billing_country)
		
SELECT cc.billing_country,cc.total_spending,cc.first_name,cc.last_name
FROM customer_with_country cc
JOIN country_max_spending ms
ON cc.billing_country=ms.billing_country
WHERE cc.total_spending=ms.max_spending
ORDER BY 1;
```

## Conclusion

This project helped me understand how databases work and how to use SQL for real-world analysis.By querying multiple related tables,I was able to find key patterns and insights.

