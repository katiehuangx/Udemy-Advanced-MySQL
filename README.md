# 📊 Udemy Advanced SQL: MySQL Data Analysis & Business Intelligence

## Table of Contents

- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Analyzing Traffic Sources](#analyzing-traffic-sources)
- [Analyzing Website Performance](#analyzing-website-performance)
- [Mid-Course Project](#mid-course-project)
- [Analysis for Channel Portfolio Management](#analysis-for-channel-portfolio-management)
- [Analyzing Business Patterns and Seasonality](#analyzing-business-patterns-and-seasonality)
- [Product Analysis](#product-analysis)
- [User Analysis](#user-analysis)
- [Final Project](#final-project)

***

## Introduction

**👩🏻‍💼 The Situation** 

You’ve just been hired as an eCommerce Database Analyst for Maven Fuzzy Factory, an online retailer which has just launched their first product.

**📈 The Brief**

As a member of the startup team, you will work with the CEO, the Head of Marketing, and the Website Manager to help steer the business. 

You will analyze and optimize marketing channels, measure and test website conversion performance, and use data to understand the impact of new product launches.

**✏️ The Objective**

Use SQL to:
- Access and explore the Maven Fuzzy Factory database
- Become the data expert for the company, and the go-to person for mission critical analyses
- Analyze and optimize the business’ marketing channels, website, and product portfolio

***

## Entity Relationship Diagram

<kbd><img src="https://user-images.githubusercontent.com/81607668/139528817-09766746-e26b-4aa6-9465-bd5cc58a34cc.png" alt="Image" width="750" height="480"></kbd>

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

## Analyzing Traffic Sources

Traffic Source Analysis
- Understanding where the customers are coming from and which channels are driving the highest quality traffic. 
- Typically, traffic sources are email, social media, search engine, and direct traffic. 
- Looking at conversion rate (CVR) which is the percentage of the traffic that converts into sales or revenue activity.

Why is it important to conduct Conversion Rate Analysis?
- Shift budget towards search engines, campaigns, or keywords driving the highest conversion rate.
- Compare user behaviour across traffic sources to customize messaging strategy.
- Identify opportunities to eliminate paid marketing channel or scale performing traffic.

### 📌 What is the conversion rate of successful orders?

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
GROUP BY 
	utm_source, 
  utm_campaign, 
  http_referer
ORDER BY total_sessions DESC;	
```

<img width="424" alt="image" src="https://user-images.githubusercontent.com/81607668/139542889-0aaaf833-d4fe-4013-add5-e2eb65090a27.png">

Insight: Learn more about `gsearch` `nonbrand` campaign to explore potential optimization opportunities and scale on it.

### 📌 What is the conversion rate for `gsearch` `nonbrand` campaign?

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

***

## Analyse Bid Optimization

Analysing for bid optimization is understanding the value of various segments of paid traffic to optimize marketing budget.
- Our job is to figure out the right amount of bid for various segments of traffic based on our potential revenue. 
- How do we do that?
  - Use conversion rate and revenue per click to figure out how much you should spend per click to acquire customers. 
  - Understand how website and products are performing for subsegments of traffic to optimise within channels. 
  - Analyse impact of bid changes have on ranking 

### 📌 After bidding down on Apr 15, 2021, what is the trend and impact on sessions for `gsearch` `nonbrand` campaign?
- ST request: Find the weekly sessions before 2012-05-10. _I'm going a step forward and providing the conversion rate as well in anticipation that ST will ask for this information._
- Filter to < 2012-05-10, utm_source = gsearch, utm_campaign = nonbrand
- Table: week_start | session (count) | orders (count) | conversion_rate

```sql
SELECT
  MIN(DATE(wb.created_at)) AS week_start,
	COUNT(DISTINCT wb.website_session_id) AS sessions,
	COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS conversion_rate
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-05-10'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY WEEK(wb.created_at);
```

<img width="275" alt="image" src="https://user-images.githubusercontent.com/81607668/139573298-51e1fb02-1a8c-4746-ba67-925a2405853a.png">

Insight: The sessions and conversion rate after 2021-04-15 has dropped - the campaign is highly sensitive to bid changes. 

Next steps: Continue to monitor session volume. Want to make campaigns more efficient by maximising volume at the lowest possible bid.

### 📌 What is the conversion rate from session to order by device type?
- Table: device_type | sessions (count) | orders (count) | conversion_rate

```sql
SELECT
  wb.device_type,
	COUNT(DISTINCT wb.website_session_id) AS sessions,
	COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS conversion_rate
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-05-10'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY wb.device_type;
```

<img width="260" alt="image" src="https://user-images.githubusercontent.com/81607668/139573682-d2ab9eb0-1cd7-457a-884f-bd58ccf1c738.png">

Insight: Desktop bids were driving nearly 4% session to successful orders rate, so we should reduce mobile bids and transfer the paid traffic to desktop channel instead.

### 📌 After bidding up on desktop channel on 2012-05-19, what is the weekly session trend for both desktop and mobile?
- Filter to between 2012-04-15 to 2012-06-19, utm_source = gsearch, utm_campaign = nonbrand
- Table: week_start_date | desktop_sessions | mobile_sessions

```sql
SELECT
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) AS desktop_sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions
FROM website_sessions
WHERE created_at BETWEEN '2012-04-15' AND '2012-06-09'
	AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand'
GROUP BY WEEK(created_at);
```

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/139574114-532ab3a1-457b-4559-9130-6df7c237b932.png">

Insight: After biding up desktop channel on 2012-05-19, there were obvious increase in desktop volume whereas mobile volume has also dropped considerably. ST made the right decision to focus on desktop and able to optimize spend efficiently.

***

## Analyzing Website Performances


