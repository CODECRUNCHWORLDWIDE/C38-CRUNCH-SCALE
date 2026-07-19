# Lecture 1 — Why Metrics, and the North Star

> **Duration:** ~2 hours. **Outcome:** You can define what a metric is, explain why a growth team needs exactly one north-star metric, apply the five-question test for picking a good one, and derive the input metrics that actually move it — for StreakLab and for any product you're handed.

Every growth team eventually asks the same question in a room full of disagreeing people: *"are we winning?"* Without a metric, that question has no answer — only opinions. This lecture is about picking the **one number** that answers it honestly, and the smaller numbers underneath it that tell you *why* it's moving.

## 1. What a metric actually is

A **metric** is a number, computed the same way every time, that summarizes something about your business or your users. That definition has three parts worth taking seriously:

- **A number.** Not a feeling, not "engagement seems up." A metric is something you can put in a `SELECT` statement and get the identical answer twice.
- **Computed the same way every time.** If "active user" means something different in this week's dashboard than last week's, you don't have a metric — you have a story someone is telling with numbers. **A metric needs a written definition** (a SQL query is the best possible definition, because it's unambiguous and re-runnable).
- **Summarizes something real.** A metric is a lossy compression of thousands of raw events into one number. The compression is the whole value — and the whole danger, because compression always throws information away. Growth work is largely about *which* information you're comfortable losing.

Put concretely: "revenue" is not a metric until you've decided whether it includes refunds, whether it's booked or recognized, and what currency and time zone it's in. Once you've decided and written it as a query, it's a metric.

## 2. Why one north star, not a dashboard of forty numbers

A **dashboard** answers "what's happening." A **north-star metric (NSM)** answers a sharper question: *if this single number goes up, is the company definitely getting healthier?* It's the one metric a whole company — not just growth — organizes around.

Why not just track everything? Two failure modes, both common:

1. **Forty metrics, zero focus.** If every team can point to *some* metric that's up, nobody has to explain the ones that are down. A north star forces a single, shared definition of progress that overrides local optimization.
2. **The wrong single metric (usually revenue).** Picking pure revenue as the north star is tempting and often wrong *for the growth team specifically*, because revenue is a **lagging** and **manipulable** signal — you can hit a quarterly revenue number by discounting hard, running one huge enterprise deal, or delaying refunds, all while the actual product is getting worse for typical users. A good north star is a **leading indicator of durable value delivered**, which revenue eventually follows.

### The five-question test for a north-star candidate

A candidate metric earns the job only if it passes all five:

| # | Question | Why it matters |
|--:|----------|-----------------|
| 1 | **Does it reflect real value delivered to the customer**, not just value captured by the company? | "Ad impressions served" is company-value; "problems solved" is customer-value. Customer-value metrics compound; company-value-only metrics can be gamed against the customer. |
| 2 | **Is it a leading indicator**, not a lagging one? | Revenue, churn, and NPS all show up *after* the damage or the win. A north star should move first and predict those. |
| 3 | **Can a team actually move it** through product and growth work — this week, not this decade? | "GDP of the country we operate in" is real and important and useless as a north star: nobody on the team can influence it. |
| 4 | **Is it hard to game in isolation?** | If one team can inflate the number without creating real value (spam invites to pump "referrals sent"), it will get gamed — not out of malice, but because people optimize what's measured. |
| 5 | **Does everyone in the company understand it in one sentence**, without a glossary? | "Weekly Engaged Users" (defined once, clearly) beats "Composite Growth Index (proprietary weighted blend)" every time. If the CEO can't explain it to a new hire in 30 seconds, it will not survive contact with a board meeting. |

### Famous north stars, and why they were chosen

- **Facebook (early years):** "7 friends in 10 days" — not DAU. DAU is a lagging *result* of retention; "7 friends in 10 days" was the discovered **activation threshold** after which retention became dramatically more likely. It was a leading indicator the team could directly influence through onboarding.
- **Airbnb:** **nights booked** — not signups, not listings created. Signups and listings are supply-side vanity; nights booked is the moment value is actually exchanged between a guest and a host.
- **Spotify:** **time spent listening** — not song plays (gameable by counting 3-second skips) and not subscriber count alone (a lagging, low-frequency number that hides whether people are actually enjoying the product between renewal dates).
- **Slack:** **messages sent between teammates** (an early proxy) evolved toward measures of teams reaching sustained daily usage — because a single signup with zero teammates invited is not "adopted," it's "curious."

Notice the pattern: none of these is revenue, and none of these is a raw count of signups. All are close to the moment the product actually delivers on its promise.

## 3. Picking a north star for StreakLab

StreakLab is a habit-tracking app: users create habits, log daily check-ins, and can upgrade to a $9/month Pro tier. What should its north star be? Walk the five-question test against three candidates:

| Candidate | Q1 real value? | Q2 leading? | Q3 movable? | Q4 hard to game? | Q5 simple? | Verdict |
|---|---|---|---|---|---|---|
| **Total signups (all time)** | No — signing up delivers no value yet | No — cumulative counts only ever go up | Partially (marketing can move it) | No — trivially inflated by paid acquisition of low-intent users | Yes | **Rejected: vanity** |
| **Monthly Recurring Revenue (MRR)** | Partially — money changed hands, but that's company-value | No — lagging; a subscription can renew for months after someone stopped actually using the habit tracker | Yes | Somewhat (discounting games it short-term) | Yes | **Rejected as *the* NSM — track it, but as a lagging outcome metric, not the star** |
| **Weekly Engaged Users** (users logging **3+ check-ins in the trailing 7 days**) | Yes — you can't fake building a streak; the value (the habit forming) is the metric | Yes — engaged users convert to Pro and stick around; disengaged users churn regardless of what MRR says this month | Yes — onboarding, reminders, and streak design all move this directly | Hard to game — you can't check in 3 times without actually opening the app and using the core feature | Yes — one sentence, no glossary needed | **Selected north star** |

**StreakLab's north star for the rest of this course: Weekly Engaged Users (WEU)** — the count of distinct users with **3 or more `checkin_logged` events in the trailing 7 days**. The "3+" threshold isn't arbitrary — it's StreakLab's analogue of Facebook's "7 friends in 10 days": the activation depth past which a user is meaningfully forming a habit, not just poking at the app once.

A quick sanity query against the seed data — how many users are engaged as of the last day we have data for (2026-06-21)?

```sql
SELECT COUNT(*) AS weekly_engaged_users
FROM (
    SELECT user_id
    FROM events
    WHERE event_name = 'checkin_logged'
      AND event_time >= TIMESTAMP '2026-06-21' - INTERVAL '6 days'
      AND event_time <  TIMESTAMP '2026-06-21' + INTERVAL '1 day'
    GROUP BY user_id
    HAVING COUNT(*) >= 3
) engaged;
```

That returns **10** — out of 15 total signups. That gap (10 engaged vs. 15 signed up) is the entire point of choosing WEU over total signups: it's telling you the truth that 5 people signed up and are *not* forming a habit, a fact a signups-only dashboard would hide completely.

## 4. From north star to input metrics: building the tree

A north star alone doesn't tell a team *what to build this week*. You get there by decomposing it into **input metrics** — smaller numbers that mechanically combine (usually by multiplication or funnel narrowing) to produce the north star, each one small enough for a specific team to own.

The decomposition for StreakLab's WEU:

```
Weekly Engaged Users
  = New Activated Users (this week)
      × [conversion from "activated" to "engaged" within the week]
  + Returning Engaged Users (activated in a prior week, still checking in 3+ times)
```

Which unpacks one level further into metrics a team can actually work on:

| Input metric | Definition | Which team owns moving it |
|---|---|---|
| **Signups** | New rows in `users` per week | Marketing / acquisition (Week 2) |
| **Activation rate** | % of new signups with a `habit_created` event within 24 hours | Onboarding / product (Week 3) |
| **Check-in frequency** | Avg. `checkin_logged` events per activated user per week | Core product / notifications |
| **Week-2 retention** | % of an activated cohort still logging 3+ check-ins one week later | Retention / lifecycle (Week 4) |
| **Reactivation rate** | % of previously-engaged users who lapsed and came back | Lifecycle / re-engagement |

Notice what this table buys you: a marketer moving signups from 50/week to 100/week does **not** by itself move the north star if activation rate is 20% — the team's honest next question becomes "should we spend on more signups, or fix activation first?" That question is unanswerable without the tree. This is Week 1's whole purpose: turn "grow the business" from a vibe into a numbered, ownable list.

### A worked check: is StreakLab's funnel signups-bottlenecked or activation-bottlenecked?

```sql
SELECT
    u.user_id,
    u.signup_channel,
    MAX(CASE WHEN e.event_name = 'habit_created' THEN 1 ELSE 0 END) AS activated
FROM users u
LEFT JOIN events e ON e.user_id = u.user_id
GROUP BY u.user_id, u.signup_channel
ORDER BY u.user_id;
```

Aggregate that (you'll do the full version in Exercise 3) and you'll find **12 of 15 users (80%) activated** — the 3 who didn't are all on the same behavioral pattern: they open the app once (`session_start`) and never create a habit. That's an **activation** problem, not an acquisition problem — a signal you could never see from a "total signups" chart, but that falls straight out of a two-line join against raw events.

## 5. Check yourself

- In your own words, what three things does a number need to actually be a "metric" (not just a statistic someone mentioned once)?
- Why is picking pure revenue as a growth team's north star usually a mistake, even though revenue obviously matters?
- Walk "Total signups" through the five-question test out loud. Which question does it fail hardest, and why?
- What does "leading indicator" mean, and name one lagging metric StreakLab should still track even though it isn't the north star?
- Explain StreakLab's north-star definition (Weekly Engaged Users) precisely enough that someone could write the `WHERE` clause from your description alone, with no other context.
- Why does a north star need input metrics underneath it instead of standing alone?

If those are solid, Lecture 2 takes the vague idea of "the customer journey" and turns it into five precise, queryable stages: the AARRR funnel.

## Further reading

- **Sean Ellis & Morgan Brown, *Hacking Growth*, Ch. 2** — the original "North Star Metric" framing (library/purchase; see `resources.md` for a free companion talk).
- **Reforge — "How to Choose a North Star Metric":** referenced in `resources.md`.
- **PostgreSQL — Date/Time functions** (you'll need `INTERVAL` arithmetic constantly this course): <https://www.postgresql.org/docs/current/functions-datetime.html>
