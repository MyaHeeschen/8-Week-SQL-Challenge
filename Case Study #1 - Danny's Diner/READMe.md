<h1 align=center>Danny's Diner</h1>

<p align = center>
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">
</p>

_All information, datasets, and images came from [Data With Danny](https://8weeksqlchallenge.com/case-study-1/)_

## Business Objective
Danny's Diner is a small restaurant that sells three of Danny's favorite foods: sushi, curry, and ramen. Danny needs help to make sure he stays in business. Using the basic data from the first couple of months Danny's Diner has been in business, we will analyze the different customer trends to help deliver a better and more personalized experience.

Danny has given us three datasets to explore:
- `sales`
- `menu`
- `members`

## Entity Relationship Diagram
<p align = center>
<a href="url"><img src="https://github.com/MyaHeeschen/8-Week-SQL-Challenge/assets/135869732/2baf6e93-eea5-44d8-b5c4-c96ab5a7211a" align="center" height="350" width="500" ></a>
</p>

## Datasets
All of the datasets are in the dannys_diner database schema.

<h3><ins>Table 1: sales</ins></h3>

This table contains the `customer_id`, `order_date`, and `product_id` for all purchases so far.
|customer_id|order_date|product_id|
|---|---|---|
|A|2021-01-01|1|
|A|2021-01-01|2|
|A|2021-01-07|2|
|A|2021-01-10|3|
|A|2021-01-11|3|
|A|2021-01-11|3|
|B|2021-01-01|2|
|B|2021-01-02|2|
|B|2021-01-04|1|
|B|2021-01-11|1|
|B|2021-01-16|3|
|B|2021-02-01|3|
|C|2021-01-01|3|
|C|2021-01-01|3|
|C|2021-01-07|3|

<h3><ins>Table 2: menu</ins></h3>

This table contains the `product_id`, `product_name`, and `price` for each menu item.
|product_id|product_name|price|
|---|---|---|
|1|sushi|10|
|2|curry|15|
|3|ramen|12|

<h3><ins>Table 3: members</ins></h3>

The last table contains the `customer_id` and `join_date` for when a customer joins Danny's Diner's loyalty program.
|customer_id|join_date|
|---|---|
|A|2021-01-07|
|B|2021-01-09|

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
   ```sql
        select 
        	sales.customer_id customer, 
            sum(menu.price) money_spent
        from dannys_diner.sales
        	join dannys_diner.menu on sales.product_id = menu.product_id
        group by sales.customer_id
        order by sales.customer_id;
   ```
2. How many days has each customer visited the restaurant?
   ```sql
        select 
        	 customer_id customer, 
            count(distinct order_date) day_count
        from dannys_diner.sales
        group by customer_id;
3. What was the first item from the menu purchased by each customer?
   ```sql
       with sales_rank as (
          select 
            sales.customer_id, 
            sales.order_date, 
            menu.product_name,
            dense_rank() over (
              partition by sales.customer_id 
              order by sales.order_date) as rank
          from dannys_diner.sales
             join dannys_diner.menu on sales.product_id = menu.product_id
        )
        
        select 
          customer_id customer, 
          product_name product
        from sales_rank
        where rank = 1
        group by customer_id, product_name;
   ```
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
   ```sql
        select
        	menu.product_name product,
           count(sales.product_id) count
        from dannys_diner.sales
        	join dannys_diner.menu on sales.product_id = menu.product_id
        group by menu.product_name
        limit 1;
   ```
5. Which item was the most popular for each customer?
   ```sql
        select
        	 menu.product_name product,
            count(sales.product_id) count
        from dannys_diner.sales
        	join dannys_diner.menu on sales.product_id = menu.product_id
        group by menu.product_name
        limit 1;
   ```
   Or
   ```sql
       with pop_menu as (
           select 
          	  sales.customer_id,
          	  menu.product_name,
          	  count(sales.product_id) as purchased,
          	  dense_rank() over (
                partition by sales.customer_id
                order by count(sales.customer_id) desc) as rank
           from dannys_diner.sales
               join dannys_diner.menu on sales.product_id = menu.product_id
           group by sales.customer_id, menu.product_name
        )
        
        select 
            customer_id customer,
            product_name product,
            purchased
        from pop_menu 
        	where rank = 1;
   ```
6. Which item was purchased first by the customer after they became a member?
   ```sql
   with mem_items as(
      select
      	  sales.customer_id,
      	  sales.product_id,
      	  row_number() over 
      		  (partition by members.customer_id
               order by sales.order_date) as date_row
       from dannys_diner.sales
         join dannys_diner.members on sales.customer_id = members.customer_id
       where sales.order_date > members.join_date
     )
     
     select
     	 mem_items.customer_id customer,
        menu.product_name product
     from mem_items
       join dannys_diner.menu on mem_items.product_id = menu.product_id
     where date_row = 1
     order by customer;
   ```
7. Which item was purchased just before the customer became a member?
   ```sql
   with prior_mem_items as(
      select
      	sales.customer_id,
      	sales.product_id,
      	row_number() over 
      		(partition by members.customer_id
      order by sales.order_date desc) as date_row
      from dannys_diner.sales
         join dannys_diner.members on sales.customer_id = members.customer_id
      where sales.order_date < members.join_date
     )
     
     select 
        prior_mem_items.customer_id customer,
        menu.product_name product
     from prior_mem_items
        join dannys_diner.menu on prior_mem_items.product_id = menu.product_id
     where date_row = 1
     order by customer;
   ```
8. What are the total items and amount spent for each member before they became a member?
   ```sql
      select
         sales.customer_id customer,
         count(sales.customer_id) total_items,
         sum(menu.price) amount_spent
      from dannys_diner.sales
         join dannys_diner.menu on sales.product_id = menu.product_id
         join dannys_diner.members on sales.customer_id = members.customer_id
      where sales.order_date < members.join_date
      group by sales.customer_id
      order by sales.customer_id;
   ```
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
   ```sql
      select
         sales.customer_id customer,
         sum(case 
               when menu.product_name = 'sushi'
                  then menu.price * 20
               else menu.price * 10
               end) points
      from dannys_diner.sales
         join dannys_diner.menu on sales.product_id = menu.product_id
      group by sales.customer_id
      order by sales.customer_id;
   ```
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
   ```sql
   with offer_dates as (
      select 
         sales.customer_id customer,
         sales.order_date,
         menu.product_name,
         menu.price,
         members.join_date,
         (join_date + interval '6 day') offer_end
      from dannys_diner.sales
         join dannys_diner.menu on sales.product_id = menu.product_id
         join dannys_diner.members on sales.customer_id = members.customer_id
    )
    
      select
         customer,
         sum(case 
               when order_date between join_date and offer_end
                  then price * 20
               when product_name = 'sushi'
                  then price * 20
               else price * 10
               end) jan_member_points
      from offer_dates
      where order_date < '2021-02-01'
      group by customer;
   ```
