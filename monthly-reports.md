## Automating Executive Insights: Ontario.ca Analytics Reporting

### Summary:

I created an automated monthly analytics report for Ontario’s flagship website, Ontario.ca, to provide senior executives with data-driven insights into digital performance. The report combines quantitative metrics from Google Analytics 4 (GA4) with written storytelling and context. This helps leadership teams quickly understand key trends, public interests, and the impact of digital initiatives.

It is distributed monthly to senior audiences, including the Premier’s Office, Ontario.ca Executive Table, and Directors of Communications.

### Tools used:

- **Python and Google Colab** – for data extraction, cleaning, and transformation
- **GA4 API** – to query analytics data directly from Google Analytics
- **Google Sheets** – to store, organize, and connect data outputs
- **Looker Studio** – to design interactive dashboards and compile the final report
- **pandas and NumPy** – for data manipulation and calculations

### Purpose of the work:

The goal of the project was to **streamline executive reporting** and make digital insights more actionable across the Ontario government. Previously, data had to be manually gathered and summarized, which limited the complexity of analysis and made it harder to highlight emerging trends.

This new reporting process:
- Automates monthly data collection from GA4
- Enables complex queries that aren’t possible in the GA4 interface
- Reduces manual effort and ensures consistent visualizations month to month
- Provides clear, narrative insights that connect analytics to real-world events, marketing activity, and new digital launches

### Steps taken to build the reports:

**1. Data Pipeline Development**

I built Python scripts in Google Colab to connect directly to the GA4 API, allowing for flexible, advanced queries without relying on BigQuery. The scripts pull raw data, clean and transform it, calculate metrics such as month-over-month percentage change, and then automatically update a linked Google Sheet.

**2. Data Visualization and Reporting Framework**

Looker Studio was linked to the Google Sheet to visualize metrics through charts, scorecards, and trend lines. By maintaining a consistent data schema, I could refresh the visuals each month without needing to rebuild the dashboard.

**3. Written Insights and Storytelling**

I authored the narrative content directly in Looker Studio, combining visual data with written analysis to explain trends, seasonal shifts, or the impact of major announcements and campaigns.

**4. Key Report Features**

- **New & Notable Page:** Highlights a new feature or service launched on Ontario.ca each month. This section showcases impact, user adoption, and next steps — helping raise visibility for the Digital and Data team’s work across government.
- **Trending Pages Analysis:** Displays which Ontario.ca pages are rising or falling in popularity compared to the previous month. Insights often tie to current events, seasonal demand, or active marketing campaigns, developed through collaboration with product and communications teams.

### Results and impact

This project modernized how executive stakeholders consume digital performance insights. It helped ensure that data-informed storytelling became a regular part of strategic discussions and decision-making.

Key outcomes include:
- **16 hour reduction** in manual data preparation time each month
- **60% improvement** in data accuracy and reporting consistency
- Enhanced awareness across senior leadership of key public interests and digital priorities
- Regular positive feedback from managers on the “New & Notable” section for effectively showcasing team impact

Overall, the report has become a trusted and anticipated deliverable each month. One that blends **technical automation, data visualization, and strategic communication** to tell the story of how Ontarians interact with government online.
