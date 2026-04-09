# SQL for Data Science Interviews

SQL is tested at every MAANG company for Data Scientist roles and most ML Engineer roles. Expect 1–2 SQL problems per interview at Meta, Google, and Amazon DS.

---

## What They're Testing

- **Basic**: joins, aggregations, GROUP BY, HAVING, subqueries
- **Intermediate**: window functions, CTEs, self-joins, date arithmetic
- **Advanced**: complex aggregations, funnel analysis, cohort analysis, sessionization

At senior level: they expect clean, readable SQL (aliases, CTEs over nested subqueries), and you should be able to explain your approach and discuss performance.

---

## Core Concepts

### Window Functions (Most Important)

```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT 
    user_id,
    product_id,
    purchase_date,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY purchase_date) AS purchase_rank
FROM purchases;

-- LAG / LEAD: access previous/next row
SELECT 
    user_id,
    event_date,
    event_date - LAG(event_date) OVER (PARTITION BY user_id ORDER BY event_date) AS days_since_last_visit
FROM user_events;

-- Running totals
SELECT 
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_revenue
FROM daily_revenue;

-- Moving average
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_7day_avg
FROM daily_revenue;
```

### CTEs (Common Table Expressions)

Always prefer CTEs over nested subqueries for readability:

```sql
WITH 
active_users AS (
    SELECT user_id
    FROM user_events
    WHERE event_date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY user_id
    HAVING COUNT(*) >= 5
),
user_revenue AS (
    SELECT user_id, SUM(amount) AS total_revenue
    FROM purchases
    GROUP BY user_id
)
SELECT 
    a.user_id,
    COALESCE(r.total_revenue, 0) AS revenue
FROM active_users a
LEFT JOIN user_revenue r ON a.user_id = r.user_id;
```

---

## Classic Interview Patterns

### Pattern 1: User Retention / Cohort Analysis

**"Calculate 30-day retention rate by signup cohort"**

```sql
WITH cohorts AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
activity AS (
    SELECT DISTINCT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_events
),
cohort_activity AS (
    SELECT
        c.cohort_month,
        a.activity_month,
        COUNT(DISTINCT c.user_id) AS active_users,
        DATEDIFF('month', c.cohort_month, a.activity_month) AS months_since_signup
    FROM cohorts c
    LEFT JOIN activity a ON c.user_id = a.user_id
    GROUP BY 1, 2, 4
)
SELECT
    cohort_month,
    months_since_signup,
    active_users,
    FIRST_VALUE(active_users) OVER (PARTITION BY cohort_month ORDER BY months_since_signup) AS cohort_size,
    active_users * 100.0 / FIRST_VALUE(active_users) OVER (PARTITION BY cohort_month ORDER BY months_since_signup) AS retention_pct
FROM cohort_activity
ORDER BY cohort_month, months_since_signup;
```

---

### Pattern 2: Funnel Analysis

**"Calculate conversion rate at each step of the checkout funnel"**

```sql
WITH funnel AS (
    SELECT
        user_id,
        MAX(CASE WHEN event_type = 'view_product' THEN 1 ELSE 0 END) AS viewed,
        MAX(CASE WHEN event_type = 'add_to_cart' THEN 1 ELSE 0 END) AS added_to_cart,
        MAX(CASE WHEN event_type = 'start_checkout' THEN 1 ELSE 0 END) AS started_checkout,
        MAX(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS purchased
    FROM user_events
    WHERE event_date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY user_id
)
SELECT
    SUM(viewed) AS step1_view,
    SUM(added_to_cart) AS step2_cart,
    SUM(started_checkout) AS step3_checkout,
    SUM(purchased) AS step4_purchase,
    ROUND(SUM(added_to_cart) * 100.0 / NULLIF(SUM(viewed), 0), 2) AS view_to_cart_pct,
    ROUND(SUM(started_checkout) * 100.0 / NULLIF(SUM(added_to_cart), 0), 2) AS cart_to_checkout_pct,
    ROUND(SUM(purchased) * 100.0 / NULLIF(SUM(started_checkout), 0), 2) AS checkout_to_purchase_pct
FROM funnel;
```

---

### Pattern 3: Sessionization

**"Calculate number of sessions per user where a session ends after 30 minutes of inactivity"**

```sql
WITH event_gaps AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS prev_event_time,
        DATEDIFF('minute', LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time), event_time) AS minutes_since_prev
    FROM user_events
),
session_flags AS (
    SELECT
        user_id,
        event_time,
        CASE 
            WHEN prev_event_time IS NULL OR minutes_since_prev > 30 THEN 1 
            ELSE 0 
        END AS is_session_start
    FROM event_gaps
),
sessions AS (
    SELECT
        user_id,
        event_time,
        SUM(is_session_start) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id
    FROM session_flags
)
SELECT
    user_id,
    COUNT(DISTINCT session_id) AS num_sessions,
    AVG(session_length_minutes) AS avg_session_length
FROM (
    SELECT
        user_id,
        session_id,
        DATEDIFF('minute', MIN(event_time), MAX(event_time)) AS session_length_minutes
    FROM sessions
    GROUP BY user_id, session_id
) session_summary
GROUP BY user_id;
```

---

### Pattern 4: Running Metrics & Rankings

**"Find the top 3 products by revenue for each category"**

```sql
WITH product_revenue AS (
    SELECT
        p.category,
        p.product_id,
        p.product_name,
        SUM(o.amount) AS total_revenue,
        RANK() OVER (PARTITION BY p.category ORDER BY SUM(o.amount) DESC) AS revenue_rank
    FROM products p
    JOIN order_items o ON p.product_id = o.product_id
    GROUP BY p.category, p.product_id, p.product_name
)
SELECT category, product_id, product_name, total_revenue
FROM product_revenue
WHERE revenue_rank <= 3
ORDER BY category, revenue_rank;
```

---

### Pattern 5: Self-Join (User Referrals, Social Graph)

**"Find all users who were referred by a user who was themselves referred"**

```sql
-- users table: user_id, referred_by
SELECT 
    u2.user_id AS second_degree_user,
    u1.user_id AS referrer,
    root.user_id AS original_referrer
FROM users u2
JOIN users u1 ON u2.referred_by = u1.user_id
JOIN users root ON u1.referred_by = root.user_id
WHERE u1.referred_by IS NOT NULL;
```

---

### Pattern 6: Date/Time Analysis

**"Calculate DAU, WAU, MAU and the DAU/MAU ratio for the last 30 days"**

```sql
WITH daily_active AS (
    SELECT 
        DATE(event_time) AS date,
        COUNT(DISTINCT user_id) AS dau
    FROM user_events
    WHERE event_time >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY 1
),
monthly_active AS (
    SELECT COUNT(DISTINCT user_id) AS mau
    FROM user_events
    WHERE event_time >= CURRENT_DATE - INTERVAL '30 days'
)
SELECT
    d.date,
    d.dau,
    m.mau,
    ROUND(d.dau * 100.0 / m.mau, 2) AS dau_mau_ratio
FROM daily_active d
CROSS JOIN monthly_active m
ORDER BY d.date;
```

---

## Common Pitfalls

**NULL handling:**
```sql
-- NULLIF prevents division by zero
ROUND(numerator * 100.0 / NULLIF(denominator, 0), 2)

-- COALESCE fills NULLs
COALESCE(revenue, 0)
```

**Counting distinct users vs events:**
```sql
COUNT(*) -- counts rows (events)
COUNT(DISTINCT user_id) -- counts unique users
```

**HAVING vs WHERE:**
- WHERE filters before GROUP BY (on individual rows)
- HAVING filters after GROUP BY (on aggregated groups)

```sql
-- Wrong: can't use aggregate in WHERE
SELECT user_id, COUNT(*) FROM orders WHERE COUNT(*) > 5 GROUP BY user_id;

-- Right:
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id HAVING COUNT(*) > 5;
```

**LEFT JOIN vs INNER JOIN:**
- LEFT JOIN: keeps all rows from left table, NULLs for no match on right
- Use LEFT JOIN when you need to include users/items with zero events

---

## Practice Problems

1. Calculate 7-day rolling retention for each user cohort
2. Find pairs of users who both visited the site on the same day
3. Calculate the median order value per customer segment (tricky — SQL doesn't have MEDIAN in all dialects)
4. Find users who have increased their purchase frequency month-over-month for 3 consecutive months
5. Given a table of A/B experiment assignments and a conversions table, calculate statistical significance of the experiment result in SQL

Platforms:
- **StrataScratch** — real interview questions from MAANG
- **Mode Analytics SQL Tutorial** — good for window functions
- **LeetCode SQL section** — 50+ database problems
- **HackerRank SQL** — structured difficulty progression
