# A/B Testing Analysis Portfolio Project

## Project Overview
This project focuses on evaluating the performance of new user interface configurations and algorithms (Variant B) compared to the baseline system (Variant A) across four independent A/B tests. The goal is to optimize the conversion funnel by analyzing data sliced by device categories and traffic channels, verifying statistical significance, and providing actionable business recommendations.

The analysis consists of:
1. **Data Extraction:** Running standard database syntax and queries to aggregate session-level user interactions from a BigQuery database.
2. **Statistical Analysis:** Implementing an automated segmentation and evaluation function using Python (Pandas & Statsmodels) in Google Colab to run two-sided Z-tests for proportions.
3. **Data Visualization:** Developing dynamic monitoring frameworks and visualization panels.

---

## Data & Resources
* **Analysis Notebook:** [Portfolio Project 2 - Colab](https://colab.research.google.com/drive/1NiUcdnNsq0FWRnamYTzOntisY5TUr_8k#scrollTo=9Ug9qks7wmWC)
* **Interactive Dashboard:** [Tableau Public Dashboard](https://public.tableau.com/app/profile/ihor.petrsyhyn/viz/Portfolio2_17821301696490/Dashboard1?publish=yes)
* **Processed Dataset:** [AB_test_results.csv](https://drive.google.com/file/d/1RvWbN5xmTDkjfsW2tS0_8I7iXF2IZJ4g/view?usp=sharing)

---

## Core Funnel Metrics Evaluated
The automated function evaluates statistical significance ($\alpha = 0.05$) for the following metrics relative to the total number of sessions:
* `add_shipping_info / session`
* `add_payment_info / session`
* `begin_checkout / session`
* `new account / session`

---

## Key Findings & Executive Summary

### Test 1: Highly Successful (Partial Rollout Recommended)
* **Device Segment:** **Desktop** and **Mobile** demonstrated outstanding conversion growth across core metrics (shipping, payment, checkout). Mobile `add_payment_info` was exceptionally strong with a lift of **+17.14%**.
* **Channel Segment:** **Direct** traffic showed stable growth along the entire funnel (`begin_checkout` up by **+14.72%**). However, **Organic Search** experienced a critical drop (e.g., **-19.46%** on payment), which requires separate isolation since organic search accounts for ~34.5% of total traffic.
* **Recommendation:** Deploy Variant B for Desktop and Mobile immediately (covering 98% of user traffic). Run an isolated test for Organic Search traffic and run a technical audit on Tablet layout due to severe conversion drops.

### Test 2: Inconclusive / Neutral (Rollback Recommended)
* **Device Segment:** **Desktop** traffic pulled overall numbers into a slight visual increase (+5.45% to +8.00%), but given the broader neutral picture, this remains statistically insufficient to justify the release.
* **Channel Segment:** **Organic Search** repeated a negative trend, showing a steady decline across the entire funnel (e.g., **-14.28%** on payment). 
* **Recommendation:** Conclude the test as neutral/unsuccessful. Roll back 100% of the traffic to Variant A. Variant B does not bring visible profit and degrades experience for search engine users.

### Test 3: Unsuccessful (Reject Variant B)
* **Device Segment:** **Mobile** was the main driver of the decline, with `begin_checkout` dropping by **-4.70%** and `add_shipping_info` falling by **-2.10%**.
* **Channel Segment:** **Organic Search** (~35.5% of traffic) continuously degraded, dropping by **-8.02%** at the checkout stage.
* **Recommendation:** Completely reject Variant B. The changes significantly harm the checkout funnel on mobile devices and organic channels. Perform a detailed UX/UI audit of the mobile version.

### Test 4: Unsuccessful (Reject Variant B)
* **Device Segment:** **Desktop** traffic (58.43% share) was the main reason for the failure, showing solid drops across the entire funnel: payment (**-7.39%**), shipping (**-6.40%**), and checkout (**-5.69%**). Interestingly, Mobile responded positively (+4.29% checkout lift).
* **Channel Segment:** **Organic Search** sank by **-7.96%** for checkout. **Social Search** dropped significantly by **-20.78%** at the payment step.
* **Recommendation:** Reject Variant B. The interface changes were clearly designed without desktop optimization in mind, introducing heavy visual obstacles or bugs that disrupt the primary conversion journey on larger screens.

---

## Tech Stack & Libraries
* **SQL** (BigQuery) — Raw data aggregation and session formatting
* **Python** (Google Colab Environment)
  * `pandas` & `numpy` — Data manipulation and structuring
  * `statsmodels` — `proportions_ztest` implementation for statistical calculations
  * `matplotlib` & `seaborn` — Visualization and plotting
