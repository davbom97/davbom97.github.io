---
layout: post
title:  "Single facility location problem"
---
# Facility Location Problem, Single facility
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
import plotly.express as px
import plotly.graph_objects as go
from scipy import optimize as opt
from IPython.display import display
import time

```


```python
data = pd.read_excel("Brescia_province_data.xlsx",sep = ";")
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
  </tbody>
</table>
</div>


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
  </tbody>
</table>
</div>


Here is a map with all the cities to check if the coordinates are right :


```python
data = pd.read_csv("Brecia_province_data_complete.csv",sep = ";",index_col = 0)
fig = go.Figure(go.Scattermapbox(lat=data.loc[:,"lat"],
                                 lon=data.loc[:,"long"],
                                 text = data.loc[:,"City"],
                                marker_size = 8))
#fig = px.scatter_mapbox(data, lat="lat", lon="long", size = "Pop size",text ="City",size_max = 12)
fig.update_layout(mapbox_style="open-street-map")
fig.show()
```


<div>


            <div id="537d9f19-b055-4319-bce9-b705182bb669" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("537d9f19-b055-4319-bce9-b705182bb669")) {
                    Plotly.newPlot(
                        '537d9f19-b055-4319-bce9-b705182bb669',
                        [{"lat": [45.307378, 45.6224015, 45.650676000000004, 45.266975, 45.76639, 45.891036299999996, 45.8489819, 45.454722, 45.429078999999994, 45.823561299999994, 45.405077, 45.681886999999996, 45.327176, 45.5105711, 45.50298, 46.0934179, 45.931693, 45.935562, 45.672786, 45.346975, 45.477261600000006, 45.9473303, 45.549680200000005, 45.7933072, 45.5931372, 45.454178000000006, 45.9900343, 45.9563241, 45.539802200000004, 45.6429453, 45.6126766, 45.457182, 45.540040000000005, 45.3480525, 46.030996, 45.7544353, 45.454869099999996, 45.6372565, 45.3646065, 45.56071420000001, 45.4995922, 45.4984126, 45.471576299999995, 45.6945498, 45.512504799999995, 45.580508, 46.0766465, 45.5841062, 46.002508500000005, 46.002595, 46.081396999999996, 45.5364624, 45.308636299999996, 46.024096, 45.942493, 45.5634485, 45.58131470000001, 45.8093124, 45.5815203, 45.471278000000005, 45.605184, 45.6374172, 46.166198, 45.443778, 45.883486299999994, 45.41811, 45.4694851, 46.178199, 45.5984995, 45.925692, 45.231475, 45.4828867, 45.2552458, 45.61878529999999, 45.6879136, 45.688138, 45.584627399999995, 45.401081, 45.8645299, 45.291876, 45.5926638, 45.737745399999994, 46.2203, 45.771317100000005, 45.6595554, 45.3091529, 45.739689, 45.3670119, 45.814119399999996, 45.719687, 45.4842379, 45.4610389, 45.437678000000005, 45.983494, 45.9894381, 45.6482576, 45.4769983, 45.781392, 45.449179, 45.9506025, 46.122581, 45.556043100000004, 45.355777, 45.709374, 45.75545, 45.736086, 45.518803999999996, 45.273575, 45.528523, 46.2115706, 45.711994, 45.632189700000005, 45.415181, 45.44638, 45.7139802, 45.562484999999995, 45.5877271, 45.975994, 45.545784000000005, 45.532183, 45.64685, 45.3861328, 45.627283, 46.016895, 45.403976, 45.4206595, 45.5552224, 45.946191999999996, 45.509584000000004, 45.589186600000005, 46.07286015, 45.553076000000004, 45.599068700000004, 45.659876399999995, 46.030996, 45.599486999999996, 45.301255600000005, 45.7444018, 45.7544894, 45.7741272, 45.843752200000004, 45.919692, 45.805731200000004, 45.661484, 45.551185, 45.431977, 45.460662799999994, 46.258603, 45.274374, 45.569379999999995, 45.4059696, 45.265575, 45.668287, 45.930858799999996, 45.5514816, 45.6359706, 45.6848636, 45.5656922, 45.310875, 45.27147, 45.514182, 45.4642998, 45.597645799999995, 45.6167638, 45.5276081, 45.5670385, 45.488578000000004, 45.658587, 45.715453000000004, 45.605677500000006, 45.5895461, 45.307376, 45.370576, 45.4915723, 45.6618702, 46.080496999999994, 46.055988899999996, 45.242674, 45.570284, 45.4688494, 45.527784000000004, 45.6531729, 45.690385, 45.746987, 46.2492143, 45.7397806, 45.5053097, 45.634530299999994, 45.5239055, 45.79190555, 45.477278999999996, 45.71193515, 45.515978000000004, 45.610193200000005, 45.76290825, 45.328581400000004, 45.3270123, 45.7106412, 46.240213399999995, 45.632214000000005, 45.355375, 45.6016804, 46.248402, 45.318378, 45.6436432, 45.763444799999995], "lon": [10.414677000000001, 9.961376, 10.3533352, 10.148172, 10.493771, 10.146098599999998, 10.1645561, 10.1168233, 10.18437, 10.462730800000001, 10.054367, 10.41217, 10.12837, 10.423032300000001, 10.043664999999999, 10.336266, 10.2805867, 10.2947906, 10.338869, 9.968467, 10.2408108, 10.2064036, 10.313259044548099, 10.273755900000001, 10.2434802, 10.052666, 10.3431903, 10.304040400000002, 10.2200214, 10.1469445, 10.315085400000001, 10.413574, 10.445833, 10.345921, 10.345862, 10.54469, 10.130347800000001, 9.9336046, 10.429384599999999, 10.115308800000001, 10.1450083, 9.943281800000001, 10.29831, 10.3212431, 9.977855, 10.0262148, 10.350067999999998, 10.1802449, 10.3257912, 10.351963, 10.368561999999999, 9.928846, 10.189374699999998, 10.365663, 10.277462, 9.9782721, 10.2102711, 10.332872300000002, 9.9411725, 9.950064, 10.216967, 10.0064681, 10.243457000000001, 10.007366000000001, 10.180706800000001, 10.0752034, 10.538946699999999, 10.329158999999999, 9.9722164, 10.250862, 10.324177, 10.175008199999999, 10.2934106, 10.559100500000001, 10.183724400000001, 10.663500599999999, 10.438236999999999, 10.279276, 10.1836526, 10.272374000000001, 10.15345, 10.4783256, 10.358759, 10.2841681, 10.0510489, 10.3221645, 10.43907, 10.2184518, 10.7915786, 10.277667, 10.0532758, 10.4845346, 10.059667, 10.316163000000001, 10.2456097, 10.2645464, 10.0427148, 10.615973, 10.079566999999999, 10.273339300000002, 10.3192116, 10.5625934, 10.13517, 10.2153259, 10.2873147, 10.093762, 10.3541615, 10.199273, 10.5375734, 10.3398783, 10.082235398202599, 10.0985704, 10.390675, 10.228771, 10.3436499, 10.461874, 10.2892353, 10.333263, 10.386772, 10.369172, 10.3884485, 10.117457, 10.121364999999999, 10.328362, 9.921564, 9.9617289, 10.073946000000001, 10.230561, 10.506376, 10.076411, 10.229215757122999, 10.4012548, 9.884966499999999, 9.955070300000001, 10.370963, 10.064917900000001, 10.2084011, 10.3439356, 10.3746705, 10.2373771, 10.1578872, 10.225261999999999, 10.1081901, 10.123963999999999, 10.504875, 9.989265, 10.173909199999999, 10.508662, 10.092471000000002, 9.85346, 10.6313729, 10.217574, 10.39677, 10.3103078, 10.421857000000001, 10.0441674, 10.4366014, 10.51147, 10.008567999999999, 10.3688024528361, 10.316671000000001, 9.9140564, 10.1094679, 10.4958703, 10.149701199999999, 9.9986008, 9.887062, 10.420571, 10.1111391, 10.520069, 10.551732699999999, 10.148671, 10.026767, 10.2170575, 10.195876199999999, 10.399663, 10.3439901, 10.177973, 10.366071, 10.607823699999999, 10.512675, 10.2777117, 10.100963, 10.240565, 10.46805, 10.7217649, 10.1024645, 10.609520800000002, 10.0797833, 10.7311310679556, 10.010665, 10.4622942169115, 9.869661, 10.3901746, 10.5910267515906, 10.076086300000002, 10.0572562, 10.4034707, 10.399732499999999, 10.1944422, 9.930966, 10.4538721, 10.447161, 10.372376, 10.499454, 10.1158821], "marker": {"size": 8}, "text": ["Acquafredda", "Adro", "Agnosine", "Alfianello", "Anfo", "Angolo Terme", "Artogne", "Azzano Mella", "Bagnolo Mella", "Bagolino", "Barbariga", "Barghe", "Bassano Bresciano", "Bedizzole", "Berlingo", "Berzo Demo", "Berzo Inferiore", "Bienno", "Bione", "Borgo San Giacomo", "Borgosatollo", "Borno", "Botticino", "Bovegno", "Bovezzo", "Brandico", "Braone", "Breno", "Brescia", "Brione", "Caino", "Calcinato", "Calvagese della Riviera", "Calvisano", "Capo di Ponte", "Capovalle", "Capriano del Colle", "Capriolo", "Carpenedolo", "Castegnato", "Castel Mella", "Castelcovati", "Castenedolo", "Casto", "Castrezzato", "Cazzago San Martino", "Cedegolo", "Cellatica", "Cerveno", "Ceto", "Cevo", "Chiari", "Cigole", "Cimbergo", "Cividate Camuno", "Coccaglio", "Collebeato", "Collio", "Cologne", "Comezzano-Cizzago", "Concesio", "Corte Franca", "Corteno Golgi", "Corzano", "Darfo Boario Terme", "Dello", "Desenzano del Garda", "Edolo", "Erbusco", "Esine", "Fiesse", "Flero", "Gambara", "Gardone Riviera", "Gardone Val Trompia", "Gargnano", "Gavardo", "Ghedi", "Gianico", "Gottolengo", "Gussago", "Idro", "Incudine", "Irma", "Iseo", "Isorella", "Lavenone", "Leno", "Limone sul Garda", "Lodrino", "Lograto", "Lonato del Garda", "Longhena", "Losine", "Lozio", "Lumezzane", "Maclodio", "Magasa", "Mairano", "Malegno", "Malonno", "Manerba del Garda", "Manerbio", "Marcheno", "Marmentino", "Marone", "Mazzano", "Milzano", "Moniga del Garda", "Monno", "Monte Isola", "Monticelli Brusati", "Montichiari", "Montirone", "Mura", "Muscoline", "Nave", "Niardo", "Nuvolento", "Nuvolera", "Odolo", "Offlaga", "Ome", "Ono San Pietro", "Orzinuovi", "Orzivecchi", "Ospitaletto", "Ossimo", "Padenghe sul Garda", "Paderno Franciacorta", "Paisco Loveno", "Paitone", "Palazzolo sull'Oglio", "Paratico", "Paspardo", "Passirano", "Pavone del Mella", "Pertica Alta", "Pertica Bassa", "Pezzaze", "Pian Camuno", "Piancogno", "Pisogne", "Polaveno", "Polpenazze del Garda", "Pompiano", "Poncarale", "Ponte di Legno", "Pontevico", "Pontoglio", "Pozzolengo", "Pralboino", "Preseglie", "Prestine", "Prevalle", "Provaglio d'Iseo", "Provaglio Val Sabbia", "Puegnago sul Garda", "Quinzano d'Oglio", "Remedello", "Rezzato", "Roccafranca", "Rodengo Saiano", "Ro\u00e8 Volciano", "Roncadelle", "Rovato", "Rudiano", "Sabbio Chiese", "Sale Marasino", "Sal\u00f2", "San Felice del Benaco", "San Gervasio Bresciano", "San Paolo", "San Zeno Naviglio", "Sarezzo", "Saviore dell'Adamello", "Sellero", "Seniga", "Serle", "Sirmione", "Soiano del Lago", "Sonico", "Sulzano", "Tavernole sul Mella", "Tem\u00f9", "Tignale", "Torbole Casaglia", "Toscolano-Maderno", "Travagliato", "Tremosine", "Trenzano", "Treviso Bresciano", "Urago d'Oglio", "Vallio Terme", "Valvestino", "Verolanuova", "Verolavecchia", "Vestone", "Vezza d'Oglio", "Villa Carcina", "Villachiara", "Villanuova sul Clisi", "Vione", "Visano", "Vobarno", "Zone"], "type": "scattermapbox"}],
                        {"mapbox": {"style": "open-street-map"}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('537d9f19-b055-4319-bce9-b705182bb669');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Solving the problem

Now that we have acquired all the necessary data, we can start describing the distance, the distance i've chosen is the rectilinear (also called "Manhattan") distance, due to the fact that is more realistic than the euclidean distance and takes into account for multiple possible paths to reach a city, the formula is:  

$ d_i = |x-x_i| + |y-y_i| $  

$ d_i $ = rectilinear distance between the location the i-th point   
$x$ = x coordinate of our location  
$y$ = y coordinate of our location  
$x_i$ = x coordinate of our i-th point  
$y_i$ = y coordinate of our i-th point  

then we weigh each distance by a factor $ w_i $  
 
$f_i = d_iw_i $

our weight in this case is the city population
  
now we can write the sum of weighted distances between the location and the points as:  

$ F = \sum_{i = 0}^{n}f_i = \sum_{i = 0}^{n}d_iw_i = \sum_{i = 0}^{n}(|x-x_i| + |y-y_i|)w_i$  

and so our objective function will be:  

$min(F) = min(\sum_{i = 0}^{n}(|x-x_i| + |y-y_i|)w_i) $  

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

Now that we've got the latitude and longitude of the optimal location we can plot it on the map


```python
fig = go.Figure(go.Scattermapbox(lat=data.loc[:,"lat"],
                                 lon=data.loc[:,"long"],
                                 name = "Cities",
                                 text = data.loc[:,"City"],
                                 marker_size = 8))
fig.add_trace(go.Scattermapbox(lat = [location[0]],
                               lon = [location[1]],
                               marker_color =  "green",
                               name = "Facility",
                               marker_size = 12))
fig.update_layout(mapbox_style="open-street-map")
fig.show()
```


<div>


            <div id="a72e963b-93c4-414d-9eff-fc55c498a1dd" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("a72e963b-93c4-414d-9eff-fc55c498a1dd")) {
                    Plotly.newPlot(
                        'a72e963b-93c4-414d-9eff-fc55c498a1dd',
                        [{"lat": [45.307378, 45.6224015, 45.650676000000004, 45.266975, 45.76639, 45.891036299999996, 45.8489819, 45.454722, 45.429078999999994, 45.823561299999994, 45.405077, 45.681886999999996, 45.327176, 45.5105711, 45.50298, 46.0934179, 45.931693, 45.935562, 45.672786, 45.346975, 45.477261600000006, 45.9473303, 45.549680200000005, 45.7933072, 45.5931372, 45.454178000000006, 45.9900343, 45.9563241, 45.539802200000004, 45.6429453, 45.6126766, 45.457182, 45.540040000000005, 45.3480525, 46.030996, 45.7544353, 45.454869099999996, 45.6372565, 45.3646065, 45.56071420000001, 45.4995922, 45.4984126, 45.471576299999995, 45.6945498, 45.512504799999995, 45.580508, 46.0766465, 45.5841062, 46.002508500000005, 46.002595, 46.081396999999996, 45.5364624, 45.308636299999996, 46.024096, 45.942493, 45.5634485, 45.58131470000001, 45.8093124, 45.5815203, 45.471278000000005, 45.605184, 45.6374172, 46.166198, 45.443778, 45.883486299999994, 45.41811, 45.4694851, 46.178199, 45.5984995, 45.925692, 45.231475, 45.4828867, 45.2552458, 45.61878529999999, 45.6879136, 45.688138, 45.584627399999995, 45.401081, 45.8645299, 45.291876, 45.5926638, 45.737745399999994, 46.2203, 45.771317100000005, 45.6595554, 45.3091529, 45.739689, 45.3670119, 45.814119399999996, 45.719687, 45.4842379, 45.4610389, 45.437678000000005, 45.983494, 45.9894381, 45.6482576, 45.4769983, 45.781392, 45.449179, 45.9506025, 46.122581, 45.556043100000004, 45.355777, 45.709374, 45.75545, 45.736086, 45.518803999999996, 45.273575, 45.528523, 46.2115706, 45.711994, 45.632189700000005, 45.415181, 45.44638, 45.7139802, 45.562484999999995, 45.5877271, 45.975994, 45.545784000000005, 45.532183, 45.64685, 45.3861328, 45.627283, 46.016895, 45.403976, 45.4206595, 45.5552224, 45.946191999999996, 45.509584000000004, 45.589186600000005, 46.07286015, 45.553076000000004, 45.599068700000004, 45.659876399999995, 46.030996, 45.599486999999996, 45.301255600000005, 45.7444018, 45.7544894, 45.7741272, 45.843752200000004, 45.919692, 45.805731200000004, 45.661484, 45.551185, 45.431977, 45.460662799999994, 46.258603, 45.274374, 45.569379999999995, 45.4059696, 45.265575, 45.668287, 45.930858799999996, 45.5514816, 45.6359706, 45.6848636, 45.5656922, 45.310875, 45.27147, 45.514182, 45.4642998, 45.597645799999995, 45.6167638, 45.5276081, 45.5670385, 45.488578000000004, 45.658587, 45.715453000000004, 45.605677500000006, 45.5895461, 45.307376, 45.370576, 45.4915723, 45.6618702, 46.080496999999994, 46.055988899999996, 45.242674, 45.570284, 45.4688494, 45.527784000000004, 45.6531729, 45.690385, 45.746987, 46.2492143, 45.7397806, 45.5053097, 45.634530299999994, 45.5239055, 45.79190555, 45.477278999999996, 45.71193515, 45.515978000000004, 45.610193200000005, 45.76290825, 45.328581400000004, 45.3270123, 45.7106412, 46.240213399999995, 45.632214000000005, 45.355375, 45.6016804, 46.248402, 45.318378, 45.6436432, 45.763444799999995], "lon": [10.414677000000001, 9.961376, 10.3533352, 10.148172, 10.493771, 10.146098599999998, 10.1645561, 10.1168233, 10.18437, 10.462730800000001, 10.054367, 10.41217, 10.12837, 10.423032300000001, 10.043664999999999, 10.336266, 10.2805867, 10.2947906, 10.338869, 9.968467, 10.2408108, 10.2064036, 10.313259044548099, 10.273755900000001, 10.2434802, 10.052666, 10.3431903, 10.304040400000002, 10.2200214, 10.1469445, 10.315085400000001, 10.413574, 10.445833, 10.345921, 10.345862, 10.54469, 10.130347800000001, 9.9336046, 10.429384599999999, 10.115308800000001, 10.1450083, 9.943281800000001, 10.29831, 10.3212431, 9.977855, 10.0262148, 10.350067999999998, 10.1802449, 10.3257912, 10.351963, 10.368561999999999, 9.928846, 10.189374699999998, 10.365663, 10.277462, 9.9782721, 10.2102711, 10.332872300000002, 9.9411725, 9.950064, 10.216967, 10.0064681, 10.243457000000001, 10.007366000000001, 10.180706800000001, 10.0752034, 10.538946699999999, 10.329158999999999, 9.9722164, 10.250862, 10.324177, 10.175008199999999, 10.2934106, 10.559100500000001, 10.183724400000001, 10.663500599999999, 10.438236999999999, 10.279276, 10.1836526, 10.272374000000001, 10.15345, 10.4783256, 10.358759, 10.2841681, 10.0510489, 10.3221645, 10.43907, 10.2184518, 10.7915786, 10.277667, 10.0532758, 10.4845346, 10.059667, 10.316163000000001, 10.2456097, 10.2645464, 10.0427148, 10.615973, 10.079566999999999, 10.273339300000002, 10.3192116, 10.5625934, 10.13517, 10.2153259, 10.2873147, 10.093762, 10.3541615, 10.199273, 10.5375734, 10.3398783, 10.082235398202599, 10.0985704, 10.390675, 10.228771, 10.3436499, 10.461874, 10.2892353, 10.333263, 10.386772, 10.369172, 10.3884485, 10.117457, 10.121364999999999, 10.328362, 9.921564, 9.9617289, 10.073946000000001, 10.230561, 10.506376, 10.076411, 10.229215757122999, 10.4012548, 9.884966499999999, 9.955070300000001, 10.370963, 10.064917900000001, 10.2084011, 10.3439356, 10.3746705, 10.2373771, 10.1578872, 10.225261999999999, 10.1081901, 10.123963999999999, 10.504875, 9.989265, 10.173909199999999, 10.508662, 10.092471000000002, 9.85346, 10.6313729, 10.217574, 10.39677, 10.3103078, 10.421857000000001, 10.0441674, 10.4366014, 10.51147, 10.008567999999999, 10.3688024528361, 10.316671000000001, 9.9140564, 10.1094679, 10.4958703, 10.149701199999999, 9.9986008, 9.887062, 10.420571, 10.1111391, 10.520069, 10.551732699999999, 10.148671, 10.026767, 10.2170575, 10.195876199999999, 10.399663, 10.3439901, 10.177973, 10.366071, 10.607823699999999, 10.512675, 10.2777117, 10.100963, 10.240565, 10.46805, 10.7217649, 10.1024645, 10.609520800000002, 10.0797833, 10.7311310679556, 10.010665, 10.4622942169115, 9.869661, 10.3901746, 10.5910267515906, 10.076086300000002, 10.0572562, 10.4034707, 10.399732499999999, 10.1944422, 9.930966, 10.4538721, 10.447161, 10.372376, 10.499454, 10.1158821], "marker": {"size": 8}, "name": "Cities", "text": ["Acquafredda", "Adro", "Agnosine", "Alfianello", "Anfo", "Angolo Terme", "Artogne", "Azzano Mella", "Bagnolo Mella", "Bagolino", "Barbariga", "Barghe", "Bassano Bresciano", "Bedizzole", "Berlingo", "Berzo Demo", "Berzo Inferiore", "Bienno", "Bione", "Borgo San Giacomo", "Borgosatollo", "Borno", "Botticino", "Bovegno", "Bovezzo", "Brandico", "Braone", "Breno", "Brescia", "Brione", "Caino", "Calcinato", "Calvagese della Riviera", "Calvisano", "Capo di Ponte", "Capovalle", "Capriano del Colle", "Capriolo", "Carpenedolo", "Castegnato", "Castel Mella", "Castelcovati", "Castenedolo", "Casto", "Castrezzato", "Cazzago San Martino", "Cedegolo", "Cellatica", "Cerveno", "Ceto", "Cevo", "Chiari", "Cigole", "Cimbergo", "Cividate Camuno", "Coccaglio", "Collebeato", "Collio", "Cologne", "Comezzano-Cizzago", "Concesio", "Corte Franca", "Corteno Golgi", "Corzano", "Darfo Boario Terme", "Dello", "Desenzano del Garda", "Edolo", "Erbusco", "Esine", "Fiesse", "Flero", "Gambara", "Gardone Riviera", "Gardone Val Trompia", "Gargnano", "Gavardo", "Ghedi", "Gianico", "Gottolengo", "Gussago", "Idro", "Incudine", "Irma", "Iseo", "Isorella", "Lavenone", "Leno", "Limone sul Garda", "Lodrino", "Lograto", "Lonato del Garda", "Longhena", "Losine", "Lozio", "Lumezzane", "Maclodio", "Magasa", "Mairano", "Malegno", "Malonno", "Manerba del Garda", "Manerbio", "Marcheno", "Marmentino", "Marone", "Mazzano", "Milzano", "Moniga del Garda", "Monno", "Monte Isola", "Monticelli Brusati", "Montichiari", "Montirone", "Mura", "Muscoline", "Nave", "Niardo", "Nuvolento", "Nuvolera", "Odolo", "Offlaga", "Ome", "Ono San Pietro", "Orzinuovi", "Orzivecchi", "Ospitaletto", "Ossimo", "Padenghe sul Garda", "Paderno Franciacorta", "Paisco Loveno", "Paitone", "Palazzolo sull'Oglio", "Paratico", "Paspardo", "Passirano", "Pavone del Mella", "Pertica Alta", "Pertica Bassa", "Pezzaze", "Pian Camuno", "Piancogno", "Pisogne", "Polaveno", "Polpenazze del Garda", "Pompiano", "Poncarale", "Ponte di Legno", "Pontevico", "Pontoglio", "Pozzolengo", "Pralboino", "Preseglie", "Prestine", "Prevalle", "Provaglio d'Iseo", "Provaglio Val Sabbia", "Puegnago sul Garda", "Quinzano d'Oglio", "Remedello", "Rezzato", "Roccafranca", "Rodengo Saiano", "Ro\u00e8 Volciano", "Roncadelle", "Rovato", "Rudiano", "Sabbio Chiese", "Sale Marasino", "Sal\u00f2", "San Felice del Benaco", "San Gervasio Bresciano", "San Paolo", "San Zeno Naviglio", "Sarezzo", "Saviore dell'Adamello", "Sellero", "Seniga", "Serle", "Sirmione", "Soiano del Lago", "Sonico", "Sulzano", "Tavernole sul Mella", "Tem\u00f9", "Tignale", "Torbole Casaglia", "Toscolano-Maderno", "Travagliato", "Tremosine", "Trenzano", "Treviso Bresciano", "Urago d'Oglio", "Vallio Terme", "Valvestino", "Verolanuova", "Verolavecchia", "Vestone", "Vezza d'Oglio", "Villa Carcina", "Villachiara", "Villanuova sul Clisi", "Vione", "Visano", "Vobarno", "Zone"], "type": "scattermapbox"}, {"lat": [45.59934120275753], "lon": [10.220021399923084], "marker": {"color": "green", "size": 12}, "name": "Facility", "type": "scattermapbox"}],
                        {"mapbox": {"style": "open-street-map"}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('a72e963b-93c4-414d-9eff-fc55c498a1dd');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Analyzing the result

The method provides the best location to minimize the average distance between the location and the clients,this is why we take in account the population size (it would be best if we knew the number of clients in each city but it is a necessary approximation in case we have no data), and also minimizes the average travel time.

In the case we have a set of location to choose for our facility, the problem becomes trivial, in fact, with a few adjustments we can take in account the investment necessary to build in a specific location, this will be subject to another work.

A non-trivial problem is the multiple facility problem, which will require an approximate solution through clustering.

It is important to point out that this method is general purpose whenever our problem requires to minimize the distance between a point and a set of other points (whatever and wherever they could be).



