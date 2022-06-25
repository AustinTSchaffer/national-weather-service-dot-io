# Data Notes

Notes on the data formats and schemas provided by weather.gov through its API endpoints and its UI.

## Data Format

After selecting your location/zip/whatever, there's a link to an "Hourly Weather Forecast", which looks like:

![](attachments/hourly-weather-forecast-page-weather-dot-gov.png)

No one in their right mind would use this page daily. This page appears to use 100% SSR, but there's a link to the data in an XML format. This URL has the format:

```
https://forecast.weather.gov/MapClick.php?
    lat=<latitute>&
    lon=<longitude>&
    FcstType=digitalDWML
```

`digitalDWML` means "digital Digital Weather Markup Language", which appears to be an XML format with a somewhat insane tabular data format. Unfortunately the weather.gov endpoint doesn't care if you try setting "Accept: text/json,application/json". The endpoint only returns XML.

The DWML XML Schema is defined here: https://graphical.weather.gov/xml/DWMLgen/schema/DWML.xsd

## More Data Formats

Apparently Weather.gov actually does have an API:

- Documentation: https://www.weather.gov/documentation/services-web-api
- Barely any additional info: https://weather-gov.github.io/api/
- GitHub repo: https://github.com/weather-gov/api
- OpenAPI Spec: https://api.weather.gov/openapi.json

It seems like the standard "what's the weather going to be like today/tomorrow?" kind of data/analysis should be provided by the `/gridpoints/{wfo}/{x},{y}` endpoints. There's a few endpoints in this series:

- `GET /gridpoints/{wfo}/{x},{y}`
- `GET /gridpoints/{wfo}/{x},{y}/forecast`
- `GET /gridpoints/{wfo}/{x},{y}/forecast/hourly`
- `GET /gridpoints/{wfo}/{x},{y}/stations`

I don't know what else I would want for a first pass at a weather monitoring app, to be honest.

There's also a collection of endpoints dedicated to getting the current weather at a particular location.

- `GET /stations/{stationId}/observations`
- `GET /stations/{stationId}/observations/latest`
- `GET /stations/{stationId}/observations/{time}`
- `GET /stations`
- `GET /stations/{stationId}`

There's thousands of stations though, so you need to be sure to provide a query string to the `GET /stations` endpoint. Or use `GET /gridpoints/{wfo}/{x},{y}/stations` to get the stations in your gridpoint region.

Since there's so much data to parse through, they have a handy `/points/{point}` endpoint, which returns a meta-object containing the routes to retrieve actual forecast data for the requested `lat,lon`.

```json
// GET https://api.weather.gov/points/38.0311,-78.4742
// Accept: application/geo+json

{
    "@context": { /* [...] */ },
    "id": "https://api.weather.gov/points/38.0311,-78.4742",
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [
            -78.474199999999996,
            38.031100000000002
        ]
    },
    "properties": {
        "@id": "https://api.weather.gov/points/38.0311,-78.4742",
        "@type": "wx:Point",
        "cwa": "LWX",
        "forecastOffice": "https://api.weather.gov/offices/LWX",
        "gridId": "LWX",
        "gridX": 51,
        "gridY": 26,
        "forecast": "https://api.weather.gov/gridpoints/LWX/51,26/forecast",
        "forecastHourly": "https://api.weather.gov/gridpoints/LWX/51,26/forecast/hourly",
        "forecastGridData": "https://api.weather.gov/gridpoints/LWX/51,26",
        "observationStations": "https://api.weather.gov/gridpoints/LWX/51,26/stations",
        "relativeLocation": {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -78.485381000000004,
                    38.037658
                ]
            },
            "properties": {
                "city": "Charlottesville",
                "state": "VA",
                "distance": {
                    "unitCode": "wmoUnit:m",
                    "value": 1220.9395048353001
                },
                "bearing": {
                    "unitCode": "wmoUnit:degree_(angle)",
                    "value": 126
                }
            }
        },
        "forecastZone": "https://api.weather.gov/zones/forecast/VAZ037",
        "county": "https://api.weather.gov/zones/county/VAC540",
        "fireWeatherZone": "https://api.weather.gov/zones/fire/VAZ037",
        "timeZone": "America/New_York",
        "radarStation": "KLWX"
    }
```

It seems that the `GET /gridpoints/LWX/51,26` endpoint contains all data that we might want is the  endpoint.
