# Weather Analysis: Little Rock, AR vs. Wellington, NZ

## Introduction
This report compares the seasonal climates of **Little Rock, Arkansas** and **Wellington, New Zealand** over the last 5 years. 

**Note:** The request specified 'Wellington, NX'. We have assumed this refers to Wellington, NZ (New Zealand).

### Metrics Analyzed
1. **Averages** (Highs/Lows) by Season
2. **Standard Deviation & Variance** (Volatility)
3. **Covariance** (Relationship between locations and daily temperature spreads)
4. **Visualization** of High/Low trends


```python
import requests
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

# Setup Plotting Style
plt.style.use('ggplot')
%matplotlib inline
```

    Matplotlib is building the font cache; this may take a moment.


## 1. Data Ingestion
Fetching daily historical weather data from the **Open-Meteo API** for the last 5 years.


```python
# 1. Configuration
locations = {
    "Little Rock, AR": {"lat": 34.7445, "lon": -92.2880, "hemisphere": "North"},
    "Wellington, NZ":  {"lat": -41.2865, "lon": 174.7762, "hemisphere": "South"}
}

end_date = datetime.now().strftime('%Y-%m-%d')
start_date = (datetime.now() - timedelta(days=5*365)).strftime('%Y-%m-%d')

# 2. Fetch Data Function
def get_weather_data(lat, lon, city, start, end):
    url = "https://archive-api.open-meteo.com/v1/archive"
    params = {
        "latitude": lat,
        "longitude": lon,
        "start_date": start,
        "end_date": end,
        "daily": ["temperature_2m_max", "temperature_2m_min"],
        "timezone": "auto"
    }
    response = requests.get(url, params=params)
    data = response.json()
    
    df = pd.DataFrame({
        'Date': pd.to_datetime(data['daily']['time']),
        'Max_Temp': data['daily']['temperature_2m_max'],
        'Min_Temp': data['daily']['temperature_2m_min']
    })
    df['Avg_Temp'] = (df['Max_Temp'] + df['Min_Temp']) / 2
    df['City'] = city
    return df

# 3. Compile Data
data_frames = []
for city, coords in locations.items():
    print(f"Fetching data for {city}...")
    df = get_weather_data(coords['lat'], coords['lon'], city, start_date, end_date)
    df['Hemisphere'] = coords['hemisphere']
    data_frames.append(df)

weather_df = pd.concat(data_frames).reset_index(drop=True)
weather_df.head()
```

    Fetching data for Little Rock, AR...
    Fetching data for Wellington, NZ...





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Max_Temp</th>
      <th>Min_Temp</th>
      <th>Avg_Temp</th>
      <th>City</th>
      <th>Hemisphere</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-11-19</td>
      <td>20.7</td>
      <td>8.3</td>
      <td>14.50</td>
      <td>Little Rock, AR</td>
      <td>North</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-11-20</td>
      <td>21.5</td>
      <td>11.9</td>
      <td>16.70</td>
      <td>Little Rock, AR</td>
      <td>North</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-11-21</td>
      <td>21.8</td>
      <td>13.0</td>
      <td>17.40</td>
      <td>Little Rock, AR</td>
      <td>North</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-11-22</td>
      <td>16.9</td>
      <td>6.1</td>
      <td>11.50</td>
      <td>Little Rock, AR</td>
      <td>North</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-11-23</td>
      <td>12.8</td>
      <td>2.1</td>
      <td>7.45</td>
      <td>Little Rock, AR</td>
      <td>North</td>
    </tr>
  </tbody>
</table>
</div>



## 2. Seasonal Analysis
We must calculate seasons differently based on the hemisphere.
- **North:** Winter (Dec-Feb), Spring (Mar-May), Summer (Jun-Aug), Fall (Sep-Nov)
- **South:** Summer (Dec-Feb), Fall (Mar-May), Winter (Jun-Aug), Spring (Sep-Nov)


```python
def get_season(row):
    month = row['Date'].month
    hemi = row['Hemisphere']
    
    if month in [12, 1, 2]:
        return 'Winter' if hemi == 'North' else 'Summer'
    elif month in [3, 4, 5]:
        return 'Spring' if hemi == 'North' else 'Fall'
    elif month in [6, 7, 8]:
        return 'Summer' if hemi == 'North' else 'Winter'
    else:
        return 'Fall' if hemi == 'North' else 'Spring'

weather_df['Season'] = weather_df.apply(get_season, axis=1)

# Calculate Aggregates: Mean, Std Dev, Variance
stats = weather_df.groupby(['City', 'Season'])['Avg_Temp'].agg(['mean', 'std', 'var'])

# Reorder for readability
season_order = ['Spring', 'Summer', 'Fall', 'Winter']
stats = stats.reindex(pd.MultiIndex.from_product([weather_df['City'].unique(), season_order], names=['City', 'Season']))

display(stats.style.format("{:.2f}").background_gradient(cmap='coolwarm'))
```


<style type="text/css">
#T_2a8a4_row0_col0 {
  background-color: #dbdcde;
  color: #000000;
}
#T_2a8a4_row0_col1 {
  background-color: #f5a081;
  color: #000000;
}
#T_2a8a4_row0_col2 {
  background-color: #f4c6af;
  color: #000000;
}
#T_2a8a4_row1_col0, #T_2a8a4_row2_col1, #T_2a8a4_row2_col2 {
  background-color: #b40426;
  color: #f1f1f1;
}
#T_2a8a4_row1_col1 {
  background-color: #6f92f3;
  color: #f1f1f1;
}
#T_2a8a4_row1_col2 {
  background-color: #5572df;
  color: #f1f1f1;
}
#T_2a8a4_row2_col0 {
  background-color: #edd2c3;
  color: #000000;
}
#T_2a8a4_row3_col0, #T_2a8a4_row7_col1, #T_2a8a4_row7_col2 {
  background-color: #3b4cc0;
  color: #f1f1f1;
}
#T_2a8a4_row3_col1 {
  background-color: #dc5d4a;
  color: #f1f1f1;
}
#T_2a8a4_row3_col2 {
  background-color: #ea7b60;
  color: #f1f1f1;
}
#T_2a8a4_row4_col0 {
  background-color: #98b9ff;
  color: #000000;
}
#T_2a8a4_row4_col1 {
  background-color: #4e68d8;
  color: #f1f1f1;
}
#T_2a8a4_row4_col2 {
  background-color: #4358cb;
  color: #f1f1f1;
}
#T_2a8a4_row5_col0 {
  background-color: #d6dce4;
  color: #000000;
}
#T_2a8a4_row5_col1 {
  background-color: #4055c8;
  color: #f1f1f1;
}
#T_2a8a4_row5_col2 {
  background-color: #3d50c3;
  color: #f1f1f1;
}
#T_2a8a4_row6_col0 {
  background-color: #b6cefa;
  color: #000000;
}
#T_2a8a4_row6_col1 {
  background-color: #5b7ae5;
  color: #f1f1f1;
}
#T_2a8a4_row6_col2 {
  background-color: #4a63d3;
  color: #f1f1f1;
}
#T_2a8a4_row7_col0 {
  background-color: #7597f6;
  color: #f1f1f1;
}
</style>
<table id="T_2a8a4">
  <thead>
    <tr>
      <th class="blank" >&nbsp;</th>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_2a8a4_level0_col0" class="col_heading level0 col0" >mean</th>
      <th id="T_2a8a4_level0_col1" class="col_heading level0 col1" >std</th>
      <th id="T_2a8a4_level0_col2" class="col_heading level0 col2" >var</th>
    </tr>
    <tr>
      <th class="index_name level0" >City</th>
      <th class="index_name level1" >Season</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_2a8a4_level0_row0" class="row_heading level0 row0" rowspan="4">Little Rock, AR</th>
      <th id="T_2a8a4_level1_row0" class="row_heading level1 row0" >Spring</th>
      <td id="T_2a8a4_row0_col0" class="data row0 col0" >17.26</td>
      <td id="T_2a8a4_row0_col1" class="data row0 col1" >5.31</td>
      <td id="T_2a8a4_row0_col2" class="data row0 col2" >28.20</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row1" class="row_heading level1 row1" >Summer</th>
      <td id="T_2a8a4_row1_col0" class="data row1 col0" >27.64</td>
      <td id="T_2a8a4_row1_col1" class="data row1 col1" >2.61</td>
      <td id="T_2a8a4_row1_col2" class="data row1 col2" >6.82</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row2" class="row_heading level1 row2" >Fall</th>
      <td id="T_2a8a4_row2_col0" class="data row2 col0" >18.78</td>
      <td id="T_2a8a4_row2_col1" class="data row2 col1" >6.60</td>
      <td id="T_2a8a4_row2_col2" class="data row2 col2" >43.55</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row3" class="row_heading level1 row3" >Winter</th>
      <td id="T_2a8a4_row3_col0" class="data row3 col0" >7.12</td>
      <td id="T_2a8a4_row3_col1" class="data row3 col1" >6.02</td>
      <td id="T_2a8a4_row3_col2" class="data row3 col2" >36.27</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level0_row4" class="row_heading level0 row4" rowspan="4">Wellington, NZ</th>
      <th id="T_2a8a4_level1_row4" class="row_heading level1 row4" >Spring</th>
      <td id="T_2a8a4_row4_col0" class="data row4 col0" >12.96</td>
      <td id="T_2a8a4_row4_col1" class="data row4 col1" >2.12</td>
      <td id="T_2a8a4_row4_col2" class="data row4 col2" >4.51</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row5" class="row_heading level1 row5" >Summer</th>
      <td id="T_2a8a4_row5_col0" class="data row5 col0" >16.85</td>
      <td id="T_2a8a4_row5_col1" class="data row5 col1" >1.91</td>
      <td id="T_2a8a4_row5_col2" class="data row5 col2" >3.67</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row6" class="row_heading level1 row6" >Fall</th>
      <td id="T_2a8a4_row6_col0" class="data row6 col0" >14.71</td>
      <td id="T_2a8a4_row6_col1" class="data row6 col1" >2.32</td>
      <td id="T_2a8a4_row6_col2" class="data row6 col2" >5.38</td>
    </tr>
    <tr>
      <th id="T_2a8a4_level1_row7" class="row_heading level1 row7" >Winter</th>
      <td id="T_2a8a4_row7_col0" class="data row7 col0" >10.82</td>
      <td id="T_2a8a4_row7_col1" class="data row7 col1" >1.81</td>
      <td id="T_2a8a4_row7_col2" class="data row7 col2" >3.28</td>
    </tr>
  </tbody>
</table>



## 3. Covariance Analysis
1. **Location Covariance:** Comparing the Avg Temps of the two cities. (Expected: Negative, as seasons are opposite).
2. **Daily Range Covariance:** Comparing Highs vs Lows within the same city.


```python
# Cross-Location Covariance
pivot = weather_df.pivot(index='Date', columns='City', values='Avg_Temp')
loc_cov = pivot.cov()

print("Covariance: Little Rock vs Wellington (Avg Temp)")
display(loc_cov)

print("\nCovariance: Daily Highs vs Lows (Within City)")
for city in locations.keys():
    subset = weather_df[weather_df['City'] == city]
    cov_val = np.cov(subset['Max_Temp'], subset['Min_Temp'])[0][1]
    print(f"{city}: {cov_val:.2f}")
```

    Covariance: Little Rock vs Wellington (Avg Temp)



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>City</th>
      <th>Little Rock, AR</th>
      <th>Wellington, NZ</th>
    </tr>
    <tr>
      <th>City</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Little Rock, AR</th>
      <td>81.483648</td>
      <td>-18.957354</td>
    </tr>
    <tr>
      <th>Wellington, NZ</th>
      <td>-18.957354</td>
      <td>9.132370</td>
    </tr>
  </tbody>
</table>
</div>


    
    Covariance: Daily Highs vs Lows (Within City)
    Little Rock, AR: 79.08
    Wellington, NZ: 8.35


## 4. 5-Year High/Low Graph
Visualizing the temperature trends.


```python
plt.figure(figsize=(14, 6))

# Resample to Monthly Mean for cleaner visualization
monthly = weather_df.groupby(['City', pd.Grouper(key='Date', freq='M')]).agg({'Max_Temp':'mean', 'Min_Temp':'mean'}).reset_index()

colors = {'Little Rock, AR': 'red', 'Wellington, NZ': 'blue'}

for city in locations.keys():
    subset = monthly[monthly['City'] == city]
    plt.plot(subset['Date'], subset['Max_Temp'], label=f'{city} High', color=colors[city], linestyle='-')
    plt.plot(subset['Date'], subset['Min_Temp'], label=f'{city} Low', color=colors[city], linestyle='--', alpha=0.6)

plt.title('Avg Monthly Highs & Lows (Last 5 Years)')
plt.ylabel('Temperature (°C)')
plt.legend()
plt.show()
```


```python

```
