# 📊 Udemy Advanced SQL: MySQL Data Analysis & Business Intelligence

## Introduction

<details> 
<summary>
Click here for Introduction! 👋🏻
	
</summary>
	
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

</details> 
	
***

## Table of Contents

- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Analyzing Traffic Sources](#analyzing-traffic-sources)
- [Analyzing Website Performances](#analyzing-website-performances)
- [Mid-Course Project](#mid-course-project)
- [Analysis for Channel Portfolio Management](#analysis-for-channel-portfolio-management)
- [Analyzing Business Patterns and Seasonality](#analyzing-business-patterns-and-seasonality)
- [Product Analysis](#product-analysis)
- [User Analysis](#user-analysis)
- [Final Project](#final-project)

## Entity Relationship Diagram
<details>
<summary>
Click here to expand tables!
</summary>
	
<kbd><img src="https://user-images.githubusercontent.com/81607668/139528817-09766746-e26b-4aa6-9465-bd5cc58a34cc.png" alt="Image" width="750" height="480"></kbd>
  
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

**Traffic Source Analysis**
- Understanding where the customers are coming from and which channels are driving the highest quality traffic. 
- Typically, traffic sources are email, social media, search engine, and direct traffic. 
- Looking at conversion rate (CVR) which is the percentage of the traffic that converts into sales or revenue activity.

**Why is it important to conduct Conversion Rate Analysis?**
- Shift budget towards search engines, campaigns, or keywords driving the highest conversion rate.
- Compare user behaviour across traffic sources to customize messaging strategy.
- Identify opportunities to eliminate paid marketing channel or scale performing traffic.

In this section, we will learn about
- Analyzing top traffic source
- Bid optimization and trend analysis

### 📌 Q1: Finding Top Traffic Sources
<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171317382-d6cb4387-3a0b-4c72-a2fc-a0425fe359ca.png">

- **Stakeholder (ST)'s request:** Breakdown of sessions by UTM source, campaign and referring domain up to 12-04-2012
- **Results:** utm_source | utm_campaign | http_referer | sessions (count)
- **Steps:** Filter results up to sessions before '2012-04-12' and group results by `utm_source`, `utm_campaign` and `http_referer`

```sql
SELECT 
  utm_source, 
  utm_campaign, 
  http_referer,
  COUNT(website_session_id) AS total_sessions
FROM website_sessions
WHERE created_at < '2012-04-12' -- Take sessions up to 2012-04-12 only
GROUP BY utm_source, utm_campaign, http_referer -- Group results
ORDER BY total_sessions DESC; -- Sort total sessions by descending showing the highest to lowest session count
```

<img width="421" alt="Screenshot 2022-05-26 at 2 07 42 PM" src="https://user-images.githubusercontent.com/81607668/170427566-cc8b5f7c-72b2-4076-91b3-2725c5c67461.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318070-1151876c-31a6-40ea-ab20-4ac26a8d3623.png">

**Insights:** Drill into gsearch nonbrand campaign traffic to explore potential optimization opportunities.

### 📌 Q2: Traffic Conversion Rates
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318096-8facb274-2d69-468f-adc5-0fc6056a8bc0.png">

- **ST request:** Calculate CVR from session to order. If CVR is 4% >=, then increase bids to drive volume, otherwise reduce bids.
- **Results:** sessions (count) | orders (count) | conversion rate
- **Steps:** Filter sessions < 2012-04-12, `utm_source` = gsearch and `utm_campaign` = nonbrand

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

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171318139-08f4d78b-a070-46f2-9b99-1c919f466ece.png">

**Insights:** Conversion rate is 2.92%, which is less than 4%, hence we're overspending on gsearch nonbrand campaign and have to reduce bids. To monitor impact of bid reduction for the campaign.

### Bid Optimization & Trend Analysis

Analysing for bid optimization is understanding the value of various segments of paid traffic to optimize marketing budget.
- Our job is to figure out the right amount of bid for various segments of traffic based on our potential revenue. 
- How do we do that?
  - Use conversion rate and revenue per click to figure out how much you should spend per click to acquire customers. 
  - Understand how website and products are performing for subsegments of traffic to optimise within channels. 
  - Analyse impact of bid changes have on ranking 

### 📌 Q3: Traffic Source Trending
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318231-7deec300-c768-4029-98b0-53acd7bb684f.png">

- **ST request:** After bidding down on Apr 15, 2021, what is the trend and impact on sessions for `gsearch` `nonbrand` campaign? Find weekly sessions before 2012-05-10. _I'm going a step forward and provide the conversion rate as well in anticipation that ST will ask for this information._
- **Result:** week_start | session (count) | orders (count) | conversion_rate
- **Steps:** Filter to < 2012-05-10, utm_source = gsearch, utm_campaign = nonbrand

```sql
SELECT
  MIN(DATE(wb.created_at)) AS week_start, -- 1st day of each week
	COUNT(DISTINCT wb.website_session_id) AS sessions, -- Unique count of sessions
	COUNT(DISTINCT o.order_id) AS orders, -- Unique count of successful orders resulting from the sessions
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS conversion_rate -- Conversion rate of orders from sessions
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-05-10'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY WEEK(wb.created_at);
```

<img width="275" alt="image" src="https://user-images.githubusercontent.com/81607668/139573298-51e1fb02-1a8c-4746-ba67-925a2405853a.png">

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318289-ab49b997-1c41-40e2-98f8-4d6ad0e6f239.png">

**Insights**: The sessions and conversion rate after 2021-04-15 has dropped - the campaign is highly sensitive to bid changes. Continue to monitor session volume. We want to make campaigns more efficient by maximising volume at the lowest possible bid.

### 📌 Q4: Traffic Source Bid Optimization
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171318335-32002c91-8990-42e1-8a93-b0196c5ceb70.png">

- **ST request:** What is the conversion rate from session to order by device type?
- **Result:** device_type | sessions (count) | orders (count) | conversion_rate

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
WHERE wb.created_at < '2012-05-11'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY wb.device_type; -- Group by device type
```

<img width="260" alt="image" src="https://user-images.githubusercontent.com/81607668/139573682-d2ab9eb0-1cd7-457a-884f-bd58ccf1c738.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171318369-5fe5b121-c2ee-4a20-b148-87d7bc6a877f.png">

**Insights:** Desktop bids were driving nearly 4% of session to successful orders rate, so we should reduce mobile bids and transfer the paid traffic spent to desktop channel instead.

### 📌 Q5: Traffic Source Segment Trending
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171318395-a4a722b8-41f6-4a9d-a376-6eb9adde159b.png">

- **ST request:** After bidding up on desktop channel on 2012-05-19, what is the weekly session trend for both desktop and mobile?
- **Result:** week_start_date | desktop_sessions | mobile_sessions
- **Steps:** Filter to between 2012-04-15 to 2012-06-19, utm_source = gsearch, utm_campaign = nonbrand

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

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318424-4b33a3e3-8b43-4a34-a875-6778da3542f4.png">

**Insights:** After biding up desktop channel on 2012-05-19, there were obvious increase in desktop volume whereas mobile volume has also dropped considerably. ST made the right decision to focus on desktop and able to optimize spend efficiently.

***

## Analyzing Website Performances

Website content analysis is about understanding which pages are seen most by the users and to identify where to focus on improving your business.
- Identify the most common landing page and the first thing a user sees.
- For most viewed pages and common landing pages, understand how those pages perform for business objectives.
- Does the pages tell a story or resonate with business values?

In this section, we will learn about
- Analyzing top website pages and entry/landing pages
- Analyzing bounce rates and landing pages test
- Building conversion funnels and testing conversion paths

### 📌 Q6: Identifying Top Website Pages
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318612-5344a53c-052a-4647-8e11-0c6d8e9cf215.png">

- **ST request:** Identify most viewed website pages ranked by session volume
- **Result:** pageview_url | sessions (count)
- **Steps:** Find page view count by page view url, filter date < 2012-06-09 and sort session count descencing

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

<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171318642-72591bb0-7067-44c3-9149-5802e80298e5.png">

- **Insights:** Most viewed pages with highest traffic are **homepage, products, and original Mr Fuzzy**. Is traffic for top landing pages the same?

### 📌 Q7: Identifying Top Entry Pages
<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171318667-f31c2781-c7fd-4cbe-821a-8a1a085cfa6c.png">

Entry/landing page is the page where customer lands on website for the first time.
- **ST request:** Pull a list of top entry pages
- **Result:** landing_page_url | landing_page_count
- **Steps:** Filter date to < 2012-06-12

```sql
WITH landing_page_cte AS (
SELECT
  website_session_id,
  MIN(website_pageview_id) AS landing_page_count -- Find the first page landed for each session
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

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318697-24420909-8d62-4433-b350-9dd56cc698e8.png">

- **Insights:** Home page is the top landing page. What other metrics can we use to analyse landing page performance? How do we know the metric can appropriately judge whether a page is performing well or not?
- **Possible metrics:** Repeat sessions? Which day and time most viewed? By source, campaign, device type?

### Landing Page Performance and Testing

### 📌 Q8: Calculating Bounce Rates
<img width="296" alt="image" src="https://user-images.githubusercontent.com/81607668/171318738-a557166e-97bb-432b-be56-6b005e85e4b5.png">

Now that we have the sessions for the landing pages, let's find out their bounce rate.

We breakdown the steps to get a clear picture of what we're trying to find.
- **Result:** landing page | sessions (count) | bounced sessions (count) | bounced rate
- **Step 1:** Find the first `website_pageview_id` or landing page for associated session
- **Step 2:** Count page views for each session to identify bounces
- **Step 3:** Summarize total sessions and bounced sessions and calculate bounce rate

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
```
<img width="326" alt="image" src="https://user-images.githubusercontent.com/81607668/171318902-6972b40c-91bc-489e-adcd-b7f1876f44d3.png">

```sql
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
```
<img width="309" alt="image" src="https://user-images.githubusercontent.com/81607668/171319068-2597bb86-eabc-4fd9-8a2d-b7ea97a2d925.png">

```sql
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

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171318771-cca1bc9a-b5ed-4847-bd56-6ed91d2a392c.png">

**Insights:** 60% bounce rate is pretty high especially for paid search.

### 📌 Q9: Analyzing Landing Page Tests
<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171319348-c663000e-69b4-4c89-8193-a48c9853675c.png">

ST is running a A/B test on `\lander-1` and `\home` for `gsearch nonbrand` campaign and would like to find out the bounce rates for both pages.
- **Criteria:** Limit time period to when `\lander-1` started receiving traffic and limit results to < 2012-07-28 to ensure fair comparison.
- **Result:** landing_page | total sessions | bounced sessions | bounce rate
- **Step 1:** Find when `/lander-1` was created and first displayed to user on the website. Use either date or pageview id to limit the results.
- **Step 2:** Find first landing page and filter to test time period (after '2012-06-19') and as prescribed by ST (before '2012-07-28') for gsearch and nonbrand campaign.
- **Step 3:** Count page views for each session to identify bounces
- **Step 4:** Summarize total sessions and bounced sessions and calculate bounce rate

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
```
<img width="301" alt="image" src="https://user-images.githubusercontent.com/81607668/171319187-d0635e13-12e8-40f2-a130-b0e18bbf7e75.png">

```sql
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
```
<img width="233" alt="image" src="https://user-images.githubusercontent.com/81607668/171319265-62a7e901-3958-49cf-abe9-ccfef22b5f73.png">

```sql
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

<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171319373-59adc93b-7be6-4141-849e-a43f04ff5cdb.png">

**Insights:** Looks like the newly created `/lander-1`'s traffic has improved and bounce rate has reduced too, meaning fewer customers has bounced on the page.  

**Next steps:** Ensure that all new campaigns are directed to the new lander-1 page and monitor the bounce rates.

### 📌 Q10: Landing Page Trend Analysis
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319399-477106ef-b077-4ed7-920c-190ff5299d81.png">

- **ST request:** Pull paid gsearch nonbrand campaign traffic on `/home` and `/lander-1` pages, trended weekly since 2012-06-01 and the bounce rates.
- **Criteria:** Email received on 31 Aug 2021, so limit results between 2012-06-01 to 2012-08-31
- **Result:** landing page | week start dates | sessions (count) | bounced sessions (count) | bounce rate | 

- **Step 1:** Find first website_pageview_id for first landing page id
- **Step 2:** Identify landing page of each session
- **Step 3:** Count page views for each session to identify bounces
- **Step 4:** Summarize sessions, bounced sessions and bounce rate by week 

```sql
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
```
<img width="297" alt="image" src="https://user-images.githubusercontent.com/81607668/171319449-a8a5da76-9885-4af9-ab9d-a5e556b0a1bd.png">

```sql
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
```
<img width="525" alt="image" src="https://user-images.githubusercontent.com/81607668/171319543-6cbc7cb1-0d75-4b59-96fa-b0d208245706.png">

```sql
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
```

<img width="558" alt="image" src="https://user-images.githubusercontent.com/81607668/170437950-75753ca5-4d34-42de-9bd5-491cd58588fd.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319595-c6cf09eb-0eea-41e0-887b-0c9fb055bf9e.png">

**Insights:** Before 2012-06-17, all traffic were routed to home, then after 2012-08-05 all traffic is routed to lander-1. Bounce rate dropped from 60%+ to nearing 50% so there is improvement. Changes made to `/lander-1` page is working well.

### Analyzing and Testing Conversion Funnels

Conversion funnel analysis is about understanding and optimizing each step of user's experience on their journey towards purchasing our products.

**Homepage > Product Page > Add to Cart > Sale**

- Identify most common paths customers take before purchasing our products.
- Identify how many of the users continue to the next step in the conversion funnel and how many users abandon at each step.
- Optimize critical pain points where users are abandoning so that you can convert more users and sell more products.

### 📌 Q11: Build Conversion Funnels for `gsearch nonbrand` traffic from `/lander-1` to `/thank you` page
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319628-a14f9841-56f0-496a-b1d9-389a077c3988.png">

- **Criteria:** Limit results from 2012-08-05 to 2012-09-05
- **Result:** sessions | lander1 click % | product click % | mrfuzzy click % | cart click % | shipping click % | billing click % | thank you click %
- **Step 1:** Select all pageviews fr relevant session
- **Step 2:** Identify each relevant pageview as specific funnel step
- **Step 3:** Create session-level conversion funnel view
- **Step 4:** Aggregate data to assess funnel performance

```sql
-- Step 1: Select all pageviews fr relevant session
WITH pageview_cte AS (
SELECT 
  s.website_session_id,
  p.pageview_url,
  CASE WHEN p.pageview_url = '/products' THEN 1 ELSE 0 END AS to_products,
  CASE WHEN p.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS to_mrfuzzy,
  CASE WHEN p.pageview_url = '/cart' THEN 1 ELSE 0 END AS to_cart,
  CASE WHEN p.pageview_url = '/shipping' THEN 1 ELSE 0 END AS to_shipping,
  CASE WHEN p.pageview_url = '/billing' THEN 1 ELSE 0 END AS to_billing,
  CASE WHEN p.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS to_thankyou
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at BETWEEN '2012-08-05' AND '2012-09-05'
  AND s.utm_campaign = 'nonbrand'
  AND s.utm_source = 'gsearch'
),
```
<img width="647" alt="image" src="https://user-images.githubusercontent.com/81607668/171319726-9c0cfae3-d9a1-48c4-87c8-78b46d84b0e6.png">

```sql
-- Step 2: Identify each relevant pageview as specific funnel step
-- Step 3: Create session-level conversion funnel view
summary_cte AS (
SELECT
  website_session_id, 
  MAX(to_products) AS product_views,
  MAX(to_mrfuzzy) AS mrfuzzy_views,
  MAX(to_cart) AS cart_views,
  MAX(to_shipping) AS shipping_views,
  MAX(to_billing) AS billing_views,
  MAX(to_thankyou) AS thankyou_views
FROM pageview_cte
GROUP BY website_session_id
)
```
<img width="683" alt="image" src="https://user-images.githubusercontent.com/81607668/171319827-fb6c151d-d7e1-40d9-93c0-ac58292bf205.png">

```sql
-- Step 4: Aggregate data to assess funnel performance
SELECT
  COUNT(DISTINCT website_session_id) AS sessions, -- total sessions
  ROUND(100 * COUNT(CASE WHEN product_views = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS products_click_rate, --   product views / total sessions
  ROUND(100 * COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN product_views = 1 THEN website_session_id ELSE NULL END),2) AS mrfuzzy_click_rate, -- mrfuzzy views / product views
  ROUND(100 * COUNT(CASE WHEN cart_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END),2) AS cart_click_rate, -- cart views / mrfuzzy views
  ROUND(100 * COUNT(CASE WHEN shipping_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END),2) AS shipping_click_rate, -- shipping views / cart views
  ROUND(100 * COUNT(CASE WHEN billing_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN shipping_views = 1 THEN website_session_id ELSE NULL END),2) AS billing_click_rate, -- billing views / shipping views
  ROUND(100 * COUNT(CASE WHEN thankyou_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN billing_views = 1 THEN website_session_id ELSE NULL END),2) AS cart_click_rate -- thankyou views / billing views
FROM summary_cte;
```

<img width="677" alt="image" src="https://user-images.githubusercontent.com/81607668/170442720-694c553e-e962-492f-a3ba-e981929d8c4e.png">

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319870-fbeb1dc4-aefe-4d88-8181-8d107e4111ec.png">

**Insights: **
- To focus on /lander-1, mrfuzzy and billing pages with lowest clickthrough rate - why are users dropping off on these pages?
- More information on billing page will make customers more comfortable to insert their credit card information.

### 📌 Q12: Analyze Conversion Funnel Tests for `/billing` vs. new `/billing-2` pages
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319887-cf73ec4b-1b9e-401d-8e31-c533d9bb9dd8.png">

- **ST request:** ST developed a new `/billing-2` page and wants to test the traffic and billing to order conversion rate of both pages.
- **Result:** billing_version | sessions | orders | billing_to_order_rate

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

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319913-0004fe00-957f-4453-98c5-361f81d8cd30.png">

**Insights:** `/billing-2` page has session to order converstion rate at 62%; much better than billing page at 46%. To request engineering team to roll out new  `/billing-2` page to customers immediately.

**Next step:** Monitor overall sales performance.

***

## Mid-Course Project

**👩🏻‍💼 THE SITUATION**

Maven Fuzzy Factory has been live for ~8 months, and your CEO is due to present company performance metrics to the board next week. You’ll be the one tasked with preparing relevant metrics to show the company’s promising growth.

**✏️ THE OBJECTIVE**

Use SQL to:
- Extract and analyze website traffic and performance data from the Maven Fuzzy Factory database to **quantify the company’s growth**, and to tell the story of **how you have been able to generate that growth**.
- As an Analyst, the first part of your job is extracting and analyzing the data, and the next part of your job is effectively communicating the story to your stakeholders.

### Project Questions

### 📌 Q1: Gsearch seems to be the biggest driver of our business. Could you pull monthly trends for gsearch sessions and orders so that we can showcase the growth there?

- **Table:** year month | sessions | orders | session to order rate

```sql
SELECT
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/ 
    COUNT(DISTINCT s.website_session_id),2) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="271" alt="image" src="https://user-images.githubusercontent.com/81607668/140636799-322905ad-5e44-492f-a5ad-009d23253bac.png">

**Insights:** Steady growth of gsearch session to order rate from 3.27% in Mar 2012 to 4.23% to Nov 2012. 

### 📌 Q2: Next, it would be great to see a similar monthly trend for Gsearch, but this time splitting out nonbrand and brand campaigns separately. I am wondering if brand is picking up at all. If so, this is a good story to tell.

- **Table:** year_month | nonbrand_sessions | nonbrand_orders | nonbrand_conversion_r | brand_sessions | brand_orders | brand_conversion_r

```sql
SELECT 
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN s.website_session_id ELSE NULL END) AS nonbrand_sessions,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END) AS nonbrand_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN s.website_session_id ELSE NULL END),2) AS nonbrand_cvr,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN s.website_session_id ELSE NULL END) AS brand_sessions,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN o.order_id ELSE NULL END) AS brand_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN s.website_session_id ELSE NULL END),2) AS brand_cvr
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign IN ('nonbrand', 'brand')
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="603" alt="image" src="https://user-images.githubusercontent.com/81607668/140636838-1a81092c-a7ac-4eba-abbc-aa46493d0f3c.png">

**Insights:** Nonbrand session to order rate is steadily growing from 3.28% to 4.19%. Brand conversion rate is slightly inconsistent with highest 9.84% in Apr 2012 and recently averaging at 4-5% in late 2012. Could be due to different brands being promoted in different month or season. It would help to have further breakdown of brands in the campaign.

### 📌 Q3: While we’re on Gsearch, could you dive into nonbrand, and pull monthly sessions and orders split by device type? I want to flex our analytical muscles a little and show the board we really know our traffic sources.

- **Table:** year_month | mobile_sessions | mobile_orders | mobile_cvr | desktop_sessions | desktop_orders | desktop_cvr

```sql
SELECT
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN s.website_session_id ELSE NULL END) AS mobile_sessions,
  COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN o.order_id ELSE NULL END) AS mobile_orders, 
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN s.website_session_id ELSE NULL END),2) AS mobile_cvr,
  COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN s.website_session_id ELSE NULL END) AS desktop_sessions,
  COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN o.order_id ELSE NULL END) AS desktop_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN s.website_session_id ELSE NULL END),2) AS desktop_cvr
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="538" alt="image" src="https://user-images.githubusercontent.com/81607668/140636873-4db9e55e-3522-4d1e-a867-d6080c622b0b.png">

Insights: Bulk of session to orders are contributed by users on desktop devices with good growth from 4.45% in Mar 2012 to 5.04% in Nov 2021. Investigate why mobile rate is low - maybe not user-friendly or pages are not displayed properly on mobile phone browser. Consider moving bids to desktop to optimize traffic and marketing spend.

### 📌 Q4: I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?

- Table: yearmonth | sessions | gsearch_session | gsearch_rate | bsearch_session | bsearch_rate

```sql
SELECT
  DISTINCT utm_source,
  utm_campaign,
  http_referer
FROM website_sessions
WHERE created_at < '2012-11-27';
```
<img width="338" alt="image" src="https://user-images.githubusercontent.com/81607668/171320004-a26391a5-bc7a-417c-a983-c28a551a8608.png">

- If source, campaign and http referral is NULL, then it is direct traffic - users type in the website link in the browser's search bar.
- If source and campaign is NULL, but there is http referral, then it is organic search - coming from search engine and not tagged with paid parameters.

```sql 
SELECT
  EXTRACT(YEAR_MONTH FROM created_at) AS yr_mth,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_paid_session,
  COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_paid_session,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_session_id ELSE NULL END) AS organic_search_session,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_session_id ELSE NULL END) AS direct_search_session
FROM website_sessions
WHERE created_at < '2012-11-27'
GROUP BY EXTRACT(YEAR_MONTH FROM created_at);
```

<img width="652" alt="image" src="https://user-images.githubusercontent.com/81607668/140637462-2180a38d-c133-4b34-839c-1c6248f2514c.png">

Insights: Gsearch traffic started out high in Mar 2012 with 98%, then slowly dropped to 70%. Bsearch traffic picked up in Aug 2012 and grow to 22% in Nov 20122. There are 10-15% null traffic. Identify what it means by null traffic.

### 📌 Q5: I’d like to tell the story of our website performance improvements over the course of the first 8 months. Could you pull session to order conversion rates, by month?

- ST request: Calculate conversion rate of orders based on sessions
- Result: yearmonth | sessions | orders | conversion_rate

```sql
SELECT
  YEAR(s.created_at) AS yr,
  MONTH(s.created_at) AS mth,
  COUNT(DISTINCT CASE WHEN s.website_session_id IS NOT NULL THEN s.website_session_id ELSE NULL END) AS sessions,
  COUNT(DISTINCT CASE WHEN o.order_id IS NOT NULL THEN o.order_id ELSE NULL END) AS orders, 
  ROUND(100 * COUNT(DISTINCT CASE WHEN o.order_id IS NOT NULL THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.website_session_id IS NOT NULL THEN s.website_session_id ELSE NULL END),2) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
GROUP BY YEAR(s.created_at), MONTH(s.created_at);
```

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/170808375-03f0a66c-4f2f-480b-886c-590e1ecd1625.png">

Insights: Conversion rate increased from 3.23% in March 2012 to 4.45% in November. Rates start to hit 4% in September.

### 📌 Q6: For the gsearch lander test, please estimate the revenue that test earned us (Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)

- Revenue from Jun 19 – Jul 28 vs same period

```sql
-- Find the first lander-1 website_pageview_id
SELECT
  MIN(website_pageview_id)
FROM website_pageviews
WHERE pageview_url = '/lander-1';
```

The first pageview url website_pageview_id = 23504

```sql
SELECT
  p.pageview_url AS landing_page,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT s.website_session_id),2) AS conversion_rate
FROM website_sessions s
INNER JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE p.website_pageview_id >= 23504
  AND s.created_at < '2012-07-28'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND p.pageview_url IN ('/home', '/lander-1')
GROUP BY p.pageview_url;
``` 
<img width="297" alt="image" src="https://user-images.githubusercontent.com/81607668/170808139-e5c195bc-b9e6-4c7c-aa7f-ceb11a71a269.png">

Homepage's conversion rate is 3.11% and the new page lander-1's conversion rate is 4.14%. Incremental difference in website performance is 1.03% using lander-1.

```sql
SELECT
  MAX(s.website_session_id)
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND p.pageview_url = '/home';
-- the last website session with gsearch nonbrand paid campaign = 17145

SELECT
  COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE created_at < '2012-11-27'
  AND website_session_id > 17145 -- last home session
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand';
```

<img width="145" alt="image" src="https://user-images.githubusercontent.com/81607668/170808159-a39d180a-824f-47d4-bc53-f78e61386ee5.png">

**Insights:** 
- Improved total sessions using `\lander-1` = 21,729
- 21,729 x 1.03% (incremental % of order) = estimated at least 223 incremental orders since 29 Jul using `\lander-1` page
- 223/4 months = 55 additional orders per month!
- Increased performance of website and quantified the performance of the additional website sessions.


### 📌 Q7: For the landing page test you analyzed previously, it would be great to show a full conversion funnel from each of the two pages to orders. You can use the same time period you analyzed last time (Jun 19 – Jul 28).

```sql
CREATE TEMPORARY TABLE flagged_sessions_summary
SELECT
  website_session_id,
  MAX(homepage) AS saw_homepage,
  MAX(custom_lander) AS saw_custom_lander,
  MAX(products_page) AS product_made_it,
  MAX(mrfuzzy_page) AS mrfuzzy_page_made_it,  
  MAX(cart_page) AS cart_page_made_it,  
  MAX(shipping_page) AS shipping_page_made_it,
  MAX(billing_page) AS billing_page_made_it,  
  MAX(thankyou_page) AS thankyou_page_made_it 
FROM (
SELECT
  s.website_session_id,
  CASE WHEN p.pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
  CASE WHEN p.pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
  CASE WHEN p.pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
  CASE WHEN p.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
  CASE WHEN p.pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
  CASE WHEN p.pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
  CASE WHEN p.pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
  CASE WHEN p.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND s.created_at BETWEEN '2012-06-19' AND '2012-07-28'
ORDER BY s.website_session_id, p.created_at) AS flagged_sessions
GROUP BY website_session_id;

SELECT
  CASE WHEN saw_homepage = 1 THEN 'saw_homepage'
    WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
    ELSE 'check logic' END AS segment,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
  COUNT(DISTINCT CASE WHEN mrfuzzy_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
  COUNT(DISTINCT CASE WHEN cart_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
  COUNT(DISTINCT CASE WHEN shipping_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
  COUNT(DISTINCT CASE WHEN billing_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
  COUNT(DISTINCT CASE WHEN thankyou_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM flagged_sessions_summary
GROUP BY segment;

-- Categorise website sessions under `segment` by 'saw_homepage' or 'saw_custom_lander'
-- Convert aggregated website sessions to percentage of click rate by dividing by total sessions
SELECT
  CASE WHEN saw_homepage = 1 THEN 'saw_homepage'
    WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
    ELSE 'check logic' END AS segment,
  COUNT(DISTINCT website_session_id) AS sessions,
  ROUND(100*COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS product_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN mrfuzzy_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS mrfuzzy_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN cart_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS cart_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN shipping_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS shipping_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN billing_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS billing_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN thankyou_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS thankyou_click_rt
FROM flagged_sessions_summary
GROUP BY segment;
```

<img width="693" alt="image" src="https://user-images.githubusercontent.com/81607668/170445488-afb17bea-b93b-4290-aa96-29f73eb99bac.png">

**Insights:** Custom lander page has overall better click through rate as compared to the original homepage. 

### 📌 Q8: I’d love for you to quantify the impact of our billing test, as well. Please analyze the lift generated from the test (Sep 10 – Nov 10), in terms of revenue per billing page session, and then pull the number of billing page sessions for the past month to understand monthly impact.

- ST request: Calculate the sessions for `/billing` and `/billing-2` and revenue per billing page from Sep 10 – Nov 10
- Result: billing_version | sessions (count) | revenue per billing session

```sql
-- Pull out relevant data ie. website session id, page url, order id and prices associated with the billing pages
WITH billing_revenue AS (
SELECT 
  p.website_session_id, 
  p.pageview_url AS billing_version, -- Page url associated with the website session id
  o.order_id, -- 
  o.price_usd -- Billing associated with each order id. It represents the revenue
FROM website_pageviews p
LEFT JOIN orders o
  ON p.website_session_id = o.website_session_id
WHERE p.created_at BETWEEN '2012-09-10' AND '2012-11-10'
  AND p.pageview_url IN ('/billing', '/billing-2'))
```

<img width="329" alt="image" src="https://user-images.githubusercontent.com/81607668/170806763-43e92bbb-88ae-42b9-bdc4-d5423305810a.png">

```sql
SELECT 
  billing_version,
  COUNT(DISTINCT website_session_id) AS sessions, -- 
  SUM(price_usd)/COUNT(DISTINCT website_session_id) AS revenue_per_billing_page
FROM billing_revenue
GROUP BY billing_version;
```
<img width="327" alt="image" src="https://user-images.githubusercontent.com/81607668/170806793-d4491997-3519-4697-891a-002cd9331d1d.png">

```sql
SELECT 
  COUNT(website_session_id) AS sessions
FROM website_pageviews
WHERE pageview_url IN ('/billing', '/billing-2')
  AND created_at BETWEEN '2012-10-27' AND '2012-11-27';
```

<img width="85" alt="image" src="https://user-images.githubusercontent.com/81607668/170807652-e6c50539-ded2-4da3-96bd-97b3e5a36304.png">

**Insights:**
- $23.04 for old '\billing' page and $31.31 for new '\billing-2' page. Lift of $8.27 per billing page view; increased by 35%.
- Over the past month, there are 1,021 sessions and with the increase of $8.27 average revenue per session, we are looking at a positive impact of $8,443.67 increase in revenue.

***

## Analysis for Channel Portfolio Management

Analyzing a portfolio of marketing channels is about bidding efficiently and using data to maximise the effectiveness of your marketing budget.
- Understanding which marketing channels are driving the most sessions and orders through your website.
- Understanding differences in user characteristics and conversion performance across marketing channels.
- Optimising bids and allocating marketing spend across multi-channel portfolio to achieve marketing performance.

### Analyzing Channel Portfolios

### 📌 Q13: Analyzing Channel Portfolios 
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171320103-01635bd5-154f-4bda-b619-b75e79e5f5d6.png">

- **ST request:** Pull weekly sessions from 22 Aug to 29 Nov for gsearch and bsearch
- **Result:** weekly | gsearch_sessions | bsearch_sessions

```sql
SELECT
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_sessions,
  COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/ -- Added additional conversion rate
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS bsearch_rate
FROM website_sessions
WHERE utm_campaign = 'nonbrand'
  AND created_at BETWEEN '2012-08-22' AND '2012-11-29'
GROUP BY WEEK(created_at);
```

<img width="368" alt="image" src="https://user-images.githubusercontent.com/81607668/170911693-dae09a00-2c5f-4fe2-81bf-536f30eea6fa.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171320142-3ae5a7ba-c5cb-48f3-9fea-9472f69bdd3a.png">

**Insights:** Conversion rate is showing a consistent 3x impact of new bsearch campaign over original gsearch campaigns in the past 3 months. 

### 📌 Q14: Comparing Channel Characteristics

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171320158-e1cef2d3-93bc-4322-827b-1ae3ae6874cc.png">

- **ST request:** Pull mobile sessions for gsearch and bsearch non brand campaign from 22 Aug - 30 Nov
- **Result:** utm_source | sessions | mobile_sessions | mobile_perc

```sql
SELECT
  utm_source,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END)/ 
    COUNT(DISTINCT website_session_id) AS mobile_perc
FROM website_sessions
WHERE created_at BETWEEN '2012-08-22' AND '2012-11-30'
  AND utm_campaign = 'nonbrand'
  AND utm_source IN ('gsearch', 'bsearch')
GROUP BY utm_source;
```

<img width="358" alt="image" src="https://user-images.githubusercontent.com/81607668/170913941-cd559385-f654-4c61-99c3-dee3232109bd.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171320194-e45ee7fe-5555-4811-aa99-ba43fdc11b6d.png">

### 📌 Q15: Cross-Channel Bid Optimization
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171320250-76563d99-943e-414c-b2b2-e4902aeda20c.png">

- **ST request:** Pull gsearch and bsearch nonbrand conversion rates from session to orders and slice by device type from 22 Aug - 18 Sep
- **Result:** device_type | utm_source | sessions | orders | conversion_rate

```sql
SELECT 
  device_type,
  utm_source,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT s.website_session_id) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.utm_campaign = 'nonbrand'
  AND s.created_at BETWEEN '2012-08-22' AND '2012-09-18'
GROUP BY device_type, utm_source;
```

<img width="434" alt="image" src="https://user-images.githubusercontent.com/81607668/170936255-6103f002-22ca-4bdd-9215-81477cd8a824.png">

**Insights:** `gsearch` campaign has higher conversion rate in both desktop and mobile at 4.75% and 1.16% compared to 3.71% and 0.8% for `bsearch` campaign. Suggest to bid down on 'bsearch' campaigns.

### 📌 Q16: Channel Portfolio Trends
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320267-066e5b60-c81b-468e-b8fa-50d5f71c668d.png">

- **ST request:** Pull gsearch and bsearch nonbrand sessions by device type from 4 Nov - 22 Dec
- **Result:** week_start_date | device_type | utm_source | sessions | bsearch_comparison

```sql
SELECT 
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_desktop_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_desktop_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS b_perc_of_g_desktop,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_mob_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_mob_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS b_perc_of_g_mob
FROM website_sessions
WHERE created_at > '2012-11-04' AND created_at < '2012-12-22'
  AND utm_campaign = 'nonbrand'
GROUP BY WEEK(created_at);
```

<img width="767" alt="image" src="https://user-images.githubusercontent.com/81607668/170945303-9ca68616-8450-49c4-8253-f34ec3962fd6.png">

<img width="287" alt="image" src="https://user-images.githubusercontent.com/81607668/171320377-2472b08d-4f24-4cfb-8995-c5df48b3f7f4.png">

**Insights:**
- The desktop `bsearch` sessions was consistent at 4% of `gsearch` sessions, but dropped after reduced bids on 2 December. However, it could also be impacted by Black Friday, Cyber Monday and other seasonality factors. 
- As for mobile sessions, there was a sharp fall after bids were reduced, but was fluctuating throughout December, so it was hard to isolate whether it was due to reduced bids or other factors as well.

### Analying Direct Traffic

### 📌 Q17: Analyzing Free Channels
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320410-60dd7279-1b6a-44ee-83ff-da49fb1871ed.png">

- **ST request:** Pull organic search, direct type in and paid brand sessions by month. Present in % of paid nonbrand
- **Result:** yr | mo | nonbrand | brand | brand_pct_of_nonbrand | direct | direct_pct_of_nonbrand | organic | organic_pct_of_nonbrand

```sql
SELECT
  YEAR(created_at) AS yr,
  MONTH(created_at) AS mo,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS nonbrand,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id ELSE NULL END) AS brand,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS brand_pct_of_nonbrand,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL and http_referer IS NULL THEN website_session_id ELSE NULL END) AS direct,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL and http_referer IS NULL THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS direct_pct_of_nonbrand,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IN ('https://www.bsearch.com', 'https://www.gsearch.com') THEN website_session_id END) AS organic,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IN ('https://www.bsearch.com', 'https://www.gsearch.com') THEN website_session_id END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS organic_pct_of_nonbrand
FROM website_sessions
WHERE created_at < '2012-12-23'
GROUP BY YEAR(created_at), MONTH(created_at);
```

<img width="696" alt="image" src="https://user-images.githubusercontent.com/81607668/170958982-da5530ed-b4e4-4ca3-aeea-b7099f9e4853.png">

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171320439-477ac083-9bc0-496b-b69e-fdf046416d3c.png">

**Insights:**

***

## Analyzing Seasonality & Business Patterns

### 📌 Q18: Analyzing Seasonality
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320599-d3a19ced-b3ed-4e3e-b904-ba6f8d69798d.png">

- **ST request:** Pull out sessions and orders by year, monthly and weekly for 2012
- **Result:** Table 1 - yr | mo| sessions | orders | order_rate; Table 2 - week_start | sessions | orders | order_rate

```sql
SELECT 
  YEAR(s.created_at) AS yr,
  MONTH(s.created_at) AS mo,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/COUNT(DISTINCT s.website_session_id) AS order_rate -- added a column to observe trend
FROM website_sessions s
LEFT JOIN orders o 
  ON s.website_session_id = o.website_session_id
WHERE YEAR(s.created_at) = 2012 -- Limit to year 2012 only
GROUP BY YEAR(s.created_at), MONTH(s.created_at);
```

<img width="263" alt="image" src="https://user-images.githubusercontent.com/81607668/171373410-7b635c8c-42f8-42b8-be8a-43eaae69aca0.png">

```sql
SELECT 
  MIN(WEEK(s.created_at)) AS week_start,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/COUNT(DISTINCT s.website_session_id) AS order_rate -- added a column to observe trend
FROM website_sessions s
LEFT JOIN orders o 
  ON s.website_session_id = o.website_session_id
WHERE YEAR(s.created_at) = 2012
GROUP BY WEEK(s.created_at);
```

<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171373703-d6db366c-b506-423e-8540-d896cc612335.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171320624-af4fbe70-93a2-40c9-a6b7-cd548f0e0151.png">

**Insights:**

### 📌 Q19: Analyzing Business Patterns
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320711-90fc2f5c-2ef6-434d-a8af-2d41a153351e.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320724-e1741577-6c30-45ec-bc88-bd7991af729e.png">

**Insights:**

***

## Product Analysis

### 📌 Q20: Product Level Sales Analysis
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320944-87fc4400-1153-4a8d-9493-4b692ad8073e.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320976-23ec861c-37f5-4fe8-a675-94d316d5da01.png">

**Insights:**

### 📌 Q21: Product Launch Sales Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321008-b69e6b38-9cbf-42bd-aa42-01b88a635cb1.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321028-cc786a9c-91d5-4f21-b108-d81e45f4cbff.png">

**Insights:**

### Product Level Website Analysis

### 📌 Q22: Product Pathing Analysis
<img width="287" alt="image" src="https://user-images.githubusercontent.com/81607668/171321218-7c8db4f4-3384-48c5-b5a1-d2b86a1186a2.png">

- **ST request:**
- **Result:**

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171321239-e0b4aed0-d2a2-4418-b96c-a4d435ffcb0c.png">

**Insights:**

### 📌 Q23: Product Conversion Funnels
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321285-5df87937-7fd9-48fb-a103-3790de1d793a.png">

- **ST request:**
- **Result:**

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171321307-771b508e-4ac9-4c5c-afb8-2cfa240a88a1.png">

**Insights:**

### Cross-Selling Products

### 📌 Q24: Cross-Sell Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321353-c792df1b-b851-41ae-93df-1ff136874b05.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321373-9f314744-af27-4f91-9881-74cbf3d34951.png">

**Insights:**

### 📌 Q25: Portfolio Expansion Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321393-c9de555f-e2ed-4302-af15-f0dd306e3878.png">

- **ST request:**
- **Result:**

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171321438-1b4ac1c9-e81b-4cfb-b89e-7b935ad904f6.png">

**Insights:**

### Product Refund Analysis

### 📌 Q26: Product Refund Rates
<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171321511-85b37542-2653-41d8-9298-7db26e5ad7c1.png">

- **ST request:**
- **Result:**

<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321546-834cd702-d32d-45c1-b49a-9a946321f52d.png">

**Insights:**

***

## User Analysis

### Analyse Repeat Behaviour

### 📌 Q27: Identifying Repeat Visitors
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171321676-ac59d791-ef96-45ac-be26-be66d5611139.png">

- **ST request:**
- **Result:**

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171321700-27f961a7-6ad4-47a0-9f66-fe13bcd4aa79.png">

**Insights:**

### 📌 Q28: Analyzing Repeat Behaviour
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171321756-020a5bf9-04b8-4299-a61c-8163b2ff966d.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321784-c057525c-034d-459e-be13-dcbd0be45559.png">

**Insights:**

### 📌 Q29: New Vs. Repeat Channel Patterns
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321840-d2724912-2e76-45a7-bda6-9cfb1c240d59.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321872-41c1cee2-2a0a-4b2e-992f-7ca762d22c73.png">

**Insights:**

### 📌 Q30: New Vs. Repeat Performance
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171321900-c182791a-6d0f-4bd7-ae65-da6e7df8b42e.png">

- **ST request:**
- **Result:**

<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171322014-a4376283-9cbe-44ec-91f0-775b983b8090.png">

**Insights:**

***

## Final Project

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171322246-c494af57-1fd1-4009-a8fc-9dec714e6f9c.png">

**👩🏻‍💼 THE SITUATION** 

Cindy is close to securing Maven Fuzzy Factory’s next round of funding, and she needs your help to tell a compelling story to investors. You’ll need to pull the relevant data, and help your CEO craft a story about a data-driven company that has been producing rapid growth.

**📈 THE OBJECTIVE**

Extract and analyze **traffic and website performance data** to craft a **growth story** that your CEO can sell. Dive in to the **marketing channel activities and the website improvements** that have contributed to your success to date, and use the opportunity to flex your analytical skills for the investors while you’re at it.

As an Analyst, the first part of your job is extracting and analyzing the data. The next (equally important) part is **communicating the story effectively to your stakeholders.**

### 📌 Q1: First, I’d like to show our volume growth. Can you pull overall session and order volume, trended by quarter for the life of the business? Since the most recent quarter is incomplete, you can decide how to handle it.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q2: Next, let’s showcase all of our efficiency improvements. I would love to show quarterly figures since we launched, for session-to-order conversion rate, revenue per order, and revenue per session.
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q3: I’d like to show how we’ve grown specific channels. Could you pull a quarterly view of orders from Gsearch nonbrand, Bsearch nonbrand, brand search overall, organic search, and direct type-in?
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q4: Next, let’s show the overall session-to-order conversion rate trends for those same channels, by quarter. Please also make a note of any periods where we made major improvements or optimizations.
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q5: We’ve come a long way since the days of selling a single product. Let’s pull monthly trending for revenue and margin by product, along with total sales and revenue. Note anything you notice about seasonality.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q6: Let’s dive deeper into the impact of introducing new products. Please pull monthly sessions to the /products page, and show how the % of those sessions clicking through another page has changed over time, along with a view of how conversion from /products to placing an order has improved.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q7: We made our 4th product available as a primary product on December 05, 2014 (it was previously only a cross-sell item). Could you please pull sales data since then, and show how well each product cross-sells from one another?

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q8: In addition to telling investors about what we’ve already achieved, let’s show them that we still have plenty of gas in the tank. Based on all the analysis you’ve done, could you share some recommendations and opportunities for us going forward? No right or wrong answer here – I’d just like to hear your perspective!

- **ST request:**
- **Result:**

**Insights:**

***
