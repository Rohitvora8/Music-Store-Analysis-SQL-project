# Music_Store_Analysis_SQL_Project

## Overview
The Music Store Data Analysis project involves analyzing a music store's dataset to derive valuable business insights. This project involves analyzing customer spending on different genre of music, globally popular music genres, integrating multiple tables, top artists across the country and analyzing employee rankings.

## Schema
![](https://github.com/Rohitvora8/Music-Store-Analysis-SQL-project/blob/main/MusicDatabaseSchema.png?raw=true)

## Question set : 1 

### Q1: Who is the senior most employee based on job title? 

select first_name, last_name, levels
from employee
order by  levels desc
limit 1


/* Q2: Which countries have the most Invoices? */

select billing_country, count(1) cnt 
from invoice
group by 1
order by 2 desc 


/* Q3: What are top 3 values of total invoice? */
select * from 
		(select *, dense_rank() over(order by total desc) rnk
		from invoice)
where rnk <= 3


/* Q4: Which city has the best customers? We would like to throw a promotional Music
Festival in the city we made the most money. 
Write a query that returns one city that has the highest sum of invoice totals. 
Return both the city name & sum of all invoice totals */

select billing_city as city, sum(total) invoice_total
from invoice
group by 1
order by 2 desc
limit 1



/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best 
       customer. 
       Write a query that returns the person who has spent the most money.*/

select c.customer_id, first_name, last_name, sum(total) total_spending
from customer c 
join invoice i on i.customer_id = c.customer_id
group by 1
order by 4 desc
-- limit 1


/* Question set : 2 */

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
Return your list ordered alphabetically by email starting with A. */

create view  diff_genre_listner
as
select email, first_name, last_name, g.name 
from genre g
join track t on g.genre_id = t.genre_id
join invoice_line il on t.track_id = il.track_id
join invoice i on i.invoice_id = il.invoice_id
join customer c on c.customer_id = i.customer_id

select  distinct * from diff_genre_listner
where name = 'Rock'
order by 1;




/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

select a.name, count(1) cnt 
from artist a 
join album ab on a.artist_id = ab.artist_id
join track t on t.album_id = ab.album_id
join genre g on g.genre_id = t.genre_id
where g.name = 'Rock'
group by 1
order by 2 desc
limit 10




/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track.
Order by the song length with the longest songs listed first. */

select name, milliseconds
from track
where milliseconds > (select avg(milliseconds) from track )
order by 2 desc





/* Question Set :3  */

/* Q1: Find how much amount spent by each customer on artists?
 Write a query to return customer name, artist name and total spent */

select distinct c.customer_id,c.first_name, c.last_name, a.name, SUM(il.unit_price*il.quantity) AS amount_spent
from customer c 
join invoice i on i.customer_id = c.customer_id
join invoice_line il on i.invoice_id = il.invoice_id
join track t on t.track_id  = il.track_id
join album ab on t.album_id = ab.album_id
join artist a on a.artist_id = ab.artist_id
group by 1,2,3,4
order by 5 desc 




/* Q2: We want to find out the most popular music Genre for each country. 
       We determine the most popular genre as the genre with the highest amount 
			 of purchases. Write a query that returns each country along with the top 
			 Genre. For countries where the maximum number of purchases is shared return all Genres. */


select country, name, cnt
from 
		(select billing_country as country, g.name, count(1) cnt 
		, row_number() over(partition by billing_country order by count(1) desc) rn 
		from genre g
		join track t on g.genre_id = t.genre_id
		join invoice_line il on t.track_id = il.track_id
		join invoice i on i.invoice_id = il.invoice_id
		group by 1,2)  x
where rn = 1 
order by 1




/* Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount. */

with cte as 
			(select c.customer_id, c.first_name, c.last_name , billing_country as country, sum(total) total_spendings
			, row_number() over(partition by billing_country order by sum(total) desc) rn
			from invoice i
			join customer c on i.customer_id = c.customer_id
			group by 1,2,3,4
			order by  4, 5 desc) 
select * from cte 
where rn =1

