## Automated Fraud Detection Engine: Fintech Transaction Analytics

### Summary
This project is a fully automated system designed to detect financial fraud in real-time. I built a pipeline that takes raw banking transaction data, cleans it, and checks it for suspicious patterns like users spending unusually high amounts. The final result is a live dashboard that alerts fraud analysts to high-risk activity the moment it happens.

### Tools Used

- Coding & Logic: _Python, SQL_
- Data Warehouse: _Google BigQuery_
- Automation: _GitHub Actions_
- Data Transformation: _dbt (Data Build Tool)_
- Dashboard: _Looker Studio_

### Purpose of the Work

I built this project to expand my technical analytics skillset with modern data engineering. Specifically, I wanted to:

- Build robust pipelines: Moving away from manual exports to building “set and forget” pipelines using Python and cloud tools.
- Implement software best practices: Treating data as code by using dbt to version-control my SQL logic and automate data quality testing.
- Engineer Risk Models: Moving beyond simple reporting to build new features (like "Velocity Attacks") that can flag fraud as it happens.

### Key Project Phases

**1. Creating the Data (Simulation)**

Since I couldn't use real private banking data, I wrote a Python script to generate realistic "fake" data.

- I programmed the script to behave like a real bank ledger, creating thousands of normal transactions mixed with hidden "fraud attacks".
- I built the system to generate new data every single day, mimicking a live business environment.

**2. Analyzing the Data (Logic & Rules)**
   
I used dbt (data build tool) to transform raw data, run tests and generate documentation.

- Feature Engineering: Calculated complex rolling-window metrics, such as velocity_last_hour and avg_spend_30_days, to identify anomalies.
- Automated Testing: configured tests to run on every build, ensuring amount is never negative and transaction_id is always unique before the data hits the dashboard.
- Documentation: Generated a self-documenting data dictionary that defines every column and risk metric for stakeholders.

**3. Automating the Process (Orchestration)**

I set up the entire project to run on autopilot using GitHub Actions.

- Every morning at 8:00 AM, the system wakes up, generates the new day's data, processes the fraud rules, and updates the dashboard.
- This proves I can build reliable, "set-it-and-forget-it" pipelines that don't require manual maintenance.

**4. Building the Dashboard (Visualization)**

I designed a clean, simple tool for Fraud Analysts to use.

- Instead of a cluttered report, I built a "Feed" that highlights high-risk transactions in red.
- This allows an analyst to log in, instantly see who the top 5 suspicious customers are, and take action to freeze their accounts.

### SQL Example

This is a snippet of the code I wrote to catch 'Velocity Fraud'. It tells the database: "Look at every transaction, look back one hour, and count how many times this specific customer has appeared."

```
SQL
SELECT
    customer_id,
    transaction_at,
    -- Count how many times this customer used their card in the last 1 hour
    COUNT(transaction_id) OVER (
        PARTITION BY customer_id
        ORDER BY unix_seconds(transaction_at)
        RANGE BETWEEN 3600 PRECEDING AND CURRENT ROW
    ) as velocity_last_hour
FROM
    raw_transactions
```

### Results and Impact

- Actionable Intelligence: Transformed raw logs into a prioritized "Investigator Feed," reducing the time it takes to identify a bot attack from hours to seconds.
- Automated Data Quality: Integrated dbt tests caught 100% of schema mismatches during development, preventing "silent failures" in the reporting layer.
- Cost Efficiency: Delivered a fully automated, daily-updating pipeline for $0.00/month by leveraging serverless tiers of BigQuery and GitHub Actions.
- Production-Grade Reliability: Established a "Dev" vs. "Prod" environment strategy, ensuring that experimental code never breaks the live dashboard.

### Links

- <a href="{{ 'https://github.com/justinduckett/banking-fraud-monitor' | relative_url }}" target="_blank" rel="noopener noreferrer">
  Link to the Banking Fraud Monitor GitHub Repository
</a>

- <a href="{{ 'https://lookerstudio.google.com/s/kWp8xAUTeTU' | relative_url }}" target="_blank" rel="noopener noreferrer">
  Link to the Live Dashboard
</a>





