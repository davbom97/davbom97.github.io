---
layout: posts
title:  "Single facility location problem"
---

## Problem description  
Suppose we pose ourself the problem : given some known factors, where should we build our facility(a supermarket, a shop, a distribution center, etc.) so we maximize our profits.

This is a common question that must be asked everytime we plan to make a investment to expand a business in a new region.
Also often the available data might be the bare minimum, so we have to make assumptions and make the most out of the data.

In this case, our problem is the following : we want to build a new supermarket in the province of Brescia, Italy.
The only data given is :
* The cities in the province
* Their population
* Their georaphical location (latitude and longitude)
We will try to find the optimal location to build our new supermarket.

In our solution we are making the following hypotheses:
1. All the costs are assumed the same for every place, which means that **the only relevant cost is the transportation cost**, proportional to the distance between the facility and each town.
2. We don't have data on the clients, so **we weigh our decision based on the total population of each region.**
3. The supermarket alone can supply all the possible demand, so it is necessary to build **one and only one facility.**
 

## Data Retrieval

First we import the relevant libraries and the cities data (from the year 2011)



```python
import pandas as pd
from geopy.geocoders import Nominatim,Photon
import numpy as np
from scipy import optimize as opt
from PIL import Image
from IPython.display import display
import time
```


```python
data = pd.read_excel("Brescia_province_data.xlsx",sep = ";")
display(data.head())
```

Now we need to get the latitudes and longitudes for the capital cities to compute the distances,we do so using a geolocator:


```python
geoloc = Nominatim(user_agent="Facility_loc_problem")
coordinates_df = pd.DataFrame(np.zeros((len(data.index),2)),index = data.index, columns = ["lat","long"])
cities = data.loc[:,"City"].values
i = 0
while i != (len(cities)): #gets the latitude and longitude of every point
    try:
        location = geoloc.geocode(cities[i] + ", Brescia, Italy")
        coordinates_df.loc[i,"lat"] = location.latitude 
        coordinates_df.loc[i,"long"] = location.longitude
        i+=1
        time.sleep(1)
    except:
        time.sleep(20)
        continue
data = data.join(coordinates_df)
```

Now let's see the updated dataframe with the coordinates of each city:


```python
display(data.head())
data.to_csv("Brecia_province_data_complete.csv",sep = ";")
```

Here is a map with all the cities to check if the coordinates are right :


```python
data = pd.read_csv("Brecia_province_data_complete.csv",sep = ";",index_col = 0)
fig = Image.open("images/sflpioc.png")
display(fig)
```

![sflpioc.png](/assets/Single-facility-location-problem-p/sflpioc.png)

## Solving the problem

Now that we have acquired all the necessary data, we can start describing the distance, the distance i've chosen is the rectilinear (also called "Manhattan") distance, due to the fact that is more realistic than the euclidean distance and takes into account for multiple possible paths to reach a city, the formula is:  

$$ d_i = |x-x_i| + |y-y_i| $$ 

$$ d_i $$ = rectilinear distance between the location the i-th point   
$$x$$ = x coordinate of our location  
$$y$$ = y coordinate of our location  
$$x_i$$ = x coordinate of our i-th point  
$$y_i$$ = y coordinate of our i-th point  

then we weigh each distance by a factor $$ w_i $$  
 
$$f_i = d_iw_i $$

our weight in this case is the city population
  
now we can write the sum of weighted distances between the location and the points as:  

$$ F = \sum_{i = 0}^{n}f_i = \sum_{i = 0}^{n}d_iw_i = \sum_{i = 0}^{n}(|x-x_i| + |y-y_i|)w_i$$

and so our objective function will be:  

$$min(F) = min(\sum_{i = 0}^{n}(|x-x_i| + |y-y_i|)w_i) $$

Now we will write the code that will find the values x,y of our location so that the sum of weighted distances is minimized:


```python
def objective_func(location,points_coords,points_weights):
    x = location[0]
    y = location[1]
    x_i = points_coords[:,0]
    y_i = points_coords[:,1]
    w_i = points_weights
    return sum((abs((x_i-x))+abs((y_i-y))*w_i))
location = [0,0] #our starting x and y points for the location
coordinates_df = data.loc[:,["lat","long"]]
res = opt.minimize(objective_func,location,args = (coordinates_df.values, data.loc[:,"Pop size"].values),method ="Nelder-Mead")
location = res.x
print("The latitude and the longitude of the optimal location is = " + str((res.x[0],res.x[1])))
```

## Visualizing our result

Now that we've got the latitude and longitude of the optimal location we can get its address and plot it on the map:



```python
print(geoloc.reverse(location))
```

![sflpiwf.png](/assets/Single-facility-location-problem-p/sflpiwf.png)

## Analyzing the result

The method provides the best location to minimize the average distance between the location and the clients,this is why we take in account the population size (it would be best if we knew the number of clients in each city but it is a necessary approximation in case we have no data), and also minimizes the average travel time.

In the case we have a set of location to choose for our facility, the problem becomes trivial, in fact, with a few adjustments we can take in account the investment necessary to build in a specific location, this will be subject to another work.

A non-trivial problem is the multiple facility problem, which will require an approximate solution through clustering.

It is important to point out that this method is general purpose whenever our problem requires to minimize the distance between a point and a set of other points (whatever and wherever they could be).



