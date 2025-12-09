## End-to-End Serverless Data Pipeline: Toronto Bike Share Analytics

### Summary 
This project involved designing a fully automated, serverless data pipeline to capture real-time transit data from the Toronto Parking Authority. It moves away from manual data extraction to a cloud-native architecture, building a historical data warehouse in Google BigQuery and visualizing network performance in a live dashboard.

### Tools used

- **Data Ingestion:** Python (Pandas, Requests, Pandas-GBQ)
- **Automation:** GitHub Actions (YAML Configuration, Cron Scheduling)
- **Cloud Warehousing:** Google BigQuery (Serverless SQL)
- **Visualization:** Looker Studio (Geospatial Mapping & Time-Series Analysis)

### Purpose of the work 
I built this project to expand my technical analytics skillset with modern data engineering. Specifically, I wanted to learn how to:

- **Automate data collection:** Moving away from manual exports to building "set and forget" pipelines using Python and cloud tools.
- **Work with cloud infrastructure:** Gaining hands-on experience with Google BigQuery and serverless architecture without incurring high costs.
- **Solve real-world data problems:** Taking raw, transient API data and transforming it into a permanent historical record for trend analysis.

### Key project phases

**1. ETL Pipeline Development:** 

I wrote a Python script to handle the full data lifecycle. 

- Extracting live station data (status and metadata) from the Toronto Bike Share GBFS API. 
- Transforming it by merging JSON feeds, enriching inventory counts with geospatial coordinates, and normalizing timestamps. 
- Loading the clean, structured data into Google BigQuery with automated schema handling.

**2. Serverless Orchestration:** 

I replaced traditional, resource-heavy schedulers (like Airflow) with GitHub Actions. I configured a YAML-based cron schedule to execute the pipeline hourly, utilizing GitHub Secrets to securely manage cloud authentication credentials.

[Screenshot Idea: Add a screenshot of your GitHub Actions log showing a successful pipeline run (Green Checkmarks).]

**3. Data Warehousing Strategy:** 

I configured the BigQuery table to support schema evolution, ensuring the database automatically adapts to new data fields from the API without manual intervention. I implemented timestamp normalization logic to standardize irregular data arrival times into clean hourly buckets, optimizing the warehouse for time-series analysis.

[Screenshot Idea: Add a screenshot of your BigQuery Schema showing the columns (station_id, lat, lon, num_bikes_available) or a preview of the populated table.]

**4 Advanced Analytics & Visualization:** 

I wrote custom SQL queries to perform aggregate filtering that native BI tools like Looker Studio could not handle, such as creating histograms of station failure rates. I developed calculated fields to identify and filter out 'Zombie Stations' (inactive/offline units) to focus analysis on active fleet performance.

[Screenshot Idea: Add a screenshot of your Network Reliability Map or the Station Failure Histogram from Looker Studio.]

**SQL Example:** 

In order to identify 'at-risk' Stations, I wrote custom SQL to categorize stations based on their failure frequency. This query segments the network into performance buckets. This helps group stations that are completely empty from those that are critically failing but operational.

```
SELECT
  CASE
    WHEN stockout_rate = 1.0 THEN '100% (Inactive)'
    WHEN stockout_rate >= 0.9 THEN '90% - 99% (Critical)'
    WHEN stockout_rate >= 0.8 THEN '80% - 90%'
    WHEN stockout_rate >= 0.7 THEN '70% - 80%'
    ELSE '0% - 70% (Healthy)'
  END as reliability_bucket,
  COUNT(*) as station_count
FROM (
  SELECT
    name,
    AVG(CASE WHEN num_bikes_available = 0 THEN 1.0 ELSE 0.0 END) as stockout_rate
  FROM
    `toronto-bikeshare-analytics.bike_data.status_history`
  GROUP BY
    name
)
GROUP BY 1
ORDER BY 1
```
### Results and impact

- **100% Automation:** Successfully deployed a "set and forget" pipeline that ingests data 24/7 with zero manual intervention, ensuring continuous data availability.
- **Cost Optimization:** Architected the entire solution to run for $0.00/month by leveraging the Free Tier limits of Google Cloud and GitHub Actions, demonstrating high-value delivery with zero infrastructure cost.
- **Operational Insight:** The analysis revealed that ~8% of the network consists of inactive 'Zombie Stations'. Furthermore, it identified the Top 10 active stations with the highest failure rates, providing a prioritized target list for operational rebalancing teams to improve service reliability.

### Links

- <a href="{{ 'https://github.com/justinduckett/toronto-bikeshare-pipeline' | relative_url }}" target="_blank" rel="noopener noreferrer">
  Link to the Bike Share Toronto GitHub Repository
</a>

- <a href="{{ 'https://lookerstudio.google.com/s/tGmMFc8I_jU' | relative_url }}" target="_blank" rel="noopener noreferrer">
  Link to the Bike Share Toronto Looker Studio report
</a>




