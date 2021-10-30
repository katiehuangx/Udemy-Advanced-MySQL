# Udemy Advanced SQL: MySQL Data Analysis & Business Intelligence

## Entity Relationship Diagram

<img width="886" alt="image" src="https://user-images.githubusercontent.com/81607668/139528817-09766746-e26b-4aa6-9465-bd5cc58a34cc.png">

`orders` - Records consist of customers' orders with order id, time when the order is created, website session id, unique user id, product id, count of products purchased, price (revenue), and cost in USD. 

<img width="659" alt="image" src="https://user-images.githubusercontent.com/81607668/139528238-dade7402-2d4e-4a66-911e-ad1bd949f2ed.png">

`order_items` - Records show various items ordered by customer with order item id, when the order is created, whether it is primary or non-primary item, product info, individual product price, and cost in USD. 

<img width="531" alt="image" src="https://user-images.githubusercontent.com/81607668/139528127-64ad1eff-abdb-4e3d-add5-47477ee0d84d.png">

`order_item_refunds` - Refund information including when creation date and time and refund amount in USD.

<img width="527" alt="image" src="https://user-images.githubusercontent.com/81607668/139528283-926c7663-bdfb-4649-ab37-8c2ca814f47d.png">

`products` - Product id, creation date of product in system, and product name.

<img width="330" alt="image" src="https://user-images.githubusercontent.com/81607668/139528155-b1c44957-d369-41af-a9d7-3da3928f0564.png">

`website_sessions` - Table is showing where the traffic is coming from and which source is helping to generate the orders. Records consist of unique website session id, UTM fields (associated with paid traffic and measure the effectiveness).

<img width="792" alt="image" src="https://user-images.githubusercontent.com/81607668/139528190-361da1c5-6512-4df7-860e-e0fdbd45f097.png">

`website_pageviews`

<img width="427" alt="image" src="https://user-images.githubusercontent.com/81607668/139528173-eb792b62-c98a-47aa-bd80-bfa5c432063b.png">

