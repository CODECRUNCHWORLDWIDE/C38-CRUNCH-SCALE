# Week 4 — Retention & Cohort Analysis

> **Goal:** by Sunday you can take a raw `events` table and answer the one question every growth team eventually has to face on evidence, not vibes — *does this product's growth compound, or does it leak?* You'll build a cohort-retention triangle, split a month's active users into new/retained/resurrected/churned, and read a survival curve well enough to tell a durable engine from a leaky bucket.

Weeks 1–3 got people **in the door and to value** — acquisition funnels, activation, time-to-value. Retention asks the harder question: once someone gets there, do they *stay*? A product can have a beautiful funnel and a rotten core — acquire 1,000 users a month and lose 900 of them by month two — and every acquisition dollar after that is just refilling a bucket with a hole in it. Retention is where you find that hole, or find out there isn't one.

We work against one seed dataset — a fictional product called **Crunch** — for the whole week: a `users` table (48 people who signed up across four monthly cohorts) and an `events` table (366 activity events). You load it once (below); every lecture, exercise, challenge, and the mini-project queries it.

## Learning objectives

By the end of this week, you will be able to:

- **Build** a cohort-retention table (the "triangle") in SQL — group users by acquisition period, measure activity in each period after signup, and lay the result out so the shape is visible at a glance.
- **Distinguish** N-day, unbounded, and rolling retention, and pick the right one for a given question instead of reaching for whichever is easiest to query.
- **Read** a retention curve's shape — smiling, flattening, or terminal (heading to zero) — and say in one sentence what each shape means for the business.
- **Separate** a month's active users into **new**, **retained**, **resurrected**, and **churned**, and check your split against the accounting identity that has to hold.
- **Build** a survival curve, handle the right-censoring that immature cohorts create, and reason about long-run stickiness with a DAU/MAU ratio.

## Prerequisites

- **C33 Crunch SQL** through window functions and CTEs — this week leans hard on `GROUP BY`, `JOIN`, `CASE`, date arithmetic, and self-joins across time.
- **C38 Weeks 1–3** — you should already be comfortable with events-based instrumentation, funnels, and activation. This week is instrumentation's payoff: the same `events` shape, a new question.
- PostgreSQL 16+ **or** SQLite 3.35+ (see [`resources.md`](./resources.md)). Every query in this week runs unchanged on both, with call-outs where the date syntax differs.
- **No spreadsheets.** Every table, cohort grid, and curve this week is built and queried in SQL. If you want to chart the numbers, do it in Python + `matplotlib`/`pandas` against the same tables — never paste rows into Excel to eyeball them.

## Set up the seed dataset (do this first)

Two tables. `users` is who signed up and when; `events` is what they did afterward. One row in `events` = one day a user took a meaningful in-product action (`core_action`) — we don't need every click, just "were they here or not, on this day."

**PostgreSQL:**

```bash
createdb crunch_scale
psql crunch_scale
```

**SQLite:**

```bash
sqlite3 crunch_scale.db
```

Then paste this into the shell (unchanged on both engines):

```sql
CREATE TABLE users (
    user_id      INTEGER PRIMARY KEY,
    signup_date  DATE    NOT NULL,
    cohort_month CHAR(7) NOT NULL,   -- 'YYYY-MM', the calendar month of signup_date
    segment      TEXT    NOT NULL,   -- 'self_serve' | 'sales_assisted'
    channel      TEXT    NOT NULL,
    country      TEXT    NOT NULL
);

CREATE TABLE events (
    event_id    INTEGER PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(user_id),
    event_date  DATE    NOT NULL,
    event_type  TEXT    NOT NULL     -- 'core_action' for every row this week
);

INSERT INTO users (user_id, signup_date, cohort_month, segment, channel, country) VALUES
(1,'2025-02-08','2025-02','self_serve','paid_search','UK'),
(2,'2025-02-15','2025-02','self_serve','referral','Canada'),
(3,'2025-02-22','2025-02','self_serve','partner','Brazil'),
(4,'2025-02-05','2025-02','self_serve','organic','India'),
(5,'2025-02-12','2025-02','self_serve','paid_search','Germany'),
(6,'2025-02-19','2025-02','self_serve','referral','USA'),
(7,'2025-02-02','2025-02','self_serve','partner','UK'),
(8,'2025-02-09','2025-02','self_serve','organic','Canada'),
(9,'2025-02-16','2025-02','self_serve','paid_search','Brazil'),
(10,'2025-02-23','2025-02','sales_assisted','referral','India'),
(11,'2025-02-06','2025-02','sales_assisted','partner','Germany'),
(12,'2025-02-13','2025-02','sales_assisted','organic','USA'),
(13,'2025-03-20','2025-03','self_serve','paid_search','UK'),
(14,'2025-03-03','2025-03','self_serve','referral','Canada'),
(15,'2025-03-10','2025-03','self_serve','partner','Brazil'),
(16,'2025-03-17','2025-03','self_serve','organic','India'),
(17,'2025-03-24','2025-03','self_serve','paid_search','Germany'),
(18,'2025-03-07','2025-03','self_serve','referral','USA'),
(19,'2025-03-14','2025-03','self_serve','partner','UK'),
(20,'2025-03-21','2025-03','self_serve','organic','Canada'),
(21,'2025-03-04','2025-03','self_serve','paid_search','Brazil'),
(22,'2025-03-11','2025-03','sales_assisted','referral','India'),
(23,'2025-03-18','2025-03','sales_assisted','partner','Germany'),
(24,'2025-03-01','2025-03','sales_assisted','organic','USA'),
(25,'2025-04-08','2025-04','self_serve','paid_search','UK'),
(26,'2025-04-15','2025-04','self_serve','referral','Canada'),
(27,'2025-04-22','2025-04','self_serve','partner','Brazil'),
(28,'2025-04-05','2025-04','self_serve','organic','India'),
(29,'2025-04-12','2025-04','self_serve','paid_search','Germany'),
(30,'2025-04-19','2025-04','self_serve','referral','USA'),
(31,'2025-04-02','2025-04','self_serve','partner','UK'),
(32,'2025-04-09','2025-04','self_serve','organic','Canada'),
(33,'2025-04-16','2025-04','self_serve','paid_search','Brazil'),
(34,'2025-04-23','2025-04','sales_assisted','referral','India'),
(35,'2025-04-06','2025-04','sales_assisted','partner','Germany'),
(36,'2025-04-13','2025-04','sales_assisted','organic','USA'),
(37,'2025-05-20','2025-05','self_serve','paid_search','UK'),
(38,'2025-05-03','2025-05','self_serve','referral','Canada'),
(39,'2025-05-10','2025-05','self_serve','partner','Brazil'),
(40,'2025-05-17','2025-05','self_serve','organic','India'),
(41,'2025-05-24','2025-05','self_serve','paid_search','Germany'),
(42,'2025-05-07','2025-05','self_serve','referral','USA'),
(43,'2025-05-14','2025-05','self_serve','partner','UK'),
(44,'2025-05-21','2025-05','self_serve','organic','Canada'),
(45,'2025-05-04','2025-05','self_serve','paid_search','Brazil'),
(46,'2025-05-11','2025-05','sales_assisted','referral','India'),
(47,'2025-05-18','2025-05','sales_assisted','partner','Germany'),
(48,'2025-05-01','2025-05','sales_assisted','organic','USA');

INSERT INTO events (event_id, user_id, event_date, event_type) VALUES
(1,1,'2025-02-13','core_action'), (2,1,'2025-02-20','core_action'), (3,1,'2025-02-26','core_action'),
(4,1,'2025-02-28','core_action'), (5,1,'2025-04-21','core_action'), (6,1,'2025-04-25','core_action'),
(7,1,'2025-04-29','core_action'), (8,2,'2025-02-11','core_action'), (9,2,'2025-02-16','core_action'),
(10,2,'2025-02-20','core_action'), (11,3,'2025-02-13','core_action'), (12,3,'2025-02-26','core_action'),
(13,3,'2025-02-27','core_action'), (14,4,'2025-02-22','core_action'), (15,4,'2025-02-26','core_action'),
(16,4,'2025-02-28','core_action'), (17,5,'2025-02-03','core_action'), (18,5,'2025-02-28','core_action'),
(19,6,'2025-02-11','core_action'), (20,6,'2025-02-19','core_action'), (21,6,'2025-02-25','core_action'),
(22,6,'2025-03-31','core_action'), (23,6,'2025-05-08','core_action'), (24,6,'2025-05-11','core_action'),
(25,6,'2025-05-14','core_action'), (26,6,'2025-05-21','core_action'), (27,7,'2025-02-04','core_action'),
(28,7,'2025-02-10','core_action'), (29,7,'2025-02-27','core_action'), (30,7,'2025-06-06','core_action'),
(31,7,'2025-06-11','core_action'), (32,7,'2025-06-14','core_action'), (33,8,'2025-02-23','core_action'),
(34,8,'2025-02-24','core_action'), (35,8,'2025-06-29','core_action'), (36,9,'2025-02-13','core_action'),
(37,9,'2025-02-17','core_action'), (38,9,'2025-02-25','core_action'), (39,10,'2025-02-01','core_action'),
(40,10,'2025-02-03','core_action'), (41,10,'2025-02-09','core_action'), (42,10,'2025-02-11','core_action'),
(43,10,'2025-02-12','core_action'), (44,10,'2025-02-13','core_action'), (45,10,'2025-02-15','core_action'),
(46,10,'2025-02-19','core_action'), (47,10,'2025-02-23','core_action'), (48,10,'2025-02-28','core_action'),
(49,10,'2025-03-02','core_action'), (50,10,'2025-03-07','core_action'), (51,10,'2025-03-08','core_action'),
(52,10,'2025-03-09','core_action'), (53,10,'2025-03-19','core_action'), (54,10,'2025-03-20','core_action'),
(55,10,'2025-03-21','core_action'), (56,10,'2025-04-03','core_action'), (57,10,'2025-04-23','core_action'),
(58,10,'2025-04-24','core_action'), (59,10,'2025-04-26','core_action'), (60,10,'2025-04-28','core_action'),
(61,11,'2025-02-02','core_action'), (62,11,'2025-02-13','core_action'), (63,11,'2025-02-21','core_action'),
(64,11,'2025-02-22','core_action'), (65,11,'2025-02-26','core_action'), (66,12,'2025-02-03','core_action'),
(67,12,'2025-02-09','core_action'), (68,12,'2025-02-11','core_action'), (69,12,'2025-02-14','core_action'),
(70,12,'2025-02-16','core_action'), (71,12,'2025-02-19','core_action'), (72,12,'2025-02-21','core_action'),
(73,12,'2025-02-24','core_action'), (74,12,'2025-02-25','core_action'), (75,12,'2025-02-26','core_action'),
(76,12,'2025-06-09','core_action'), (77,13,'2025-03-10','core_action'), (78,13,'2025-03-30','core_action'),
(79,14,'2025-03-15','core_action'), (80,14,'2025-03-16','core_action'), (81,14,'2025-03-18','core_action'),
(82,14,'2025-03-20','core_action'), (83,15,'2025-03-04','core_action'), (84,15,'2025-03-17','core_action'),
(85,15,'2025-03-30','core_action'), (86,16,'2025-03-09','core_action'), (87,16,'2025-03-16','core_action'),
(88,16,'2025-03-29','core_action'), (89,16,'2025-04-01','core_action'), (90,16,'2025-05-02','core_action'),
(91,16,'2025-05-16','core_action'), (92,16,'2025-05-25','core_action'), (93,16,'2025-05-31','core_action'),
(94,17,'2025-03-07','core_action'), (95,17,'2025-04-03','core_action'), (96,17,'2025-04-09','core_action'),
(97,17,'2025-04-16','core_action'), (98,18,'2025-03-03','core_action'), (99,18,'2025-03-10','core_action'),
(100,18,'2025-03-14','core_action'), (101,18,'2025-05-04','core_action'), (102,18,'2025-05-05','core_action'),
(103,18,'2025-05-17','core_action'), (104,18,'2025-05-26','core_action'), (105,18,'2025-06-22','core_action'),
(106,18,'2025-06-25','core_action'), (107,19,'2025-03-20','core_action'), (108,19,'2025-03-26','core_action'),
(109,20,'2025-03-08','core_action'), (110,20,'2025-03-19','core_action'), (111,20,'2025-03-28','core_action'),
(112,21,'2025-03-02','core_action'), (113,21,'2025-03-28','core_action'), (114,21,'2025-03-30','core_action'),
(115,22,'2025-03-07','core_action'), (116,22,'2025-03-09','core_action'), (117,22,'2025-03-12','core_action'),
(118,22,'2025-03-20','core_action'), (119,22,'2025-03-23','core_action'), (120,22,'2025-03-24','core_action'),
(121,22,'2025-04-02','core_action'), (122,22,'2025-04-03','core_action'), (123,22,'2025-04-19','core_action'),
(124,22,'2025-04-24','core_action'), (125,22,'2025-04-25','core_action'), (126,22,'2025-04-27','core_action'),
(127,22,'2025-05-05','core_action'), (128,22,'2025-05-07','core_action'), (129,22,'2025-05-14','core_action'),
(130,22,'2025-05-24','core_action'), (131,22,'2025-05-31','core_action'), (132,22,'2025-06-05','core_action'),
(133,23,'2025-03-10','core_action'), (134,23,'2025-03-12','core_action'), (135,23,'2025-03-14','core_action'),
(136,23,'2025-03-17','core_action'), (137,23,'2025-03-18','core_action'), (138,23,'2025-03-31','core_action'),
(139,23,'2025-04-11','core_action'), (140,23,'2025-05-01','core_action'), (141,24,'2025-03-05','core_action'),
(142,24,'2025-03-06','core_action'), (143,24,'2025-03-08','core_action'), (144,24,'2025-03-11','core_action'),
(145,24,'2025-03-12','core_action'), (146,24,'2025-03-14','core_action'), (147,24,'2025-03-18','core_action'),
(148,24,'2025-03-21','core_action'), (149,24,'2025-03-23','core_action'), (150,24,'2025-03-31','core_action'),
(151,24,'2025-04-02','core_action'), (152,24,'2025-04-10','core_action'), (153,24,'2025-04-15','core_action'),
(154,24,'2025-04-17','core_action'), (155,24,'2025-04-21','core_action'), (156,24,'2025-04-28','core_action'),
(157,24,'2025-04-29','core_action'), (158,24,'2025-05-03','core_action'), (159,24,'2025-05-05','core_action'),
(160,24,'2025-05-07','core_action'), (161,24,'2025-05-08','core_action'), (162,24,'2025-05-11','core_action'),
(163,24,'2025-05-15','core_action'), (164,24,'2025-05-27','core_action'), (165,24,'2025-05-28','core_action'),
(166,24,'2025-05-29','core_action'), (167,24,'2025-05-31','core_action'), (168,24,'2025-06-03','core_action'),
(169,24,'2025-06-07','core_action'), (170,24,'2025-06-15','core_action'), (171,24,'2025-06-16','core_action'),
(172,24,'2025-06-22','core_action'), (173,24,'2025-06-23','core_action'), (174,24,'2025-06-24','core_action'),
(175,24,'2025-06-25','core_action'), (176,24,'2025-06-27','core_action'), (177,25,'2025-04-06','core_action'),
(178,25,'2025-04-20','core_action'), (179,25,'2025-04-23','core_action'), (180,25,'2025-05-22','core_action'),
(181,25,'2025-05-25','core_action'), (182,25,'2025-05-28','core_action'), (183,25,'2025-05-31','core_action'),
(184,26,'2025-04-06','core_action'), (185,26,'2025-04-24','core_action'), (186,26,'2025-04-27','core_action'),
(187,26,'2025-06-07','core_action'), (188,26,'2025-06-17','core_action'), (189,26,'2025-06-29','core_action'),
(190,27,'2025-04-16','core_action'), (191,27,'2025-04-22','core_action'), (192,28,'2025-04-02','core_action'),
(193,28,'2025-04-03','core_action'), (194,28,'2025-04-17','core_action'), (195,28,'2025-04-27','core_action'),
(196,29,'2025-04-04','core_action'), (197,29,'2025-04-08','core_action'), (198,29,'2025-05-12','core_action'),
(199,29,'2025-05-21','core_action'), (200,29,'2025-05-28','core_action'), (201,29,'2025-05-29','core_action'),
(202,30,'2025-04-13','core_action'), (203,30,'2025-04-21','core_action'), (204,31,'2025-04-06','core_action'),
(205,31,'2025-04-23','core_action'), (206,31,'2025-05-05','core_action'), (207,31,'2025-05-09','core_action'),
(208,31,'2025-05-27','core_action'), (209,31,'2025-05-28','core_action'), (210,32,'2025-04-04','core_action'),
(211,32,'2025-04-11','core_action'), (212,33,'2025-04-03','core_action'), (213,33,'2025-04-05','core_action'),
(214,33,'2025-04-06','core_action'), (215,33,'2025-04-08','core_action'), (216,33,'2025-06-03','core_action'),
(217,33,'2025-06-04','core_action'), (218,33,'2025-06-11','core_action'), (219,33,'2025-06-27','core_action'),
(220,34,'2025-04-09','core_action'), (221,34,'2025-04-13','core_action'), (222,34,'2025-04-14','core_action'),
(223,34,'2025-04-15','core_action'), (224,34,'2025-04-17','core_action'), (225,34,'2025-04-18','core_action'),
(226,34,'2025-04-21','core_action'), (227,34,'2025-04-25','core_action'), (228,34,'2025-04-30','core_action'),
(229,34,'2025-05-02','core_action'), (230,34,'2025-05-10','core_action'), (231,34,'2025-05-12','core_action'),
(232,34,'2025-05-18','core_action'), (233,34,'2025-06-08','core_action'), (234,34,'2025-06-13','core_action'),
(235,34,'2025-06-15','core_action'), (236,34,'2025-06-16','core_action'), (237,34,'2025-06-24','core_action'),
(238,34,'2025-06-26','core_action'), (239,34,'2025-06-28','core_action'), (240,34,'2025-06-29','core_action'),
(241,35,'2025-04-03','core_action'), (242,35,'2025-04-04','core_action'), (243,35,'2025-04-11','core_action'),
(244,35,'2025-04-19','core_action'), (245,35,'2025-04-22','core_action'), (246,35,'2025-04-23','core_action'),
(247,35,'2025-04-28','core_action'), (248,35,'2025-04-29','core_action'), (249,35,'2025-04-30','core_action'),
(250,35,'2025-05-03','core_action'), (251,35,'2025-05-06','core_action'), (252,35,'2025-05-10','core_action'),
(253,35,'2025-05-19','core_action'), (254,35,'2025-05-23','core_action'), (255,35,'2025-05-30','core_action'),
(256,35,'2025-05-31','core_action'), (257,35,'2025-06-01','core_action'), (258,35,'2025-06-03','core_action'),
(259,35,'2025-06-04','core_action'), (260,35,'2025-06-09','core_action'), (261,35,'2025-06-13','core_action'),
(262,35,'2025-06-15','core_action'), (263,35,'2025-06-17','core_action'), (264,35,'2025-06-18','core_action'),
(265,35,'2025-06-21','core_action'), (266,35,'2025-06-30','core_action'), (267,36,'2025-04-03','core_action'),
(268,36,'2025-04-06','core_action'), (269,36,'2025-04-08','core_action'), (270,36,'2025-04-10','core_action'),
(271,36,'2025-04-13','core_action'), (272,36,'2025-04-15','core_action'), (273,36,'2025-04-19','core_action'),
(274,36,'2025-04-22','core_action'), (275,36,'2025-04-26','core_action'), (276,36,'2025-04-29','core_action'),
(277,36,'2025-05-03','core_action'), (278,36,'2025-05-11','core_action'), (279,36,'2025-05-30','core_action'),
(280,36,'2025-05-31','core_action'), (281,36,'2025-06-03','core_action'), (282,36,'2025-06-05','core_action'),
(283,36,'2025-06-14','core_action'), (284,36,'2025-06-15','core_action'), (285,36,'2025-06-17','core_action'),
(286,36,'2025-06-24','core_action'), (287,36,'2025-06-29','core_action'), (288,37,'2025-05-07','core_action'),
(289,37,'2025-05-09','core_action'), (290,37,'2025-05-25','core_action'), (291,37,'2025-06-03','core_action'),
(292,37,'2025-06-04','core_action'), (293,37,'2025-06-07','core_action'), (294,37,'2025-06-09','core_action'),
(295,38,'2025-05-04','core_action'), (296,38,'2025-06-24','core_action'), (297,39,'2025-05-14','core_action'),
(298,39,'2025-05-19','core_action'), (299,39,'2025-05-23','core_action'), (300,39,'2025-05-29','core_action'),
(301,39,'2025-06-02','core_action'), (302,39,'2025-06-03','core_action'), (303,39,'2025-06-06','core_action'),
(304,39,'2025-06-30','core_action'), (305,40,'2025-05-09','core_action'), (306,40,'2025-05-22','core_action'),
(307,40,'2025-05-30','core_action'), (308,41,'2025-05-10','core_action'), (309,41,'2025-05-11','core_action'),
(310,41,'2025-05-27','core_action'), (311,42,'2025-05-14','core_action'), (312,42,'2025-05-15','core_action'),
(313,42,'2025-05-26','core_action'), (314,42,'2025-06-03','core_action'), (315,42,'2025-06-11','core_action'),
(316,42,'2025-06-20','core_action'), (317,42,'2025-06-24','core_action'), (318,43,'2025-05-25','core_action'),
(319,43,'2025-06-04','core_action'), (320,43,'2025-06-18','core_action'), (321,44,'2025-05-17','core_action'),
(322,44,'2025-05-20','core_action'), (323,44,'2025-05-21','core_action'), (324,44,'2025-05-27','core_action'),
(325,45,'2025-05-16','core_action'), (326,45,'2025-05-19','core_action'), (327,45,'2025-05-26','core_action'),
(328,45,'2025-05-29','core_action'), (329,46,'2025-05-06','core_action'), (330,46,'2025-05-07','core_action'),
(331,46,'2025-05-09','core_action'), (332,46,'2025-05-10','core_action'), (333,46,'2025-05-16','core_action'),
(334,46,'2025-05-21','core_action'), (335,46,'2025-05-24','core_action'), (336,46,'2025-05-25','core_action'),
(337,46,'2025-05-28','core_action'), (338,46,'2025-05-29','core_action'), (339,46,'2025-06-15','core_action'),
(340,47,'2025-05-01','core_action'), (341,47,'2025-05-03','core_action'), (342,47,'2025-05-10','core_action'),
(343,47,'2025-05-12','core_action'), (344,47,'2025-05-14','core_action'), (345,47,'2025-05-20','core_action'),
(346,47,'2025-06-02','core_action'), (347,47,'2025-06-07','core_action'), (348,47,'2025-06-09','core_action'),
(349,47,'2025-06-17','core_action'), (350,47,'2025-06-28','core_action'), (351,48,'2025-05-04','core_action'),
(352,48,'2025-05-05','core_action'), (353,48,'2025-05-11','core_action'), (354,48,'2025-05-14','core_action'),
(355,48,'2025-05-16','core_action'), (356,48,'2025-05-25','core_action'), (357,48,'2025-05-27','core_action'),
(358,48,'2025-05-31','core_action'), (359,48,'2025-06-01','core_action'), (360,48,'2025-06-06','core_action'),
(361,48,'2025-06-14','core_action'), (362,48,'2025-06-17','core_action'), (363,48,'2025-06-18','core_action'),
(364,48,'2025-06-19','core_action'), (365,48,'2025-06-20','core_action'), (366,48,'2025-06-21','core_action');
```

Sanity check — this should print `48` and `366`:

```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM events;
```

### What's baked into this dataset (don't peek before the exercises if you want to discover it yourself)

<details>
<summary>Reveal</summary>

- **Four monthly signup cohorts**: Feb, Mar, Apr, and May 2025, 12 users each. "Today" for every query this week is **2025-06-30** — the observation cutoff.
- **Two segments**: `self_serve` (36 users, the majority — low-touch, product-led) and `sales_assisted` (12 users — onboarded with a human). They behave very differently after signup.
- **An onboarding change shipped 2025-04-01.** Watch whether the Feb/Mar cohorts (pre-change) and Apr/May cohorts (post-change) retain differently. Nobody tells you this in the data — you find it by reading the curve.
- Because the cohorts have different amounts of history as of the cutoff (Feb has ~5 months, May has ~2), the retention **triangle** you build will genuinely be a triangle — some cells simply don't exist yet. That's not a bug; that's what a live retention table always looks like.

</details>

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed data; cohorts + the retention triangle | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | N-day / unbounded / rolling; growth accounting | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Survival curves + DAU/MAU stickiness | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Exercises; start challenges | 0h | 1h | 1.5h | 0.5h | 1h | 0.5h | 4.5h |
| Friday | Challenges; mini-project setup | 0h | 0h | 1.5h | 0.5h | 1h | 1.5h | 4.5h |
| Saturday | Mini-project (dashboard + survival curve + write-up) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **4.5h** | **~27h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-cohorts-and-retention-curves.md](./lecture-notes/01-cohorts-and-retention-curves.md) | Cohorts, the retention triangle in SQL, reading the curve's shape | 2h |
| 2 | [lecture-notes/02-retention-definitions.md](./lecture-notes/02-retention-definitions.md) | N-day vs. unbounded vs. rolling retention; the growth-accounting split | 2h |
| 3 | [lecture-notes/03-survival-and-stickiness.md](./lecture-notes/03-survival-and-stickiness.md) | Survival curves, censoring, DAU/MAU stickiness | 2h |
| 4 | [exercises/exercise-01-build-a-cohort-triangle.md](./exercises/exercise-01-build-a-cohort-triangle.md) | Build the full retention triangle from raw events | 1.5h |
| 5 | [exercises/exercise-02-growth-accounting-split.md](./exercises/exercise-02-growth-accounting-split.md) | New / retained / resurrected / churned, month by month | 1.5h |
| 6 | [exercises/exercise-03-stickiness-ratio.md](./exercises/exercise-03-stickiness-ratio.md) | DAU/MAU by segment and over time | 1h |
| 7 | [challenges/challenge-01-diagnose-a-leaky-bucket.md](./challenges/challenge-01-diagnose-a-leaky-bucket.md) | Find and quantify the leak in one segment | 1.5h |
| 8 | [challenges/challenge-02-compare-retention-across-segments.md](./challenges/challenge-02-compare-retention-across-segments.md) | Self-serve vs. sales-assisted, argued from the numbers | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Cohort-retention dashboard query + survival curve + one-page read | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + further reading | — |

## By the end of this week you can…

- Turn a raw `events` table into a cohort-retention triangle, and explain every cell in it.
- Say, precisely, what "40% retention" means — over what window, measured which way — instead of leaving it ambiguous.
- Split any month's active users into new/retained/resurrected/churned and prove the split sums correctly.
- Look at a retention curve and say "this flattens at X% — durable" or "this is heading to zero — leaky" and defend it with a query.
- Compute a stickiness ratio and know whether it's describing a daily habit or a monthly errand.

## Up next

[Week 5 — LTV, CAC & unit economics](../week-05-ltv-cac-and-unit-economics/) — once you know whether users stick around, you can finally put a dollar value on them.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
