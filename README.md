

```python
#dependencies
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine 
import pandas as pd
import numpy as np
import matplotlib.patches as mpatches
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine, inspect, func

```


```python
engine = create_engine("sqlite:///hawaii.sqlite")


```


```python
Base = automap_base()
Base.prepare(engine, reflect=True)
Base.classes.keys()
```




    ['measurements', 'stations']




```python
Measurements = Base.classes.measurements
```


```python
Stations = Base.classes.stations
```


```python
session = Session(engine)
```

Precipitation Analysis


```python
check_measurements = session.query(Measurements).first()
```


```python
check_measurements.__dict__
```




    {'_sa_instance_state': <sqlalchemy.orm.state.InstanceState at 0x17ebde7b0f0>,
     'date': '2010-01-01',
     'id': 1,
     'prcp': 0.08,
     'station': 'USC00519397',
     'tobs': 65}




```python
#Most recent date by descending.first
most_recent = session.query(Measurements.date).order_by(Measurements.date.desc()).first()
most_recent_list = most_recent[0].split("-")#split on "-"
most_recent_list#check
```




    ['2017', '08', '23']




```python
#convert index 0 value to integer to perform math
year_before = int(most_recent_list [0])-1
year_before#check
```




    2016




```python
#convert back to string and combine
year_ago = str(year_before)+"-"+most_recent_list[1]+"-"+most_recent_list[2]
year_ago#check
```




    '2016-08-23'




```python
#session query
year_prcp = session.query(Measurements.date, Measurements.prcp).order_by(Measurements.date).filter(Measurements.date > year_ago)
```


```python
year_prcp#check
```




    <sqlalchemy.orm.query.Query at 0x17ebef8f470>




```python
#write to data frame
year_prcp_df = pd.read_sql_query(year_prcp.statement, engine,index_col = 'date')
year_prcp_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
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
      <td>0.08</td>
    </tr>
    <tr>
      <th>2016-08-24</th>
      <td>2.15</td>
    </tr>
    <tr>
      <th>2016-08-24</th>
      <td>2.28</td>
    </tr>
    <tr>
      <th>2016-08-24</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-08-24</th>
      <td>1.22</td>
    </tr>
  </tbody>
</table>
</div>




```python
#plot graph no.1
sns.set()
year_prcp_df.plot(figsize = (10,7),rot = 45, use_index = True, legend=True, color='G')
plt.ylabel('Precipation', fontweight = 'bold')
plt.xlabel('Date', fontweight = 'bold')
plt.title("Precipition in Hawaii from %s to %s" % (year_ago,most_recent[0]), fontweight = 'bold')
green_patch = mpatches.Patch(color='Green', label='precipitation')
plt.legend(handles=[green_patch])
plt.show()
```


![png](output_15_0.png)


Station Analysis


```python
#inspect station data
station_data = session.query(Stations).first()
station_data.__dict__
```




    {'_sa_instance_state': <sqlalchemy.orm.state.InstanceState at 0x17ebdb80f98>,
     'elevation': '3',
     'id': 1,
     'latitude': '21.2716',
     'location': 'POINT(21.2716 -157.8168)',
     'longitude': -157.8168,
     'name': 'WAIKIKI 717.2, HI US',
     'station': 'USC00519397'}




```python
#get station count, has been checked with measurement station count
station_count = session.query(Stations.station).group_by(Stations.station).count()
```


```python
station_count#check
```




    9




```python
#query tables to get count of daily report, all temp data is complete for each record, so the count
#reflects a count of a station giving temp data, prcp data may or may not have been reported on that date
temp_data_query = session.query(Stations.station, Stations.name, Measurements.station, func.count(Measurements.tobs)).filter(Stations.station == Measurements.station).group_by(Measurements.station).order_by(func.count(Measurements.tobs).desc()).all()
```


```python
temp_data_query
```




    [('USC00519281', 'WAIHEE 837.5, HI US', 'USC00519281', 2772),
     ('USC00519397', 'WAIKIKI 717.2, HI US', 'USC00519397', 2724),
     ('USC00513117', 'KANEOHE 838.1, HI US', 'USC00513117', 2709),
     ('USC00519523', 'WAIMANALO EXPERIMENTAL FARM, HI US', 'USC00519523', 2669),
     ('USC00516128', 'MANOA LYON ARBO 785.2, HI US', 'USC00516128', 2612),
     ('USC00514830',
      'KUALOA RANCH HEADQUARTERS 886.9, HI US',
      'USC00514830',
      2202),
     ('USC00511918', 'HONOLULU OBSERVATORY 702.2, HI US', 'USC00511918', 1979),
     ('USC00517948', 'PEARL CITY, HI US', 'USC00517948', 1372),
     ('USC00518838', 'UPPER WAHIAWA 874.3, HI US', 'USC00518838', 511)]




```python
#hard code most active station
most_activity = temp_data_query[0][0:2]
most_activity
```




    ('USC00519281', 'WAIHEE 837.5, HI US')




```python
# the number of reports from the most active station
temps_mosact = session.query(Measurements.station, Measurements.tobs).filter(Measurements.station == most_activity[0], Measurements.date > year_ago).all()
```


```python
len(temps_mosact)
```




    351




```python
# list created from temperature data query from the most active station
sns.set()
temps = [x[1] for x in temps_mosact]
plt.hist(temps, bins=12, color='R')
plt.xlabel("Temperature (F)", fontweight = 'bold')
plt.ylabel("Frequency", fontweight = 'bold')
plt.title("Temperature Frequency at %s" % (most_activity[1]), fontweight = 'bold')
red_patch = mpatches.Patch(color='red', label='tobs')
plt.legend(handles=[red_patch])
plt.show()
```


![png](output_25_0.png)


Temperature Analysis


```python
def calc_temps(start_date, end_date):
    #create dates 1 year prior
    dates = [start_date, end_date]
    new_dates = []
    for date in dates:
        date_list = date.split("-")
        date_list[0] = str(int(date_list[0]) - 1)
        new_date = "-".join(date_list)
        new_dates.append(new_date)
    print(new_dates) 
    
    #query database for temps from those dates
    temp_values = session.query(Measurements.tobs).filter(Measurements.date >= new_dates[0], Measurements.date <= new_dates[1]).all()
    temp_values_list = [x for (x,) in temp_values]
    avg_temp = np.mean(temp_values_list)
    max_temp = max(temp_values_list)
    min_temp = min(temp_values_list)
    
    # create bar graph
    plt.figure(figsize=(2,5))
    plt.title("Trip Avg Temp", fontweight = 'bold')
    plt.ylabel("Temperature (F)",fontweight = 'bold')
    plt.bar(1, avg_temp, yerr = (max_temp - min_temp), tick_label = "",color='Y')
    plt.show()
```


```python
calc_temps('2018-08-01', '2018-08-14')
```

    ['2017-08-01', '2017-08-14']
    


![png](output_28_1.png)


Optional Recommended Analysis


```python
#query to return list of temps for each date
def daily_normals(chosen_date):
    temps = session.query(Measurements.tobs).filter(Measurements.date.like('%'+chosen_date)).all()
    obs = [x for (x), in temps]
    return obs
    
start_date = '08-23'
end_date = '09-04'

#function to generate list of dates given any start and end date
def create_date_list(start_date, end_date):
    start_month = start_date.split("-")[0]
    end_month = end_date.split("-")[0]
    
    start_day = int(start_date.split("-")[1])
    end_day = int(end_date.split("-")[1])
    
    if start_month == end_month:
        diff = end_day - start_day
        days = [start_day + x for x in range(0,diff + 1) ]
    
    else:
        diff1 = 31 - start_day
        days1 = [start_day + x for x in range(0,diff1 + 1)]
        days2 = [x for x in range(1, end_day + 1)]
        days = days1 + days2
        
    days_str = [('%s-%s' % (start_month, str(x))) if len(str(x)) == 2 else ('%s-0%s' % (end_month, str(x))) for x in days]
    return days_str

#uses functions above to return dictionary of normals, skips dates for which there is no data (false dates)
def query_results(start, end):
    dates = create_date_list(start, end)
    master_dict = {"Date": [], "tmax": [], "tmin": [], "tavg": []}
    for date in dates:
        data_list = []
        observations = daily_normals(date)
        if observations != []:
            for temp in observations:
                data_list.append(temp)
            master_dict['Date'].append(date)
            master_dict['tmax'].append(max(data_list))
            master_dict['tmin'].append(min(data_list))
            master_dict['tavg'].append(round(np.mean(data_list),2))
            master_dict
    return(master_dict)
    
normals_df = pd.DataFrame(query_results('08-03', '08-15')).set_index('Date')
normals_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tavg</th>
      <th>tmax</th>
      <th>tmin</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>08-03</th>
      <td>76.61</td>
      <td>85</td>
      <td>70</td>
    </tr>
    <tr>
      <th>08-04</th>
      <td>76.71</td>
      <td>84</td>
      <td>69</td>
    </tr>
    <tr>
      <th>08-05</th>
      <td>76.15</td>
      <td>82</td>
      <td>69</td>
    </tr>
    <tr>
      <th>08-06</th>
      <td>76.25</td>
      <td>83</td>
      <td>67</td>
    </tr>
    <tr>
      <th>08-07</th>
      <td>77.16</td>
      <td>83</td>
      <td>71</td>
    </tr>
    <tr>
      <th>08-08</th>
      <td>76.56</td>
      <td>83</td>
      <td>68</td>
    </tr>
    <tr>
      <th>08-09</th>
      <td>75.98</td>
      <td>81</td>
      <td>69</td>
    </tr>
    <tr>
      <th>08-10</th>
      <td>76.42</td>
      <td>83</td>
      <td>65</td>
    </tr>
    <tr>
      <th>08-11</th>
      <td>75.98</td>
      <td>82</td>
      <td>67</td>
    </tr>
    <tr>
      <th>08-12</th>
      <td>76.53</td>
      <td>83</td>
      <td>67</td>
    </tr>
    <tr>
      <th>08-13</th>
      <td>76.98</td>
      <td>84</td>
      <td>71</td>
    </tr>
    <tr>
      <th>08-14</th>
      <td>76.78</td>
      <td>82</td>
      <td>71</td>
    </tr>
    <tr>
      <th>08-15</th>
      <td>76.47</td>
      <td>83</td>
      <td>69</td>
    </tr>
  </tbody>
</table>
</div>




```python
normals_df = normals_df[['tmax', 'tavg', 'tmin']]

normals_df.plot(kind = 'area', stacked = False, alpha = .25, rot = 45, color = ['brown', 'orange', 'gold'], figsize = (10,7), linestyle = 'solid')
plt.xlabel('Date')
plt.ylabel('Temperature (F)')
plt.legend(frameon = True)
plt.show()
```


![png](output_31_0.png)

