# 📊 Udemy Advanced SQL: MySQL Data Analysis & Business Intelligence

## Introduction

**The Situation** 

You’ve just been hired as an eCommerce Database Analyst for Maven Fuzzy Factory, an online retailer which has just launched their first product.

**The Brief**

As a member of the startup team, you will work with the CEO, the Head of Marketing, and the Website Manager to help steer the business. 

You will analyze and optimize marketing channels, measure and test website conversion performance, and use data to understand the impact of new product launches.

**The Objective**

Use SQL to:
- Access and explore the Maven Fuzzy Factory database
- Become the data expert for the company, and the go-to person for mission critical analyses
- Analyze and optimize the business’ marketing channels, website, and product portfolio

## Entity Relationship Diagram

<kbd><img width="886" alt="image" src="https://user-images.githubusercontent.com/81607668/139528817-09766746-e26b-4aa6-9465-bd5cc58a34cc.png"></kbd>

<details>
<summary>
Click here to expand tables!
</summary>
  
`orders` - Records consist of customers' orders with order id, time when the order is created, website session id, unique user id, product id, count of products purchased, price (revenue), and cost in USD. 

<kbd><img width="659" alt="image" src="https://user-images.githubusercontent.com/81607668/139528238-dade7402-2d4e-4a66-911e-ad1bd949f2ed.png"></kbd>

`order_items` - Records show various items ordered by customer with order item id, when the order is created, whether it is primary or non-primary item, product info, individual product price, and cost in USD. 

<kbd><img width="531" alt="image" src="https://user-images.githubusercontent.com/81607668/139528127-64ad1eff-abdb-4e3d-add5-47477ee0d84d.png"></kbd>

`order_item_refunds` - Refund information including when creation date and time and refund amount in USD.

<kbd><img width="527" alt="image" src="https://user-images.githubusercontent.com/81607668/139528283-926c7663-bdfb-4649-ab37-8c2ca814f47d.png"></kbd>

`products` - Product id, creation date of product in system, and product name.

<kbd><img width="330" alt="image" src="https://user-images.githubusercontent.com/81607668/139528155-b1c44957-d369-41af-a9d7-3da3928f0564.png"></kbd>

`website_sessions` - Table is showing where the traffic is coming from and which source is helping to generate the orders. Records consist of unique website session id, UTM (Urchin Tracking Module) fields. UTMs tracking parameters used by Google Analytics to track paid marketing activity. 

<kbd><img width="792" alt="image" src="https://user-images.githubusercontent.com/81607668/139528190-361da1c5-6512-4df7-860e-e0fdbd45f097.png"></kbd>

`website_pageviews`

<kbd><img width="427" alt="image" src="https://user-images.githubusercontent.com/81607668/139528173-eb792b62-c98a-47aa-bd80-bfa5c432063b.png"></kbd>

</details>
 
***

# Analyzing Traffic Sources

Traffic Source Analysis
- Understanding where the customers are coming from and which channels are driving the highest quality traffic. 
- Typically, traffic sources are email, social media, search engine, and direct traffic. 
- Looking at conversion rate (CVR) which is the percentage of the traffic that converts into sales or revenue activity.

Why is it important to conduct Conversion Rate Analysis?
- Shift budget towards search engines, campaigns, or keywords driving the highest conversion rate.
- Compare user behaviour across traffic sources to customize messaging strategy.
- Identify opportunities to eliminate paid marketing channel or scale performing traffic.

## What is the conversion rate of successful orders?

- ST request: We want to know which traffic source is generating website sessions and driving the orders.
- Get distinct website session ids and count the total of website sessions, orders and find the percentage of successful orders from active sessions.
- Table: utm_content | sessions (count) | orders (count) | conversion_rate

```sql
SELECT 
	utm_source, 
	utm_campaign, 
	http_referer, 
  COUNT(website_session_id) AS total_sessions
FROM website_sessions
WHERE created_at < '2012-04-12'
GROUP BY utm_source, utm_campaign, http_referer
ORDER BY total_sessions DESC;
```

<img width="424" alt="image" src="https://user-images.githubusercontent.com/81607668/139542889-0aaaf833-d4fe-4013-add5-e2eb65090a27.png">

Insight: Learn more about `gsearch` `nonbrand` campaign to explore potential optimization opportunities and scale on it.

## What is the conversion rate for `gsearch` `nonbrand` campaign?

We want to know from the total web sessions for `gsearch` `nonbrand` campaign, how many turned out to be successful orders and converted to sales?
- ST Request: If CVR >= 4%, then campaign is effective. Otherwise, bid down or reduce the paid marketing cost.
- Filter to sessions < 2012-04-12, utm_source = gsearch, and utm_campaign = nonbrand
- Table: sessions (count) | orders (count) | conversion rate

```sql
SELECT
  COUNT(DISTINCT wb.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS session_to_order_cvr
FROM mavenfuzzyfactory.website_sessions wb
LEFT JOIN mavenfuzzyfactory.orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-04-12'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand';
```

<img width="220" alt="image" src="https://user-images.githubusercontent.com/81607668/139543484-4c50ff9c-7302-4189-897c-9760872f2811.png">

Insights: Conversion rate is less than 4%, hence overspending on gsearch nonbrand campaign. Analysis has saved company's money.

Next steps: Monitor impact of bid reduction for the campaign.

 
