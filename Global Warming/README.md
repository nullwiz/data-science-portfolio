Global warming is an important subject that takes place all over the world, affecting crops, enviroments,
and is suspected just to get worse in the future. 

I will walk throught the analyisis of this Dataset and create an animation that shows the median change
in temperature over the years for random cities all around the world. 

Lets start off by analyzing the average raise of temperature over the years:

```python
year_temp = globaltemp.groupby(globaltemp.dt.dt.year).mean()
pd.stats.moments.ewma(year_temp.LandAverageTemperature, 5).plot()
year_temp.LandAverageTemperature.plot(linewidth=1)
plt.title('Average temperature by year')
plt.xlabel('year')
```

[image]


We can see an upward trend. We will now check the temperatures for the cities.
For that, we need to convert to a city-year table and calculate mean year temperature.

~~~~python
bycities = pd.read_csv('../input/GlobalLandTemperaturesByCity.csv', parse_dates=['dt'])
# there are some cities with the same name but in different countries 
bycities[['City', 'Country']].drop_duplicates()
bycities.City = bycities.City.str.cat(bycities.Country, sep=' ')
bycities = bycities[bycities.dt.dt.year >= 1900]
bycities.head()
~~~~

We need to normalize cities temperatures. For that we shift them by the means for the first 5 years. 

~~~~python
first_years_mean = city_means.iloc[:, :5].mean(axis=1) # mean temperature for the first 5 years
city_means_shifted = city_means.subtract(first_years_mean, axis=0)

def plot_temps(cities, city_ser, ax):
    first_years_mean = city_ser.iloc[:, :5].mean(axis=1)
    city_ser = city_ser.subtract(first_years_mean, axis=0)
    for city in random_cities:
        row = city_ser.loc[city]
        pd.stats.moments.ewma(row, 10).plot(label=row.name, ax=ax)
    ax.set_xlabel('')
    ax.legend(loc='best')

fig, axes = plt.subplots(3,1, figsize=(10,10))

n = 5
random_cities = city_means_shifted.sample(n).index

plot_temps(random_cities, city_means, axes[0])
plot_temps(random_cities, city_mins, axes[1])
plot_temps(random_cities, city_maxs, axes[2])

axes[0].set_title("Year's mean temperature increase for random cities")
axes[1].set_title("Year's min temperature increase for random cities")
axes[2].set_title("Year's max temperature increase for random cities")
~~~~

[image_tempbycity]




We want to create an animation showing markers which will have the following characteristics:
Color: indicates if the year temperature is near to a recorded one for the city  (dark blue for coldest temperatures, dark red for highest temperatures).
Size: Represents absolute difference between city median temperature and current year temperature.
~~~python
def get_temp_markers(city_names, year):
    points = np.zeros(len(city_names), dtype=[('lon', float, 1),
                                      ('lat', float, 1),
                                      ('size',     float, 1),
                                      ('color',    float, 1)])
    cmap = plt.get_cmap('coolwarm')
    
    for i, city in enumerate(city_names):
        city_temps = city_means.loc[city]
        _MIN, _MAX, _MEDIAN = city_temps.min(), city_temps.max(), city_temps.median()
        temp = city_temps.loc[year]
        
        coords = cities_info.loc[city][['Latitude', 'Longitude']].values
        lat = float(coords[0][:-1]) * (-1 if coords[0][-1] == 'S' else 1)
        lon = float(coords[1][:-1]) * (-1 if coords[1][-1] == 'W' else 1)
        
        points['lat'][i] = lat
        points['lon'][i] = lon
        points['size'][i] = 100 * abs(temp - _MEDIAN)
        points['color'][i] = (temp - _MIN) / (_MAX - _MIN)
            
    return points   
    ~~~~
