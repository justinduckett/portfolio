# **Fintech data engineering: Detecting high-value banking fraud**

### **Summary**

I built a fully automated fraud detection pipeline that simulates how a bank monitors transactions for suspicious activity. Every morning it generates simulated banking transactions mixed with two realistic fraud patterns. It loads them into Google BigQuery, scores every transaction against version controlled detection rules in dbt, and serves the results to a live triage dashboard built for fraud investigators. The system runs hands off on a daily schedule, tests its own data quality on every build, documents every column in a generated data dictionary, monitors its own freshness, and costs $0.00 per month.

> **[Screenshot 1: The daily triage page (page 1 of the dashboard), showing the flagged transaction feed with evidence columns and the daily trend by type. This is the strongest visual hook, lead with it.]**

[**View the live dashboard**](https://lookerstudio.google.com/s/kWp8xAUTeTU) | [**View the code on GitHub**](https://github.com/justinduckett/banking-fraud-monitor)

### **Why I built this**

I built this project to apply analytics engineering practices to the fraud and risk domain. Three priorities shaped the build:

- **Engineering features, not just reports.** Using SQL window functions to calculate behavioral metrics that fraud detection actually looks for. This included customer spending baselines and transaction velocity. These models form a small feature store, the same pattern fraud data science teams use to feed both rules and machine learning models.

- **Putting business logic in tested code, not in a BI tool.** Every fraud feature I worked on lives in version controlled SQL with automated tests and documentation, so the pipeline, the dashboard, and any future consumer all work from one definition.

- **Designing for the investigator, not the chart.** Fraud rules produce false positives by design. The dashboard treats flags as review priorities rather than verdicts, and shows the evidence behind every flag.

### **How it works**

I structured the project around the standard components of modern data infrastructure, following the ELT approach: land raw data in the warehouse first, then transform it there.

**1. Data source.** Real banking data is private, so a Python script simulates a bank's daily transaction feed: everyday spending across 20 customers and 9 merchant categories, seeded with two fraud signatures. The first fraud feature is a 'high amount spike' far above a customer's normal spending. The second is a 'velocity attack' of 5 to 9 small purchases packed into a 45 minute window, to mimic a fraudster validating a stolen card before big spending. The two patterns look completely different in the data, which is the point: different fraud types need different detectors.

**2. Ingestion.** A GitHub Actions workflow runs the python script every morning and appends roughly 100 new transactions to BigQuery. The script reads configuration from environment variables, logs with timestamps, and alerts failures. If an upload errors, the exception is re raised so the run turns red and sends an alert instead of quietly passing.

**3. Storage.** Google BigQuery acts as the warehouse. Raw transactions land untouched in an append only table with its own documentation and quality tests, so any problem can be traced cleanly to either ingestion or transformation.

**4. Transformation and modeling.** This is the analytics engineering core of the project. dbt Cloud rebuilds the models daily: a staging layer applies explicit type casts as a data contract, and a mart layer computes the fraud features with SQL window functions. The detection rules live in the model too. 

- Transactions over 5x the customer's 30 day baseline are flagged as high amount spikes.
- More than 3 transactions in an hour flags a velocity attack.

Every row receives a single classification the dashboard filters on. Every build runs a suite of dbt tests, including an accepted values test that halts the pipeline if the classification logic ever produces a category the dashboard does not expect. Every column is also documented in a data dictionary.

> **[Screenshot 2: The dbt documentation page for the fraud_features model, showing column descriptions and test indicators. This is the shot that shows engineering rigor to a technical reviewer.]**

**5. Orchestration.** Ingestion and transformation run in a fixed daily sequence. 

- The GitHub Actions workflow runs around 7:00 UTC (2am EST) each day and ingests new data.
- The dbt job builds at 14:00 UTC (9am EST), a seven hour buffer.

The buffer exists because I ran into the ingestion ordering failure firsthand. At one point the transform ran before ingestion, and the dashboard silently served stale data. Two schedules with a gap is workable at this scale, but it is not a true orchestrator. This project is relying on Cron scheduling and my repo documents exactly what that tradeoff means.

**6. Analytics and visualization.** A two page Looker Studio dashboard serves the results, and is organized by an investigator's workflow. A daily triage page lists every flagged transaction with the evidence behind it (amount, spending baseline, transactions in the past hour). A pattern analysis page shows where risk concentrates across customers and merchant categories. A freshness indicator also shows exactly how current the data is at a glance.

> **[Screenshot 3: The category mix chart from page 2, showing normal categories as a wall of blue and the high risk categories dominated by flags. The two fraud signatures are visibly different patterns.]**

### **Problems I solved along the way**

Real pipelines break in instructive ways. Three examples from this build:

- **A pipeline that died silently.** GitHub disables scheduled workflows after 60 days of repository inactivity. Nothing anywhere reported a problem and I diagnosed it from the data itself. I updated the project so errors now fail loudly, and the dashboard monitors its own freshness.

- **Updating a credential broke the pipeline.** Rotating the BigQuery service account key took down dbt Cloud, because three systems held copies of the old key and I only updated two. Credential rotation now starts with an inventory of every consumer.

- **A leaky baseline that weakened detection where risk is highest.** The 30 day average originally included the transaction being scored, so a large fraud inflated the very baseline it was compared against. The fix was a one word change to the window frame, and it is the same leakage problem machine learning teams guard against in feature engineering.

### **Results**

- **Zero touch automation.** The pipeline generates, ingests, transforms, tests, and publishes daily without manual intervention.

- **Trustworthy by design.** Every model carries automated tests that run on every build, from primary key checks to logical assertions, and every column is documented in a browsable data dictionary generated by dbt.

- **Both fraud signatures detected.** The dashboard shows spike victims and card testing attacks as visibly different events with different evidence, and the category mix chart makes false positives and false negatives visible rather than hiding them.

- **Zero cost.** The full system runs at $0.00 per month on free tier resources across GitHub, BigQuery, dbt Cloud, and Looker Studio.

### **Example: the spending baseline feature**

One window function computes each customer's 30 day average spend, the baseline the spike rule compares against:

```
avg(amount) over (
    partition by customer_id
    order by unix_seconds(transaction_at)
    range between 2592000 preceding and 1 preceding
) as avg_spend_30_days
```

An improtant detail I included is `1 preceding` instead of `current row`. This helps make sure the baseline deliberately excludes the transaction being scored. Without that, a $5,000 fraud gets averaged into the spending history it is judged against, diluting the signal exactly when it matters most. The velocity feature next to it keeps `current row` on purpose, because a transaction is part of the burst it belongs to.

### **What I'd tell you about it in five minutes**

This pipeline failed in three quiet ways while I built and maintained it. The GitHub Action stopped running, credentials broke and the orchestration had a scheduling gap. Each failure became a fix you can see in the finished pipeline. There are now loud errors, freshness monitoring, sequenced schedules, and documentation that is honest about what cron scheduling cannot do. The dashboard is the visible part, but the mistakes and fixes I made while building the pipeline are where I learned the most.

### **Links**

- [Live dashboard](https://lookerstudio.google.com/s/kWp8xAUTeTU)
- [GitHub repository](https://github.com/justinduckett/banking-fraud-monitor)






