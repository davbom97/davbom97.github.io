---
layout: posts
title:  "Capacitated multi facility location problem"
description: "In this post we will find how to choose the best location for multiple capacitated facilities given a set of points of interest"
category: "Optimization"
#tags: "optimization","location","logistics"
---
## Problem description

The capacitated multi-facility location problem is an extension of the single facility problem to **n facilities simultaneously**.
In this project, the objective is to find the optimal location of multiple facilities at the same time, given a maximum output capacity, in Italy.

The hypotheses are quite the same as the single facility location problem but **we dropped the hypothesis that a facility can supply infinite demand.**



First we load the data about all the italian cities with more than 5000 population (ISTAT 2013 data)


```python
import pandas as pd
from geopy.geocoders import Nominatim
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from sklearn.cluster import KMeans
from IPython.display import display
import time
import pickle
```


```python
data = pd.read_csv("cities_data_fromMax_to5mil.csv") 
display(data.head())
display(data.tail())
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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city_name</th>
      <th>pop_size</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Varallo Pombia</td>
      <td>5004</td>
      <td>45.665904</td>
      <td>8.632693</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Certosa di Pavia</td>
      <td>5004</td>
      <td>45.257015</td>
      <td>9.148114</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mistretta</td>
      <td>5014</td>
      <td>37.929885</td>
      <td>14.362873</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Santa Teresa Gallura</td>
      <td>5018</td>
      <td>41.241377</td>
      <td>9.188518</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alzate Brianza</td>
      <td>5019</td>
      <td>45.769778</td>
      <td>9.182041</td>
    </tr>
  </tbody>
</table>
</div>



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
      <th>city_name</th>
      <th>pop_size</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2370</th>
      <td>Palermo</td>
      <td>657561</td>
      <td>38.111227</td>
      <td>13.352443</td>
    </tr>
    <tr>
      <th>2371</th>
      <td>Torino</td>
      <td>872367</td>
      <td>45.067755</td>
      <td>7.682489</td>
    </tr>
    <tr>
      <th>2372</th>
      <td>Napoli</td>
      <td>962003</td>
      <td>40.835934</td>
      <td>14.248783</td>
    </tr>
    <tr>
      <th>2373</th>
      <td>Milano</td>
      <td>1242123</td>
      <td>45.466800</td>
      <td>9.190500</td>
    </tr>
    <tr>
      <th>2374</th>
      <td>Roma</td>
      <td>2617175</td>
      <td>41.894802</td>
      <td>12.485338</td>
    </tr>
  </tbody>
</table>
</div>


The latitude and longitude are retrieved the same way as explained in the single facility location problem.  
Next it is important to check if the data is representative, we sum all the population of the cities we are considering and divide them by the total population of Italy at that time.


```python
np.round(data.loc[:,"pop_size"].sum()/59433744,3)
```




    0.822



The cities represent the 82.2% of all the population of italy which is representative enough.  
Suppose this time that the number of people buying our product is the 10% of the population, equally distributed across all the cities.
Then we must calculate the weight of each city by dividing the number of potential clients in each city by the total number of clients


```python
pt_clients_perc = 0.1
clients = np.round(data.loc[:,"pop_size"].values*pt_clients_perc)
clients_perc = pd.DataFrame(clients/clients.sum(),columns = ["weight"])
data = data.join(clients_perc)
data = data.join(pd.DataFrame(clients,columns = ["number_of_clients"]))
#data = data.join(clients)
display(data.head())
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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city_name</th>
      <th>pop_size</th>
      <th>lat</th>
      <th>long</th>
      <th>weight</th>
      <th>number_of_clients</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Varallo Pombia</td>
      <td>5004</td>
      <td>45.665904</td>
      <td>8.632693</td>
      <td>0.000102</td>
      <td>500.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Certosa di Pavia</td>
      <td>5004</td>
      <td>45.257015</td>
      <td>9.148114</td>
      <td>0.000102</td>
      <td>500.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mistretta</td>
      <td>5014</td>
      <td>37.929885</td>
      <td>14.362873</td>
      <td>0.000103</td>
      <td>501.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Santa Teresa Gallura</td>
      <td>5018</td>
      <td>41.241377</td>
      <td>9.188518</td>
      <td>0.000103</td>
      <td>502.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alzate Brianza</td>
      <td>5019</td>
      <td>45.769778</td>
      <td>9.182041</td>
      <td>0.000103</td>
      <td>502.0</td>
    </tr>
  </tbody>
</table>
</div>


The way we will find the solution is by using a clustering method, in particular the K-Means algorithm, which seeks to minimize the distance between each point in a cluster (the cities assigned to a particular facility) and the centre of the cluster (the facility itself)

First we initialize the clustering process by putting all the cities in one cluster


```python
coordinates_df = data.loc[:,["lat","long"]]
n = 1

#solver.fit(coordinates_df,sample_weight = data.loc[:,"%clients"] )
#location = solver.cluster_centers_
#labels = data.set_index(solver.labels_)
```

Now we introduce a constraint max_capacity, defined as the maximum amount of product units the facility can output in one year, divided by the total demand of the output in one year, for a single product 

suppose the maximum output is 1500 units/day the maximum capacity is:


```python
units_day = 1500
units_year = units_day*365
max_capacity = np.round(units_year/clients.sum(),6)
print(f"The maximum capacity for each facility is {max_capacity}")
print(f"Which equals to a maximum yearly output of {units_year} units/year ")
```

    The maximum capacity for each facility is 0.112119
    Which equals to a maximum yearly output of 547500 units/year 
    

The total sum of the demand (or weights) for i-th facility must be less or equal than the units_year (or max_capacity).


Next we define the function find_optimum that takes in the minimum amount of facilities we want to locate and find an optimal amount of facilities with the aforementioned contraint.
Then, among the possible solutions we calculate the the average saturation rate, defined as : 

$$ \textrm{Saturation rate}_j = \frac{\textrm{output}_j}{\textrm{maximum output}} = \frac{\sum w_{ij}}{\textrm{maximum output}} $$  

$$ \textrm{output}_j = \textrm{The sum of the weights of the cities supplied by the j-th facility} $$
$$ w_{ij} = \textrm{The weight of the i-th city supplied by the j-th facility} $$
$$ \textrm{maximum output} = \textrm{The maximum sum of weights a facility can supply} $$

In this case, given a maximum daily output of 1500 units/day:

$$ \textrm{Saturation rate}_j = \frac{\textrm{output}_j}{0.1129} $$

In the end we pick the combination of facilities with the maximum saturation rate.


```python
def find_optimum(starting_n):
    n = starting_n
    optimal_solution = None
    optimal_cluster_n = 10000  #initializes the variables that will hold the optimum
    optimal_array = [] #optimal array will contain the all the possible optimum solutions
    optimal_capacity = [] #optimal capacity will contain the output of the faclities in a given solution
    while True:
        print (f"Trying number of facilities = {n}")
        labels = data.set_index(np.zeros(len(data.index)))
        #These lines will initialize the labels array which will hold the clusters 
        for j in range(200):
            solver = KMeans(n,n_init = 1,max_iter = 1000,tol = 1e-9,init = "k-means++")
            solver.fit(coordinates_df,sample_weight = data.loc[:,"weight"])
            labels = pd.DataFrame(solver.labels_,columns = ["label"])
            labels = data.set_index(solver.labels_)
            #fits the kmeans algorithm to the data points

            for i in range(0,n):
                used_capacity = labels.groupby(level=0)["weight"].sum().values
            #computes used_capacity which measures how much the capacity of each facility is saturated
            if all([i <= max_capacity for i in used_capacity]): 
                #if the capacity of all the facilities is not saturated, it is considered the minimum amount of facilities
                #that satisfy the capacity contraint,and will search an optimum among them,
                #otherwise the loop continues
                optimal_cluster_n = n
                optimal_array.append(solver)
                optimal_capacity.append(used_capacity)
                
        if optimal_array:
            #calculates the average output of each found solution, then picks the one with the maximum avg output
            optimal_capacity_avg = [facility_used_capacity.mean() for facility_used_capacity in optimal_capacity]
            optimal_saturation = max(optimal_capacity_avg)
            indx = optimal_capacity_avg.index(optimal_saturation)
            used_capacity = optimal_capacity[indx]
            optimal_solution = optimal_array[indx]
            print("Done!")
            return optimal_cluster_n,optimal_solution,used_capacity
        if optimal_solution == None:
            n+=1
            continue 
        else:
            break
```

We run the algorithm starting from one facility :


```python
cluster_n,solver,used_capacity = find_optimum(9)
location = solver.cluster_centers_
labels = data.set_index(solver.labels_)
```

    Trying number of facilities = 9
    Trying number of facilities = 10
    Trying number of facilities = 11
    Trying number of facilities = 12
    Trying number of facilities = 13
    Trying number of facilities = 14
    Trying number of facilities = 15
    Trying number of facilities = 16
    Trying number of facilities = 17
    Done!
    

**We might run the algorithm multiple times, becouse it does not guarantee that it outputs a feasible location for each facility.**  
Then we plot our found facilities on a map to check feasibility of the results:
![cmflp_lm.png](/assets/Capacitated-multi-facility-location-problem/cmflp_lm.png)

Next we locate the facilities from the coordinates

```python
geoloc = Nominatim(user_agent="Facility_loc_problem")
addresses = []
i = 0
while True : 
    try:
        if i == len(location):
            break
        else:
            loc = geoloc.reverse(location[i],exactly_one = True)
            addresses.append(loc[0])
            time.sleep(2)
            i+=1
    except:
        time.sleep(20)
    
facilities = pd.DataFrame(location,index = addresses,columns = ["Latitude","Longitude"])
facilities = facilities.join(pd.DataFrame(np.arange(cluster_n),index = addresses, columns = ["Cluster number"]))
```

The last step is to calculate the saturation rate and display the results


```python
print(f"The optimal number of facilities is = {cluster_n}")
saturation_avg = np.around(sum(used_capacity*100)/(cluster_n*max_capacity),2)
saturation_pctg = np.around(used_capacity*100/max_capacity,2)
facility_output = np.round(saturation_pctg/100*units_day,0)
print(f"The average saturation rate is = {saturation_avg}%")
facilities = facilities.join(pd.DataFrame(saturation_pctg,
                                          index = facilities.index,
                                          columns = ["Saturation rate(%)"]))
facilities = facilities.join(pd.DataFrame(facility_output,
                                          index = facilities.index,
                                          columns = ["Daily facility output(pieces/day)"]))
facilities = facilities.join(pd.DataFrame(facility_output*365,
                                          index = facilities.index,
                                          columns = ["Yearly facility output(pieces/year)"]))
```

    The optimal number of facilities is = 17
    The average saturation rate is = 52.47%
    


```python
labels.index = labels.index.set_names("Cluster number")
display(facilities)
display(labels.head())
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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster number</th>
      <th>Saturation rate(%)</th>
      <th>Daily facility output(pieces/day)</th>
      <th>Yearly facility output(pieces/year)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Poggio del Lupo, Misterbianco, Catania, Sicilia, 95045, Italia</th>
      <td>37.545541</td>
      <td>15.038274</td>
      <td>0</td>
      <td>48.32</td>
      <td>725.0</td>
      <td>264625.0</td>
    </tr>
    <tr>
      <th>Via Simoncini, Ville, Serravalle Pistoiese, Pistoia, Toscana, 51130, Italia</th>
      <td>43.889483</td>
      <td>10.859511</td>
      <td>1</td>
      <td>70.31</td>
      <td>1055.0</td>
      <td>385075.0</td>
    </tr>
    <tr>
      <th>Cartiera, Loreto Aprutino, Pescara, Abruzzo, 65014, Italia</th>
      <td>42.405641</td>
      <td>13.996831</td>
      <td>2</td>
      <td>22.45</td>
      <td>337.0</td>
      <td>123005.0</td>
    </tr>
    <tr>
      <th>Santa Maria in Prato, San Zenone al Lambro, Milano, Lombardia, 26852, Italia</th>
      <td>45.316446</td>
      <td>9.363823</td>
      <td>3</td>
      <td>95.78</td>
      <td>1437.0</td>
      <td>524505.0</td>
    </tr>
    <tr>
      <th>Erchie, Brindisi, Puglia, 72028, Italia</th>
      <td>40.451648</td>
      <td>17.710318</td>
      <td>4</td>
      <td>32.12</td>
      <td>482.0</td>
      <td>175930.0</td>
    </tr>
    <tr>
      <th>Circonvallazione sud, Asuni, Aristanis/Oristano, Sardegna, Italia</th>
      <td>39.861941</td>
      <td>8.945599</td>
      <td>5</td>
      <td>19.93</td>
      <td>299.0</td>
      <td>109135.0</td>
    </tr>
    <tr>
      <th>Via Brigata Emilia, Pero, Breda di Piave, Treviso, Veneto, 31048, Italia</th>
      <td>45.700320</td>
      <td>12.342201</td>
      <td>6</td>
      <td>70.84</td>
      <td>1063.0</td>
      <td>387995.0</td>
    </tr>
    <tr>
      <th>Via Prenestina, Quartiere XIX Prenestino-Centocelle, Roma, Roma Capitale, Lazio, 00171, Italia</th>
      <td>41.894788</td>
      <td>12.560785</td>
      <td>7</td>
      <td>91.55</td>
      <td>1373.0</td>
      <td>501145.0</td>
    </tr>
    <tr>
      <th>Parcheggio Stadio, Via Palermo, San Bernardino, Oltrestazione, Legnano, Milano, Lombardia, 20025, Italia</th>
      <td>45.588261</td>
      <td>8.910030</td>
      <td>8</td>
      <td>55.69</td>
      <td>835.0</td>
      <td>304775.0</td>
    </tr>
    <tr>
      <th>Mulino Drago, Strada statale Corleonese Agrigentina, Casa Sant'Ippolito, Corleone, Palermo, Sicilia, 90030, Italia</th>
      <td>37.860989</td>
      <td>13.297069</td>
      <td>9</td>
      <td>36.99</td>
      <td>555.0</td>
      <td>202575.0</td>
    </tr>
    <tr>
      <th>Via Tetti Laghi, Cascina Reggenza, Carmagnola, Torino, Piemonte, 10022, Italia</th>
      <td>44.885312</td>
      <td>7.767223</td>
      <td>10</td>
      <td>50.88</td>
      <td>763.0</td>
      <td>278495.0</td>
    </tr>
    <tr>
      <th>Via Paradiso, Maldritto, Castelbelforte, Mantova, Lombardia, 46032, Italia</th>
      <td>45.223828</td>
      <td>10.887862</td>
      <td>11</td>
      <td>77.83</td>
      <td>1167.0</td>
      <td>425955.0</td>
    </tr>
    <tr>
      <th>Valcarecce, Cingoli, Macerata, Marche, 60039, Italia</th>
      <td>43.402066</td>
      <td>13.177101</td>
      <td>12</td>
      <td>22.85</td>
      <td>343.0</td>
      <td>125195.0</td>
    </tr>
    <tr>
      <th>strada Liardi Iunci, Iunci Soprano, Catanzaro, Calabria, 88049, Italia</th>
      <td>39.096233</td>
      <td>16.353780</td>
      <td>13</td>
      <td>20.05</td>
      <td>301.0</td>
      <td>109865.0</td>
    </tr>
    <tr>
      <th>Masseria Friuli, Corato, Bari, Puglia, Italia</th>
      <td>41.106532</td>
      <td>16.329850</td>
      <td>14</td>
      <td>44.45</td>
      <td>667.0</td>
      <td>243455.0</td>
    </tr>
    <tr>
      <th>Cutinelli, Sant'Anastasia, Napoli, Campania, 80038, Italia</th>
      <td>40.889406</td>
      <td>14.389779</td>
      <td>15</td>
      <td>95.82</td>
      <td>1437.0</td>
      <td>524505.0</td>
    </tr>
    <tr>
      <th>Turrito, Sarsina, Unione dei comuni Valle del Savio, Forlì-Cesena, Emilia-Romagna, Italia</th>
      <td>43.907781</td>
      <td>12.121604</td>
      <td>16</td>
      <td>36.05</td>
      <td>541.0</td>
      <td>197465.0</td>
    </tr>
  </tbody>
</table>
</div>



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
      <th>city_name</th>
      <th>pop_size</th>
      <th>lat</th>
      <th>long</th>
      <th>weight</th>
      <th>number_of_clients</th>
    </tr>
    <tr>
      <th>Cluster number</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>Varallo Pombia</td>
      <td>5004</td>
      <td>45.665904</td>
      <td>8.632693</td>
      <td>0.000102</td>
      <td>500.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Certosa di Pavia</td>
      <td>5004</td>
      <td>45.257015</td>
      <td>9.148114</td>
      <td>0.000102</td>
      <td>500.0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Mistretta</td>
      <td>5014</td>
      <td>37.929885</td>
      <td>14.362873</td>
      <td>0.000103</td>
      <td>501.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Santa Teresa Gallura</td>
      <td>5018</td>
      <td>41.241377</td>
      <td>9.188518</td>
      <td>0.000103</td>
      <td>502.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Alzate Brianza</td>
      <td>5019</td>
      <td>45.769778</td>
      <td>9.182041</td>
      <td>0.000103</td>
      <td>502.0</td>
    </tr>
  </tbody>
</table>
</div>


Then we save the facilities and the clusters for use or for further analysis using pickle


```python
save = [labels,facilities]
pickle.dump(save,open("Multi_facility_clients_and_facilities.pkl","wb"))
```
