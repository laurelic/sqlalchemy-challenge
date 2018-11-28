# SQL Alchemy Challenge

![hawaiipic](Resources/wild-away-589644-unsplash.jpg)

Photo by [Wild & Away](https://unsplash.com/photos/wTgk34xyTgg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/hawaii?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## The Challenge
In order to plan a trip to Hawaii, use SQL Alchemy to query a weather database for historical data. The goal is to survey typical weather patterns during the desired dates.

### Step 1: Precipitation Analysis
Queries 12 months of precipitation data and visualizes rainfall in a line graph.

### Step 2: Station Analysis
Review the stations in the database and visuaize the temperature observations from the station with the highest number of observations.

### Step 3: Temperature Analysis
Plot minimum, average, and max temperature for the date range on a bar chart with error bar.

### Step 4: Daily Normals
Calculate the year-over-year daily normal avg, min, and max temperature for the date range and visualize in a stacked area chart.

```python
%matplotlib notebook
from matplotlib import style
style.use('fivethirtyeight')
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
```


```python
import numpy as np
import pandas as pd
```


```python
import datetime as dt
```

# Reflect Tables into SQLAlchemy ORM


```python
# Python SQL toolkit and Object Relational Mapper
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
```


```python
engine = create_engine("sqlite:///Resources/hawaii.sqlite")
```


```python
# reflect an existing database into a new model
Base = automap_base()
# reflect the tables
Base.prepare(engine, reflect=True)
```


```python
# We can view all of the classes that automap found
Base.classes.keys()
```




    ['measurement', 'station']




```python
# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
```


```python
# Create our session (link) from Python to the DB
session = Session(engine)
```

# Exploratory Climate Analysis


```python
#get the column names and datatypes for the measurement table
md = sqlalchemy.MetaData()
measure_table = sqlalchemy.Table('measurement', md, autoload=True, autoload_with=engine)
measure_columns = measure_table.c
for c in measure_columns:
    print(c.name, c.type)
```

    id INTEGER
    station TEXT
    date TEXT
    prcp FLOAT
    tobs FLOAT
    


```python
#get the column names and datatypes for the station table
station_table = sqlalchemy.Table('station', md, autoload=True, autoload_with=engine)
station_columns = station_table.c
for c in station_columns:
    print(c.name, c.type)
```

    id INTEGER
    station TEXT
    name TEXT
    latitude FLOAT
    longitude FLOAT
    elevation FLOAT
    


```python
# Prompt: Design a query to retrieve the last 12 months of precipitation data and plot the results
# Step 1: Calculate the date 1 year ago from today
# FYI - the latest date in the db is 8.24.17, so the following queries are based around that max date
# Here is the query I would write to base it on today: ly = dt.today() - dt.timedelta(days=365)

#find the most recent date in the dataset by ordering according the set and selecting first object
ty = session.query(Measurement.date).order_by(Measurement.date.desc()).first()[0]

#convert this point into a datetime object
tyd = dt.datetime.strptime(ty, '%Y-%m-%d')

#subtract 365 days from this date in order to select a year previous
ly = tyd - dt.timedelta(days=365)

#convert the LY number back into a string in order to communicate with the db accordingly
lys = ly.strftime('%Y-%m-%d')
```


```python
# Step 2: Perform a query to retrieve the data and precipitation scores according to TY/LY in Step 1

year_history = session.query(Measurement.date, Measurement.prcp).\
    filter(Measurement.date > lys).\
    order_by(Measurement.date).all()
```


```python
# Step 3: Save the query results as a Pandas DataFrame and set the index to the date column

precip = pd.DataFrame(year_history)
#convert the 'date' column to datetime in order to parse over the year
precip['date'] = precip['date'].astype('datetime64[ns]')
#group by date in order to clean up the data
precip_df = pd.DataFrame(precip.groupby(['date']).agg({'prcp':'sum'}))
precip_df.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>prcp</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-08-24</th>
      <td>9.33</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.54</td>
    </tr>
    <tr>
      <th>2016-08-26</th>
      <td>0.10</td>
    </tr>
    <tr>
      <th>2016-08-27</th>
      <td>0.32</td>
    </tr>
    <tr>
      <th>2016-08-28</th>
      <td>3.10</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Step 4: Use Pandas Plotting with Matplotlib to plot the data and rotate the xticks for the dates

fig, ax = plt.subplots(1, 1)

line1, = ax.plot(precip_df.index, precip_df['prcp'], color='thistle')
ax.legend(['Precipitation'])
plt.title(f"Precipitation From {lys} To {ty}", fontsize=14)
plt.xlabel("Date")
plt.ylabel("Preciptation Level")

ax.set_xlim(ly, tyd)
ax.format_xdata = mdates.DateFormatter('%Y-%m-%d')
plt.xticks(rotation=25)

for item in ([ax.xaxis.label, ax.yaxis.label] + ax.get_xticklabels() + ax.get_yticklabels()):
    item.set_fontsize(8)

plt.show()
```

![precip](Resources/preciprange.png)



```python
# Prompt: Use Pandas to calcualte the summary statistics for the precipitation data
summ_df = pd.DataFrame(precip_df['prcp'].describe())
summ_df
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>prcp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>365.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.974164</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1.776466</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.050000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.080000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>14.280000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Prompt: How many stations are available in this dataset?
stations = (session.query(func.count(Measurement.station.distinct()))).scalar()
print(stations)
```

    9
    


```python
# Prompt: What are the most active stations? List the stations and the counts in descending order.
station_activity = engine.execute('SELECT station, COUNT(station) FROM measurement GROUP BY station ORDER BY COUNT(station) DESC')
for s in station_activity:
    print(s)
```

    ('USC00519281', 2772)
    ('USC00519397', 2724)
    ('USC00513117', 2709)
    ('USC00519523', 2669)
    ('USC00516128', 2612)
    ('USC00514830', 2202)
    ('USC00511918', 1979)
    ('USC00517948', 1372)
    ('USC00518838', 511)
    


```python
# Using the station id from the previous query, calculate the lowest temperature recorded, 
# highest temperature recorded, and average temperature most active station?
min_temp = session.query(func.min(Measurement.tobs)).filter_by(station='USC00519281').scalar()
max_temp = session.query(func.max(Measurement.tobs)).filter_by(station='USC00519281').scalar()
avg_temp = session.query(func.avg(Measurement.tobs)).filter_by(station='USC00519281').scalar()
print(f'Min Temp: {min_temp}, Max Temp: {max_temp}, Avg Temp: {avg_temp}')
```

    Min Temp: 54.0, Max Temp: 85.0, Avg Temp: 71.66378066378067
    


```python
# Choose the station with the highest number of temperature observations.
# Query the last 12 months of temperature observation data for this station and plot the results as a histogram

#pull the data for USC00519281 into a DataFrame
station_df = pd.read_sql('SELECT * FROM measurement WHERE station=\'USC00519281\' AND date>=(?)', engine.connect(), params=(lys,))
station_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>station</th>
      <th>date</th>
      <th>prcp</th>
      <th>tobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>14608</td>
      <td>USC00519281</td>
      <td>2016-08-23</td>
      <td>1.79</td>
      <td>77.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14609</td>
      <td>USC00519281</td>
      <td>2016-08-24</td>
      <td>2.15</td>
      <td>77.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>14610</td>
      <td>USC00519281</td>
      <td>2016-08-25</td>
      <td>0.06</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>14611</td>
      <td>USC00519281</td>
      <td>2016-08-26</td>
      <td>0.01</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>14612</td>
      <td>USC00519281</td>
      <td>2016-08-27</td>
      <td>0.12</td>
      <td>75.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Choose the station with the highest number of temperature observations.
# Query the last 12 months of temperature observation data for this station and plot the results as a histogram

fig, ax = plt.subplots()

ax.hist(station_df['tobs'], color='thistle', ec='grey', alpha=0.7)
ax.legend(['Temperature'])
plt.title(f"Temperature Frequency From {lys} To {ty}", fontsize=14)
plt.xlabel("Temperature (F)")
plt.ylabel("Frequency (days)")

for item in ([ax.xaxis.label, ax.yaxis.label] + ax.get_xticklabels() + ax.get_yticklabels()):
    item.set_fontsize(8)

plt.show()
```
![temp](Resources/tempfreq.png)

```python
# Write a function called `calc_temps` that will accept start date and end date in the format '%Y-%m-%d' 
# and return the minimum, average, and maximum temperatures for that range of dates
def calc_temps(start_date, end_date):
    """TMIN, TAVG, and TMAX for a list of dates.
    
    Args:
        start_date (string): A date string in the format %Y-%m-%d
        end_date (string): A date string in the format %Y-%m-%d
        
    Returns:
        TMIN, TAVE, and TMAX
    """
    
    return session.query(func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)).\
        filter(Measurement.date >= start_date).filter(Measurement.date <= end_date).all()
print(calc_temps('2012-02-28', '2012-03-05'))
```

    [(62.0, 69.57142857142857, 74.0)]
    


```python
# Use your previous function `calc_temps` to calculate the tmin, tavg, and tmax 
# for your trip using the previous year's data for those same dates.
trip = calc_temps('2017-01-15', '2017-01-25')
print(trip)
```

    [(66.0, 71.6086956521739, 80.0)]
    


```python
# Plot the results from your previous query as a bar chart. 
# Use "Trip Avg Temp" as your Title
# Use the average temperature for the y value
# Use the peak-to-peak (tmax-tmin) value as the y error bar (yerr)

fig, ax = plt.subplots()
ax.bar("",trip[0][1], yerr=(trip[0][2]-trip[0][0]), alpha=0.7, color='thistle', ecolor='grey')
plt.title('Trip Avg Temp', fontsize=14)
plt.ylabel('Temperature (F)')

for item in ([ax.yaxis.label] + ax.get_yticklabels()):
    item.set_fontsize(8)
plt.show()
```

![avg](Resources/avgtemp.png)



```python
# Calculate the rainfall per weather station for your trip dates using the previous year's matching dates.
# Sort this in descending order by precipitation amount and list the station, name, latitude, longitude, and elevation

sel = [Station.station, Station.name, func.sum(Measurement.prcp), Station.latitude, Station.longitude, Station.elevation]
rainfall = session.query(*sel).filter(Measurement.station == Station.station).filter(Measurement.date >= '2017-01-15').filter(Measurement.date <= '2017-01-25').group_by(Station.station).order_by(Measurement.prcp.desc()).all()

for r in rainfall:
    print(r)
```

    ('USC00516128', 'MANOA LYON ARBO 785.2, HI US', 6.220000000000001, 21.3331, -157.8025, 152.4)
    ('USC00519281', 'WAIHEE 837.5, HI US', 1.07, 21.45167, -157.84888999999998, 32.9)
    ('USC00513117', 'KANEOHE 838.1, HI US', 0.4, 21.4234, -157.8015, 14.6)
    ('USC00519397', 'WAIKIKI 717.2, HI US', 0.23, 21.2716, -157.8168, 3.0)
    ('USC00519523', 'WAIMANALO EXPERIMENTAL FARM, HI US', 0.22999999999999998, 21.33556, -157.71139, 19.5)
    ('USC00514830', 'KUALOA RANCH HEADQUARTERS 886.9, HI US', 0.02, 21.5213, -157.8374, 7.0)
    ('USC00517948', 'PEARL CITY, HI US', 0.0, 21.3934, -157.9751, 11.9)
    

## Optional Challenge Assignment


```python
# Create a query that will calculate the daily normals 
# (i.e. the averages for tmin, tmax, and tavg for all historic data matching a specific month and day)

def daily_normals(date):
    """Daily Normals.
    
    Args:
        date (str): A date string in the format '%m-%d'
        
    Returns:
        A list of tuples containing the daily normals, tmin, tavg, and tmax
    
    """
    
    sel = [func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)]
    return session.query(*sel).filter(func.strftime("%m-%d", Measurement.date) == date).all()
    
daily_normals("01-01")
```




    [(62.0, 69.15384615384616, 77.0)]




```python
# calculate the daily normals for your trip
# push each tuple of calculations into a list called `normals`

# Set the start and end date of the trip
s_date = dt.date(2018, 1, 15)
e_date = dt.date(2018, 1, 25)
```


```python
# Use the start and end date to create a range of dates
trip_dates = []

while s_date <= e_date:
    trip_dates.append(s_date)
    s_date += dt.timedelta(days=1)

print(trip_dates)
```

    [datetime.date(2018, 1, 15), datetime.date(2018, 1, 16), datetime.date(2018, 1, 17), datetime.date(2018, 1, 18), datetime.date(2018, 1, 19), datetime.date(2018, 1, 20), datetime.date(2018, 1, 21), datetime.date(2018, 1, 22), datetime.date(2018, 1, 23), datetime.date(2018, 1, 24), datetime.date(2018, 1, 25)]
    


```python
# Stip off the year and save a list of %m-%d strings
normals = []

for day in trip_dates:
    d = day.strftime('%m-%d')
    temp = daily_normals(d)[0]
    d_stat = {'date': day, 'tmin': temp[0], 'tavg': temp[1], 'tmax': temp[2]}
    normals.append(d_stat)
print(normals)
```

    [{'date': datetime.date(2018, 1, 15), 'tmin': 56.0, 'tavg': 69.31372549019608, 'tmax': 78.0}, {'date': datetime.date(2018, 1, 16), 'tmin': 54.0, 'tavg': 68.62962962962963, 'tmax': 80.0}, {'date': datetime.date(2018, 1, 17), 'tmin': 61.0, 'tavg': 69.07407407407408, 'tmax': 76.0}, {'date': datetime.date(2018, 1, 18), 'tmin': 57.0, 'tavg': 68.63157894736842, 'tmax': 77.0}, {'date': datetime.date(2018, 1, 19), 'tmin': 60.0, 'tavg': 68.26315789473684, 'tmax': 78.0}, {'date': datetime.date(2018, 1, 20), 'tmin': 61.0, 'tavg': 68.86666666666666, 'tmax': 78.0}, {'date': datetime.date(2018, 1, 21), 'tmin': 61.0, 'tavg': 70.14545454545454, 'tmax': 76.0}, {'date': datetime.date(2018, 1, 22), 'tmin': 60.0, 'tavg': 69.26415094339623, 'tmax': 76.0}, {'date': datetime.date(2018, 1, 23), 'tmin': 57.0, 'tavg': 69.50909090909092, 'tmax': 79.0}, {'date': datetime.date(2018, 1, 24), 'tmin': 58.0, 'tavg': 68.76271186440678, 'tmax': 78.0}, {'date': datetime.date(2018, 1, 25), 'tmin': 61.0, 'tavg': 67.94915254237289, 'tmax': 75.0}]
    


```python
# Load the previous query results into a Pandas DataFrame and add the `trip_dates` range as the `date` index
df = pd.DataFrame(normals).set_index('date')
df
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tavg</th>
      <th>tmax</th>
      <th>tmin</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-01-15</th>
      <td>69.313725</td>
      <td>78.0</td>
      <td>56.0</td>
    </tr>
    <tr>
      <th>2018-01-16</th>
      <td>68.629630</td>
      <td>80.0</td>
      <td>54.0</td>
    </tr>
    <tr>
      <th>2018-01-17</th>
      <td>69.074074</td>
      <td>76.0</td>
      <td>61.0</td>
    </tr>
    <tr>
      <th>2018-01-18</th>
      <td>68.631579</td>
      <td>77.0</td>
      <td>57.0</td>
    </tr>
    <tr>
      <th>2018-01-19</th>
      <td>68.263158</td>
      <td>78.0</td>
      <td>60.0</td>
    </tr>
    <tr>
      <th>2018-01-20</th>
      <td>68.866667</td>
      <td>78.0</td>
      <td>61.0</td>
    </tr>
    <tr>
      <th>2018-01-21</th>
      <td>70.145455</td>
      <td>76.0</td>
      <td>61.0</td>
    </tr>
    <tr>
      <th>2018-01-22</th>
      <td>69.264151</td>
      <td>76.0</td>
      <td>60.0</td>
    </tr>
    <tr>
      <th>2018-01-23</th>
      <td>69.509091</td>
      <td>79.0</td>
      <td>57.0</td>
    </tr>
    <tr>
      <th>2018-01-24</th>
      <td>68.762712</td>
      <td>78.0</td>
      <td>58.0</td>
    </tr>
    <tr>
      <th>2018-01-25</th>
      <td>67.949153</td>
      <td>75.0</td>
      <td>61.0</td>
    </tr>
  </tbody>
</table>
</div>


```python
# Plot the daily normals as an area plot with `stacked=False`
fig, ax = plt.subplots()
ax.plot(df.index, df['tmin'], color='lightsteelblue')
ax.plot(df.index, df['tavg'], color='lightsalmon')
ax.plot(df.index, df['tmax'], color='khaki')
ax.fill_between(df.index, df['tmax'], color='khaki', alpha=0.3)
ax.fill_between(df.index, df['tavg'], color='lightsalmon', alpha=0.3)
ax.fill_between(df.index, df['tmin'], color='lightsteelblue', alpha=0.3)

plt.title(f"Daily Normals For Trip Dates", fontsize=14)

for item in ([ax.xaxis.label, ax.yaxis.label] + ax.get_xticklabels() + ax.get_yticklabels()):
    item.set_fontsize(8)

plt.xticks(rotation=15)
plt.legend()

plt.show()
```
![dailies](Resources/dailynormals.png)

