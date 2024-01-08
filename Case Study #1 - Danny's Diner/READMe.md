<h1 align=center>Danny's Diner</h1>

<p align = center>
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">
</p>

<p align = center><em>
   All information, datasets, and images come from <a href="https://8weeksqlchallenge.com/case-study-1/">Data With Danny</a></em>
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
<h3>1. <ins>What is the total amount each customer spent at the restaurant?</ins></h3>

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
<h3>2. <ins>How many days has each customer visited the restaurant?</ins></h3>

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

   We can see that customer B visited the most days at 6 while customer C visited the least at 2 days.
<br/><br/>
<h3>3. <ins>What was the first item from the menu purchased by each customer?</ins></h3>

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
- To more easily write our query, we will create a CTE (Common Table Expression) called `sales_rank` with all of the attributes we need to answer the question. DENSE_RANK is used to find the row number which is then partitioned by each `customer_id` and sorted by the `order_date` as we want to know what the customers ordered first.
- We then write a query to select the `customer_id` and `product_name` columns from the CTE using the command `WHERE rank = 1` to find the first items that were ordered.
- The query is then grouped by `customer_id` and `product_name`.

### Result
| customer | product |
| -------- | ------- |
| A        | curry   |
| A        | sushi   |
| B        | curry   |
| C        | ramen   |

   - Customer A has curry and sushi as their first items as they purchased both on the same day.
   - Customer B's first order was curry.
   - Customer C's first order was ramen.
<br/><br/>
<h3>4. <ins>What is the most purchased item on the menu and how many times was it purchased by all customers?</ins></h3>

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
- We will JOIN the `menu` with the `sales` table as we need the `product_name`.
- COUNT() will be used to determine how many times each item was ordered and GROUP BY will group the results by each menu item.
- We only want the most popular one, so we will LIMIT the query to just one result.

### Result
| product | count |
| ------- | ----- |
| ramen   | 8     |

   Ramen is the most popular item and was purchased 8 times.
<br/><br/>
<h3>5. <ins>Which item was the most popular for each customer?</ins></h3>

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
- Similar to the query in question 3, we will create a CTE table, but this time we will use DENSE_RANK() to rank the COUNT() of rows, while grouping by `customer_id` and `product_name`.
   - The `sales` table is joined with the `menu` table on `product_id`.
- The outside query will result in the customer, product, and the amount of items purchased with the rank being equal to one.
### Result
| customer | product | purchased |
| -------- | ------- | --------- |
| A        | ramen   | 3         |
| B        | ramen   | 2         |
| B        | curry   | 2         |
| B        | sushi   | 2         |
| C        | ramen   | 3         |

   - Customers A and C purchased ramen the most.
   - Customer B purchased ramen, curry, and sushi an equal amount of times.
<br/><br/>
<h3>6. <ins>Which item was purchased first by the customer after they became a member?</ins></h3>

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
- Create CTE with the needed columns and use ROW_NUMBER() to rank the rows. We will PARTITION() by `customer_id` and ORDER_BY() the `order_date`. We are only looking at the items purchased after customers became members, so we will use the WHERE clause to only get results where the `order_date` > `join_date`.
- We then select the product that was purchased first for each customer.
### Result
| customer | product |
| -------- | ------- |
| A        | ramen   |
| B        | sushi   |

   After becoming a member at Danny's Diner, customer A purchased ramen, and customer B purchased sushi.
<br/><br/>
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
- We create almost the same CTE table but where `order_date` < `join_date` and ordered by a descending `order_date` as we want the date that was most recent before they became a member.
- Then we selected the product with the row number of 1.
### Result
| customer | product |
| -------- | ------- |
| A        | sushi   |
| B        | sushi   |

   Right before becoming a member at Danny's Diner, customers A and B both purchased sushi.
<br/><br/>
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
### Steps
### Result
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
### Steps
### Result
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
