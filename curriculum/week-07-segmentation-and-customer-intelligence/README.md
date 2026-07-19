# Week 7 — Segmentation & Customer Intelligence

> **Goal:** by Sunday you can take a warehouse of customers, purchases, and product usage and answer the question every VP of Growth eventually asks — *who, specifically, should we spend the next dollar on?* — with RFM value tiers built in SQL, behavioral cohorts built from feature-usage patterns, and a k-means clustering pass in pandas that turns thirty rows of numbers into personas you can actually act on.

Week 6 built the marts. This week you finally point them at the question they exist to answer: not "how much revenue do we have" but "which customers *are* the revenue, and what do the rest need from us?" Most teams answer that with a gut feeling and a slide titled "Our Customer Personas" that nobody can trace back to a query. You're going to do it properly — three techniques, three levels of rigor, one dataset:

1. **RFM segmentation** — score every customer on how recently, how often, and how much they buy, then turn those three scores into value tiers a sales or CS team can act on today.
2. **Behavioral segmentation** — forget spend for a moment and segment by what customers *do* in the product: which features they've adopted, how deep they've gone, whether they've gone quiet.
3. **Clustering** — stop hand-picking thresholds and let k-means find the groups in the data, then do the harder work of turning "cluster 3" into a persona a human can target.

You'll also learn where each technique lies to you. RFM's quintile cutoffs are arbitrary by construction. Behavioral tiers are only as good as the thresholds you pick. And k-means will confidently draw boundaries around groups that, on inspection, aren't actually different in any way that matters. Knowing *when* a segmentation is real versus decorative is the actual skill this week teaches — not the SQL, which by Week 7 you can already write.

We work all week against **Crunch Flow**, the same B2B project-management SaaS from Week 6 — now with two new tables: **`orders`** (a la carte add-on purchases — extra seats, integration packs, onboarding services — the purchase history that makes RFM possible) and **`product_events`** (feature-usage logs — the behavior that makes behavioral segmentation and clustering possible). 30 customers, the same company, seen from two new angles.

## Learning objectives

By the end of this week, you will be able to:

- **Build** RFM (recency, frequency, monetary) segments in SQL — score every customer with `NTILE`, combine the three scores into a value tier, and defend every tier boundary you draw.
- **Construct** behavioral segments from feature-usage patterns — feature breadth, "power" actions, and days-since-last-active — and connect behavior back to revenue and churn risk.
- **Cluster** customers with k-means in pandas — build a feature matrix from SQL output, scale it correctly, choose `k` with the elbow and silhouette methods, and interpret what each cluster actually represents.
- **Profile** segments by retention risk, MRR, and add-on spend to decide which ones are worth a team's limited attention.
- **Translate** a segmentation — any of the three — into a concrete, defensible targeting action, and know the difference between a segment and a decoration.

## Prerequisites

- **C33 Crunch SQL** through window functions and CTEs (`NTILE`, `ROW_NUMBER`, `CASE`, `GROUP BY`) — this week leans on `NTILE` constantly.
- **C38 Weeks 1–6** — comfortable with events-based schemas, retention concepts, and the raw → staging → marts mental model. Week 6's `dim_user`/`fct_mrr_monthly` pattern is exactly the shape this week's `customers` table plays.
- **Python 3.10+ with `pandas` and `scikit-learn`** (`pip install pandas scikit-learn`) — Lecture 3 and Exercise 3 build a k-means model. If you've never used scikit-learn, don't worry — Lecture 3 walks every line.
- PostgreSQL 16+ **or** SQLite 3.35+. **No spreadsheets, anywhere, at any point** — every score, tier, and cluster this week is a SQL query or a pandas `DataFrame`, never a hand-sorted list in Excel. Spreadsheets are taught only in C41 Crunch Excel, and never as a data store.

## Set up the seed database (do this first)

Three tables this week. `customers` is who's paying and how much; `orders` is what they've bought on top of their subscription (the source for RFM); `product_events` is what they actually do in the product (the source for behavioral segmentation and clustering).

**PostgreSQL:**

```bash
createdb crunch_scale_w7
psql crunch_scale_w7
```

**SQLite:**

```bash
sqlite3 crunch_scale_w7.db
```

Then paste this into the shell (it works unchanged on both engines):

```sql
-- ── Customers: 30 Crunch Flow accounts, their plan and subscription MRR ──
CREATE TABLE customers (
    customer_id   INTEGER PRIMARY KEY,
    company_name  TEXT    NOT NULL,
    plan          TEXT    NOT NULL,   -- 'Starter' | 'Growth' | 'Scale'
    mrr           NUMERIC NOT NULL,   -- subscription MRR: 29 / 99 / 299
    signup_date   DATE    NOT NULL,
    country       TEXT    NOT NULL,
    industry      TEXT    NOT NULL
);

INSERT INTO customers (customer_id, company_name, plan, mrr, signup_date, country, industry) VALUES
(1,'Northwind Traders','Scale',299,'2025-01-15','USA','Retail'),
(2,'Lumen Studio','Scale',299,'2025-01-19','USA','Design'),
(3,'Fjord Tech','Scale',299,'2025-01-23','Norway','Software'),
(4,'BrightPath Consulting','Scale',299,'2025-01-27','USA','Consulting'),
(5,'Stavro AB','Scale',299,'2025-01-31','Sweden','Manufacturing'),
(6,'Marlowe & Co','Growth',99,'2025-01-20','UK','Legal'),
(7,'Rio Verde Software','Growth',99,'2025-01-25','Mexico','Software'),
(8,'Baumgarten GmbH','Growth',99,'2025-01-30','Germany','Manufacturing'),
(9,'Del Mar Analytics','Growth',99,'2025-02-04','Spain','Software'),
(10,'Hanuri Labs','Growth',99,'2025-02-09','South Korea','Software'),
(11,'NovaSys','Scale',299,'2025-10-05','Canada','Software'),
(12,'Westfold Digital','Growth',99,'2025-10-11','Ireland','Marketing'),
(13,'Suncrest Farms','Scale',299,'2025-10-17','Australia','Agriculture'),
(14,'Tidewater Logistics','Scale',299,'2025-10-23','USA','Logistics'),
(15,'BrightLoop','Scale',299,'2025-10-29','USA','Software'),
(16,'Cinder & Ash','Growth',99,'2025-02-01','USA','Retail'),
(17,'Palmwood Studios','Growth',99,'2025-02-06','Canada','Media'),
(18,'Kestrel Partners','Growth',99,'2025-02-11','UK','Consulting'),
(19,'Verdant Foods','Growth',99,'2025-02-16','France','Retail'),
(20,'Ashgrove Legal','Growth',99,'2025-02-21','UK','Legal'),
(21,'Ironclad Freight','Starter',29,'2025-03-10','USA','Logistics'),
(22,'Halcyon Health','Starter',29,'2025-03-17','USA','Healthcare'),
(23,'Milltown Robotics','Starter',29,'2025-03-24','Germany','Manufacturing'),
(24,'Bluepeak Media','Starter',29,'2025-03-31','USA','Media'),
(25,'Coral & Vine','Starter',29,'2025-04-07','Australia','Retail'),
(26,'Driftwood Analytics','Growth',99,'2025-01-25','USA','Software'),
(27,'Falkirk Systems','Starter',29,'2025-01-31','Scotland','Software'),
(28,'Amberline Goods','Starter',29,'2025-02-06','USA','Retail'),
(29,'Thistledown Farms','Growth',99,'2025-02-12','Ireland','Agriculture'),
(30,'Nightingale Clinics','Starter',29,'2025-02-18','USA','Healthcare');

-- ── Orders: a la carte add-on purchases on top of the subscription ──
-- (extra seats, premium templates, integration packs, onboarding, support, reporting)
CREATE TABLE orders (
    order_id     INTEGER PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date   DATE    NOT NULL,
    item         TEXT    NOT NULL,
    amount       NUMERIC NOT NULL
);

INSERT INTO orders (order_id, customer_id, order_date, item, amount) VALUES
(1,1,'2025-08-05','priority_support',97.5), (2,1,'2025-08-05','priority_support',97.5), (3,1,'2025-08-08','priority_support',97.5),
(4,1,'2025-09-05','advanced_reporting',76.7), (5,1,'2025-10-01','premium_template',32.5), (6,1,'2025-10-20','extra_seats',36.97),
(7,1,'2025-10-27','onboarding_service',390.0), (8,1,'2025-12-22','premium_template',32.5), (9,2,'2025-04-03','integration_pack',63.7),
(10,2,'2025-06-16','advanced_reporting',76.7), (11,2,'2025-08-22','premium_template',32.5), (12,2,'2025-10-31','integration_pack',63.7),
(13,2,'2025-11-01','advanced_reporting',76.7), (14,2,'2025-11-27','priority_support',97.5), (15,2,'2025-12-26','advanced_reporting',76.7),
(16,3,'2025-04-01','onboarding_service',390.0), (17,3,'2025-04-18','premium_template',32.5), (18,3,'2025-07-23','priority_support',97.5),
(19,3,'2025-08-10','extra_seats',37.0), (20,3,'2025-08-11','priority_support',97.5), (21,3,'2025-11-18','integration_pack',63.7),
(22,3,'2025-12-08','advanced_reporting',76.7), (23,3,'2025-12-19','onboarding_service',390.0), (24,4,'2025-05-21','onboarding_service',390.0),
(25,4,'2025-05-26','premium_template',32.5), (26,4,'2025-06-08','priority_support',97.5), (27,4,'2025-08-21','extra_seats',23.7),
(28,4,'2025-09-19','premium_template',32.5), (29,4,'2025-11-01','onboarding_service',390.0), (30,4,'2025-11-26','integration_pack',63.7),
(31,4,'2025-12-14','priority_support',97.5), (32,4,'2025-12-22','onboarding_service',390.0), (33,5,'2025-03-12','integration_pack',63.7),
(34,5,'2025-04-23','priority_support',97.5), (35,5,'2025-08-21','premium_template',32.5), (36,5,'2025-08-30','premium_template',32.5),
(37,5,'2025-09-11','extra_seats',38.57), (38,5,'2025-10-12','extra_seats',75.59), (39,5,'2025-10-17','integration_pack',63.7),
(40,5,'2025-12-12','integration_pack',63.7), (41,5,'2025-12-17','extra_seats',64.98), (42,6,'2025-05-12','extra_seats',36.71),
(43,6,'2025-06-21','extra_seats',20.67), (44,6,'2025-07-24','advanced_reporting',59.0), (45,6,'2025-08-21','premium_template',25.0),
(46,6,'2025-10-07','onboarding_service',300.0), (47,7,'2025-04-21','priority_support',75.0), (48,7,'2025-04-25','extra_seats',25.91),
(49,7,'2025-06-08','priority_support',75.0), (50,7,'2025-07-03','priority_support',75.0), (51,7,'2025-10-09','onboarding_service',300.0),
(52,8,'2025-03-21','premium_template',25.0), (53,8,'2025-04-16','premium_template',25.0), (54,8,'2025-05-24','premium_template',25.0),
(55,8,'2025-06-03','premium_template',25.0), (56,8,'2025-06-22','extra_seats',24.34), (57,8,'2025-08-19','priority_support',75.0),
(58,8,'2025-09-14','priority_support',75.0), (59,9,'2025-03-04','premium_template',25.0), (60,9,'2025-04-30','premium_template',25.0),
(61,9,'2025-07-18','premium_template',25.0), (62,9,'2025-10-08','priority_support',75.0), (63,9,'2025-10-09','priority_support',75.0),
(64,9,'2025-10-15','extra_seats',49.57), (65,10,'2025-02-21','extra_seats',35.63), (66,10,'2025-02-22','onboarding_service',300.0),
(67,10,'2025-04-30','priority_support',75.0), (68,10,'2025-07-19','advanced_reporting',59.0), (69,10,'2025-08-14','onboarding_service',300.0),
(70,10,'2025-09-29','advanced_reporting',59.0), (71,10,'2025-10-18','priority_support',75.0), (72,11,'2025-11-05','priority_support',82.5),
(73,11,'2025-11-25','onboarding_service',330.0), (74,11,'2025-12-28','onboarding_service',330.0), (75,12,'2025-12-06','onboarding_service',330.0),
(76,12,'2025-12-24','priority_support',82.5), (77,13,'2025-12-10','onboarding_service',330.0), (78,13,'2025-12-21','onboarding_service',330.0),
(79,13,'2025-12-27','priority_support',82.5), (80,14,'2025-11-07','premium_template',27.5), (81,14,'2025-12-27','extra_seats',60.14),
(82,15,'2025-10-29','premium_template',27.5), (83,15,'2025-11-18','priority_support',82.5), (84,15,'2025-12-27','integration_pack',53.9),
(85,16,'2025-02-01','integration_pack',49.0), (86,16,'2025-02-09','integration_pack',49.0), (87,16,'2025-05-06','integration_pack',49.0),
(88,16,'2025-07-20','extra_seats',20.4), (89,16,'2025-07-31','premium_template',25.0), (90,17,'2025-02-06','priority_support',75.0),
(91,17,'2025-04-02','onboarding_service',300.0), (92,17,'2025-07-11','priority_support',75.0), (93,17,'2025-07-24','premium_template',25.0),
(94,17,'2025-08-03','extra_seats',18.85), (95,18,'2025-02-11','integration_pack',49.0), (96,18,'2025-02-11','advanced_reporting',59.0),
(97,18,'2025-04-23','integration_pack',49.0), (98,18,'2025-06-27','extra_seats',35.3), (99,18,'2025-07-30','priority_support',75.0),
(100,18,'2025-08-13','integration_pack',49.0), (101,19,'2025-02-16','advanced_reporting',59.0), (102,19,'2025-06-08','onboarding_service',300.0),
(103,19,'2025-06-24','priority_support',75.0), (104,19,'2025-06-30','onboarding_service',300.0), (105,19,'2025-07-19','extra_seats',16.02),
(106,20,'2025-02-21','advanced_reporting',59.0), (107,20,'2025-02-21','advanced_reporting',59.0), (108,20,'2025-02-21','integration_pack',49.0),
(109,20,'2025-03-16','extra_seats',38.45), (110,20,'2025-04-25','onboarding_service',300.0), (111,20,'2025-07-18','integration_pack',49.0),
(112,21,'2025-08-01','advanced_reporting',47.2), (113,21,'2025-10-19','onboarding_service',240.0), (114,23,'2025-10-11','premium_template',20.0),
(115,25,'2025-09-24','advanced_reporting',47.2), (116,25,'2025-10-19','premium_template',20.0), (117,26,'2025-01-25','onboarding_service',270.0),
(118,26,'2025-04-07','advanced_reporting',53.1), (119,27,'2025-05-13','priority_support',67.5), (120,28,'2025-03-14','priority_support',67.5),
(121,29,'2025-02-17','extra_seats',48.73), (122,29,'2025-02-25','extra_seats',29.79), (123,29,'2025-04-14','onboarding_service',270.0),
(124,30,'2025-02-18','integration_pack',44.1), (125,30,'2025-03-10','premium_template',22.5);

-- ── Product events: feature-usage log, the source for behavioral segmentation ──
CREATE TABLE product_events (
    event_id     INTEGER PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(customer_id),
    event_name   TEXT    NOT NULL,   -- see the 8-event vocabulary below
    event_date   DATE    NOT NULL
);

INSERT INTO product_events (event_id, customer_id, event_name, event_date) VALUES
(1,1,'integration_connected','2025-02-08'), (2,1,'invite_sent','2025-02-24'), (3,1,'comment_added','2025-03-12'),
(4,1,'api_call','2025-03-17'), (5,1,'task_created','2025-03-28'), (6,1,'template_used','2025-04-27'),
(7,1,'invite_sent','2025-04-29'), (8,1,'comment_added','2025-05-02'), (9,1,'automation_triggered','2025-06-08'),
(10,1,'task_created','2025-06-13'), (11,1,'template_used','2025-07-18'), (12,1,'invite_sent','2025-07-23'),
(13,1,'report_created','2025-07-24'), (14,1,'report_created','2025-08-07'), (15,1,'automation_triggered','2025-08-09'),
(16,1,'automation_triggered','2025-08-13'), (17,1,'report_created','2025-09-03'), (18,1,'invite_sent','2025-10-19'),
(19,1,'comment_added','2025-10-21'), (20,1,'integration_connected','2025-11-03'), (21,1,'integration_connected','2025-11-24'),
(22,1,'invite_sent','2025-12-23'), (23,1,'automation_triggered','2025-12-27'), (24,1,'api_call','2025-12-30'),
(25,2,'template_used','2025-02-15'), (26,2,'invite_sent','2025-02-16'), (27,2,'task_created','2025-03-01'),
(28,2,'integration_connected','2025-03-09'), (29,2,'report_created','2025-03-23'), (30,2,'report_created','2025-03-28'),
(31,2,'api_call','2025-03-28'), (32,2,'report_created','2025-04-06'), (33,2,'api_call','2025-04-23'),
(34,2,'api_call','2025-05-23'), (35,2,'comment_added','2025-06-03'), (36,2,'automation_triggered','2025-06-09'),
(37,2,'comment_added','2025-06-21'), (38,2,'comment_added','2025-07-03'), (39,2,'comment_added','2025-08-09'),
(40,2,'api_call','2025-08-13'), (41,2,'automation_triggered','2025-08-20'), (42,2,'automation_triggered','2025-09-19'),
(43,2,'integration_connected','2025-10-13'), (44,2,'integration_connected','2025-11-01'), (45,2,'automation_triggered','2025-11-02'),
(46,2,'comment_added','2025-12-27'), (47,3,'invite_sent','2025-01-23'), (48,3,'integration_connected','2025-02-23'),
(49,3,'template_used','2025-03-05'), (50,3,'api_call','2025-03-05'), (51,3,'template_used','2025-03-17'),
(52,3,'comment_added','2025-04-06'), (53,3,'automation_triggered','2025-05-04'), (54,3,'automation_triggered','2025-05-31'),
(55,3,'api_call','2025-05-31'), (56,3,'api_call','2025-07-05'), (57,3,'report_created','2025-07-09'),
(58,3,'task_created','2025-07-24'), (59,3,'api_call','2025-08-06'), (60,3,'integration_connected','2025-08-07'),
(61,3,'comment_added','2025-08-31'), (62,3,'api_call','2025-09-03'), (63,3,'api_call','2025-09-12'),
(64,3,'automation_triggered','2025-09-20'), (65,3,'comment_added','2025-11-11'), (66,3,'api_call','2025-11-26'),
(67,3,'comment_added','2025-12-25'), (68,3,'integration_connected','2025-12-28'), (69,3,'integration_connected','2025-12-28'),
(70,3,'report_created','2025-12-29'), (71,4,'api_call','2025-02-06'), (72,4,'integration_connected','2025-03-27'),
(73,4,'task_created','2025-04-04'), (74,4,'template_used','2025-04-24'), (75,4,'comment_added','2025-05-06'),
(76,4,'report_created','2025-05-06'), (77,4,'report_created','2025-05-26'), (78,4,'task_created','2025-06-02'),
(79,4,'template_used','2025-06-15'), (80,4,'automation_triggered','2025-06-17'), (81,4,'integration_connected','2025-06-21'),
(82,4,'invite_sent','2025-08-13'), (83,4,'report_created','2025-08-22'), (84,4,'report_created','2025-09-29'),
(85,4,'automation_triggered','2025-10-26'), (86,4,'report_created','2025-11-03'), (87,4,'automation_triggered','2025-12-15'),
(88,4,'comment_added','2025-12-15'), (89,4,'api_call','2025-12-20'), (90,4,'task_created','2025-12-26'),
(91,5,'invite_sent','2025-01-31'), (92,5,'comment_added','2025-01-31'), (93,5,'api_call','2025-02-01'),
(94,5,'integration_connected','2025-03-18'), (95,5,'report_created','2025-03-30'), (96,5,'comment_added','2025-04-07'),
(97,5,'report_created','2025-04-12'), (98,5,'integration_connected','2025-04-13'), (99,5,'template_used','2025-04-26'),
(100,5,'comment_added','2025-05-01'), (101,5,'comment_added','2025-05-03'), (102,5,'automation_triggered','2025-05-25'),
(103,5,'comment_added','2025-06-12'), (104,5,'integration_connected','2025-06-23'), (105,5,'integration_connected','2025-07-23'),
(106,5,'report_created','2025-07-28'), (107,5,'invite_sent','2025-08-31'), (108,5,'comment_added','2025-09-15'),
(109,5,'task_created','2025-09-18'), (110,5,'integration_connected','2025-09-21'), (111,5,'api_call','2025-09-25'),
(112,5,'integration_connected','2025-11-03'), (113,5,'template_used','2025-11-09'), (114,5,'integration_connected','2025-12-25'),
(115,5,'report_created','2025-12-29'), (116,6,'api_call','2025-01-30'), (117,6,'task_created','2025-02-11'),
(118,6,'comment_added','2025-04-20'), (119,6,'template_used','2025-05-01'), (120,6,'task_created','2025-05-04'),
(121,6,'comment_added','2025-05-19'), (122,6,'comment_added','2025-05-21'), (123,6,'comment_added','2025-06-27'),
(124,6,'comment_added','2025-08-10'), (125,6,'comment_added','2025-08-10'), (126,6,'integration_connected','2025-09-01'),
(127,6,'task_created','2025-09-17'), (128,6,'comment_added','2025-09-21'), (129,6,'task_created','2025-10-16'),
(130,6,'comment_added','2025-11-26'), (131,6,'task_created','2025-12-26'), (132,7,'comment_added','2025-01-29'),
(133,7,'comment_added','2025-02-16'), (134,7,'task_created','2025-03-13'), (135,7,'task_created','2025-05-03'),
(136,7,'comment_added','2025-05-14'), (137,7,'comment_added','2025-05-22'), (138,7,'task_created','2025-06-20'),
(139,7,'task_created','2025-06-27'), (140,7,'task_created','2025-07-22'), (141,7,'report_created','2025-07-28'),
(142,7,'comment_added','2025-08-03'), (143,7,'task_created','2025-08-05'), (144,7,'task_created','2025-10-13'),
(145,7,'integration_connected','2025-10-15'), (146,7,'task_created','2025-12-05'), (147,7,'comment_added','2025-12-10'),
(148,7,'task_created','2025-12-10'), (149,7,'comment_added','2025-12-19'), (150,8,'task_created','2025-02-05'),
(151,8,'comment_added','2025-02-07'), (152,8,'automation_triggered','2025-02-15'), (153,8,'task_created','2025-02-21'),
(154,8,'task_created','2025-03-02'), (155,8,'task_created','2025-03-11'), (156,8,'report_created','2025-04-15'),
(157,8,'report_created','2025-04-29'), (158,8,'template_used','2025-06-15'), (159,8,'comment_added','2025-07-09'),
(160,8,'report_created','2025-08-02'), (161,8,'comment_added','2025-08-13'), (162,8,'comment_added','2025-08-25'),
(163,8,'comment_added','2025-09-14'), (164,8,'task_created','2025-10-03'), (165,8,'comment_added','2025-10-30'),
(166,8,'task_created','2025-11-04'), (167,8,'task_created','2025-11-29'), (168,8,'integration_connected','2025-12-26'),
(169,9,'task_created','2025-03-07'), (170,9,'api_call','2025-03-09'), (171,9,'task_created','2025-04-04'),
(172,9,'comment_added','2025-04-22'), (173,9,'task_created','2025-05-26'), (174,9,'comment_added','2025-06-21'),
(175,9,'template_used','2025-07-03'), (176,9,'comment_added','2025-07-10'), (177,9,'comment_added','2025-07-24'),
(178,9,'report_created','2025-08-17'), (179,9,'task_created','2025-08-20'), (180,9,'task_created','2025-09-01'),
(181,9,'task_created','2025-10-06'), (182,9,'comment_added','2025-10-25'), (183,9,'task_created','2025-10-27'),
(184,9,'comment_added','2025-11-27'), (185,9,'comment_added','2025-12-07'), (186,9,'template_used','2025-12-26'),
(187,10,'report_created','2025-02-19'), (188,10,'task_created','2025-03-08'), (189,10,'task_created','2025-03-14'),
(190,10,'report_created','2025-03-15'), (191,10,'comment_added','2025-03-30'), (192,10,'comment_added','2025-04-28'),
(193,10,'task_created','2025-05-04'), (194,10,'invite_sent','2025-05-21'), (195,10,'template_used','2025-06-06'),
(196,10,'task_created','2025-07-09'), (197,10,'report_created','2025-07-26'), (198,10,'comment_added','2025-08-06'),
(199,10,'task_created','2025-10-13'), (200,10,'task_created','2025-10-28'), (201,10,'task_created','2025-10-28'),
(202,10,'comment_added','2025-11-29'), (203,10,'template_used','2025-12-25'), (204,11,'task_created','2025-10-18'),
(205,11,'comment_added','2025-11-07'), (206,11,'report_created','2025-11-09'), (207,11,'template_used','2025-11-21'),
(208,11,'report_created','2025-11-29'), (209,11,'integration_connected','2025-12-01'), (210,11,'report_created','2025-12-04'),
(211,11,'automation_triggered','2025-12-04'), (212,11,'task_created','2025-12-06'), (213,11,'automation_triggered','2025-12-13'),
(214,11,'integration_connected','2025-12-15'), (215,11,'template_used','2025-12-20'), (216,11,'invite_sent','2025-12-27'),
(217,12,'comment_added','2025-10-17'), (218,12,'invite_sent','2025-10-20'), (219,12,'report_created','2025-10-23'),
(220,12,'integration_connected','2025-10-23'), (221,12,'api_call','2025-10-29'), (222,12,'task_created','2025-10-30'),
(223,12,'report_created','2025-11-02'), (224,12,'task_created','2025-11-02'), (225,12,'automation_triggered','2025-11-20'),
(226,12,'task_created','2025-11-26'), (227,12,'task_created','2025-12-03'), (228,12,'api_call','2025-12-13'),
(229,12,'task_created','2025-12-16'), (230,12,'api_call','2025-12-18'), (231,12,'api_call','2025-12-21'),
(232,12,'automation_triggered','2025-12-26'), (233,12,'task_created','2025-12-30'), (234,13,'task_created','2025-10-17'),
(235,13,'report_created','2025-10-19'), (236,13,'invite_sent','2025-10-29'), (237,13,'api_call','2025-11-02'),
(238,13,'template_used','2025-11-06'), (239,13,'report_created','2025-11-09'), (240,13,'template_used','2025-11-11'),
(241,13,'comment_added','2025-11-13'), (242,13,'comment_added','2025-11-21'), (243,13,'comment_added','2025-11-23'),
(244,13,'template_used','2025-11-24'), (245,13,'comment_added','2025-12-09'), (246,13,'report_created','2025-12-10'),
(247,13,'automation_triggered','2025-12-11'), (248,13,'template_used','2025-12-16'), (249,13,'api_call','2025-12-23'),
(250,13,'task_created','2025-12-30'), (251,14,'automation_triggered','2025-10-23'), (252,14,'integration_connected','2025-10-23'),
(253,14,'comment_added','2025-10-23'), (254,14,'task_created','2025-10-26'), (255,14,'task_created','2025-11-11'),
(256,14,'comment_added','2025-11-22'), (257,14,'api_call','2025-11-23'), (258,14,'task_created','2025-11-25'),
(259,14,'report_created','2025-12-03'), (260,14,'automation_triggered','2025-12-13'), (261,14,'report_created','2025-12-15'),
(262,14,'comment_added','2025-12-20'), (263,14,'template_used','2025-12-20'), (264,14,'template_used','2025-12-23'),
(265,14,'invite_sent','2025-12-23'), (266,14,'report_created','2025-12-29'), (267,15,'invite_sent','2025-10-29'),
(268,15,'automation_triggered','2025-10-29'), (269,15,'task_created','2025-11-01'), (270,15,'task_created','2025-11-13'),
(271,15,'report_created','2025-11-16'), (272,15,'task_created','2025-11-25'), (273,15,'comment_added','2025-11-26'),
(274,15,'integration_connected','2025-11-27'), (275,15,'template_used','2025-11-28'), (276,15,'invite_sent','2025-11-30'),
(277,15,'api_call','2025-12-04'), (278,15,'report_created','2025-12-10'), (279,15,'api_call','2025-12-19'),
(280,15,'integration_connected','2025-12-25'), (281,15,'invite_sent','2025-12-30'), (282,16,'task_created','2025-02-05'),
(283,16,'comment_added','2025-02-12'), (284,16,'task_created','2025-04-26'), (285,16,'task_created','2025-05-04'),
(286,16,'task_created','2025-05-18'), (287,16,'task_created','2025-06-01'), (288,16,'task_created','2025-06-18'),
(289,16,'comment_added','2025-08-27'), (290,16,'task_created','2025-10-08'), (291,17,'comment_added','2025-02-13'),
(292,17,'task_created','2025-03-12'), (293,17,'report_created','2025-04-02'), (294,17,'template_used','2025-05-13'),
(295,17,'task_created','2025-05-27'), (296,17,'task_created','2025-06-04'), (297,17,'report_created','2025-06-11'),
(298,17,'integration_connected','2025-06-20'), (299,17,'task_created','2025-08-05'), (300,17,'comment_added','2025-09-18'),
(301,17,'task_created','2025-09-24'), (302,17,'comment_added','2025-10-31'), (303,18,'report_created','2025-02-11'),
(304,18,'comment_added','2025-06-07'), (305,18,'comment_added','2025-06-11'), (306,18,'invite_sent','2025-06-15'),
(307,18,'comment_added','2025-06-22'), (308,18,'report_created','2025-06-26'), (309,18,'template_used','2025-08-02'),
(310,18,'comment_added','2025-09-02'), (311,18,'report_created','2025-09-08'), (312,18,'comment_added','2025-10-18'),
(313,18,'comment_added','2025-10-21'), (314,19,'task_created','2025-02-16'), (315,19,'task_created','2025-02-16'),
(316,19,'task_created','2025-02-16'), (317,19,'task_created','2025-02-16'), (318,19,'task_created','2025-06-08'),
(319,19,'template_used','2025-06-29'), (320,19,'comment_added','2025-08-26'), (321,19,'task_created','2025-09-26'),
(322,19,'task_created','2025-10-25'), (323,20,'task_created','2025-03-04'), (324,20,'comment_added','2025-04-23'),
(325,20,'task_created','2025-05-06'), (326,20,'task_created','2025-05-25'), (327,20,'task_created','2025-05-31'),
(328,20,'comment_added','2025-06-15'), (329,20,'template_used','2025-09-14'), (330,20,'comment_added','2025-09-21'),
(331,20,'template_used','2025-10-14'), (332,21,'report_created','2025-05-18'), (333,21,'task_created','2025-09-03'),
(334,21,'task_created','2025-09-05'), (335,21,'report_created','2025-10-17'), (336,21,'comment_added','2025-12-01'),
(337,21,'template_used','2025-12-21'), (338,22,'comment_added','2025-03-17'), (339,22,'report_created','2025-04-17'),
(340,22,'task_created','2025-06-22'), (341,22,'task_created','2025-08-25'), (342,22,'task_created','2025-09-13'),
(343,22,'task_created','2025-11-05'), (344,22,'task_created','2025-11-23'), (345,23,'automation_triggered','2025-03-24'),
(346,23,'comment_added','2025-04-04'), (347,23,'task_created','2025-06-14'), (348,23,'task_created','2025-09-15'),
(349,23,'comment_added','2025-10-29'), (350,23,'task_created','2025-11-29'), (351,23,'task_created','2025-12-04'),
(352,24,'task_created','2025-03-31'), (353,24,'task_created','2025-03-31'), (354,24,'task_created','2025-04-13'),
(355,24,'task_created','2025-04-17'), (356,24,'comment_added','2025-05-01'), (357,24,'template_used','2025-05-02'),
(358,24,'task_created','2025-05-11'), (359,24,'report_created','2025-08-13'), (360,24,'task_created','2025-12-13'),
(361,25,'template_used','2025-04-07'), (362,25,'comment_added','2025-04-07'), (363,25,'comment_added','2025-05-12'),
(364,25,'comment_added','2025-05-31'), (365,25,'task_created','2025-08-11'), (366,25,'task_created','2025-09-14'),
(367,25,'task_created','2025-11-11'), (368,25,'task_created','2025-11-21'), (369,26,'task_created','2025-01-25'),
(370,26,'task_created','2025-02-24'), (371,26,'task_created','2025-07-11'), (372,26,'report_created','2025-07-24'),
(373,27,'comment_added','2025-01-31'), (374,27,'report_created','2025-01-31'), (375,27,'task_created','2025-01-31'),
(376,27,'comment_added','2025-03-13'), (377,27,'task_created','2025-04-27'), (378,28,'report_created','2025-03-11'),
(379,28,'api_call','2025-04-05'), (380,28,'comment_added','2025-04-16'), (381,28,'template_used','2025-06-09'),
(382,29,'invite_sent','2025-02-12'), (383,29,'task_created','2025-02-12'), (384,29,'task_created','2025-02-12'),
(385,29,'task_created','2025-04-01'), (386,29,'task_created','2025-05-26'), (387,29,'integration_connected','2025-07-13'),
(388,30,'task_created','2025-02-27'), (389,30,'automation_triggered','2025-03-08'), (390,30,'task_created','2025-03-30'),
(391,30,'task_created','2025-06-08'), (392,30,'comment_added','2025-06-15'), (393,30,'comment_added','2025-07-08');
```

Sanity check — this should print `30`, `125`, `393`:

```sql
SELECT COUNT(*) FROM customers;
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM product_events;
```

Two facts to hold onto all week:

- **"Today" is 2025-12-31.** Every recency calculation (RFM's `R`, behavioral "days since last active") is measured against this cutoff.
- **The `event_name` vocabulary is 8 values**, each with a clear meaning: `task_created`, `comment_added`, `report_created`, `template_used`, `invite_sent` are **core** actions any plan can do; `automation_triggered`, `integration_connected`, `api_call` are **power** actions that separate a casual user from a deeply-embedded one. Lecture 2 leans on this split constantly — three of eight event types are the ones that predict whether a customer is hard to churn.
- **Two customers (`22` Halcyon Health, `24` Bluepeak Media) have never placed an order.** That's on purpose — it's the `NULL`-recency case RFM has to handle honestly, and it's exactly the kind of row a sloppy `NTILE` query silently mishandles. Lecture 1 spends real time on it.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed data; RFM scoring in SQL | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | RFM value tiers; drills against the seed | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Behavioral segments from feature usage | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | k-means clustering in pandas; choosing k | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | Challenges: profile segments, plan targeting | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (RFM + behavioral + clustering) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-rfm-segmentation.md](./lecture-notes/01-rfm-segmentation.md) | Recency/frequency/monetary in SQL, `NTILE` scoring, `NULL`-safe recency, value tiers | 2h |
| 2 | [lecture-notes/02-behavioral-segments.md](./lecture-notes/02-behavioral-segments.md) | Feature-adoption breadth, power vs. core actions, days-since-last-active, behavior tiers | 2h |
| 3 | [lecture-notes/03-clustering-for-intelligence.md](./lecture-notes/03-clustering-for-intelligence.md) | k-means in pandas/scikit-learn, scaling, elbow + silhouette, interpreting clusters into personas | 2h |
| 4 | [exercises/exercise-01-build-rfm-scores.md](./exercises/exercise-01-build-rfm-scores.md) | Build the full RFM scoring query yourself against the seed | 1h |
| 5 | [exercises/exercise-02-behavioral-cohort.md](./exercises/exercise-02-behavioral-cohort.md) | Build a behavioral cohort from `product_events` | 1h |
| 6 | [exercises/exercise-03-kmeans-clusters.md](./exercises/exercise-03-kmeans-clusters.md) | Cluster Crunch Flow's customers with k-means in pandas | 1h |
| 7 | [challenges/challenge-01-profile-segments-by-value.md](./challenges/challenge-01-profile-segments-by-value.md) | Profile every segment by MRR, add-on spend, and retention risk — rank them | 1h |
| 8 | [challenges/challenge-02-turn-segments-into-actions.md](./challenges/challenge-02-turn-segments-into-actions.md) | Write the actual targeting plan a CS/growth team would run per segment | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Combine RFM + behavioral in SQL, cluster in pandas, recommend the segment to target | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Build an RFM value tier from raw purchase history in SQL — `NTILE`-scored, `NULL`-safe, and defensible tier boundary by tier boundary.
- Turn a feature-usage log into a behavioral segment that predicts engagement risk before churn shows up in the revenue numbers.
- Stand up a k-means model in pandas, justify your choice of `k` with the elbow and silhouette methods, and know when a cluster is a real pattern versus noise the algorithm was forced to name.
- Profile any segmentation by revenue and risk, and say — with a query behind the claim — which segment deserves the next dollar of attention.
- Explain, precisely, the difference between a segment and a "persona slide": one has a runnable query and a targeting action behind it, the other is decoration.

## Up next

[Week 8 — Pricing & Packaging Experiments](../week-08-pricing-and-packaging-experiments/) — now that you know who your customers *are*, you can test what they're willing to pay.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
