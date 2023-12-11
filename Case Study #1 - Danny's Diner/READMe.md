<h1 align=center>Danny's Diner</h1>

<p align = center>
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">
</p>

<p align = center><em>
   All information, datasets, and images came from <a href="https://8weeksqlchallenge.com/case-study-1/">Data With Danny</a></em>
</p>

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
1. __What is the total amount each customer spent at the restaurant?__
   ```sql
        select 
            sales.customer_id customer, 
            sum(menu.price) money_spent
        from dannys_diner.sales
        	join dannys_diner.menu on sales.product_id = menu.product_id
        group by sales.customer_id
        order by sales.customer_id;
   ```
### Steps
   - For this question, we will need the price from the `menu` table, so we will use JOIN to merge the `sales` and `menu` tables on `product_id`.
   - SUM is used to add up the price for each `customer_id` and GROUP BY will group and sum the price for each customer.
### Result
| customer | money_spent |
| -------- | ----------- |
| A        | 76          |
| B        | 74          |
| C        | 36          |

   From this query, we can see that customer A spent the most and customer C spent the least.
<br/><br/>
2. __How many days has each customer visited the restaurant?__
   ```sql
        select 
        	 customer_id customer, 
            count(distinct order_date) day_count
        from dannys_diner.sales
        group by customer_id;
   ```
   ### Steps
   - We will just need the `sales` table for this question to find out how many days the customer has visited Danny's Diner.
   - It is important to use COUNT(DISTINCT `order_date`) as the command distinct will only select unique dates.
   - For example, customer A visited twice on '2021-01-01' and we just want the days a customer visited, not the number of times a customer ordered.
  
   ### Result
| customer | day_count |
| -------- | --------- |
| A        | 4         |
| B        | 6         |
| C        | 2         |

   We can see that customer B has visited the most days at 6 while customer C visited the least at 2 days.
<br/><br/>
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
### Steps
### Result
| customer | product |
| -------- | ------- |
| A        | curry   |
| A        | sushi   |
| B        | curry   |
| C        | ramen   |

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
### Steps
### Result
| product | count |
| ------- | ----- |
| ramen   | 8     |

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
   ### Steps
   ### Result
| customer | product | purchased |
| -------- | ------- | --------- |
| A        | ramen   | 3         |
| A        | curry   | 2         |
| A        | sushi   | 1         |
| B        | ramen   | 2         |
| B        | curry   | 2         |
| B        | sushi   | 2         |
| C        | ramen   | 3         |

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
   ### Steps
   ### Result
| customer | product | purchased |
| -------- | ------- | --------- |
| A        | ramen   | 3         |
| B        | ramen   | 2         |
| B        | curry   | 2         |
| B        | sushi   | 2         |
| C        | ramen   | 3         |

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
   ### Steps
   ### Result
| customer | product |
| -------- | ------- |
| A        | ramen   |
| B        | sushi   |

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
   ### Steps
   ### Result
| customer | product |
| -------- | ------- |
| A        | sushi   |
| B        | sushi   |

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
   ### Steps
   ### Result
| customer | total_items | amount_spent |
| -------- | ----------- | ------------ |
| A        | 2           | 25           |
| B        | 3           | 40           |

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
   ### Steps
   ### Result
| customer | points |
| -------- | ------ |
| A        | 860    |
| B        | 940    |
| C        | 360    |

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
   ### Steps
   ### Result
| customer | jan_member_points |
| -------- | ----------------- |
| A        | 1370              |
| B        | 820               |

## Bonus Questions
- Join All The Things
   - Danny also wants a table that includes the `customer_id`, `order_date`, `product_name`, `price`, and whether the customer is a member. This table will help him and his team to draw general insights quickly.
   ``` sql
      select
         sales.customer_id, sales.order_date,
         menu.product_name, menu.price,
         case
            when sales.order_date >= members.join_date
               then 'Y'
            when sales.order_date < members.join_date
               then 'N'
            else 'N'
            end member
      from dannys_diner.sales
         join dannys_diner.menu on sales.product_id = menu.product_id
         left join dannys_diner.members on sales.customer_id = members.customer_id
      order by sales.customer_id, sales.order_date;
   ```
| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |
- Rank All The Things
   - Danny also wants a table that ranks the order of products for member-only purchases.
``` sql
with mem_status as (
   Select
      sales.customer_id, sales.order_date,
      menu.product_name, menu.price,
      case
         when sales.order_date >= members.join_date
            then 'Y'
         when sales.order_date < members.join_date
            then 'N'
         else 'N'
         end member
   from dannys_diner.sales
      join dannys_diner.menu on sales.product_id = menu.product_id
      left join dannys_diner.members on sales.customer_id = members.customer_id
    )
    
   select *,
      case	
         when member = 'N'
            then null
         else
            rank() over (
               partition by customer_id, member
               order by order_date, member)
         end
   from mem_status;
```
| customer_id | order_date               | product_name | price | member | rank |
| ----------- | ------------------------ | ------------ | ----- | ------ | ---- |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1    |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2    |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1    |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2    |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |      |
