# 🧪 A/B Testing Analytics: Checkout Funnel Optimization & Z-Test Evaluation

## 📌 Project Overview
This project focuses on evaluating the performance of new user interface configurations and algorithms (Variant B) compared to the baseline system (Variant A) across four independent A/B tests. The goal is to optimize the conversion funnel by analyzing data sliced by device categories and traffic channels, verifying statistical significance, and providing actionable business recommendations.

### 🔬 Core Workflow & Methodology
1. **Data Extraction:** Running standard database syntax and queries to aggregate session-level user interactions from a BigQuery database.
2. **Statistical Analysis:** Implementing an automated segmentation and evaluation function using Python (Pandas & Statsmodels) in Google Colab to run two-sided Z-tests for proportions.
3. **Data Visualization:** Developing dynamic monitoring frameworks and visualization panels.

## 📁 Quick Links

* 📄 **Data for Testing:** [Google Drive - Query Data Folder](https://drive.google.com/file/d/1NX2a7ePRxbuzYX0bwHa-FKvk-YijHqjg/view?usp=sharing)
* 🐍 **Python Analytics & Statistical Testing:** [`Portfolio_Project_2.ipynb`](https://github.com/IhorPetryshyn/A-B-Testing-Analytics-Checkout-Funnel-Optimization-Z-Test-Evaluation/blob/main/Portfolio_Project_2.ipynb)
* 🗄️ **Colab Source Notebook:** [Google Colab Link](https://colab.research.google.com/drive/1IVlGJwd2mlzuZjR_PrvfMBRFJDplsESn)
* 📊 **Interactive Dashboard:** [Tableau Public Dashboard](https://public.tableau.com/app/profile/ihor.petrsyhyn/viz/Portfolio2_17821301696490/Dashboard1?publish=yes)
* 💾 **Processed Dataset:** [AB_test_results.csv](https://drive.google.com/file/d/1RvWbN5xmTDkjfsW2tS0_8I7iXF2IZJ4g/view?usp=sharing)

---

## BigQuery Data Extraction Script

Before executing the A/B testing statistical pipeline in Python, session log data was extracted and merged from BigQuery using the following SQL query:

<details>
    
<summary>🔍 Click to expand BigQuery Data Extraction SQL Query</summary>
    
```sql
with session_info as (
SELECT
    date,
    sp.country,
    s.ga_session_id,
    sp.device,
    sp.continent,
    sp.channel,
    ab.test,
    ab.test_group,
from `DA.session_params` sp
join `DA.ab_test` ab
on sp.ga_session_id = ab.ga_session_id
join `DA.session` s
on s.ga_session_id = sp.ga_session_id
),


events as(
select
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group,
    event_name,
    count (ep.ga_session_id) as session_with_events
from session_info
join `DA.event_params` ep
on ep.ga_session_id = session_info.ga_session_id
group by
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group,
    event_name
),
session as(
select
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group,
    count(distinct ga_session_id) as session_cnt
from session_info


group by
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group
),


orders as(
select
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group,
    count(distinct o.ga_session_id) as session_with_orders
from session_info
join `DA.order` o
on o.ga_session_id = session_info.ga_session_id
group by
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group
),


accounts as(
select
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group,
    count(distinct acs.ga_session_id) as new_account_cnt


from `DA.account_session` acs
join session_info
on session_info.ga_session_id = acs.ga_session_id
group by
    session_info.date,
    session_info.country,
    session_info.device,
    session_info.continent,
    session_info.channel,
    session_info.test,
    session_info.test_group
)


Select    
    events.date,
    events.country,
    events.device,
    events.continent,
    events.channel,
    events.test,
    events.test_group,
    event_name,
    session_with_events as value,


from events


UNION ALL


Select    
    orders.date,
    orders.country,
    orders.device,
    orders.continent,
    orders.channel,
    orders.test,
    orders.test_group,
    'session with orders' as event_name,
    orders.session_with_orders as value,


from orders


UNION ALL


Select    
    accounts.date,
    accounts.country,
    accounts.device,
    accounts.continent,
    accounts.channel,
    accounts.test,
    accounts.test_group,
    'new account' as event_name,
    accounts.new_account_cnt as value
from accounts


UNION ALL


Select    
    session.date,
    session.country,
    session.device,
    session.continent,
    session.channel,
    session.test,
    session.test_group,
    'session' as event_name,
    session.session_cnt as value
from session;
```

</details>
    
---

## 📐 Statistical Framework & Funnel Metrics

### Two-Proportion $Z$-Test Formula
To determine whether conversion rate differences between Variant A ($p_A$) and Variant B ($p_B$) were statistically significant, we evaluated the pooled proportion standard error:

$$Z = \frac{\hat{p}_B - \hat{p}_A}{\sqrt{\hat{p}(1 - \hat{p}) \left(\frac{1}{n_A} + \frac{1}{n_B}\right)}}$$

### Evaluated Funnel Metrics
Significance testing ($\alpha = 0.05$) was computed relative to total session volume across 4 key steps:
* 🛒 `add_shipping_info` / `session`
* 💳 `add_payment_info` / `session`
* 🏁 `begin_checkout` / `session`
* 👤 `new_account` / `session`

---

## 🔑 Key Findings & Executive Summary

### Test 1: Highly Successful (Partial Rollout Recommended)
* **Device Segment:** **Desktop** and **Mobile** demonstrated outstanding conversion growth across core metrics (shipping, payment, checkout). Mobile `add_payment_info` was exceptionally strong with a lift of **+17.14%**.
* **Channel Segment:** **Direct** traffic showed stable growth along the entire funnel (`begin_checkout` up by **+14.72%**). However, **Organic Search** experienced a critical drop (e.g., **-19.46%** on payment), which requires separate isolation since organic search accounts for ~34.5% of total traffic.
* **Recommendation:** Deploy Variant B for Desktop and Mobile immediately (covering 98% of user traffic). Run an isolated test for Organic Search traffic and run a technical audit on Tablet layout due to severe conversion drops.

<img width="1199" height="799" alt="Dashboard 1" src="https://github.com/user-attachments/assets/bdd1d10b-7fd4-4f18-882f-daee55895cc5" />

---

### Test 2: Inconclusive / Neutral (Rollback Recommended)
* **Device Segment:** **Desktop** traffic pulled overall numbers into a slight visual increase (+5.45% to +8.00%), but given the broader neutral picture, this remains statistically insufficient to justify the release.
* **Channel Segment:** **Organic Search** repeated a negative trend, showing a steady decline across the entire funnel (e.g., **-14.28%** on payment). 
* **Recommendation:** Conclude the test as neutral/unsuccessful. Roll back 100% of the traffic to Variant A. Variant B does not bring visible profit and degrades experience for search engine users.

<img width="1199" height="799" alt="Dashboard 2" src="https://github.com/user-attachments/assets/e7f66c53-705d-4f96-afec-484536d41ef1" />

---

### Test 3: Unsuccessful (Reject Variant B)
* **Device Segment:** **Mobile** was the main driver of the decline, with `begin_checkout` dropping by **-4.70%** and `add_shipping_info` falling by **-2.10%**.
* **Channel Segment:** **Organic Search** (~35.5% of traffic) continuously degraded, dropping by **-8.02%** at the checkout stage.
* **Recommendation:** Completely reject Variant B. The changes significantly harm the checkout funnel on mobile devices and organic channels. Perform a detailed UX/UI audit of the mobile version.

<img width="1199" height="799" alt="Dashboard 3" src="https://github.com/user-attachments/assets/5b4b8edd-f841-4dbd-9e47-4bd87d8da040" />

---

### Test 4: Unsuccessful (Reject Variant B)
* **Device Segment:** **Desktop** traffic (58.43% share) was the main reason for the failure, showing solid drops across the entire funnel: payment (**-7.39%**), shipping (**-6.40%**), and checkout (**-5.69%**). Interestingly, Mobile responded positively (+4.29% checkout lift).
* **Channel Segment:** **Organic Search** sank by **-7.96%** for checkout. **Social Search** dropped significantly by **-20.78%** at the payment step.
* **Recommendation:** Reject Variant B. The interface changes were clearly designed without desktop optimization in mind, introducing heavy visual obstacles or bugs that disrupt the primary conversion journey on larger screens.

<img width="1199" height="799" alt="Dashboard 4" src="https://github.com/user-attachments/assets/bcf8411b-05a7-49a0-a399-f55d56420189" />

---

## Tech Stack & Libraries
* **SQL** (BigQuery) — Raw data aggregation and session formatting
* **Python** (Google Colab Environment)
  * `pandas` & `numpy` — Data manipulation and structuring
  * `statsmodels` — `proportions_ztest` implementation for statistical calculations
  * `matplotlib` & `seaborn` — Visualization and plotting
