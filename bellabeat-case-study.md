# Bellabeat Case Study: How Smart Device Data Can Shape Marketing Strategy

**A Google Data Analytics Capstone Project**

---

## 1. Business Task

Bellabeat, a high-tech wellness company for women, wants to understand how consumers use non-Bellabeat smart fitness devices in order to guide marketing strategy for its own product line (Leaf, Time, Spring, and the Bellabeat app). This analysis explores daily activity and sleep data from FitBit users to surface behavioral trends, then translates those trends into specific, data-backed marketing recommendations for Bellabeat.

**Key stakeholders:** Urška Sršen (Co-founder, Chief Creative Officer), Sando Mur (Co-founder), and the Bellabeat marketing analytics team.

---

## 2. Data Source

This analysis uses the **FitBit Fitness Tracker Data** (Kaggle, CC0: Public Domain, via Mobius) — personal fitness tracker data from FitBit users, including daily activity, steps, and sleep logs, collected across two overlapping date ranges (3/12/2016–4/11/2016 and 4/12/2016–5/12/2016).

**Limitations and credibility notes (ROCCC):**
- **Sample size:** The dataset claims 30 users but actually contains 35 unique IDs in the activity data.
- **Timeframe:** Roughly two months of data from 2016 — dated, and too short to capture seasonal or long-term behavior patterns.
- **No demographic data:** No age, gender, or location fields, which limits how confidently findings can be generalized to Bellabeat's target audience (women).
- **Sleep data coverage:** Only about 30% of activity-days (410 of 1,367) have corresponding sleep data logged, limiting the reliability of sleep-related findings relative to activity-related ones, and Slee data is only present between 4/12/2016–5/12/2016.
- **Self-selected sample:** Users opted in to share their data, which may not represent typical device-user behavior.

Given these limitations, findings here are treated as **directional signals**, not statistically definitive population-level conclusions.

---

## 3. Data Cleaning & Processing

All cleaning and analysis was performed in **Google BigQuery (SQL)**, with verification in **Excel** and visualization in **Tableau**.

Key cleaning steps:

- **Removed exact duplicate rows** from the sleep dataset (3 duplicates found).
- **Resolved a date-range overlap bug:** the two source activity files overlapped on 4/12/2016, creating 48 duplicate `Id + ActivityDate` pairs with conflicting values. Diagnosed as a partial-export artifact; resolved by excluding the overlapping dates from the earlier file and keeping the correct file's version for that date.
- **Converted `SleepDay` from a malformed string/timestamp** (`4/12/2016 12:00:00 AM`) into a proper `DATE` type using `PARSE_DATE`.
- **Validated data integrity:** checked for negative values, confirmed activity-level minutes never exceeded 1,440/day (except where the duplicate-date bug was present, which was then fixed), and checked for null values across both tables.
- **Flagged likely non-wear days** (`TotalSteps = 0 AND Calories = 0`) — ultimately found only 6 of 1,373 days matched this strict definition, likely an undercount (see Finding 5).
- **Joined** cleaned activity and sleep tables via `LEFT JOIN` on `Id` and date, preserving all activity records even when sleep wasn't logged that day.
- **Added a `DayOfWeek` column** for weekday-level analysis.

Full SQL queries are available in the accompanying files / linked repository.

---

## 4. Analysis Summary & Key Findings

### Finding 1 — Most of the day is sedentary; real exercise is a small share of it
Users average **7,441 steps/day** (below the commonly cited 10,000-step benchmark). Of the tracked day: **70.0% sedentary, 13.2% light activity, and just 2.4% combined moderate-to-vigorous activity** (~33.8 min/day — slightly above the CDC's ~30 min/day guideline, but a small fraction of the day overall).

### Finding 2 — Sunday and Tuesday sit at opposite ends of the weekly pattern
**Sunday** shows the highest average sleep (7h33m), lowest steps (6,641), lowest active minutes, and the most time spent awake in bed — consistent with a rest/recovery day. **Tuesday** shows the reverse: highest steps (7,969), lowest sleep (6h45m) — a higher-output, lower-rest day.

### Finding 3 — More sleep is linked to less daytime activity, with a possible "sweet spot"
Steps and sedentary time decline steadily as sleep duration increases. However, **calories burned and vigorous-activity minutes peak at medium sleep (6–7 hrs)** rather than continuing to rise with more sleep — suggesting a possible energy "sweet spot" where moderate rest supports higher-intensity activity better than either too little or too much sleep.

### Finding 4 — No statistically significant relationship between activity level and sleep-logging consistency
A linear regression of average steps against percentage of missing-sleep days showed a slight downward trend, but it was **not statistically significant** (p = 0.14, against a standard threshold of p < 0.05, n = 35 users). This does not provide confident evidence that activity level predicts sleep-logging behavior.

### Finding 5 — Device non-wear could not be reliably measured
Using a strict definition (zero steps and zero calories), only 6 of 1,373 total day-records were flagged as likely non-wear — almost certainly an undercount, since other analysis showed legitimately low-activity days can have zero steps but non-zero calories. This limited the ability to compare wear-consistent vs. inconsistent users.

---

## 5. Visualizations

Interactive dashboard built in Tableau Public:
<img width="1998" height="1598" alt="Key Findings" src="https://github.com/user-attachments/assets/2ea060d6-28d4-4a7c-97e1-130ea865b80c" />

**🔗 [View the full interactive dashboard here]( https://public.tableau.com/views/FitBit_17906549293210/KeyFindings?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link )**

The dashboard includes:
- Average minutes per day by activity level (Sedentary/Light/Moderate/Very Active)
- Steps vs. sleep by day of week
- Activity metrics across sleep-duration buckets
- Activity level vs. sleep-logging consistency (scatter plot with trend line)

---

## 6. Top Recommendations for Bellabeat

### Recommendation 1 — Shift app messaging tone by day of week (Leaf / Time / App)
Data shows Sunday functions as a clear rest day (highest sleep, lowest activity) while Tuesday is the opposite. Rather than sending the same activity-goal messaging every day, the Bellabeat app could adapt its tone based on these patterns — framing Sunday content around recovery and sleep quality, and weekday content (especially early-to-mid week) around activity encouragement. This uses data Leaf and Time already collect; it only requires changing what the app chooses to surface.

### Recommendation 2 — Reposition marketing around light, everyday movement rather than step-count competition (Leaf)
Only 2.4% of the average day is spent in moderate-to-vigorous activity, while light activity (13.2%) and overall non-sedentary time (~3h45m/day) make up a much larger share. Rather than competing on step-count or workout intensity — territory dominated by harder-core fitness brands — Bellabeat could lean into Leaf's lifestyle/wellness positioning by marketing "every bit of movement counts," validating light daily activity as real progress. This plays to Bellabeat's existing brand identity rather than competing head-on with fitness-focused competitors.

### Recommendation 3 — Add a same-day sleep-logging reminder (App)
Sleep data was logged on only ~30% of tracked days, and this gap did not correlate with activity level — meaning it's a broad engagement issue, not isolated to one user segment. A simple, low-cost in-app notification — triggered specifically on days when activity was logged but sleep wasn't — could directly close this gap and improve the completeness of the wellness picture Bellabeat offers its users.

---

## Process & Tools

This project was completed using a **SQL → Excel → Tableau** pipeline:
- **BigQuery (SQL):** data cleaning, joining, and all aggregate analysis
- **Excel:** verification via pivot tables
- **Tableau Public:** final visualizations and dashboard

---

*This case study was completed as part of the Google Data Analytics Professional Certificate capstone project, using the publicly available FitBit Fitness Tracker dataset.*
