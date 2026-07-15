# Serverless analytics engineering: Tracking Toronto's bike share network live 

### Summary

I built a fully automated, serverless pipeline that captures live data from Toronto's bike share network every 2 hours. It ingests roughly 12,500 rows a day from a live city API into Google BigQuery, models them with version controlled SQL in Dataform, and powers a live dashboard tracking availability across 1,000+ stations.

![Bike Share dashboard](assets/bikeshare_dashboard_one.png)

- [Live Data Studio dashboard](https://lookerstudio.google.com/reporting/ee66c00f-c017-4a8d-9249-72a67b1ba3a6)
- [GitHub repository](https://github.com/justinduckett/toronto-bikeshare-pipeline)

### Why I built this

I built this project to run analytics engineering practices against live external data. I used a public API that reports only the current moment and forgets everything else. Three priorities shaped the build:

- **Pipelines, not exports.** The system ingests, models, and serves data on a fixed schedule with no manual steps.
- **Treating data as code.** Every SQL transformation is version controlled in Git, with automated assertions that halt the pipeline before bad data can reach the dashboard. 
- **Turning fleeting data into a real asset.** The bike share API only shows the current moment. By capturing snapshots every 2 hours, the pipeline turns fleeting data into a historical record you can actually analyze.

### How it works

I structured the project around the standard components of modern data infrastructure, following the ELT (Extract, Load, Transform) approach. The raw data lands in the warehouse first, then is transformed after.

![Bike Share dashboard](assets/bikeshare_architecture_diagram.png)

**1. Data source.** Toronto's bike share network publishes live station data through a public API using the global GBFS standard, the same format used by bike share systems worldwide. It reports how many bikes and docks each of the city's 1,000+ stations has right now.

**2. Ingestion.** A Python script fetches the live feeds, merges bike counts with station locations, and loads the result into the warehouse. It runs serverlessly on Google Cloud Run Functions, woken every 2 hours by Cloud Scheduler.

**3. Storage.** Google BigQuery acts as the data warehouse. Raw snapshots land untouched in an append only table, building the historical record. A scheduled cleanup keeps a rolling 30 day window, which keeps storage inside the free tier.

**4. Transformation and modeling.** This is the analytics engineering core. Google Cloud Dataform turns the raw snapshots into clean, purpose built tables using version controlled SQL. Each table answers one question: 

- bike availability by hour of day
- the live status of every station,
- each station's reliability over 30 days.

Every model carries automated assertions, tests that run on every execution and halt the pipeline if the data violates expectations, like duplicate stations or missing values.

![Bike Share dashboard](assets/bikeshare_dataform_dag.png)

**5. Orchestration.** Ingestion and transformation run in a fixed sequence every 2 hours: data lands at 14 minutes past the hour, and the SQL models run at 20 past. The transformation code itself is compiled into a release once daily at midnight. This is a deliberate separation between how often the data refreshes and how often the code changes. If a data quality test fails, the run stops and the dashboard keeps serving the last good data instead of broken numbers.

**6. Analytics and visualization.** A three page Data Studio dashboard serves the results, organized by time: a live network map refreshed every 2 hours, recent 7 day trends with week over week comparisons, and 30 day station reliability rankings.

![Bike Share dashboard](assets/bikeshare_dashboard_two.png)

### Problems I solved along the way

The pipeline broke a few times during the build:

- **A timezone bug that only showed up in the charts.** The hourly chart displayed incorrect odd hours. The cause was a hardcoded timezone offset hidden in a dashboard calculated field, which was wrong half the year because of daylight saving. I fixed it by moving the data transformation out of the dashboard and into the transformation workflow. I stored timestamps in UTC and converted to local time once, in the SQL modeling layer.
- **Code that was merged but not deployed.** New models stopped appearing in scheduled runs. The cause was Dataform's release configuration, which compiles code snapshots on its own schedule, running stale code. Debugging it taught me the difference between code being merged and code being live, a distinction every production data system has.

### Results

- **Zero touch automation.** The pipeline runs 12 cycles a day without manual intervention, capturing 1,000+ stations per cycle, roughly 12,500 rows daily, into a rolling window of about 380,000 records.
- **Trustworthy by design.** All 6 SQL models carry automated assertions that test data quality on every run, and every chart description was verified against what the SQL actually computes.
- **Operational insight.** The reliability analysis identifies chronically empty stations. At the time of writing, 10 stations sat empty more than 80% of the time, a prioritized target list for rebalancing crews.
- **Zero cost.** The full system runs at $0.00 per month on Google Cloud free tier resources.

![Bike Share dashboard](assets/bikeshare_dashboard_three.png)

### Example: the station reliability model

One SQL model calculates each station's "stockout rate," the share of time it sits completely empty, and groups stations into reliability bands:

```sql
WITH station_stats AS (
  SELECT
    station_id,
    ANY_VALUE(name) AS name,
    AVG(CASE WHEN num_bikes_available = 0 THEN 1.0 ELSE 0.0 END) AS stockout_rate
  FROM ${ref("status_history")}
  WHERE name IS NOT NULL
  GROUP BY station_id
)

SELECT
  *,
  CASE
    WHEN stockout_rate = 0 THEN '0%'
    WHEN stockout_rate <= 0.10 THEN '1-10%'
    WHEN stockout_rate <= 0.50 THEN '10-50%'
    WHEN stockout_rate < 1.0 THEN '50-99%'
    ELSE 'Always empty'
  END AS reliability_bucket
FROM station_stats
```

The `AVG(CASE WHEN ...)` pattern turns each snapshot into a 1 (empty) or 0 (has bikes), so the average is the fraction of time the station sat empty. Grouping by `station_id` rather than station name keeps the logic safe even if two stations ever share a name.

### Links

- [Live Data Studio dashboard](https://lookerstudio.google.com/reporting/ee66c00f-c017-4a8d-9249-72a67b1ba3a6)
- [GitHub repository](https://github.com/justinduckett/toronto-bikeshare-pipeline)
