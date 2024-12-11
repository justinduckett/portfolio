## Alcohol Map

### Summary:

During an LCBO strike in July 2024, the Ontario government rolled out a searchable and interactive map to help consumers find outlets to buy beer, wine, cider and spirits. For this project I was responsible for geocoding over 6,000 address locations for the map, as well as setting up web analytics and reporting.

_Tools: JupyterLab, Python, Google Maps API, Google Tag Manager, Google Analytics 4, Looker Studio_

![Alcoholic beverage map](assets/alcohol map.PNG)

### Questions Answered:

- Is there a way we can automate the geocoding of addresses for the map, rather than doing in manually?
- How many users are visiting the map and viewing specific retailer locations?
- Can users navigate the map easily? (eg searching, using filters or directly clicking pins)
- What traffic sources are driving users to the map and what percent of those users are engaging with the content?

### Steps:

#### Geocoding data
In order to build the map our team was provided a list of over 6,000 alcohol retailer addresses in Ontario. These addresses did not include data on coordinates, so I used Jupyter Notebooks and Python to write a script calling the Google Maps API. The script provides the API a csv list of addresses and returns their latitude and longitude coordinates in a separate csv file. 

The geocoding python script can be viewed on GitHub here.

#### Setting up web analytics
To set up web analytics I provided developers javascript code snippets to add to the map. I set up event tags in Tag Manager, configured event parameters in GA4 and built a dashboard visualizing key data in Looker Studio. Screenshots of the dashboard can be viewed below.

![Dashboard page 1](assets/dashboard screenshot 1.PNG)

![Dashboard page 2](assets/dashboard screenshot 2.PNG)

![Dashboard page 3](assets/dashboard screenshot 3.PNG)
### Results:

- I successfully integrated the Google Maps Geocoding API to accurately add latitude and longitude coordinates for over 6000 addresses, ensuring precise location data for improved user experience.
- Over 450,000 users visited the map and 3 million location pins were clicked in 2024. The map had a very high engagement rate over 65%.
- Web analytics data also provided insights on product features, including how users were filtering for location results most often.

### Media Coverage:

- News Release: [Let’s Make it an Ontario-Made Summer!](https://news.ontario.ca/en/release/1004813/lets-make-it-an-ontario-made-summer)
- CBC News: [Ford rolls out map to help find booze retailers amid LCBO strike](https://www.cbc.ca/news/canada/toronto/online-map-alcohol-sales-ontario-lcbo-strike-1.7257144)
- CTV News: [Ontario launches interactive map of retail locations selling alcohol during LCBO strike](https://toronto.ctvnews.ca/ontario-launches-interactive-map-of-retail-locations-selling-alcohol-during-lcbo-strike-1.6955317)
