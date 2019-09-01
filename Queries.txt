/*QUESTIONS FOR FINAL PROJECT

Question 1

We want to understand more about the movies that families are watching.
 The following categories are considered family movies: Animation, Children,
  Classics, Comedy, Family and Music.

Create a query that lists each movie, the film category it is classified in,
 and the number of times it has been rented out.*/

 WITH t1 AS( SELECT f.title AS film_title,
                    c.name AS category_name,
                    COUNT(*) AS rental_count
             FROM film f
             JOIN film_category fc
             ON fc.film_id = f.film_id
             JOIN category c
             ON fc.category_id = c.category_id
             JOIN inventory i
             ON i.film_id = f.film_id
             JOIN rental r
             ON r.inventory_id = i.inventory_id
             GROUP BY 1, 2
             HAVING c.name IN('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
             ORDER BY 2, 1)

 SELECT category_name, COUNT(*)
 FROM t1
 GROUP BY 1
 ORDER BY 2 DESC;

/*
Question 2:

We want to find out how the two stores compare in their count of rental orders
 during every month for all the years we have data for. Write a query that
  returns the store ID for the store, the year and month and the number of
   rental orders each store has fulfilled for that month. Your table should
    include a column for each of the following: year, month, store ID and count
     of rental orders fulfilled during that month. */


     WITH t1 AS(SELECT DATE_TRUNC('month', r.rental_date) AS new_date,
                i.store_id AS store_id,
                COUNT(*) AS count_rentals
                FROM rental r
                JOIN inventory i
                ON r.inventory_id = i.inventory_id
                GROUP BY 1, 2
                ORDER BY 3 DESC)

     SELECT DATE_PART('month', new_date) AS rental_month,
     	   DATE_PART('year', new_date) AS rental_year,
            store_id AS store_id,
            count_rentals AS count_rentals
     FROM t1;


     /*Question 3

We would like to know who were our top 10 paying customers, how many payments
they made on a monthly basis during 2007, and what was the amount of the
 monthly payments. Can you write a query to capture the customer name, month
  and year of payment, and total payment amount for each month by these top 10
   paying customers?*/

     WITH t1 AS(SELECT customer_id AS cust_id,
                       SUM(amount) total_amount
                FROM payment p
                GROUP BY 1
                ORDER BY 2 DESC
                LIMIT 10),

     t2 AS (SELECT p.customer_id c_id,
                   DATE_TRUNC('month', p.payment_date) AS new_date,
                   COUNT(*) no_of_payments,
                   SUM(p.amount) monthly_payment_amount
            FROM payment p
            JOIN t1
            ON p.customer_id = t1.cust_id
            GROUP BY 1, 2
            HAVING DATE_PART('year', DATE_TRUNC('month', p.payment_date)) = 2007)

     SELECT t2.new_date AS pay_mon,
            CONCAT(c.first_name, ' ', c.last_name) AS full_name,
            t2.no_of_payments AS pay_countpermon,
            t2.monthly_payment_amount AS pay_amount
     FROM customer c
     JOIN t2
     ON c.customer_id = t2.c_id
     ORDER BY 2, 1;


/*Question 4

Finally, for each of these top 10 paying customers, I would like to find out
 the difference across their monthly payments during 2007. Please go ahead and
  write a query to compare the payment amounts in each successive month.
   Repeat this for each of these 10 paying customers.  */

     WITH t1 AS(SELECT customer_id AS cust_id,
                           SUM(amount) total_amount
                    FROM payment p
                    GROUP BY 1
                    ORDER BY 2 DESC
                    LIMIT 10),

         t2 AS (SELECT p.customer_id c_id,
                       DATE_TRUNC('month', p.payment_date) AS new_date,
                       COUNT(*) no_of_payments,
                       SUM(p.amount) monthly_payment_amount
                FROM payment p
                JOIN t1
                ON p.customer_id = t1.cust_id
                GROUP BY 1, 2
                HAVING DATE_PART('year', DATE_TRUNC('month', p.payment_date)) = 2007),

    t3 AS (SELECT DATE_PART('month', t2.new_date) AS pay_mon_2007,
                CONCAT(c.first_name, ' ', c.last_name) AS full_name,
                t2.no_of_payments AS pay_countpermon,
                t2.monthly_payment_amount AS pay_amount
         FROM customer c
         JOIN t2
         ON c.customer_id = t2.c_id
         ORDER BY 2, 1)

    SELECT pay_mon_2007,
    	          full_name AS name,
                  pay_countpermon,
                  pay_amount,
                  LEAD(pay_amount) OVER w AS lead,
                  LEAD(pay_amount) OVER w - pay_amount AS lead_difference
            FROM t3
            WINDOW w AS (PARTITION BY full_name)
