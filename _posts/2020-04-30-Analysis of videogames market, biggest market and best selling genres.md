---
layout: posts
title:  "Analysis of videogames market, biggest market and best selling genres"
description: "Analyzing the videogames market, finding the biggest regional market and the best selling genres, by looking at historical data"
category: "Data Analysis"
#tags: "optimization","location","logistics"
---
We will analyze the sales data of the videogames market using the data about games with more than 100000 copies sold

source: <https://www.kaggle.com/gregorut/videogamesales> 
Code and data used: <https://github.com/davbom97/source>

I assume this data to be somewhat representative sample of the videogames market and I will use it to extract some insights

* Import Data
* Data Cleaning
* Data Analysis
    * Composition of Total Sales by Market
    * Time series of Sales by Market
    * Markets composition by Genre
    * Analysis of Genre Sales
        * Genre sales boxplot
        * Which Genre sells the most per game?




```python
import pandas as pd
import numpy as np
from IPython.display import display,Image
import plotly.graph_objs as go
from plotly.subplots import make_subplots
import plotly.figure_factory as ff
import plotly.express as px

```

## Import Data


```python
data = pd.read_csv("vgsales.csv").sort_values("Global_Sales",ascending= False)
data = data.drop("Rank",axis = 1)

display(data.head(5))
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
      <th>Name</th>
      <th>Platform</th>
      <th>Year</th>
      <th>Genre</th>
      <th>Publisher</th>
      <th>NA_Sales</th>
      <th>EU_Sales</th>
      <th>JP_Sales</th>
      <th>Other_Sales</th>
      <th>Global_Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wii Sports</td>
      <td>Wii</td>
      <td>2006.0</td>
      <td>Sports</td>
      <td>Nintendo</td>
      <td>41.49</td>
      <td>29.02</td>
      <td>3.77</td>
      <td>8.46</td>
      <td>82.74</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Super Mario Bros.</td>
      <td>NES</td>
      <td>1985.0</td>
      <td>Platform</td>
      <td>Nintendo</td>
      <td>29.08</td>
      <td>3.58</td>
      <td>6.81</td>
      <td>0.77</td>
      <td>40.24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mario Kart Wii</td>
      <td>Wii</td>
      <td>2008.0</td>
      <td>Racing</td>
      <td>Nintendo</td>
      <td>15.85</td>
      <td>12.88</td>
      <td>3.79</td>
      <td>3.31</td>
      <td>35.82</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Wii Sports Resort</td>
      <td>Wii</td>
      <td>2009.0</td>
      <td>Sports</td>
      <td>Nintendo</td>
      <td>15.75</td>
      <td>11.01</td>
      <td>3.28</td>
      <td>2.96</td>
      <td>33.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Pokemon Red/Pokemon Blue</td>
      <td>GB</td>
      <td>1996.0</td>
      <td>Role-Playing</td>
      <td>Nintendo</td>
      <td>11.27</td>
      <td>8.89</td>
      <td>10.22</td>
      <td>1.00</td>
      <td>31.37</td>
    </tr>
  </tbody>
</table>
</div>


The data reports for each game:
* The game name (Unique for every game)
* The main platform it was published (Categorical value)
* Year it was bublished (Numeric discrete value)
* Its genre (Categorical value)
* Its publisher (Categorical value)
* Sales by market (Numeric continuous value)

Entries are ranked by descending order of global sales

## Data Cleaning

Because the data is incomplete from 2016 on, we drop all the entries beyond 2015


```python
data = data[data["Year"] <= 2015]
```

Then we check if there are any NaN values in the data


```python
data_na = data.isna().sum()
display(data_na)
```


    Name             0
    Platform         0
    Year             0
    Genre            0
    Publisher       34
    NA_Sales         0
    EU_Sales         0
    JP_Sales         0
    Other_Sales      0
    Global_Sales     0
    dtype: int64


There seems to be some missing values in the "Publisher" category, may be due some minor publishers

Becouse there are only 34 missing values, I decided to delete those entries


```python
data = data.dropna()
```

## Data Analysis

### Composition of Total Sales by Market

We group the sales by year and market to find which one contributes the most to the total sales


```python
market_data = data[["Year","NA_Sales","EU_Sales","JP_Sales","Other_Sales"]].groupby("Year").sum()
display(market_data.head(5))
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
      <th>NA_Sales</th>
      <th>EU_Sales</th>
      <th>JP_Sales</th>
      <th>Other_Sales</th>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1980.0</th>
      <td>10.59</td>
      <td>0.67</td>
      <td>0.00</td>
      <td>0.12</td>
    </tr>
    <tr>
      <th>1981.0</th>
      <td>33.40</td>
      <td>1.96</td>
      <td>0.00</td>
      <td>0.32</td>
    </tr>
    <tr>
      <th>1982.0</th>
      <td>26.92</td>
      <td>1.65</td>
      <td>0.00</td>
      <td>0.31</td>
    </tr>
    <tr>
      <th>1983.0</th>
      <td>7.76</td>
      <td>0.80</td>
      <td>8.10</td>
      <td>0.14</td>
    </tr>
    <tr>
      <th>1984.0</th>
      <td>33.28</td>
      <td>2.10</td>
      <td>14.27</td>
      <td>0.70</td>
    </tr>
  </tbody>
</table>
</div>


There seems to be missing data of JP_Sales from 1980 to 1982, so we drop these years for the analysis


```python
market_data = market_data[market_data.index >1982]
```


```python
display(market_data.sum().reset_index())
fig = px.pie(market_data.sum().reset_index(),values = 0,names = "index")
fig.update_layout(title="Global Sales composition by Market")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
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
      <th>index</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NA_Sales</td>
      <td>4233.81</td>
    </tr>
    <tr>
      <th>1</th>
      <td>EU_Sales</td>
      <td>2375.65</td>
    </tr>
    <tr>
      <th>2</th>
      <td>JP_Sales</td>
      <td>1270.55</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Other_Sales</td>
      <td>780.39</td>
    </tr>
  </tbody>
</table>
</div>





![output_22_1.png](/assets/Analysis of videogames market, biggest market and best selling genres/output_22_1.png)




From the data emerges that the biggest market is the North American one, making up to almost the 50% of the total sales, followed by the European and the Japanese ones.

### Time series of Sales per Market

Now we want to see the time series of the total sales and the sales per market


```python
fig = go.Figure()

fig.add_trace(go.Scatter(x = market_data.index, y= market_data.sum(axis = 1), mode = "lines+markers"))

fig.update_layout(title="Time Series of Total Sales",
                   xaxis_title="Total Sales (Mln)",
                   yaxis_title="Year")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_26_0.png)




```python
fig = go.Figure()
fig.add_trace(go.Scatter(x = market_data.index, y= market_data["NA_Sales"], name = "NA Sales", mode = "lines+markers"))
fig.add_trace(go.Scatter(x = market_data.index, y= market_data["EU_Sales"], name = "EU Sales", mode = "lines+markers"))
fig.add_trace(go.Scatter(x = market_data.index, y= market_data["JP_Sales"], name = "JP Sales", mode = "lines+markers"))
fig.add_trace(go.Scatter(x = market_data.index, y= market_data["Other_Sales"], name = "Other Markets Sales", mode = "lines+markers"))
fig.add_trace(go.Scatter(x = market_data.index, y= market_data.sum(axis = 1), name = "Total Sales" ,mode = "lines+markers"))
fig.update_layout(title="Time Series of Market Sales",
                   xaxis_title="Total Sales (Mln)",
                   yaxis_title="Year")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_27_0.png)




```python
market_sales_change = (market_data.pct_change()*100).round(2)
total_sales_change = (market_data.sum(axis = 1).pct_change()*100).round(2)

fig = make_subplots(rows=5, cols=1, shared_yaxes = True)
fig.add_trace(go.Scatter(x = market_sales_change.index, y= market_sales_change["NA_Sales"], name = "NA Sales", mode = "lines+markers"),row =1,col = 1)
fig.add_trace(go.Scatter(x = market_sales_change.index, y= market_sales_change["EU_Sales"], name = "EU Sales", mode = "lines+markers"),row =2,col = 1)
fig.add_trace(go.Scatter(x = market_sales_change.index, y= market_sales_change["JP_Sales"], name = "JP Sales", mode = "lines+markers"),row =3,col = 1)
fig.add_trace(go.Scatter(x = market_sales_change.index, y= market_sales_change["Other_Sales"], name = "Other Markets Sales", mode = "lines+markers"),row =4,col = 1)
fig.add_trace(go.Scatter(x = total_sales_change.index, y= total_sales_change, name = "Total Sales" ,mode = "lines+markers"),row =5,col = 1)
fig.update_layout(title="Rate of change of Market Sales",
                   xaxis_title="Year",
                   yaxis_title="Rate of change (%)",
                   height=800)
img_bytes = fig.to_image(format="png")
Image(img_bytes)

```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_28_0.png)



The data shows the growth of the videogames markets and the effects of the economic cycles and bubbles (the dot-com bubble in 2000, the crisis of 2008 on the NA sales and the European debt crisis of 2010-2011 on the EU markets).

In general, the market peaked in 2008, right before the economi crisis of those years, while showing a down trend in 2015.

While it is obvious the impact of the economic cycle on the sales, it must be noted that some changes in the industry might have shifted the focus of the companies and the customers(Microtransactions, DLCs, Free to Play business models, etc), indicating that the reduction of sales does not mean that the industry is in a bad shape, but just that now there are more revenue sources for the companies besides game sales than in the past.

Looking at the average growth rates of each market:


```python
display(market_sales_change.mean().round(2))
```


    NA_Sales       28.72
    EU_Sales       38.28
    JP_Sales        7.88
    Other_Sales    46.86
    dtype: float64


And the total sales growth rate:


```python
display("The total sales average growth rate is : %.2f" % total_sales_change.mean())
```


    'The total sales average growth rate is : 19.10'


### Markets composition by Genre

Now we can see which are the best selling genres and platforms for each market using a heatmap: 


```python
market_sales_genre = data.drop(["Global_Sales","Year"], axis = 1).groupby("Genre").sum()
market_sales_genre = (market_sales_genre.div(market_sales_genre.sum(axis = 0),axis = 1)*100).round(2)
fig = ff.create_annotated_heatmap(z = market_sales_genre.values,
                                  y = market_sales_genre.index.values.tolist(),
                                  x = market_sales_genre.columns.values.tolist())
fig.update_layout(title="Composition of each market by genre (%)")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_36_0.png)




```python
print("The top 3 selling genres in the NA market are:")
print(market_sales_genre["NA_Sales"].sort_values(ascending = False)[:3])

print("\nThe top 3 selling genres in the EU market are:")
print(market_sales_genre["EU_Sales"].sort_values(ascending = False)[:3])

print("\nThe top 3 selling genres in the JP market are:")
print(market_sales_genre["JP_Sales"].sort_values(ascending = False)[:3])

print("\nThe top 3 selling genres in other markets are:")
print(market_sales_genre["Other_Sales"].sort_values(ascending = False)[:3])
```

    The top 3 selling genres in the NA market are:
    Genre
    Action     19.88
    Sports     15.46
    Shooter    13.19
    Name: NA_Sales, dtype: float64
    
    The top 3 selling genres in the EU market are:
    Genre
    Action     21.43
    Sports     15.29
    Shooter    12.72
    Name: EU_Sales, dtype: float64
    
    The top 3 selling genres in the JP market are:
    Genre
    Role-Playing    27.28
    Action          12.03
    Sports          10.55
    Name: JP_Sales, dtype: float64
    
    The top 3 selling genres in other markets are:
    Genre
    Action     23.44
    Sports     16.74
    Shooter    12.74
    Name: Other_Sales, dtype: float64
    

### Composition of Total Sales by Genre

Like we did for the markets, we group the sales by Genre and Year:


```python
genre_sales = data[["Year","Genre","Global_Sales"]].groupby(["Year","Genre"]).sum().unstack(level=-1).droplevel(0,axis = 1)
display(genre_sales.head(12))
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
      <th>Genre</th>
      <th>Action</th>
      <th>Adventure</th>
      <th>Fighting</th>
      <th>Misc</th>
      <th>Platform</th>
      <th>Puzzle</th>
      <th>Racing</th>
      <th>Role-Playing</th>
      <th>Shooter</th>
      <th>Simulation</th>
      <th>Sports</th>
      <th>Strategy</th>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>1980.0</th>
      <td>0.34</td>
      <td>NaN</td>
      <td>0.77</td>
      <td>2.71</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7.07</td>
      <td>NaN</td>
      <td>0.49</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1981.0</th>
      <td>14.84</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.93</td>
      <td>2.24</td>
      <td>0.48</td>
      <td>NaN</td>
      <td>10.04</td>
      <td>0.45</td>
      <td>0.79</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1982.0</th>
      <td>6.52</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.87</td>
      <td>5.03</td>
      <td>10.03</td>
      <td>1.57</td>
      <td>NaN</td>
      <td>3.79</td>
      <td>NaN</td>
      <td>1.05</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1983.0</th>
      <td>2.86</td>
      <td>0.40</td>
      <td>NaN</td>
      <td>2.14</td>
      <td>6.93</td>
      <td>0.78</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.48</td>
      <td>NaN</td>
      <td>3.20</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1984.0</th>
      <td>1.85</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.45</td>
      <td>0.69</td>
      <td>3.14</td>
      <td>5.95</td>
      <td>NaN</td>
      <td>31.10</td>
      <td>NaN</td>
      <td>6.18</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1985.0</th>
      <td>3.52</td>
      <td>NaN</td>
      <td>1.05</td>
      <td>NaN</td>
      <td>43.17</td>
      <td>3.21</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.00</td>
      <td>0.03</td>
      <td>1.96</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1986.0</th>
      <td>13.74</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.39</td>
      <td>NaN</td>
      <td>1.96</td>
      <td>2.52</td>
      <td>3.89</td>
      <td>NaN</td>
      <td>5.57</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1987.0</th>
      <td>1.12</td>
      <td>4.38</td>
      <td>5.42</td>
      <td>NaN</td>
      <td>1.74</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.65</td>
      <td>0.71</td>
      <td>NaN</td>
      <td>3.72</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1988.0</th>
      <td>1.75</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>27.73</td>
      <td>5.58</td>
      <td>2.14</td>
      <td>5.88</td>
      <td>0.51</td>
      <td>0.03</td>
      <td>3.60</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1989.0</th>
      <td>4.64</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.28</td>
      <td>20.66</td>
      <td>37.75</td>
      <td>NaN</td>
      <td>2.20</td>
      <td>1.20</td>
      <td>NaN</td>
      <td>5.72</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1990.0</th>
      <td>6.39</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22.97</td>
      <td>6.00</td>
      <td>6.26</td>
      <td>4.52</td>
      <td>NaN</td>
      <td>1.14</td>
      <td>2.11</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1991.0</th>
      <td>6.76</td>
      <td>2.24</td>
      <td>0.39</td>
      <td>0.08</td>
      <td>7.64</td>
      <td>3.24</td>
      <td>1.14</td>
      <td>3.25</td>
      <td>2.00</td>
      <td>2.14</td>
      <td>2.41</td>
      <td>0.94</td>
    </tr>
  </tbody>
</table>
</div>


There are missing values (probably due to the genre not being defined at the time, or not existing), so we delete all the entries before 1991.


```python
genre_sales = genre_sales[genre_sales.index >= 1991]
```


```python
fig = px.pie(genre_sales.sum(),values = 0,names = genre_sales.columns)
fig.update_layout(title="Global Sales composition by Genre")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_43_0.png)



Here we can see that the best selling Genres are:
* Action
* Sports
* Shooter and Role-Playing

### Analysis of Genre Sales

We want to ask ourselves "Which Genres are, on average, the best selling?"

Looking at the total sales does not tell us the full story, because the most of sales are done only by few very big hits and are dependent on the count of games relased:


```python
data_1991_on = data[data["Year"]>=1991]
genre_sales_desc = data_1991_on[["Genre","Global_Sales"]].groupby("Genre").describe().droplevel(0,axis = 1)
genre_sales_desc = genre_sales_desc.sort_values(by =["50%"],axis = 0,ascending = False)
display(genre_sales_desc)
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>Genre</th>
      <th></th>
      <th></th>
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
      <th>Platform</th>
      <td>829.0</td>
      <td>0.822461</td>
      <td>2.005954</td>
      <td>0.01</td>
      <td>0.090</td>
      <td>0.250</td>
      <td>0.6900</td>
      <td>30.01</td>
    </tr>
    <tr>
      <th>Shooter</th>
      <td>1220.0</td>
      <td>0.777205</td>
      <td>1.692879</td>
      <td>0.01</td>
      <td>0.080</td>
      <td>0.220</td>
      <td>0.7100</td>
      <td>14.76</td>
    </tr>
    <tr>
      <th>Sports</th>
      <td>2240.0</td>
      <td>0.562612</td>
      <td>2.127120</td>
      <td>0.01</td>
      <td>0.090</td>
      <td>0.220</td>
      <td>0.5525</td>
      <td>82.74</td>
    </tr>
    <tr>
      <th>Fighting</th>
      <td>818.0</td>
      <td>0.529279</td>
      <td>0.960339</td>
      <td>0.01</td>
      <td>0.080</td>
      <td>0.210</td>
      <td>0.5500</td>
      <td>13.04</td>
    </tr>
    <tr>
      <th>Action</th>
      <td>3063.0</td>
      <td>0.537173</td>
      <td>1.185337</td>
      <td>0.01</td>
      <td>0.070</td>
      <td>0.190</td>
      <td>0.5000</td>
      <td>21.40</td>
    </tr>
    <tr>
      <th>Racing</th>
      <td>1195.0</td>
      <td>0.591431</td>
      <td>1.689209</td>
      <td>0.01</td>
      <td>0.075</td>
      <td>0.190</td>
      <td>0.5300</td>
      <td>35.82</td>
    </tr>
    <tr>
      <th>Role-Playing</th>
      <td>1417.0</td>
      <td>0.633211</td>
      <td>1.741657</td>
      <td>0.01</td>
      <td>0.070</td>
      <td>0.190</td>
      <td>0.5300</td>
      <td>31.37</td>
    </tr>
    <tr>
      <th>Misc</th>
      <td>1660.0</td>
      <td>0.470030</td>
      <td>1.340625</td>
      <td>0.01</td>
      <td>0.060</td>
      <td>0.160</td>
      <td>0.4100</td>
      <td>29.02</td>
    </tr>
    <tr>
      <th>Simulation</th>
      <td>834.0</td>
      <td>0.464808</td>
      <td>1.216313</td>
      <td>0.01</td>
      <td>0.060</td>
      <td>0.160</td>
      <td>0.4300</td>
      <td>24.76</td>
    </tr>
    <tr>
      <th>Puzzle</th>
      <td>549.0</td>
      <td>0.315993</td>
      <td>0.839252</td>
      <td>0.01</td>
      <td>0.040</td>
      <td>0.100</td>
      <td>0.2700</td>
      <td>15.30</td>
    </tr>
    <tr>
      <th>Strategy</th>
      <td>660.0</td>
      <td>0.261773</td>
      <td>0.527860</td>
      <td>0.01</td>
      <td>0.040</td>
      <td>0.095</td>
      <td>0.2800</td>
      <td>5.45</td>
    </tr>
    <tr>
      <th>Adventure</th>
      <td>1239.0</td>
      <td>0.184036</td>
      <td>0.503521</td>
      <td>0.01</td>
      <td>0.020</td>
      <td>0.060</td>
      <td>0.1600</td>
      <td>11.18</td>
    </tr>
  </tbody>
</table>
</div>


Looking at the histogram of the distribution of global sales for all games:


```python
fig = px.histogram(data_1991_on["Global_Sales"], x="Global_Sales", nbins=50)
fig.update_layout(title="Histogram of the distribution of Global Sales",
                   xaxis_title="Global Sales")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_49_0.png)



Calculating the 95th percentile:


```python
print("The 95th percentile is equal to : %f" % (data_1991_on["Global_Sales"].quantile(0.95)))
```

    The 95th percentile is equal to : 2.008500
    

Meaning that the 95% of games in this sample made less than 2 milion copies sold


```python
relative_total = 0
i = 0
total = data_1991_on["Global_Sales"].sum()
sales = data_1991_on[["Global_Sales"]]
pareto = []
while relative_total <= 0.8:
    game_sales_perc = data_1991_on["Global_Sales"].iloc[i]/total
    relative_total += game_sales_perc
    i+=1
print("The %f%% of games makes up for the %f%% of sales" % (i/len(sales)*100, relative_total*100))

```

    The 25.572373% of games makes up for the 80.003392% of sales
    

Another evidence of the skewness is that just around 25% percent of games make the 80% total sales

#### Genre Sales boxplot

Within every Genre there is a huge variance, in support of our hypothesis we can see that the median sales are lower than the average sales, it is even more evident with a boxplot:


```python
fig = px.box(data_1991_on, y="Global_Sales",x = "Genre")
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_57_0.png)



The outliers (best selling games) make the boxplot unreadable, zooming in the boxes:


```python
Image(filename="boxplot_zoomed.png") 
```




![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_59_0.png)



#### Which Genre sells the most per game?

To answer this question we have to take in account the variability within each genre, thus we will evaluate the best selling genre per game using the quartile coefficient of dispersion:


```python
genre_sales_qcd = ((genre_sales_desc["75%"]-genre_sales_desc["25%"])/(genre_sales_desc["75%"]+genre_sales_desc["25%"])).sort_values(ascending = False)
print("The quartile coefficient of dispersion for each Genre is:")
display(genre_sales_qcd)

fig = go.Figure(go.Bar(x = genre_sales_qcd.index, y = genre_sales_qcd.values, text = genre_sales_qcd.round(2),textposition='outside'))
fig.update_layout(title="Quartile coefficient of dispersion",
                   yaxis_title="qcd",
                   height=500)
img_bytes = fig.to_image(format="png")
Image(img_bytes)
```

    The quartile coefficient of dispersion for each Genre is:
    


    Genre
    Shooter         0.797468
    Adventure       0.777778
    Platform        0.769231
    Role-Playing    0.766667
    Simulation      0.755102
    Action          0.754386
    Racing          0.752066
    Strategy        0.750000
    Fighting        0.746032
    Misc            0.744681
    Puzzle          0.741935
    Sports          0.719844
    dtype: float64





![png](/assets/Analysis of videogames market, biggest market and best selling genres/output_62_2.png)



Note: When comparing two genres, the one with the less coefficient of variation has lower variability between each game

Now, as written above, on average the best selling genres are:
* Platform
* Shooter
* Sports


```python
for genre in genre_sales_desc.index[:3]:
    print("The Genre %s qcv: %f median : %f" %(genre, genre_sales_qcd.loc[genre],genre_sales_desc.loc[genre]["50%"]))
```

    The Genre Platform qcv: 0.769231 median : 0.250000
    The Genre Shooter qcv: 0.797468 median : 0.220000
    The Genre Sports qcv: 0.719844 median : 0.220000
    

## Conclusions

Among the videogames markets, the NA is the biggest one, followed by EU and JP

The most sold Genres are Action, Sports, Shooter, and make up a significant portion of videogames sales in each market, with the exeption of Role-Playing, which is sold a lot in the JP market, making up the 27.3% of sales alone in that market.

Looking at the median copies sold by each game within a genre, We have that the Platform, Shooter and Sports games are on average the best selling, the median of the Platform genre in bigger than the Shooter one, while having a smaller quartile coefficient of dispersion, meaning that it has a lower variability of copies sold from game to game, meaning that overall the Platform Genre is better selling than the Shooting Genre.

The Shooter Genre has a equal median and a bigger qcd than the Sports, meaning that they sell on average the same, but the Shooter has more variability in sales, thus concluding that the top three selling Genres, on average, are in order:

* Platform
* Sports
* Shooter
