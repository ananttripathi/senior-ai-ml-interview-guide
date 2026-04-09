# SQL Interview Questions — Complete Guide

SQL is tested in two distinct formats at MAANG. Understanding the difference is critical for your prep.

---

## Two Types of SQL Rounds

### Type 1: SQL Coding Round
**Who gets it**: Data Scientist roles at Meta, Netflix, Amazon, Google. Occasionally ML Engineers at DS-heavy teams.  
**Format**: given a schema + sample data, write a query to answer a specific question. Timed (20–30 min per question). Evaluated on correctness, efficiency, and code clarity.  
**Platform**: CoderPad, StrataScratch, HackerRank, or custom internal tool.  
**Difficulty**: medium to hard. Window functions, multiple CTEs, self-joins, complex aggregations.

### Type 2: SQL in Product/Analytics Round  
**Who gets it**: Data Scientist roles everywhere. Also ML Engineers at Meta and Google during "product sense" rounds.  
**Format**: open-ended — "You want to measure X. Walk me through how you'd write the query." Focus is on your analytical thinking, not just syntax.  
**Evaluated on**: problem decomposition, metric definition, edge case awareness, and whether you can translate a business question into SQL.

This file covers both types with real questions reported at MAANG.

---

# PART 1: SQL CODING QUESTIONS

## Difficulty 1: Foundations (warm-up)

### Q1. Second Highest Salary
*LeetCode 176 — reported at Amazon, Meta*

```sql
-- Table: Employee(id, salary)
-- Find the second highest distinct salary. Return NULL if it doesn't exist.

SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);

-- Alternative with OFFSET (works in PostgreSQL/MySQL 8+):
SELECT salary AS SecondHighestSalary
FROM (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) t
UNION ALL
SELECT NULL
WHERE NOT EXISTS (
    SELECT 1 FROM Employee
    WHERE salary < (SELECT MAX(salary) FROM Employee)
)
LIMIT 1;
```

**Follow-up**: "Generalize to Nth highest salary."
```sql
-- Replace OFFSET 1 with OFFSET N-1
```

---

### Q2. Consecutive Numbers
*LeetCode 180 — reported at Meta, Google*

```sql
-- Table: Logs(id, num) where id is consecutive integers.
-- Find all numbers that appear at least 3 times consecutively.

SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l2.id = l1.id + 1 AND l2.num = l1.num
JOIN Logs l3 ON l3.id = l1.id + 2 AND l3.num = l1.num;

-- Window function approach (cleaner):
WITH ranked AS (
    SELECT num,
           LAG(num, 1) OVER (ORDER BY id) AS prev1,
           LAG(num, 2) OVER (ORDER BY id) AS prev2
    FROM Logs
)
SELECT DISTINCT num AS ConsecutiveNums
FROM ranked
WHERE num = prev1 AND num = prev2;
```

---

### Q3. Employees Earning More Than Managers
*LeetCode 181 — reported at Amazon*

```sql
-- Table: Employee(id, name, salary, managerId)
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m ON e.managerId = m.id
WHERE e.salary > m.salary;
```

---

### Q4. Duplicate Emails
*LeetCode 182 — reported at Amazon, Meta*

```sql
-- Table: Person(id, email)
SELECT email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## Difficulty 2: Intermediate (most common interview level)

### Q5. Rising Temperature
*LeetCode 197 — reported at Meta, Amazon*

```sql
-- Table: Weather(id, recordDate, temperature)
-- Find all dates' IDs with higher temperature than previous day.

SELECT w1.id
FROM Weather w1
JOIN Weather w2
  ON w2.recordDate = w1.recordDate - INTERVAL '1 day'  -- PostgreSQL
  -- ON DATEDIFF(w1.recordDate, w2.recordDate) = 1     -- MySQL
WHERE w1.temperature > w2.temperature;

-- Window function approach:
SELECT id
FROM (
    SELECT id,
           temperature,
           LAG(temperature) OVER (ORDER BY recordDate) AS prev_temp,
           LAG(recordDate)  OVER (ORDER BY recordDate) AS prev_date,
           recordDate
    FROM Weather
) t
WHERE temperature > prev_temp
  AND recordDate = prev_date + INTERVAL '1 day';
```

---

### Q6. Department Highest Salary
*LeetCode 184 — reported at Meta, Google, Amazon*

```sql
-- Tables: Employee(id, name, salary, departmentId), Department(id, name)
-- Find employees who earn the highest salary in each department.

SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM Employee e
JOIN Department d ON e.departmentId = d.id
WHERE (e.departmentId, e.salary) IN (
    SELECT departmentId, MAX(salary)
    FROM Employee
    GROUP BY departmentId
);

-- Window function approach (cleaner, handles ties):
WITH ranked AS (
    SELECT e.name AS emp_name, d.name AS dept_name, e.salary,
           RANK() OVER (PARTITION BY e.departmentId ORDER BY e.salary DESC) AS rnk
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
)
SELECT dept_name AS Department, emp_name AS Employee, salary AS Salary
FROM ranked
WHERE rnk = 1;
```

---

### Q7. Rank Scores
*LeetCode 178 — reported at Meta, Netflix*

```sql
-- Table: Scores(id, score)
-- Rank scores: same scores get same rank, no gaps in ranking (DENSE_RANK).

SELECT score,
       DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores
ORDER BY score DESC;

-- Difference between RANK, DENSE_RANK, ROW_NUMBER:
-- Scores: 100, 100, 90, 80
-- RANK:        1, 1, 3, 4  (gaps after ties)
-- DENSE_RANK:  1, 1, 2, 3  (no gaps)
-- ROW_NUMBER:  1, 2, 3, 4  (always unique)
```

---

### Q8. Consecutive Free Seats in a Cinema
*LeetCode 603 — reported at Netflix*

```sql
-- Table: Cinema(seat_id, free) — 1=free, 0=occupied
-- Find all free seats that are part of 2+ consecutive free seats.

SELECT DISTINCT c1.seat_id
FROM Cinema c1
JOIN Cinema c2
  ON ABS(c1.seat_id - c2.seat_id) = 1
 AND c1.free = 1
 AND c2.free = 1
ORDER BY c1.seat_id;
```

---

### Q9. Friend Requests Acceptance Rate
*LeetCode 597 — reported at Meta*

```sql
-- Tables: FriendRequest(sender_id, send_to_id, request_date)
--         RequestAccepted(requester_id, accepter_id, accept_date)
-- Find overall acceptance rate, rounded to 2 decimals.

SELECT ROUND(
    COALESCE(
        (SELECT COUNT(DISTINCT requester_id, accepter_id) FROM RequestAccepted) * 1.0
        / NULLIF((SELECT COUNT(DISTINCT sender_id, send_to_id) FROM FriendRequest), 0),
        0
    ), 2
) AS accept_rate;
```

---

### Q10. Find Users With Valid Emails
*LeetCode 1517 — reported at Meta*

```sql
-- Table: Users(user_id, name, mail)
-- Valid email: starts with letter, contains only letters/digits/./+/-/_,
-- ends with @leetcode.com

SELECT *
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_\\.\\-\\+]*@leetcode\\.com$';
```

---

## Difficulty 3: Window Functions (senior-level)

### Q11. Median Employee Salary
*LeetCode 569 — reported at Meta, Google*

```sql
-- Tables: Employee(id, company, salary)
-- Find median salary for each company (no built-in MEDIAN in most dialects).

WITH ranked AS (
    SELECT id, company, salary,
           ROW_NUMBER() OVER (PARTITION BY company ORDER BY salary)        AS row_asc,
           ROW_NUMBER() OVER (PARTITION BY company ORDER BY salary DESC)   AS row_desc,
           COUNT(*) OVER (PARTITION BY company)                            AS cnt
    FROM Employee
)
SELECT id, company, salary
FROM ranked
WHERE row_asc BETWEEN cnt/2.0 AND cnt/2.0 + 1
   OR row_desc BETWEEN cnt/2.0 AND cnt/2.0 + 1
ORDER BY company, salary;
```

**Key insight**: the median is where `row_asc` and `row_desc` are equal (odd n) or adjacent (even n). The `BETWEEN cnt/2 AND cnt/2+1` trick handles both.

---

### Q12. Cumulative Salary of an Employee
*LeetCode 579 — reported at Amazon, Meta*

```sql
-- Table: Employee(id, month, salary)
-- For each employee, compute 3-month rolling sum (current + 2 previous months).
-- Exclude the most recent month.

WITH rolling AS (
    SELECT id, month, salary,
           SUM(salary) OVER (
               PARTITION BY id
               ORDER BY month
               ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
           ) AS total,
           MAX(month) OVER (PARTITION BY id) AS max_month
    FROM Employee
)
SELECT id, month, total
FROM rolling
WHERE month != max_month
ORDER BY id, month DESC;
```

---

### Q13. Trips and Users (Cancellation Rate)
*LeetCode 262 — reported at Meta, Uber*

```sql
-- Tables: Trips(id, client_id, driver_id, city_id, status, request_at)
--         Users(users_id, banned, role)
-- Compute cancellation rate for unbanned users, Oct 1–3 2013.

SELECT t.request_at AS Day,
       ROUND(
           SUM(CASE WHEN t.status != 'completed' THEN 1 ELSE 0 END) * 1.0
           / COUNT(*),
       2) AS "Cancellation Rate"
FROM Trips t
JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
WHERE t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.request_at
ORDER BY t.request_at;
```

---

### Q14. Game Play Analysis IV
*LeetCode 550 — reported at Meta*

```sql
-- Table: Activity(player_id, device_id, event_date, games_played)
-- Find fraction of players who logged in on the day AFTER their first login.

WITH first_login AS (
    SELECT player_id, MIN(event_date) AS first_date
    FROM Activity
    GROUP BY player_id
),
next_day_logins AS (
    SELECT COUNT(DISTINCT a.player_id) AS next_day_count
    FROM Activity a
    JOIN first_login f
      ON a.player_id = f.player_id
     AND a.event_date = f.first_date + INTERVAL '1 day'
)
SELECT ROUND(
    next_day_count * 1.0 / (SELECT COUNT(DISTINCT player_id) FROM Activity),
2) AS fraction
FROM next_day_logins;
```

---

### Q15. Human Traffic of Stadium
*LeetCode 601 — reported at Google, Meta*

```sql
-- Table: Stadium(id, visit_date, people)
-- Find all rows with 3+ consecutive rows where people >= 100.

WITH flagged AS (
    SELECT *,
           CASE WHEN people >= 100 THEN 1 ELSE 0 END AS high
    FROM Stadium
),
groups AS (
    SELECT *,
           id - ROW_NUMBER() OVER (PARTITION BY high ORDER BY id) AS grp
    FROM flagged
    WHERE high = 1
),
valid_grps AS (
    SELECT grp FROM groups GROUP BY grp HAVING COUNT(*) >= 3
)
SELECT id, visit_date, people
FROM groups
WHERE grp IN (SELECT grp FROM valid_grps)
ORDER BY visit_date;
```

**Alternative using LAG/LEAD** (cleaner for interviews):
```sql
WITH windowed AS (
    SELECT id, visit_date, people,
           LAG(people, 1) OVER (ORDER BY id)  AS p1,
           LAG(people, 2) OVER (ORDER BY id)  AS p2,
           LEAD(people, 1) OVER (ORDER BY id) AS n1,
           LEAD(people, 2) OVER (ORDER BY id) AS n2
    FROM Stadium
)
SELECT DISTINCT id, visit_date, people
FROM windowed
WHERE people >= 100
  AND (
    (p1 >= 100 AND p2 >= 100) OR  -- curr is 3rd
    (p1 >= 100 AND n1 >= 100) OR  -- curr is 2nd
    (n1 >= 100 AND n2 >= 100)     -- curr is 1st
  )
ORDER BY visit_date;
```

---

### Q16. Nth Highest Salary (Generalized)
*LeetCode 177 — reported at Meta, Amazon*

```sql
-- Find Nth highest salary (handle N > number of distinct salaries → return NULL).

CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    SET N = N - 1;   -- OFFSET is 0-indexed
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET N
    );
END;
```

---

## Difficulty 4: Hard / Advanced (senior bar)

### Q17. Active Businesses
*LeetCode 1112 — reported at Google, Meta*

```sql
-- Tables: Events(business_id, event_type, occurrences)
-- An event is "above average" for a business if occurrences > avg for that event type.
-- An "active business" has more than 1 above-average event type.

WITH avg_events AS (
    SELECT event_type, AVG(occurrences) AS avg_occ
    FROM Events
    GROUP BY event_type
),
above_avg AS (
    SELECT e.business_id,
           COUNT(*) AS above_avg_count
    FROM Events e
    JOIN avg_events a ON e.event_type = a.event_type
    WHERE e.occurrences > a.avg_occ
    GROUP BY e.business_id
)
SELECT business_id
FROM above_avg
WHERE above_avg_count > 1;
```

---

### Q18. Find the Quiet Students in All Exams
*LeetCode 1412 — reported at Google*

```sql
-- Tables: Student(student_id, student_name), Exam(exam_id, student_id, score)
-- Find students who never scored the highest OR lowest on any exam.

WITH score_ranks AS (
    SELECT *,
           MAX(score) OVER (PARTITION BY exam_id) AS max_score,
           MIN(score) OVER (PARTITION BY exam_id) AS min_score
    FROM Exam
),
disqualified AS (
    SELECT DISTINCT student_id
    FROM score_ranks
    WHERE score = max_score OR score = min_score
)
SELECT s.student_id, s.student_name
FROM Student s
WHERE s.student_id IN (SELECT DISTINCT student_id FROM Exam)
  AND s.student_id NOT IN (SELECT student_id FROM disqualified)
ORDER BY s.student_id;
```

---

### Q19. Report Contiguous Dates
*LeetCode 1225 — reported at Meta, Google*

```sql
-- Tables: Failed(fail_date), Succeeded(success_date) — dates in 2019
-- Produce a report of consecutive periods and their state.

WITH all_dates AS (
    SELECT fail_date AS dt, 'failed' AS state FROM Failed
     WHERE fail_date BETWEEN '2019-01-01' AND '2019-12-31'
    UNION ALL
    SELECT success_date, 'succeeded' FROM Succeeded
     WHERE success_date BETWEEN '2019-01-01' AND '2019-12-31'
),
grouped AS (
    SELECT dt, state,
           ROW_NUMBER() OVER (ORDER BY dt) -
           ROW_NUMBER() OVER (PARTITION BY state ORDER BY dt) AS grp
    FROM all_dates
)
SELECT state AS period_state,
       MIN(dt) AS start_date,
       MAX(dt) AS end_date
FROM grouped
GROUP BY state, grp
ORDER BY start_date;
```

**Key trick**: the "islands and gaps" pattern — the difference between global row number and per-group row number is constant within a consecutive run of the same state.

---

# PART 2: SQL IN PRODUCT/ANALYTICS ROUNDS

These questions are open-ended. The interviewer wants to see your analytical thinking, not just syntax. Approach them as problem-solving conversations, not coding exercises.

---

## Meta Product Analytics Questions

### Q20. Investigate a metric drop
*Reported as a Meta DS and MLE screen question*

**"Daily active users dropped 10% last Tuesday. Walk me through how you'd investigate using SQL."**

This is not one query — it's a structured investigation. The interviewer wants to see your process:

```sql
-- Step 1: Confirm the drop is real
SELECT DATE(event_time) AS dt,
       COUNT(DISTINCT user_id) AS dau
FROM events
WHERE event_time >= CURRENT_DATE - 14
GROUP BY 1 ORDER BY 1;

-- Step 2: Segment by platform
SELECT DATE(event_time), platform,
       COUNT(DISTINCT user_id) AS dau
FROM events
GROUP BY 1, 2 ORDER BY 1, 2;

-- Step 3: Segment by geography
SELECT DATE(event_time), country,
       COUNT(DISTINCT user_id) AS dau
FROM events
GROUP BY 1, 2 ORDER BY 1, 2;

-- Step 4: Check by user tenure (new vs returning)
SELECT DATE(event_time),
       CASE WHEN DATEDIFF(event_time, created_at) < 7 THEN 'new'
            ELSE 'returning' END AS user_type,
       COUNT(DISTINCT e.user_id) AS dau
FROM events e
JOIN users u ON e.user_id = u.user_id
GROUP BY 1, 2;

-- Step 5: Check if specific features / surfaces affected
SELECT DATE(event_time), event_type,
       COUNT(*) AS event_count
FROM events
WHERE event_time >= CURRENT_DATE - 7
GROUP BY 1, 2
ORDER BY 1, 2;

-- Step 6: Check for data pipeline issues
SELECT DATE(created_at), COUNT(*) AS rows_logged
FROM events
GROUP BY 1 ORDER BY 1;
-- Sudden drop in rows = pipeline outage, not real user drop
```

**What the interviewer is looking for**: systematic breakdown, knowing to check logging/pipeline issues first, segmenting to isolate where the drop occurred.

---

### Q21. Design a metric to measure Instagram Stories health
*Reported at Meta DS*

**"How would you define and query a metric for the health of Instagram Stories?"**

Discuss before writing SQL:
- **Reach**: how many users posted a Story this week?
- **Views per Story**: average views per posted Story
- **Completion rate**: what fraction of viewers watched the full Story?
- **Replies/reactions**: engagement depth
- **Cross-post rate**: Stories shared to Feed

```sql
-- Reach: weekly creator count
SELECT DATE_TRUNC('week', created_at) AS week,
       COUNT(DISTINCT creator_id) AS weekly_creators
FROM stories
GROUP BY 1 ORDER BY 1;

-- Completion rate
SELECT DATE_TRUNC('day', view_time) AS dt,
       SUM(CASE WHEN viewed_fraction >= 0.9 THEN 1 ELSE 0 END) * 1.0
           / COUNT(*) AS completion_rate
FROM story_views
GROUP BY 1 ORDER BY 1;

-- Average views per story, by creator tier
SELECT
    CASE
        WHEN follower_count < 1000   THEN 'nano'
        WHEN follower_count < 10000  THEN 'micro'
        WHEN follower_count < 100000 THEN 'mid-tier'
        ELSE 'macro'
    END AS creator_tier,
    AVG(view_count) AS avg_views_per_story
FROM stories s
JOIN users u ON s.creator_id = u.user_id
GROUP BY 1;
```

---

### Q22. A/B Test Results Analysis
*Reported at Meta, Netflix, Amazon DS*

**"You ran an A/B test on a new feed ranking algorithm. Write a query to compute the key metrics and assess statistical significance."**

```sql
-- Tables:
-- experiment_assignments(user_id, variant, assigned_at)
-- events(user_id, event_type, event_time, session_id)

-- Step 1: Compute primary metric (CTR per user)
WITH user_metrics AS (
    SELECT
        a.user_id,
        a.variant,
        COUNT(CASE WHEN e.event_type = 'click' THEN 1 END) AS clicks,
        COUNT(CASE WHEN e.event_type = 'impression' THEN 1 END) AS impressions,
        COUNT(CASE WHEN e.event_type = 'click' THEN 1 END) * 1.0
            / NULLIF(COUNT(CASE WHEN e.event_type = 'impression' THEN 1 END), 0) AS user_ctr
    FROM experiment_assignments a
    LEFT JOIN events e ON a.user_id = e.user_id
        AND e.event_time > a.assigned_at  -- post-assignment only
    WHERE a.assigned_at >= '2024-01-01'
    GROUP BY 1, 2
),

-- Step 2: Aggregate by variant
variant_summary AS (
    SELECT
        variant,
        COUNT(DISTINCT user_id) AS n_users,
        SUM(clicks) AS total_clicks,
        SUM(impressions) AS total_impressions,
        AVG(user_ctr) AS mean_ctr,
        STDDEV(user_ctr) AS std_ctr,
        SUM(clicks) * 1.0 / NULLIF(SUM(impressions), 0) AS aggregate_ctr
    FROM user_metrics
    GROUP BY 1
)

SELECT * FROM variant_summary;

-- Step 3: Check for Sample Ratio Mismatch (SRM)
-- Expected: equal split. Actual should be within 5% of 50/50.
SELECT variant,
       COUNT(DISTINCT user_id) AS assigned_users,
       COUNT(DISTINCT user_id) * 100.0
           / SUM(COUNT(DISTINCT user_id)) OVER () AS pct_of_total
FROM experiment_assignments
GROUP BY 1;
-- If pct_of_total is not ~50%, experiment has SRM — results are invalid
```

**Key discussion points** (say out loud):
- SRM check must happen before looking at results
- Use user-level CTR (not aggregate CTR) as the unit to compute proper standard errors
- Minimum 2-week runtime for weekly seasonality
- Check guardrail metrics (session length, retention) not just the primary metric

---

### Q23. User Retention Cohort Analysis
*Reported at Meta, Netflix, Airbnb, Amazon DS*

**"Compute 30-day retention rate broken down by signup cohort (month)."**

```sql
WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', created_at) AS cohort_month
    FROM users
    WHERE created_at >= '2023-01-01'
),
activities AS (
    SELECT DISTINCT
        user_id,
        DATE_TRUNC('month', event_time) AS active_month
    FROM events
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(DISTINCT user_id) AS cohort_size
    FROM cohorts
    GROUP BY 1
),
retention_raw AS (
    SELECT
        c.cohort_month,
        a.active_month,
        DATEDIFF('month', c.cohort_month, a.active_month) AS months_since_signup,
        COUNT(DISTINCT c.user_id) AS retained_users
    FROM cohorts c
    JOIN activities a ON c.user_id = a.user_id
    GROUP BY 1, 2, 3
)
SELECT
    r.cohort_month,
    r.months_since_signup,
    cs.cohort_size,
    r.retained_users,
    ROUND(r.retained_users * 100.0 / cs.cohort_size, 1) AS retention_pct
FROM retention_raw r
JOIN cohort_sizes cs ON r.cohort_month = cs.cohort_month
WHERE r.months_since_signup BETWEEN 0 AND 6
ORDER BY r.cohort_month, r.months_since_signup;
```

**Interviewer follow-ups:**
- "How would you identify which cohort has the worst retention?" → add `ORDER BY retention_pct` for months_since_signup = 1
- "How would you compute 30-day retention instead of monthly?" → use `event_time BETWEEN created_at AND created_at + INTERVAL '30 days'`

---

### Q24. Revenue Attribution (Multi-Touch)
*Reported at Amazon, Google DS*

**"A user visited 3 pages (ad → organic search → direct) before purchasing. How do you query last-touch and first-touch attribution?"**

```sql
-- Table: sessions(user_id, session_id, channel, session_time)
--         orders(user_id, order_id, revenue, order_time)

-- Last-touch attribution
WITH session_before_order AS (
    SELECT
        o.user_id,
        o.order_id,
        o.revenue,
        s.channel,
        s.session_time,
        ROW_NUMBER() OVER (
            PARTITION BY o.order_id
            ORDER BY s.session_time DESC
        ) AS recency_rank
    FROM orders o
    JOIN sessions s
      ON o.user_id = s.user_id
     AND s.session_time <= o.order_time
)
SELECT channel,
       COUNT(DISTINCT order_id) AS attributed_orders,
       SUM(revenue) AS attributed_revenue
FROM session_before_order
WHERE recency_rank = 1  -- last touch
GROUP BY 1 ORDER BY 2 DESC;

-- First-touch: change ORDER BY session_time DESC → ASC
```

---

### Q25. Supply and Demand Balance (Marketplace)
*Reported at Uber, Airbnb, Amazon DS*

**"For each city and hour, compute the ratio of available drivers to ride requests (supply/demand ratio). Flag hours where demand exceeds supply by > 20%."**

```sql
WITH hourly_supply AS (
    SELECT city_id,
           DATE_TRUNC('hour', available_at) AS hour,
           COUNT(DISTINCT driver_id) AS available_drivers
    FROM driver_availability
    GROUP BY 1, 2
),
hourly_demand AS (
    SELECT city_id,
           DATE_TRUNC('hour', request_time) AS hour,
           COUNT(DISTINCT request_id) AS ride_requests
    FROM ride_requests
    GROUP BY 1, 2
)
SELECT
    s.city_id,
    s.hour,
    s.available_drivers,
    d.ride_requests,
    ROUND(s.available_drivers * 1.0 / NULLIF(d.ride_requests, 0), 2) AS supply_demand_ratio,
    CASE
        WHEN d.ride_requests > s.available_drivers * 1.2
        THEN 'DEMAND_EXCEEDS_SUPPLY'
        ELSE 'BALANCED'
    END AS status
FROM hourly_supply s
JOIN hourly_demand d USING (city_id, hour)
ORDER BY s.city_id, s.hour;
```

---

### Q26. Creator Revenue Share
*Reported at YouTube (Google), TikTok, Meta*

**"Find the top 10% of creators by total watch time and compute their share of overall revenue."**

```sql
WITH creator_watch_time AS (
    SELECT creator_id,
           SUM(watch_seconds) AS total_watch_seconds
    FROM video_views
    GROUP BY creator_id
),
creator_revenue AS (
    SELECT creator_id, SUM(revenue_usd) AS total_revenue
    FROM creator_payments
    GROUP BY creator_id
),
ranked AS (
    SELECT
        w.creator_id,
        w.total_watch_seconds,
        r.total_revenue,
        NTILE(10) OVER (ORDER BY w.total_watch_seconds DESC) AS decile
    FROM creator_watch_time w
    LEFT JOIN creator_revenue r ON w.creator_id = r.creator_id
)
SELECT
    SUM(CASE WHEN decile = 1 THEN total_revenue ELSE 0 END) AS top_10pct_revenue,
    SUM(total_revenue) AS total_platform_revenue,
    ROUND(
        SUM(CASE WHEN decile = 1 THEN total_revenue ELSE 0 END)
        * 100.0 / NULLIF(SUM(total_revenue), 0),
    1) AS top_10pct_revenue_share
FROM ranked;
```

---

### Q27. Session-Level Funnel with Drop-Off
*Reported at Meta, Amazon*

**"For users who started checkout, compute what % dropped off at each step. Also compute the average time spent on each step."**

```sql
WITH session_steps AS (
    SELECT
        session_id,
        user_id,
        MAX(CASE WHEN event_type = 'view_cart'       THEN event_time END) AS view_cart_time,
        MAX(CASE WHEN event_type = 'begin_checkout'  THEN event_time END) AS begin_checkout_time,
        MAX(CASE WHEN event_type = 'add_payment'     THEN event_time END) AS add_payment_time,
        MAX(CASE WHEN event_type = 'confirm_order'   THEN event_time END) AS confirm_order_time
    FROM events
    WHERE event_type IN ('view_cart','begin_checkout','add_payment','confirm_order')
    GROUP BY session_id, user_id
),
funnel AS (
    SELECT
        COUNT(*) FILTER (WHERE view_cart_time IS NOT NULL)      AS step1_view_cart,
        COUNT(*) FILTER (WHERE begin_checkout_time IS NOT NULL) AS step2_checkout,
        COUNT(*) FILTER (WHERE add_payment_time IS NOT NULL)    AS step3_payment,
        COUNT(*) FILTER (WHERE confirm_order_time IS NOT NULL)  AS step4_order,
        AVG(EXTRACT(EPOCH FROM (begin_checkout_time - view_cart_time))) AS avg_sec_step1_to_2,
        AVG(EXTRACT(EPOCH FROM (add_payment_time - begin_checkout_time))) AS avg_sec_step2_to_3,
        AVG(EXTRACT(EPOCH FROM (confirm_order_time - add_payment_time))) AS avg_sec_step3_to_4
    FROM session_steps
)
SELECT
    step1_view_cart,
    step2_checkout,
    step3_payment,
    step4_order,
    ROUND(step2_checkout * 100.0 / NULLIF(step1_view_cart, 0), 1) AS pct_to_step2,
    ROUND(step3_payment  * 100.0 / NULLIF(step2_checkout, 0), 1)  AS pct_to_step3,
    ROUND(step4_order    * 100.0 / NULLIF(step3_payment, 0), 1)   AS pct_to_step4,
    ROUND(avg_sec_step1_to_2) AS avg_sec_1_to_2,
    ROUND(avg_sec_step2_to_3) AS avg_sec_2_to_3,
    ROUND(avg_sec_step3_to_4) AS avg_sec_3_to_4
FROM funnel;
```

---

## Amazon-Specific SQL Questions

### Q28. Recommend Similar Products
*Amazon DS / Applied Scientist*

**"Find all pairs of products that were frequently bought together (co-purchased by > 100 users)."**

```sql
WITH user_purchases AS (
    SELECT DISTINCT user_id, product_id
    FROM orders
    WHERE status = 'delivered'
),
product_pairs AS (
    SELECT
        LEAST(p1.product_id, p2.product_id) AS product_a,
        GREATEST(p1.product_id, p2.product_id) AS product_b,
        COUNT(DISTINCT p1.user_id) AS co_purchase_count
    FROM user_purchases p1
    JOIN user_purchases p2
      ON p1.user_id = p2.user_id
     AND p1.product_id < p2.product_id  -- avoid duplicates
    GROUP BY 1, 2
    HAVING COUNT(DISTINCT p1.user_id) > 100
)
SELECT product_a, product_b, co_purchase_count
FROM product_pairs
ORDER BY co_purchase_count DESC
LIMIT 50;
```

---

### Q29. Customer Lifetime Value by Acquisition Channel
*Amazon, Airbnb DS*

```sql
SELECT
    u.acquisition_channel,
    COUNT(DISTINCT u.user_id) AS total_users,
    COUNT(DISTINCT o.user_id) AS buyers,
    ROUND(COUNT(DISTINCT o.user_id) * 100.0 / COUNT(DISTINCT u.user_id), 1) AS conversion_rate,
    ROUND(AVG(user_ltv.total_spend), 2) AS avg_ltv_all_users,
    ROUND(AVG(CASE WHEN user_ltv.total_spend > 0 THEN user_ltv.total_spend END), 2) AS avg_ltv_buyers
FROM users u
LEFT JOIN (
    SELECT user_id, SUM(revenue) AS total_spend
    FROM orders
    WHERE status = 'delivered'
    GROUP BY user_id
) user_ltv ON u.user_id = user_ltv.user_id
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY 1
ORDER BY avg_ltv_all_users DESC;
```

---

## Common SQL Interview Mistakes to Avoid

### 1. Filtering after GROUP BY when you need HAVING
```sql
-- WRONG: can't use aggregate in WHERE
SELECT user_id, COUNT(*) FROM orders
WHERE COUNT(*) > 5 GROUP BY user_id;

-- RIGHT:
SELECT user_id, COUNT(*) FROM orders
GROUP BY user_id HAVING COUNT(*) > 5;
```

### 2. Division by zero
```sql
-- Always use NULLIF for denominators
ROUND(numerator * 1.0 / NULLIF(denominator, 0), 2)
```

### 3. Forgetting that COUNT(*) includes NULLs but COUNT(col) doesn't
```sql
COUNT(*)          -- counts all rows including NULLs
COUNT(col)        -- counts non-NULL values of col
COUNT(DISTINCT col) -- counts unique non-NULL values
```

### 4. Self-join without alias
Always alias self-joins: `FROM users u1 JOIN users u2 ON u1.referred_by = u2.user_id`

### 5. Using LEFT JOIN when INNER JOIN is intended
LEFT JOIN keeps rows with no match (fills with NULLs). If you need only matched rows, use INNER JOIN. Common source of inflated counts.

### 6. Not handling time zones
In global products, store timestamps in UTC. Convert to local time only for display: `CONVERT_TZ(event_time, 'UTC', user_timezone)`

### 7. Confusing RANK, DENSE_RANK, ROW_NUMBER
- Use `ROW_NUMBER` when you need exactly one row per partition (e.g., latest record per user)
- Use `DENSE_RANK` when you want ties to share a rank with no gaps (e.g., leaderboard)
- Use `RANK` only when you specifically want gaps after ties

---

## Practice Platforms

### For SQL Coding Rounds
| Platform | Best for | Free? |
|----------|---------|-------|
| **StrataScratch** | Real MAANG questions, company filter | Partial |
| **LeetCode (Database section)** | 200+ questions, same format as interviews | Partial |
| **HackerRank SQL** | Structured difficulty progression | Yes |
| **DataLemur** | Modern product analytics questions | Partial |
| **Mode SQL Tutorial** | Window functions deep dive | Yes |

### For Product Analytics Round Prep
| Resource | Best for |
|----------|---------|
| **Exponent** | Meta/Google DS mock interviews with feedback |
| **InterviewQuery** | Company-specific DS interview prep |
| **Ace the Data Science Interview** (book) | 200+ real interview questions with solutions |
