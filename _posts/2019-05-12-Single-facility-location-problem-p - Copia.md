---
layout: posts
title:  "dasdasd"
description: "dadsa"
category: "nkho"
#tags: "optimization","location","logistics"
---

## Problem description  

Suppose we pose ourselves the following problem: given some known factors, where should we build our facility (a supermarket, a shop, a distribution center, etc.) so we maximize our profits.  
This is a common question that must be asked every time we plan to make an investment to expand a business in a new region. Besides, often the available data might be the bare minimum, so we must make assumptions and make the most out of the data.

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
from geopy.geocoders import Nominatim
import numpy as np
from scipy import optimize as opt
from IPython.display import display
import time
```


```python
data = pd.read_excel("Brescia_province_data.xlsx",sep = ";")
display(data.head(10))
```


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
<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Pop size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Acquafredda</td>
      <td>1615</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adro</td>
      <td>7180</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Agnosine</td>
      <td>1839</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alfianello</td>
      <td>2476</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anfo</td>
      <td>487</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Angolo Terme</td>
      <td>2563</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Artogne</td>
      <td>3545</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Azzano Mella</td>
      <td>2900</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bagnolo Mella</td>
      <td>12969</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Bagolino</td>
      <td>3968</td>
    </tr>
  </tbody>
</table>
</div>


Now we need to get the latitudes and longitudes of the cities to compute the distances,we do so using a geolocator:


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
data.to_csv("Brecia_province_data_complete.csv",sep = ";")
display(data)
```


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
<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Pop size</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Acquafredda</td>
      <td>1615</td>
      <td>45.307378</td>
      <td>10.414677</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adro</td>
      <td>7180</td>
      <td>45.622402</td>
      <td>9.961376</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Agnosine</td>
      <td>1839</td>
      <td>45.650676</td>
      <td>10.353335</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alfianello</td>
      <td>2476</td>
      <td>45.266975</td>
      <td>10.148172</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anfo</td>
      <td>487</td>
      <td>45.766390</td>
      <td>10.493771</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Angolo Terme</td>
      <td>2563</td>
      <td>45.891036</td>
      <td>10.146099</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Artogne</td>
      <td>3545</td>
      <td>45.848982</td>
      <td>10.164556</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>199</th>
      <td>Villa Carcina</td>
      <td>10997</td>
      <td>45.632214</td>
      <td>10.194442</td>
    </tr>
    <tr>
      <th>200</th>
      <td>Villachiara</td>
      <td>1456</td>
      <td>45.355375</td>
      <td>9.930966</td>
    </tr>
    <tr>
      <th>201</th>
      <td>Villanuova sul Clisi</td>
      <td>5855</td>
      <td>45.601680</td>
      <td>10.453872</td>
    </tr>
    <tr>
      <th>202</th>
      <td>Vione</td>
      <td>729</td>
      <td>46.248402</td>
      <td>10.447161</td>
    </tr>
    <tr>
      <th>203</th>
      <td>Visano</td>
      <td>1953</td>
      <td>45.318378</td>
      <td>10.372376</td>
    </tr>
    <tr>
      <th>204</th>
      <td>Vobarno</td>
      <td>8259</td>
      <td>45.643643</td>
      <td>10.499454</td>
    </tr>
    <tr>
      <th>205</th>
      <td>Zone</td>
      <td>1110</td>
      <td>45.763445</td>
      <td>10.115882</td>
    </tr>
  </tbody>
</table>
</div>


Here are all the cities plotted on a map:

![sflpioc.png](/assets/Single-facility-location-problem-p/sflpioc.png)

## Solving the problem

Now that we have acquired all the necessary data, we can start describing the distance, the distance I've chosen is the rectilinear (also called "Manhattan") distance of the latitudes and longitudes, due to the fact that is more realistic than the euclidean distance and takes into account for multiple possible paths to reach a city, the formula is:  

$$ d_i = |x-x_i| + |y-y_i| $$ 

$$ d_i $$ = rectilinear distance between the location the i-th point   
$$x$$ = longitude of our facility  
$$y$$ = latitude of our location  
$$x_i$$ = longitude of the i-th city  
$$y_i$$ = latitude of the i-th city  

then we weigh each distance by a factor $$ w_i $$  
 
$$f_i = d_iw_i $$

our weight in this case is the city population
  
now we can write the sum of weighted distances between the facility and the points as:  

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

    The latitude and the longitude of the optimal location is = (45.59934120275753, 10.220021399923084)
    

## Visualizing our result

Now that we've got the latitude and longitude of the optimal location we can get its address and plot it on the map:



```python
print(geoloc.reverse(location))
```

    Quartiere Colonnello Gherardo Vaiarini, Concesio, Comunità montana della valle Trompia, Brescia, Lombardia, 25062, Italia
    

![sflpiwf.png](/assets/Single-facility-location-problem-p/sflpiwf.png)

## Analyzing the result

The method provides the best location to minimize the average distance between the location and the clients,this is why we take in account the population size (it would be best if we knew the number of clients in each city but it is a necessary approximation in case we have no data) and also minimizes the average travel time.

In case we have a set of locations to choose for our facility, the problem becomes trivial, in fact, with a few adjustments we can take into account the investment necessary to build in a specific location, this will be subject to another work.

A non-trivial problem is the multiple facility problem, which will require an approximate solution through clustering.

It is important to point out that this method is general-purpose whenever our problem requires to minimize the distance between a point and a set of other points (whatever and wherever they could be).




