# Architecture

Creating a human readable application over this data.

Currently considering the following components:

- Grafana to view the timeseries data
    - Temp
    - Humidity
    - Precipitation
    - Wind
    - https://grafana.com/docs/grafana/next/setup-grafana/
- Optional: An API layer that proxies requests to `api.weather.gov`
    - This component will be so the app can retrieve/store cache results, and avoid hammering api.weather.gov.
    - This may be additionally important in the event Grafana can't parse the timestamp format that weather.gov sends back, which is the most important part in timeseries data.
- Optional: A caching database.
    - Redis or TimescaleDB with a retention policy.
- Super Optional: A simple web frontend that sits in front of grafana so it can easily grab your lat/lon from the browser, and/or provide a place to enter your location information. This is obviously a stretch goal.

Notes
- If we can somehow trigger an async endpoint to deposit the requisite data into a postgres-compatible database, we could have Grafana read from that database. That would mean we won't need to worry about the Grafana plugin that allows it to read API requests.
- Having a database sucks when you're trying to reduce maintenance.
