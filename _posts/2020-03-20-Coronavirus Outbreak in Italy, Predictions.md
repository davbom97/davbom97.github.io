---
layout: posts
title:  "Coronavirus Outbreak in Italy, Predictions"
description: "In this post we try to predict the trend of the SARS-CoV-2 in italy"
category: "Data Analysis"
#tags: "optimization","location","logistics"
---

In this post we try to give a prediction the of SARS-CoV-2 outbreak in Italy.

To do this we assume that the number of confirmed cases follows a logistc function:

$$ f(x) = \frac{L}{1+e^{-k(x-x_0)}} $$  

where:

$$k = \textrm{the logistic growth rate or steepness of the curve} $$
$$L = \textrm{the curve's maximum value} $$
$$x_0 = \textrm{the x-value of the curve's midpoint} $$

By estimating the values of $$ k,L,x_0 $$ from the data available we can try to predict the future cases

### Data manipulation

first we import the libraries we will use and the daily data of the italian outbreak from 24/02/2020 until 20/03/2020:


```python
import plotly.graph_objects as go
import pandas as pd
from scipy.optimize import curve_fit
import numpy as np
from sklearn import preprocessing
from IPython.display import display
```


```python
italy_data = pd.read_csv("dpc-covid19-ita-andamento-nazionale 20-03-2020.csv")
italy_data = pd.DataFrame({"date":italy_data["data"],
                           "total_cases":italy_data["totale_casi"],
                           "recovered":italy_data["dimessi_guariti"],
                           "deaths":italy_data["deceduti"],
                           "infected":italy_data["totale_attualmente_positivi"],
                           "new_infected":italy_data["nuovi_attualmente_positivi"]})
display(italy_data.head(10))

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
      <th>date</th>
      <th>total_cases</th>
      <th>recovered</th>
      <th>deaths</th>
      <th>infected</th>
      <th>new_infected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-02-24 18:00:00</td>
      <td>229</td>
      <td>1</td>
      <td>7</td>
      <td>221</td>
      <td>221</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-02-25 18:00:00</td>
      <td>322</td>
      <td>1</td>
      <td>10</td>
      <td>311</td>
      <td>90</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-02-26 18:00:00</td>
      <td>400</td>
      <td>3</td>
      <td>12</td>
      <td>385</td>
      <td>74</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-02-27 18:00:00</td>
      <td>650</td>
      <td>45</td>
      <td>17</td>
      <td>588</td>
      <td>203</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-02-28 18:00:00</td>
      <td>888</td>
      <td>46</td>
      <td>21</td>
      <td>821</td>
      <td>233</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2020-02-29 18:00:00</td>
      <td>1128</td>
      <td>50</td>
      <td>29</td>
      <td>1049</td>
      <td>228</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2020-03-01 18:00:00</td>
      <td>1694</td>
      <td>83</td>
      <td>34</td>
      <td>1577</td>
      <td>528</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020-03-02 18:00:00</td>
      <td>2036</td>
      <td>149</td>
      <td>52</td>
      <td>1835</td>
      <td>258</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2020-03-03 18:00:00</td>
      <td>2502</td>
      <td>160</td>
      <td>79</td>
      <td>2263</td>
      <td>428</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2020-03-04 18:00:00</td>
      <td>3089</td>
      <td>276</td>
      <td>107</td>
      <td>2706</td>
      <td>443</td>
    </tr>
  </tbody>
</table>
</div>


For now we delete the "date" column


```python
italy_data = italy_data.drop("date",axis = 1)
```

we scale the data between 0 and 1:


```python
scaler_y = preprocessing.MinMaxScaler()
bounds = np.array([0,100000]).reshape(-1,1)
scaler_y.fit(bounds)
italy_data_scaled = scaler_y.transform(italy_data.values)
italy_data_scaled = pd.DataFrame(italy_data_scaled, columns = italy_data.columns)
```

From the logistic function we defined in the beginning :

$$ f(x) = \frac{L}{1+e^{-k(x-x_0)}} $$  

We define the independent variable x as the number of days after the first entry, for example:

$$ f(0) = \textrm{values at 02/02/2020} $$

$$ f(1) = \textrm{values at 03/02/2020} $$

$$ f(22) = \textrm{values at 24/02/2020} $$

the values of x will also be scaled between 0 and 1


```python
scaler_x = preprocessing.MinMaxScaler()
period_x = scaler_x.fit_transform(np.arange(0,60).reshape(-1,1)).flatten()
italy_data_scaled.index = scaler_x.transform(italy_data_scaled.index.values.reshape(-1,1)).flatten()
```

## Defining and fitting the logistic curve

Finally we define code the logistic function:


```python
logistic = lambda x,k,x_0,l : l/(1+np.exp(-k*(x-x_0)))
```

And we find the best fit


```python
popt_tot_cases,pcov_tot_cases = curve_fit(f = logistic,
                                          xdata = italy_data_scaled.index.values,
                                          ydata = italy_data_scaled["total_cases"].values.flatten(),
                                          maxfev=3000)
```

To estimate the L parameter for the recovered and death curve we calculate the rate of recoveries and deaths first:

$$ \textrm{"%"}_{\textrm{recovered}} = \frac{N_{\textrm{recovered}}}{N_{\textrm{recovered}}+N_{\textrm{deaths}}} $$

$$ \textrm{"%"}_{\textrm{deaths}} = \frac{N_{\textrm{deaths}}}{N_{\textrm{recovered}}+N_{\textrm{deaths}}} $$

we use these percentages to express the maximum deaths and recoveries as a fraction of the total cases:

$$ L_{\textrm{recovered}} = L_\textrm{tot} *\textrm{"%"}_\textrm{recovered}$$

$$ L_{\textrm{deaths}} = L_\textrm{tot} *\textrm{"%"}_\textrm{deaths}$$


```python
perc_deaths = italy_data_scaled["deaths"].sum()/(italy_data_scaled["deaths"].sum()+italy_data_scaled["recovered"].values.sum())
perc_recovered = italy_data_scaled["recovered"].values.sum()/(italy_data_scaled["deaths"].sum()+italy_data_scaled["recovered"].values.sum())
print("The percentage of deaths is : %.3f \nThe percentage of recoveries is : %.3f" %(perc_deaths,perc_recovered))
```

    The percentage of deaths is : 0.427 
    The percentage of recoveries is : 0.573
    

With this information we can estimate the parameters for the recoveries and deaths curves:


```python
popt_recovered,pcov_recovered = curve_fit(f = logistic,
                                          xdata = italy_data_scaled.index.values,
                                          ydata = italy_data_scaled["recovered"].values.flatten(),
                                          bounds = ([0,0,perc_recovered*popt_tot_cases[2]-1e-12],[np.inf,np.inf,perc_recovered*popt_tot_cases[2]]),
                                          maxfev=3000)
popt_deaths,pcov_deaths = curve_fit(f = logistic,
                                    xdata = italy_data_scaled.index.values,
                                    ydata = italy_data_scaled["deaths"].values.flatten(),
                                    bounds = ([0,0,perc_deaths*popt_tot_cases[2]-1e-12],[np.inf,np.inf,perc_deaths*popt_tot_cases[2]]),
                                    maxfev=3000)
```

Finally we predict the values for the confirmed cases,recovered and deaths, the actual infected curve is calculated using the following equation: 

$$ f(x) = N_{\textrm{infected}}(x)+N_{\textrm{deaths}}(x)+N_{\textrm{recovered}}(x) $$

therefore:

$$  N_{\textrm{infected}}(x) = f(x)-N_{\textrm{deaths}}(x)-N_{\textrm{recovered}}(x) $$



```python
period_x = scaler_x.transform(np.arange(0,70).reshape(-1,1)).flatten()
predicted_cases_italy = logistic(period_x,
                                 popt_tot_cases[0],
                                 popt_tot_cases[1],
                                 popt_tot_cases[2])
predicted_recovered_italy = logistic(period_x,
                                     popt_recovered[0],
                                     popt_recovered[1],
                                     popt_recovered[2])
predicted_deaths_italy = logistic(period_x,
                                  popt_deaths[0],
                                  popt_deaths[1],
                                  popt_deaths[2])
italy_dataframe = pd.DataFrame()

italy_dataframe = pd.concat([italy_dataframe,pd.DataFrame({"predicted_cases":scaler_y.inverse_transform(predicted_cases_italy.reshape(-1,1)).flatten()})],axis = 1)
italy_dataframe["predicted_cases"] = italy_dataframe["predicted_cases"].astype(int)

italy_dataframe = pd.concat([italy_dataframe,pd.DataFrame({"predicted_recovered":scaler_y.inverse_transform(predicted_recovered_italy.reshape(-1,1)).flatten()})],axis = 1)
italy_dataframe["predicted_recovered"] = italy_dataframe["predicted_recovered"].astype(int)

italy_dataframe = pd.concat([italy_dataframe,pd.DataFrame({"predicted_deaths":scaler_y.inverse_transform(predicted_deaths_italy.reshape(-1,1)).flatten()})],axis = 1)
italy_dataframe["predicted_deaths"] = italy_dataframe["predicted_deaths"].astype(int)

predicted_infected =pd.DataFrame({"predicted_infected":italy_dataframe["predicted_cases"].values-italy_dataframe["predicted_recovered"].values-italy_dataframe["predicted_deaths"].values})
italy_dataframe = pd.concat([italy_dataframe,predicted_infected],axis = 1)

predicted_infected_change =pd.DataFrame({"predicted_infected_change":italy_dataframe["predicted_infected"].iloc[1:].values-italy_dataframe["predicted_infected"].iloc[0:-1].values})
italy_dataframe = italy_dataframe = pd.concat([italy_dataframe,predicted_infected_change],axis = 1)

italy_dataframe.index = pd.date_range(start="2020-02-24", periods=len(period_x))
italy_data.index = pd.date_range(start="2020-02-24", periods=len(italy_data))
```

Now we take a look at the predictions:


```python
display(italy_dataframe)
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
      <th>predicted_cases</th>
      <th>predicted_recovered</th>
      <th>predicted_deaths</th>
      <th>predicted_infected</th>
      <th>predicted_infected_change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-02-24</th>
      <td>536</td>
      <td>59</td>
      <td>37</td>
      <td>440</td>
      <td>101.0</td>
    </tr>
    <tr>
      <th>2020-02-25</th>
      <td>657</td>
      <td>71</td>
      <td>45</td>
      <td>541</td>
      <td>125.0</td>
    </tr>
    <tr>
      <th>2020-02-26</th>
      <td>805</td>
      <td>85</td>
      <td>54</td>
      <td>666</td>
      <td>153.0</td>
    </tr>
    <tr>
      <th>2020-02-27</th>
      <td>987</td>
      <td>102</td>
      <td>66</td>
      <td>819</td>
      <td>186.0</td>
    </tr>
    <tr>
      <th>2020-02-28</th>
      <td>1208</td>
      <td>123</td>
      <td>80</td>
      <td>1005</td>
      <td>229.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-04-29</th>
      <td>94218</td>
      <td>53695</td>
      <td>40070</td>
      <td>453</td>
      <td>-76.0</td>
    </tr>
    <tr>
      <th>2020-04-30</th>
      <td>94223</td>
      <td>53749</td>
      <td>40097</td>
      <td>377</td>
      <td>-63.0</td>
    </tr>
    <tr>
      <th>2020-05-01</th>
      <td>94228</td>
      <td>53794</td>
      <td>40120</td>
      <td>314</td>
      <td>-53.0</td>
    </tr>
    <tr>
      <th>2020-05-02</th>
      <td>94231</td>
      <td>53832</td>
      <td>40138</td>
      <td>261</td>
      <td>-43.0</td>
    </tr>
    <tr>
      <th>2020-05-03</th>
      <td>94234</td>
      <td>53863</td>
      <td>40153</td>
      <td>218</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>70 rows × 5 columns</p>
</div>


## Visualizing results

**Real number of confirmed cases vs predicted cases:**


```python
fig = go.Figure(data=go.Scatter(x=italy_dataframe.index,
                                y=italy_dataframe["predicted_cases"],
                                mode='markers',
                                name = "Predicted Cases"))
fig.add_trace(go.Scatter(x=italy_dataframe.index, y=italy_data["total_cases"],
                    mode='markers',
                    name = "Real Cases"))
fig.update_xaxes(nticks=25)
fig.update_yaxes(nticks=25)

fig.show("png", width=1024, height=768, scale=2)
```

![output_26_0.png](/assets/2020-03-20-Coronavirus Outbreak in Italy, Predictions/output_26_0.png)


**Real number of recovered vs predicted recovered:**


```python
fig = go.Figure(data = go.Scatter(x=italy_dataframe.index,
                                  y=italy_dataframe["predicted_recovered"],
                                  mode='markers',
                                  name = "Predicted Recovered"))
fig.add_trace(go.Scatter(x=italy_dataframe.index,
                         y=italy_data["recovered"],
                         mode='markers',
                         name = "Recovered"))
fig.update_xaxes(nticks=25)
fig.update_yaxes(nticks=25)

fig.show("png", width=1024, height=768, scale=2)
```


![output_28_0.png](/assets/2020-03-20-Coronavirus Outbreak in Italy, Predictions/output_28_0.png)


**Real number of deaths vs predicted deaths:**


```python
fig = go.Figure(data=go.Scatter(x=italy_dataframe.index,
                                y=italy_dataframe["predicted_deaths"],
                                mode='markers',
                                name = "Predicted Deaths"))
fig.add_trace(go.Scatter(x=italy_dataframe.index,
                         y=italy_data["deaths"],
                         mode='markers',
                         name = "Deaths"))
fig.update_xaxes(nticks=25)
fig.update_yaxes(nticks=25)

fig.show("png", width=1024, height=768, scale=2)
```


![output_30_0.png](/assets/2020-03-20-Coronavirus Outbreak in Italy, Predictions/output_30_0.png)


**Real number of infected vs predicted infected:**


```python
fig = go.Figure(data=go.Scatter(x=italy_dataframe.index,
                                y=italy_dataframe["predicted_infected"],
                                mode='markers',
                                name = "Predicted Infected"))
fig.add_trace(go.Scatter(x=italy_dataframe.index,
                         y=italy_data["infected"],
                         mode='markers',
                         name = "Infected"))
fig.update_xaxes(nticks=25)
fig.update_yaxes(nticks=25)

fig.show("png", width=1024, height=768, scale=2)
```


![output_32_0.png](/assets/2020-03-20-Coronavirus Outbreak in Italy, Predictions/output_32_0.png)


**Putting it all together :**


```python
fig = go.Figure(data=go.Scatter(x=italy_dataframe.index, y=italy_dataframe["predicted_cases"], mode='markers',name = "Predicted Cases"))

fig.add_trace(go.Scatter(x=italy_dataframe.index, y=italy_dataframe["predicted_recovered"],
                    mode='markers',
                    name = "Predicted Recovered",
                    marker_color = "green"))
fig.add_trace(go.Scatter(x=italy_dataframe.index, y=italy_dataframe["predicted_deaths"],
                    mode='markers',
                    name = "Predicted Deaths",
                    marker_color = "red"))
fig.add_trace(go.Scatter(x=italy_dataframe.index, y=italy_dataframe["predicted_infected"],
                    mode='markers',
                    name = "Predicted Infected",
                    marker_color='purple'))
fig.update_xaxes(nticks=25)
fig.update_yaxes(nticks=25)

fig.show("png", width=1024, height=768, scale=2)
```


![output_34_0.png](/assets/2020-03-20-Coronavirus Outbreak in Italy, Predictions/output_34_0.png)



```python
infected_peak_date = italy_dataframe[italy_dataframe["predicted_infected_change"]<0].index[0]
print("The expected infected peak date is : %s-%s-%s" % (infected_peak_date.day,infected_peak_date.month,infected_peak_date.year))
display(italy_dataframe[italy_dataframe.index == infected_peak_date])
```

    The expected infected peak date is : 26-3-2020
    


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
      <th>predicted_cases</th>
      <th>predicted_recovered</th>
      <th>predicted_deaths</th>
      <th>predicted_infected</th>
      <th>predicted_infected_change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-03-26</th>
      <td>72221</td>
      <td>13213</td>
      <td>10780</td>
      <td>48228</td>
      <td>-235.0</td>
    </tr>
  </tbody>
</table>
</div>


## Notes

An important observation must be made to the percentage of deaths, which seems high compared to the one observed in china and other countries during the outbreak

You can find the code used on my github repository <https://github.com/davbom97/source>

Source of data used: <https://github.com/pcm-dpc/COVID-19>
