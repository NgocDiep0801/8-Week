--  A. Digital Analysis
-- 1. How many users are there?
SELECT COUNT(DISTINCT user_id) AS user_count
FROM users

-- 2. How many cookies does each user have on average?
SELECT
	ROUND(AVG(cookie_count)) as avg_cookie
from (
    SELECT user_id, 
        COUNT(cookie_id) AS cookie_count
    FROM users
    GROUP By user_id
)

-- 3. What is the unique number of visits by all users per month?
SELECT 
  EXTRACT(MONTH FROM event_time) as month, 
  COUNT(DISTINCT visit_id) AS visit_count
FROM events
GROUP BY EXTRACT(MONTH FROM event_time)

-- 4. What is the number of events for each event type? 
SELECT 
  event_type, 
  COUNT(*) AS event_count
FROM events
GROUP BY event_type
ORDER BY event_type

-- 5. What is the percentage of visits which have a purchase event?
SELECT 
  Round(100.0 * COUNT(DISTINCT visit_id) / 
        (SELECT COUNT(DISTINCT visit_id) FROM events), 2) AS percentage_purchase
FROM events 
WHERE event_type = 3

-- 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
WITH checkout_purchase AS (
    SELECT 
      visit_id,
      Sum(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
      SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
    FROM events
    GROUP BY visit_id)
SELECT
	ROUND(100.0 * (1- (SUM(purchase) / SUM(checkout))), 2) as percent_checkout_view_with_no_purchase
FROM checkout_purchase

-- 7. What are the top 3 pages by number of views?
SELECT 
  ph.page_name,
  COUNT(e.visit_id) AS page_views
FROM events e 
join page_hierarchy ph ON e.page_id = ph.page_id
WHERE e.event_type = 1
GROUP by  ph.page_name
ORder by COUNT(e.visit_id) DESC
LIMIT 3

-- 8. What is the number of views and cart adds for each product category?
SELECT 
  ph.product_category,
  sum(CASE when e.event_type = 1 then 1 else 0 end) as page_views,
  sum(CASE when e.event_type = 2 then 1 else 0 end) as cart_adds
FROM events e 
JOIN page_hierarchy AS ph ON e.page_id = ph.page_id
GROUP by ph.product_category

-- 9. What are the top 3 products by purchases?
SELECT 
  ph.product_category,
  Count(*) as purchase
FROM events e 
JOIN page_hierarchy AS ph ON e.page_id = ph.page_id
WHERE e.event_type = 3
GROUP by ph.product_category
ORDER by purchase DESC
LIMIT 3

-- B. Product Funnel Analysis
WITH purchase_events AS (
    SELECT DISTINCT visit_id
    FROM events
    WHERE event_type = 3
),
product_info AS (
    SELECT 
        ph.page_name AS products_name,
        ph.product_category,
        SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS viewed,
        SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
        SUM(CASE WHEN e.event_type = 2 AND pe.visit_id IS NOT NULL THEN 1 ELSE 0 END) AS purchases,
        SUM(CASE WHEN e.event_type = 2 AND pe.visit_id IS NULL THEN 1 ELSE 0 END) AS abandoned
    FROM events e
    JOIN page_hierarchy ph ON e.page_id = ph.page_id
    LEFT JOIN purchase_events pe ON e.visit_id = pe.visit_id
    WHERE ph.product_id IS NOT NULL
    GROUP BY ph.page_name, ph.product_category
)
-- 1. Which product had the most views, cart adds and purchases?
SELECT 
    products_name, 
    product_category, 
    viewed + cart_adds + purchases AS most_views_cart_adds_purchases
FROM product_info
ORDER BY most_views_cart_adds_purchases DESC
LIMIT 1

-- 2. Which product was most likely to be abandoned?
SELECT 
    products_name, 
    product_category, 
    abandoned
FROM product_info
ORDER BY abandoned
LIMIT 1 

-- 3. Which product had the highest view to purchase percentage? 
SELECT 
    product_name, 
    product_category, 
    ROUND(100.0 * purchases / viewed, 2) AS purchase_per_view_percentage
FROM product_info
ORDER BY purchase_per_view_percentage DESC
	
-- 4. What is the average conversion rate from view to cart add?
SELECT 
    AVG(CASE WHEN viewed = 0 THEN 0 ELSE (cart_adds / viewed) END) AS avg_conversion_rate
FROM product_info
	
-- 5. What is the average conversion rate from cart add to purchase? 
 SELECT 
    ROUND(100.0 * AVG(cart_adds / viewed), 2) AS avg_view_to_cart_add_conversion,
    ROUND(100.0 * AVG(purchases / cart_adds), 2) AS avg_cart_add_to_purchases_conversion_rate
FROM product_info
