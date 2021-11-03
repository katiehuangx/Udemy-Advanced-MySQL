# 📊 Udemy Advanced SQL: MySQL Data Analysis & Business Intelligence

## Table of Contents

- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Analyzing Traffic Sources](#analyzing-traffic-sources)
- [Analyzing Website Performances](#analyzing-website-performances)
- [Mid-Course Project](#mid-course-project)
- [Analysis for Channel Portfolio Management](#analysis-for-channel-portfolio-management)
- [Analyzing Business Patterns and Seasonality](#analyzing-business-patterns-and-seasonality)
- [Product Analysis](#product-analysis)
- [User Analysis](#user-analysis)
- [Final Project](#final-project)

***

## Introduction

**👩🏻‍💼 THE SITUATION** 

You’ve just been hired as an eCommerce Database Analyst for Maven Fuzzy Factory, an online retailer which has just launched their first product.

**📈 THE BRIEF**

As a member of the startup team, you will work with the CEO, the Head of Marketing, and the Website Manager to help steer the business. 

You will analyze and optimize marketing channels, measure and test website conversion performance, and use data to understand the impact of new product launches.

**✏️ THE OBJECTIVE**

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

### 📌 Q21: What is the conversion rate of successful orders?

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

### 📌 Q23: What is the conversion rate for `gsearch` `nonbrand` campaign?

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
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-04-14'
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

### 📌 Q26: After bidding down on Apr 15, 2021, what is the trend and impact on sessions for `gsearch` `nonbrand` campaign?
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

### 📌 Q28: What is the conversion rate from session to order by device type?
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

### 📌 Q30: After bidding up on desktop channel on 2012-05-19, what is the weekly session trend for both desktop and mobile?
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

Website content analysis is about understanding which pages are seen most by the users and to identify where to focus on improving your business.
- Identify the most common landing page and the first thing a user sees.
- For most viewed pages and common landing pages, understand how those pages perform for business objectives.
- Does the pages tell a story or resonate with business values?

### 📌 Q34: Identify most viewed website pages ranked by session volume
- Filter date < 2012-06-09
- Table: pageview_url | sessions (count)

```sql
SELECT
  pageview_url,
  COUNT(DISTINCT website_pageview_id) AS page_views
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY page_views DESC;
```

<img width="216" alt="image" src="https://user-images.githubusercontent.com/81607668/139636941-dfca8add-4466-454a-8d76-3c5e9139bded.png">

- Insight: Most viewed pages with highest traffic are homepage, products, and original Mr Fuzzy.
- Next steps: Is traffic for top landing pages the same?

### 📌 Q36: Finding top entry/landing pages

Entry/landing page is the page where customer lands on website for the first time.
- Filter date to < 2012-06-12
- Table: landing_page_url | landing_page (count)

```sql
WITH landing_page_cte AS (
SELECT
  website_session_id,
  MIN(website_pageview_id) AS landing_page -- Find the first page landed for each session
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id
)

SELECT 
  wp.pageview_url AS landing_page_url,
  COUNT(DISTINCT lp.website_session_id) AS count
FROM landing_page_cte lp
LEFT JOIN website_pageviews wp
  ON lp.landing_page = wp.website_pageview_id
GROUP BY wp.pageview_url
ORDER BY landing_page_url DESC;
```

<img width="153" alt="image" src="https://user-images.githubusercontent.com/81607668/139637250-f60c1773-59e3-4708-9eae-3c9cd0b795fc.png">

- Insight: Home page is the top landing page.
- Next steps: What other metrics can we use to analyse landing page performance? How do we know the metric can appropriately judge whether a page is performing well or not?
- Possible metrics: Repeat sessions? Which day and time most viewed? By source, campaign, device type?

### 📌 Q39: Analzying bounce rate for landing pages

Now that we have the sessions for the landing pages, let's find out their bounce rate.

We breakdown the steps to get a clear picture of what we're trying to find.
- Table: landing page | sessions (count) | bounced sessions (count) | bounced rate
- Step 1: Find the first `website_pageview_id` or landing page for associated session
- Step 2: Count page views for each session to identify bounces
- Step 3: Summarize total sessions and bounced sessions and calculate bounce rate

```sql
-- Step 1: Find the first `website_pageview_id` or landing page for associated session
WITH landing_page_cte AS (
SELECT 
  p.website_session_id,
  MIN(p.website_pageview_id) AS first_landing_page_id,
  p.pageview_url AS landing_page -- page view of first landing page
FROM website_pageviews p
INNER JOIN website_sessions s
  ON p.website_session_id = s.website_session_id
  AND s.created_at BETWEEN '2014-01-01' AND '2014-02-01'
GROUP BY p.website_session_id
),
-- Step 2: Count page views for each session to identify bounces
bounced_views_cte AS (
SELECT 
  lp.website_session_id,
  lp.landing_page, 
  COUNT(p.website_pageview_id) AS bounced_views
FROM landing_page_cte lp
LEFT JOIN website_pageviews p
  ON lp.website_session_id = p.website_session_id
GROUP BY 
  lp.website_session_id,
  lp.landing_page
HAVING COUNT(p.website_pageview_id) = 1
)
- Step 3: Summarize total sessions and bounced sessions and calculate bounce rate
SELECT 
  COUNT(DISTINCT lp.website_session_id) AS total_sessions, -- number of sessions by landing page
  COUNT(DISTINCT b.website_session_id) AS bounced_sessions, -- number of bounced sessions by landing page
  ROUND(100 * COUNT(DISTINCT b.website_session_id)/
    COUNT(DISTINCT lp.website_session_id),2) AS bounce_rate
FROM landing_page_cte lp -- use left join to preserve all sessions with 1 home page view
LEFT JOIN bounced_views_cte b
  ON lp.website_session_id = b.website_session_id
GROUP BY lp.landing_page;
```

<img width="315" alt="image" src="https://user-images.githubusercontent.com/81607668/139800515-7b0938c8-c6d1-4411-b31a-b8dfecb78643.png">

Insight: `/lander-3` has highest bounce rate followed by `/lander-3` and `/home`. 

Next steps: To further investigate why `/lander-3` has > 50% bounce rate. 

### 📌 Calculating bounce rate for homepage

As home page is the first page that users most commonly see on the website, ST is interested to know what's the bounce rate.

- Table: sessions | bounced sessions | bounce rate 
- Step 1: Find first website_pageview_id/landing page for associated session and filter to date < 2012-06-14 and `/home`
- Step 2: Count page views for each session to identify bounces
- Step 3: Summarize total sessions and bounced sessions

```sql
-- Step 1: Find first website_pageview_id/landing page for associated session and filter to date < 2012-06-14 and '\home'
WITH landing_page_cte AS (
SELECT 
  p.website_session_id,
  MIN(p.website_pageview_id) AS first_landing_page_id,
  p.pageview_url AS landing_page -- page view of first landing page
FROM website_pageviews p
INNER JOIN website_sessions s
  ON p.website_session_id = s.website_session_id
  AND s.created_at < '2012-06-14'
WHERE p.pageview_url = '/home' 
GROUP BY p.website_session_id
),
- Step 2: Count page views for each session to identify bounces
bounced_views_cte AS (
SELECT 
  lp.website_session_id,
  lp.landing_page, 
  COUNT(p.website_pageview_id) AS bounced_views
FROM landing_page_cte lp
LEFT JOIN website_pageviews p
  ON lp.website_session_id = p.website_session_id
GROUP BY 
  lp.website_session_id,
  lp.landing_page
HAVING COUNT(p.website_pageview_id) = 1
)
- Step 3: Summarize total sessions and bounced sessions
SELECT 
  COUNT(DISTINCT lp.website_session_id) AS total_sessions, -- number of sessions by landing page
  COUNT(DISTINCT b.website_session_id) AS bounced_sessions, -- number of bounced sessions by landing page
  ROUND(100 * COUNT(DISTINCT b.website_session_id)/
    COUNT(DISTINCT lp.website_session_id),2) AS bounce_rate
FROM landing_page_cte lp -- use left join to preserve all sessions with 1 home page view
LEFT JOIN bounced_views_cte b
  ON lp.website_session_id = b.website_session_id
GROUP BY lp.landing_page;
```

<img width="268" alt="image" src="https://user-images.githubusercontent.com/81607668/139804071-9f7b4dc3-4bed-46e0-bc59-89b90b3b9747.png">

Insight: 60% bounce rate is pretty high especially for paid search.

### 📌 Q41: Analyzing Landing Page Tests

ST is running a A/B test on `\lander-1` and `\home` for `gsearch nonbrand` campaign and would like to find out the bounce rates for both pages.
- Criteria: Limit time period to when `\lander-1` started receiving traffic and limit results to < 2012-07-28 to ensure fair comparison.
- Table: landing_page | total sessions | bounced sessions | bounce rate
- Step 1: Find when `/lander-1` was created and first displayed to user on the website. Use either date or pageview id to limit the results.
- Step 2: Find first landing page and filter to test time period (after '2012-06-19') and as prescribed by ST (before '2012-07-28') for gsearch and nonbrand campaign.
- Step 3: Count page views for each session to identify bounces
- Step 4: Summarize total sessions and bounced sessions and calculate bounce rate

```sql
-- Step 1: Find when `/lander-1` was created and first displayed to user on the website
SELECT 
  MIN(created_at) AS lander1_created_at,
  MIN(website_pageview_id) AS lander1_website_pageview_id
FROM website_pageviews
WHERE pageview_url = '/lander-1';

-- Step 2: Find first landing page and filter to test time period '2012-06-19' to '2012-07-28' for gsearch and nonbrand campaign
WITH landing_page_cte AS (
SELECT 
  p.website_session_id,
  MIN(p.website_pageview_id) AS landing_page_id,
  p.pageview_url AS landing_page -- page view of first landing page
FROM website_pageviews p
INNER JOIN website_sessions s
  ON p.website_session_id = s.website_session_id
  AND s.created_at BETWEEN '2012-06-19' AND '2012-07-28' -- A/B test time period as prescribed
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand'
  AND p.pageview_url IN ('/home','/lander-1') -- A/B test on both pages
GROUP BY p.website_session_id, p.pageview_url
),
-- Step 3: Count page views for each session to identify bounces
bounced_views_cte AS (
SELECT 
  lp.website_session_id,
  COUNT(p.website_pageview_id) AS bounced_views
FROM landing_page_cte lp
LEFT JOIN website_pageviews p
  ON lp.website_session_id = p.website_session_id -- join where session id with first landing page
GROUP BY lp.website_session_id
HAVING COUNT(p.website_pageview_id) = 1 -- Filter for page views = 1 view = bounced view
)
-- Step 4: Summarize total sessions and bounced sessions and calculate bounce rate
SELECT 
  landing_page,
  COUNT(DISTINCT lp.website_session_id) AS total_sessions, -- number of sessions by landing page
  COUNT(DISTINCT b.website_session_id) AS bounced_sessions, -- number of bounced sessions by landing page
  ROUND(100 * COUNT(DISTINCT b.website_session_id)/
    COUNT(DISTINCT lp.website_session_id),2) AS bounce_rate
FROM landing_page_cte lp -- use left join to preserve all sessions with 1 page view
LEFT JOIN bounced_views_cte b
  ON lp.website_session_id = b.website_session_id
GROUP BY lp.landing_page;
```

<img width="343" alt="image" src="https://user-images.githubusercontent.com/81607668/139808742-1f307189-6633-4765-baf2-57969405f8cd.png">

Insight: Looks like the newly created `/lander-1`'s traffic has improved and bounce rate has reduced too, meaning fewer customers has bounced on the page.  

Next steps: Ensure that all new campaigns are directed to the new lander-1 page and monitor the bounce rates.

### 📌 Q43: Landing Page Trend Analysis

ST is requesting us to pull paid gsearch nonbrand campaign traffic on `/home` and `/lander-1` pages, trended weekly since 2012-06-01 and the bounce rates.
- Criteria: Email received on 31 Aug 2021, so limit results between 2012-06-01 to 2012-08-31
- Table: landing page | week start dates | sessions (count) | bounced sessions (count) | bounce rate | 

-- Step 1: Find first website_pageview_id for first landing page id
-- Step 2: Identify landing page of each session
-- Step 3: Count page views for each session to identify bounces
-- Step 4: Summarize sessions, bounced sessions and bounce rate by week 

-- Step 1: Find first website_pageview_id/landing page and no of page views
WITH landing_pages_cte AS (
SELECT 
  s.website_session_id,
  MIN(p.website_pageview_id) AS first_pageview_id, -- first page view id
  COUNT(p.website_pageview_id) AS pageview_count -- no of total view counts per session
FROM website_sessions s 
INNER JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at BETWEEN '2012-06-01' AND '2012-08-31'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
GROUP BY s.website_session_id
),
summary_cte AS (
SELECT
  lp.website_session_id,
  lp.first_pageview_id,
  lp.pageview_count,
  p.pageview_url AS landing_page, -- add new field
  p.created_at -- add new field
FROM landing_pages_cte lp
INNER JOIN website_pageviews p
  ON lp.first_pageview_id = p.website_pageview_id
)
SELECT
  YEARWEEK(created_at) AS year_week,
  MIN(DATE(created_at)) AS week_start, -- 1st day of associated week
  COUNT(DISTINCT website_session_id) AS total_sessions,
  COUNT(DISTINCT CASE WHEN pageview_count = 1 THEN website_session_id END) AS bounced_sessions,
  ROUND(100 * COUNT(DISTINCT CASE WHEN pageview_count = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS bounce_rate,
  COUNT(DISTINCT CASE WHEN landing_page = '/home' THEN website_session_id ELSE NULL END) AS home_sessions,
  COUNT(DISTINCT CASE WHEN landing_page = '/lander-1' THEN website_session_id ELSE NULL END) AS lander_sessions
FROM summary_cte s
GROUP BY WEEK(created_at);

Insight: Before 2012-06-17, all traffic were routed to home, then after 2012-08-05 all traffic is routed to lander-1. Bounce rate dropped from 60%+ to nearing 50% so there is improvement. Changes made to `/lander-1` page is working well.

### Analyzing and Testing Conversion Funnels

Conversion funnel analysis is about understanding and optimizing each step of user's experience on their journey towards purchasing our products.

Homepage > Product Page > Add to Cart > Sale

- Identify most common paths customers take before purchasing our products.
- Identify how many of the users continue to the next step in the conversion funnel and how many users abandon at each step.
- Optimize critical pain points where users are abandoning so that you can convert more users and sell more products.

### Q46: Build a conversion funnel for `gsearch nonbrand` traffic from `/lander-1` to `/thank you` page

- Criteria: Limit results from 2012-08-05 to 2012-09-05
- Table: sessions | lander1 click % | product click % | mrfuzzy click % | cart click % | shipping click % | billing click % | thank you click %
- Step 1: Select all pageviews fr relevant session
- Step 2: Identify each relevant pageview as specific funnel step
- Step 3: Create session-level conversion funnel view
- Step 4: Aggregate data to assess funnel performance

```sql
-- Step 1: Select all pageviews fr relevant session
WITH pageviews_cte AS (
SELECT
  s.website_session_id,
  p.pageview_url,
  CASE WHEN p.pageview_url = '/products' THEN 1 ELSE 0 END AS product_page,
  CASE WHEN p.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
  CASE WHEN p.pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
  CASE WHEN p.pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
  CASE WHEN p.pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
  CASE WHEN p.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at BETWEEN '2012-08-05' AND '2012-09-05'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND p.pageview_url IN ('/lander-1', '/products', '/the-original-mr-fuzzy', 
    '/cart', '/shipping', '/billing', '/thank-you-for-your-order')
),
-- Step 2: Identify each relevant pageview as specific funnel step
-- Step 3: Create session-level conversion funnel view
session_level_cte AS (
SELECT
  website_session_id, 
  MAX(product_page) AS product_madeit,
  MAX(mrfuzzy_page) AS mrfuzzy_madeit,
  MAX(cart_page) AS cart_madeit,
  MAX(shipping_page) AS shipping_madeit,
  MAX(billing_page) AS billing_madeit,
  MAX(thankyou_page) AS thankyou_madeit
FROM pageviews_cte
GROUP BY website_session_id
)
-- Step 4: Aggregate data to assess funnel performance
SELECT
  COUNT(DISTINCT website_session_id) AS sessions,
  ROUND(100 * COUNT(CASE WHEN product_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS lander_clickrate,
  ROUND(100 * COUNT(CASE WHEN mrfuzzy_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(CASE WHEN product_madeit = 1 THEN 1 ELSE NULL END),2) AS products_clickrate,
  ROUND(100 * COUNT(CASE WHEN cart_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(CASE WHEN mrfuzzy_madeit = 1 THEN 1 ELSE NULL END),2) AS mrfuzzy_clickrate,
  ROUND(100 * COUNT(CASE WHEN shipping_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(CASE WHEN cart_madeit = 1 THEN 1 ELSE NULL END),2) AS cart_clickrate,
  ROUND(100 * COUNT(CASE WHEN billing_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(CASE WHEN shipping_madeit = 1 THEN 1 ELSE NULL END),2) AS shipping_clickrate,
  ROUND(100 * COUNT(CASE WHEN thankyou_madeit = 1 THEN 1 ELSE NULL END)/
    COUNT(CASE WHEN billing_madeit = 1 THEN 1 ELSE NULL END),2) AS billing_clickrate
FROM session_level_cte;
```

<img width="649" alt="image" src="https://user-images.githubusercontent.com/81607668/140033057-cb36e058-1f0a-4cf6-921b-b2eb4ad46d1c.png">

Insights: 
- To focus on /lander-1, mrfuzzy and billing pages with lowest clickthrough rate - why are users dropping off on these pages?
- More information on billing page will make customers more comfortable to insert their credit card information.

### Q48: Analyze conversion funnel test for `/billing` vs. new `/billing-2` pages
- ST has developed a new `/billing-2` page and wants to test the traffic and billing to order conversion rate of both pages.
- table: billing_version | sessions | orders | billing_to_order_rate

```sql
-- Step 1: Find when /billing-2 was first active
SELECT 
  MIN(website_pageview_id)
FROM website_pageviews
WHERE pageview_url = '/billing-2';
-- /billing-2 is first active on website pageview id = 53550

-- Step 2: Select all pageviews for relevant session
WITH billing_cte AS (
SELECT 
  s.website_session_id,
  p.pageview_url
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE p.website_pageview_id >= 53550 -- first page view when /billing-2 is active
  AND s.created_at < '2012-11-10'
  AND p.pageview_url IN ('/billing', '/billing-2')
)
-- Step 3: Aggregate and summarise the conversion rate
SELECT
  b.pageview_url,
  COUNT(DISTINCT b.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/COUNT(DISTINCT b.website_session_id),2) AS session_to_orders_rate
FROM billing_cte b
LEFT JOIN orders o
  ON b.website_session_id = o.website_session_id
GROUP BY b.pageview_url;
```

<img width="317" alt="image" src="https://user-images.githubusercontent.com/81607668/140033607-c8f07fcf-a56a-460f-ad8d-4e8a0da2bccb.png">

Insights: `/billing-2` page has session to order converstion rate at 62%; much better than billing page at 46%. To request engineering team to roll out new  `/billing-2` page to customers immediately.

Next step: Monitor overall sales performance.

***

## Mid-Course Project

**👩🏻‍💼 THE SITUATION **

Maven Fuzzy Factory has been live for ~8 months, and your CEO is due to present company performance metrics to the board next week. You’ll be the one tasked with preparing relevant metrics to show the company’s promising growth.

**✏️ THE OBJECTIVE**

Use SQL to:
- Extract and analyze website traffic and performance data from the Maven Fuzzy Factory database to quantify the company’s growth, and to tell the story of how you have been able to generate that growth.
- As an Analyst, the first part of your job is extracting and analyzing the data, and the next part of your job is effectively communicating the story to your stakeholders.

### Project Questions

### 📌 Gsearch seems to be the biggest driver of our business. Could you pull monthly trends for gsearch sessions and orders so that we can showcase the growth there?

### 📌 Next, it would be great to see a similar monthly trend for Gsearch, but this time splitting out nonbrand and brand campaigns separately. I am wondering if brand is picking up at all. If so, this is a good story to tell.

### 📌 While we’re on Gsearch, could you dive into nonbrand, and pull monthly sessions and orders split by device type? I want to flex our analytical muscles a little and show the board we really know our traffic sources.

### 📌 I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?

### 📌 I’d like to tell the story of our website performance improvements over the course of the first 8 months. Could you pull session to order conversion rates, by month?

### 📌 For the gsearch lander test, please estimate the revenue that test earned us (Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)

### 📌 For the landing page test you analyzed previously, it would be great to show a full conversion funnel from each of the two pages to orders. You can use the same time period you analyzed last time (Jun 19 – Jul 28).

### 📌 I’d love for you to quantify the impact of our billing test, as well. Please analyze the lift generated from the test (Sep 10 – Nov 10), in terms of revenue per billing page session, and then pull the number of billing page sessions for the past month to understand monthly impact.

***

