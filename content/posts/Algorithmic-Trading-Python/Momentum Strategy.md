---
title: "Momentum Strategy S&P 500"
date: 2020-12-09T06:00:23+06:00
hero: /images/posts/writing-posts/analytics.svg
menu:
  sidebar:
    name: Momentum Strategy S&P 500
    identifier: Momentum Strategy S&P 500-algorithmic-trading-python
    parent: Algorithmic-Trading-Python
    weight: 111
---
# Quantitative Momentum Strategy

"Momentum investing" means investing in the stocks that have increased in price the most.
## Library Imports


```python
import numpy as np #The Numpy numerical computing library
import pandas as pd #The Pandas data science library
import requests #The requests library for HTTP requests in Python
import xlsxwriter #The XlsxWriter libarary for
import math #The Python math module
from scipy import stats #The SciPy stats module
from scipy.stats import percentileofscore as score
```

## Importing Our List of Stocks


```python
stocks = pd.read_csv('sp_500_stocks.csv')
from secrets import IEX_CLOUD_API_TOKEN
```

## Making Our First API Call


```python
symbol = 'AAPL'
api_url = f'https://sandbox.iexapis.com/stable/stock/{symbol}/stats?token={IEX_CLOUD_API_TOKEN}'
data = requests.get(api_url).json()
data
```




    {'companyName': 'Apple Inc',
     'marketcap': 2229290738938,
     'week52high': 138.38,
     'week52low': 57.07,
     'week52change': 0.7756040098878855,
     'sharesOutstanding': 17661460248,
     'float': 0,
     'avg10Volume': 117212549,
     'avg30Volume': 110980157,
     'day200MovingAvg': 118.16,
     'day50MovingAvg': 127.64,
     'employees': 0,
     'ttmEPS': 3.33,
     'ttmDividendRate': 0.8342779339390235,
     'dividendYield': 0.006342384272543202,
     'nextDividendDate': '0',
     'exDividendDate': '2020-10-23',
     'nextEarningsDate': '0',
     'peRatio': 40.67098281576622,
     'beta': 1.2066769092254626,
     'maxChangePercent': 50.23609650720246,
     'year5ChangePercent': 4.691615897848619,
     'year2ChangePercent': 2.686097724309814,
     'year1ChangePercent': 0.8081233825710212,
     'ytdChangePercent': -0.013032223189693985,
     'month6ChangePercent': 0.4515433981547753,
     'month3ChangePercent': 0.1302133676150156,
     'month1ChangePercent': 0.0740171668697909,
     'day30ChangePercent': 0.0748077755423013,
     'day5ChangePercent': -0.028665585006375}



## Parsing Our API Call

This API call has all the information we need. We can parse it using the same square-bracket notation as in the first project of this course. Here is an example.


```python
data['year1ChangePercent']
```




    0.8081233825710212



## Executing A Batch API Call & Building Our DataFrame


```python
# Function sourced from
# https://stackoverflow.com/questions/312443/how-do-you-split-a-list-into-evenly-sized-chunks
def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]   

symbol_groups = list(chunks(stocks['Ticker'], 100))
symbol_strings = []
for i in range(0, len(symbol_groups)):
    symbol_strings.append(','.join(symbol_groups[i]))
#     print(symbol_strings[i])

my_columns = ['Ticker', 'Price', 'One-Year Price Return', 'Number of Shares to Buy']
```

Now we need to create a blank DataFrame and add our data to the data frame one-by-one.


```python
final_dataframe = pd.DataFrame(columns = my_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=stats,quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        final_dataframe = final_dataframe.append(
                                        pd.Series([symbol,
                                                   data[symbol]['quote']['latestPrice'],
                                                   data[symbol]['stats']['year1ChangePercent'],
                                                   'N/A'
                                                   ],
                                                  index = my_columns),
                                        ignore_index = True)


final_dataframe
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
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>120.37</td>
      <td>0.441618</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.09</td>
      <td>-0.450894</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>162.97</td>
      <td>-0.00672252</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>132.66</td>
      <td>0.807433</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>107.60</td>
      <td>0.268484</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>107.18</td>
      <td>0.0591794</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.81</td>
      <td>0.0562389</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>393.17</td>
      <td>0.491743</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>44.69</td>
      <td>-0.125175</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>168.01</td>
      <td>0.245133</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



## Removing Low-Momentum Stocks


```python
final_dataframe.sort_values('One-Year Price Return', ascending = False, inplace = True)
final_dataframe = final_dataframe[:51]
final_dataframe.reset_index(drop = True, inplace = True)
final_dataframe
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
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CARR</td>
      <td>38.61</td>
      <td>2.2094</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ALB</td>
      <td>167.89</td>
      <td>1.36496</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NVDA</td>
      <td>556.61</td>
      <td>1.29116</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FCX</td>
      <td>28.48</td>
      <td>1.2533</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LB</td>
      <td>40.87</td>
      <td>1.21176</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>5</th>
      <td>PYPL</td>
      <td>238.67</td>
      <td>1.21112</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CDNS</td>
      <td>142.12</td>
      <td>0.965718</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ALGN</td>
      <td>558.85</td>
      <td>0.956706</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>8</th>
      <td>WST</td>
      <td>297.72</td>
      <td>0.943377</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ABMD</td>
      <td>335.60</td>
      <td>0.93543</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>10</th>
      <td>AMD</td>
      <td>96.21</td>
      <td>0.913639</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>11</th>
      <td>IDXX</td>
      <td>516.73</td>
      <td>0.880474</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>12</th>
      <td>SNPS</td>
      <td>262.87</td>
      <td>0.833769</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NOW</td>
      <td>533.30</td>
      <td>0.831274</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>14</th>
      <td>QCOM</td>
      <td>156.35</td>
      <td>0.828426</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ROL</td>
      <td>40.35</td>
      <td>0.80749</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>16</th>
      <td>AAPL</td>
      <td>132.66</td>
      <td>0.807433</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>17</th>
      <td>AMZN</td>
      <td>3349.02</td>
      <td>0.744492</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>18</th>
      <td>PWR</td>
      <td>71.91</td>
      <td>0.737624</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>19</th>
      <td>TWTR</td>
      <td>53.93</td>
      <td>0.7258</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>20</th>
      <td>TMUS</td>
      <td>132.78</td>
      <td>0.71335</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>21</th>
      <td>LRCX</td>
      <td>506.47</td>
      <td>0.710128</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>22</th>
      <td>DXCM</td>
      <td>385.03</td>
      <td>0.703802</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>23</th>
      <td>TTWO</td>
      <td>210.37</td>
      <td>0.688181</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>24</th>
      <td>MSCI</td>
      <td>431.61</td>
      <td>0.683176</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>25</th>
      <td>FDX</td>
      <td>263.48</td>
      <td>0.681036</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>26</th>
      <td>ADSK</td>
      <td>309.80</td>
      <td>0.66116</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>27</th>
      <td>DE</td>
      <td>285.97</td>
      <td>0.61066</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>28</th>
      <td>BIO</td>
      <td>591.57</td>
      <td>0.604644</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>29</th>
      <td>NFLX</td>
      <td>527.20</td>
      <td>0.601831</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>30</th>
      <td>PKI</td>
      <td>155.70</td>
      <td>0.596285</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>31</th>
      <td>CMG</td>
      <td>1377.02</td>
      <td>0.593953</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>32</th>
      <td>DVA</td>
      <td>121.01</td>
      <td>0.569433</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>33</th>
      <td>ATVI</td>
      <td>93.34</td>
      <td>0.565825</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>34</th>
      <td>PAYC</td>
      <td>433.87</td>
      <td>0.565437</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>35</th>
      <td>TSCO</td>
      <td>141.87</td>
      <td>0.558177</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>36</th>
      <td>IPGP</td>
      <td>227.90</td>
      <td>0.557083</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>37</th>
      <td>SIVB</td>
      <td>388.84</td>
      <td>0.547524</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ODFL</td>
      <td>194.04</td>
      <td>0.542991</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>39</th>
      <td>KLAC</td>
      <td>266.24</td>
      <td>0.531555</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>40</th>
      <td>NEM</td>
      <td>66.03</td>
      <td>0.521247</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>41</th>
      <td>QRVO</td>
      <td>173.55</td>
      <td>0.512954</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>42</th>
      <td>MTD</td>
      <td>1206.64</td>
      <td>0.501738</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>43</th>
      <td>DHR</td>
      <td>232.38</td>
      <td>0.500907</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>44</th>
      <td>TGT</td>
      <td>181.39</td>
      <td>0.498305</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>45</th>
      <td>MXIM</td>
      <td>92.16</td>
      <td>0.49808</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>46</th>
      <td>AMAT</td>
      <td>93.30</td>
      <td>0.497579</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>47</th>
      <td>TMO</td>
      <td>484.09</td>
      <td>0.4971</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>48</th>
      <td>ZBRA</td>
      <td>393.17</td>
      <td>0.491743</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>49</th>
      <td>EBAY</td>
      <td>52.77</td>
      <td>0.490331</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>50</th>
      <td>ALXN</td>
      <td>159.15</td>
      <td>0.480404</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
</div>



## Calculating the Number of Shares to Buy


```python
def portfolio_input():
    global portfolio_size
    portfolio_size = input("Enter the value of your portfolio:")

    try:
        val = float(portfolio_size)
    except ValueError:
        print("That's not a number! \n Try again:")
        portfolio_size = input("Enter the value of your portfolio:")

portfolio_input()
print(portfolio_size)
```

    Enter the value of your portfolio:1000000
    1000000



```python
position_size = float(portfolio_size) / len(final_dataframe.index)
for i in range(0, len(final_dataframe['Ticker'])):
    final_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / final_dataframe['Price'][i])
final_dataframe
```

    C:\Users\omen\anaconda3\lib\site-packages\pandas\core\indexing.py:1765: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      isetter(loc, value)





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
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CARR</td>
      <td>38.61</td>
      <td>2.2094</td>
      <td>507</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ALB</td>
      <td>167.89</td>
      <td>1.36496</td>
      <td>116</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NVDA</td>
      <td>556.61</td>
      <td>1.29116</td>
      <td>35</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FCX</td>
      <td>28.48</td>
      <td>1.2533</td>
      <td>688</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LB</td>
      <td>40.87</td>
      <td>1.21176</td>
      <td>479</td>
    </tr>
    <tr>
      <th>5</th>
      <td>PYPL</td>
      <td>238.67</td>
      <td>1.21112</td>
      <td>82</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CDNS</td>
      <td>142.12</td>
      <td>0.965718</td>
      <td>137</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ALGN</td>
      <td>558.85</td>
      <td>0.956706</td>
      <td>35</td>
    </tr>
    <tr>
      <th>8</th>
      <td>WST</td>
      <td>297.72</td>
      <td>0.943377</td>
      <td>65</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ABMD</td>
      <td>335.60</td>
      <td>0.93543</td>
      <td>58</td>
    </tr>
    <tr>
      <th>10</th>
      <td>AMD</td>
      <td>96.21</td>
      <td>0.913639</td>
      <td>203</td>
    </tr>
    <tr>
      <th>11</th>
      <td>IDXX</td>
      <td>516.73</td>
      <td>0.880474</td>
      <td>37</td>
    </tr>
    <tr>
      <th>12</th>
      <td>SNPS</td>
      <td>262.87</td>
      <td>0.833769</td>
      <td>74</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NOW</td>
      <td>533.30</td>
      <td>0.831274</td>
      <td>36</td>
    </tr>
    <tr>
      <th>14</th>
      <td>QCOM</td>
      <td>156.35</td>
      <td>0.828426</td>
      <td>125</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ROL</td>
      <td>40.35</td>
      <td>0.80749</td>
      <td>485</td>
    </tr>
    <tr>
      <th>16</th>
      <td>AAPL</td>
      <td>132.66</td>
      <td>0.807433</td>
      <td>147</td>
    </tr>
    <tr>
      <th>17</th>
      <td>AMZN</td>
      <td>3349.02</td>
      <td>0.744492</td>
      <td>5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>PWR</td>
      <td>71.91</td>
      <td>0.737624</td>
      <td>272</td>
    </tr>
    <tr>
      <th>19</th>
      <td>TWTR</td>
      <td>53.93</td>
      <td>0.7258</td>
      <td>363</td>
    </tr>
    <tr>
      <th>20</th>
      <td>TMUS</td>
      <td>132.78</td>
      <td>0.71335</td>
      <td>147</td>
    </tr>
    <tr>
      <th>21</th>
      <td>LRCX</td>
      <td>506.47</td>
      <td>0.710128</td>
      <td>38</td>
    </tr>
    <tr>
      <th>22</th>
      <td>DXCM</td>
      <td>385.03</td>
      <td>0.703802</td>
      <td>50</td>
    </tr>
    <tr>
      <th>23</th>
      <td>TTWO</td>
      <td>210.37</td>
      <td>0.688181</td>
      <td>93</td>
    </tr>
    <tr>
      <th>24</th>
      <td>MSCI</td>
      <td>431.61</td>
      <td>0.683176</td>
      <td>45</td>
    </tr>
    <tr>
      <th>25</th>
      <td>FDX</td>
      <td>263.48</td>
      <td>0.681036</td>
      <td>74</td>
    </tr>
    <tr>
      <th>26</th>
      <td>ADSK</td>
      <td>309.80</td>
      <td>0.66116</td>
      <td>63</td>
    </tr>
    <tr>
      <th>27</th>
      <td>DE</td>
      <td>285.97</td>
      <td>0.61066</td>
      <td>68</td>
    </tr>
    <tr>
      <th>28</th>
      <td>BIO</td>
      <td>591.57</td>
      <td>0.604644</td>
      <td>33</td>
    </tr>
    <tr>
      <th>29</th>
      <td>NFLX</td>
      <td>527.20</td>
      <td>0.601831</td>
      <td>37</td>
    </tr>
    <tr>
      <th>30</th>
      <td>PKI</td>
      <td>155.70</td>
      <td>0.596285</td>
      <td>125</td>
    </tr>
    <tr>
      <th>31</th>
      <td>CMG</td>
      <td>1377.02</td>
      <td>0.593953</td>
      <td>14</td>
    </tr>
    <tr>
      <th>32</th>
      <td>DVA</td>
      <td>121.01</td>
      <td>0.569433</td>
      <td>162</td>
    </tr>
    <tr>
      <th>33</th>
      <td>ATVI</td>
      <td>93.34</td>
      <td>0.565825</td>
      <td>210</td>
    </tr>
    <tr>
      <th>34</th>
      <td>PAYC</td>
      <td>433.87</td>
      <td>0.565437</td>
      <td>45</td>
    </tr>
    <tr>
      <th>35</th>
      <td>TSCO</td>
      <td>141.87</td>
      <td>0.558177</td>
      <td>138</td>
    </tr>
    <tr>
      <th>36</th>
      <td>IPGP</td>
      <td>227.90</td>
      <td>0.557083</td>
      <td>86</td>
    </tr>
    <tr>
      <th>37</th>
      <td>SIVB</td>
      <td>388.84</td>
      <td>0.547524</td>
      <td>50</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ODFL</td>
      <td>194.04</td>
      <td>0.542991</td>
      <td>101</td>
    </tr>
    <tr>
      <th>39</th>
      <td>KLAC</td>
      <td>266.24</td>
      <td>0.531555</td>
      <td>73</td>
    </tr>
    <tr>
      <th>40</th>
      <td>NEM</td>
      <td>66.03</td>
      <td>0.521247</td>
      <td>296</td>
    </tr>
    <tr>
      <th>41</th>
      <td>QRVO</td>
      <td>173.55</td>
      <td>0.512954</td>
      <td>112</td>
    </tr>
    <tr>
      <th>42</th>
      <td>MTD</td>
      <td>1206.64</td>
      <td>0.501738</td>
      <td>16</td>
    </tr>
    <tr>
      <th>43</th>
      <td>DHR</td>
      <td>232.38</td>
      <td>0.500907</td>
      <td>84</td>
    </tr>
    <tr>
      <th>44</th>
      <td>TGT</td>
      <td>181.39</td>
      <td>0.498305</td>
      <td>108</td>
    </tr>
    <tr>
      <th>45</th>
      <td>MXIM</td>
      <td>92.16</td>
      <td>0.49808</td>
      <td>212</td>
    </tr>
    <tr>
      <th>46</th>
      <td>AMAT</td>
      <td>93.30</td>
      <td>0.497579</td>
      <td>210</td>
    </tr>
    <tr>
      <th>47</th>
      <td>TMO</td>
      <td>484.09</td>
      <td>0.4971</td>
      <td>40</td>
    </tr>
    <tr>
      <th>48</th>
      <td>ZBRA</td>
      <td>393.17</td>
      <td>0.491743</td>
      <td>49</td>
    </tr>
    <tr>
      <th>49</th>
      <td>EBAY</td>
      <td>52.77</td>
      <td>0.490331</td>
      <td>371</td>
    </tr>
    <tr>
      <th>50</th>
      <td>ALXN</td>
      <td>159.15</td>
      <td>0.480404</td>
      <td>123</td>
    </tr>
  </tbody>
</table>
</div>



## Building a Better (and More Realistic) Momentum Strategy

Real-world quantitative investment firms differentiate between "high quality" and "low quality" momentum stocks:

* High-quality momentum stocks show "slow and steady" outperformance over long periods of time
* Low-quality momentum stocks might not show any momentum for a long time, and then surge upwards.

The reason why high-quality momentum stocks are preferred is because low-quality momentum can often be cause by short-term news that is unlikely to be repeated in the future (such as an FDA approval for a biotechnology company).

To identify high-quality momentum, we're going to build a strategy that selects stocks from the highest percentiles of:

* 1-month price returns
* 3-month price returns
* 6-month price returns
* 1-year price returns

Let's start by building our DataFrame. You'll notice that I use the abbreviation `hqm` often. It stands for `high-quality momentum`.


```python
hqm_columns = [
                'Ticker',
                'Price',
                'Number of Shares to Buy',
                'One-Year Price Return',
                'One-Year Return Percentile',
                'Six-Month Price Return',
                'Six-Month Return Percentile',
                'Three-Month Price Return',
                'Three-Month Return Percentile',
                'One-Month Price Return',
                'One-Month Return Percentile',
                'HQM Score'
                ]

hqm_dataframe = pd.DataFrame(columns = hqm_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=stats,quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        hqm_dataframe = hqm_dataframe.append(
                                        pd.Series([symbol,
                                                   data[symbol]['quote']['latestPrice'],
                                                   'N/A',
                                                   data[symbol]['stats']['year1ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month6ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month3ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month1ChangePercent'],
                                                   'N/A',
                                                   'N/A'
                                                   ],
                                                  index = hqm_columns),
                                        ignore_index = True)

hqm_dataframe.columns
hqm_dataframe
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
      <th>Ticker</th>
      <th>Price</th>
      <th>Number of Shares to Buy</th>
      <th>One-Year Price Return</th>
      <th>One-Year Return Percentile</th>
      <th>Six-Month Price Return</th>
      <th>Six-Month Return Percentile</th>
      <th>Three-Month Price Return</th>
      <th>Three-Month Return Percentile</th>
      <th>One-Month Price Return</th>
      <th>One-Month Return Percentile</th>
      <th>HQM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>122.54</td>
      <td>N/A</td>
      <td>0.433615</td>
      <td>N/A</td>
      <td>0.359491</td>
      <td>N/A</td>
      <td>0.166762</td>
      <td>N/A</td>
      <td>0.0416307</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.00</td>
      <td>N/A</td>
      <td>-0.445331</td>
      <td>N/A</td>
      <td>0.242938</td>
      <td>N/A</td>
      <td>0.183779</td>
      <td>N/A</td>
      <td>-0.0597672</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.76</td>
      <td>N/A</td>
      <td>-0.00670065</td>
      <td>N/A</td>
      <td>0.116792</td>
      <td>N/A</td>
      <td>0.0100361</td>
      <td>N/A</td>
      <td>0.034035</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>134.69</td>
      <td>N/A</td>
      <td>0.810656</td>
      <td>N/A</td>
      <td>0.459919</td>
      <td>N/A</td>
      <td>0.131126</td>
      <td>N/A</td>
      <td>0.07209</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>108.00</td>
      <td>N/A</td>
      <td>0.278281</td>
      <td>N/A</td>
      <td>0.106961</td>
      <td>N/A</td>
      <td>0.237019</td>
      <td>N/A</td>
      <td>-0.00739917</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>107.53</td>
      <td>N/A</td>
      <td>0.0605584</td>
      <td>N/A</td>
      <td>0.234134</td>
      <td>N/A</td>
      <td>0.129951</td>
      <td>N/A</td>
      <td>-0.000291812</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.26</td>
      <td>N/A</td>
      <td>0.0565088</td>
      <td>N/A</td>
      <td>0.298844</td>
      <td>N/A</td>
      <td>0.104245</td>
      <td>N/A</td>
      <td>0.0484151</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>398.95</td>
      <td>N/A</td>
      <td>0.510499</td>
      <td>N/A</td>
      <td>0.508562</td>
      <td>N/A</td>
      <td>0.44045</td>
      <td>N/A</td>
      <td>0.00557348</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>43.65</td>
      <td>N/A</td>
      <td>-0.125527</td>
      <td>N/A</td>
      <td>0.375817</td>
      <td>N/A</td>
      <td>0.415149</td>
      <td>N/A</td>
      <td>-0.00938373</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>167.40</td>
      <td>N/A</td>
      <td>0.239934</td>
      <td>N/A</td>
      <td>0.207671</td>
      <td>N/A</td>
      <td>0.00939442</td>
      <td>N/A</td>
      <td>0.045042</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 12 columns</p>
</div>




```python
hqm_dataframe.fillna(0, inplace=True)
```


```python
hqm_dataframe.isnull().sum()
```




    Ticker                           0
    Price                            0
    Number of Shares to Buy          0
    One-Year Price Return            0
    One-Year Return Percentile       0
    Six-Month Price Return           0
    Six-Month Return Percentile      0
    Three-Month Price Return         0
    Three-Month Return Percentile    0
    One-Month Price Return           0
    One-Month Return Percentile      0
    HQM Score                        0
    dtype: int64



## Calculating Momentum Percentiles

We now need to calculate momentum percentile scores for every stock in the universe. More specifically, we need to calculate percentile scores for the following metrics for every stock:

* `One-Year Price Return`
* `Six-Month Price Return`
* `Three-Month Price Return`
* `One-Month Price Return`

Here's how we'll do this:


```python
time_periods = [
                'One-Year',
                'Six-Month',
                'Three-Month',
                'One-Month'
                ]

for row in hqm_dataframe.index:
    for time_period in time_periods:
        hqm_dataframe.loc[row, f'{time_period} Return Percentile'] = stats.percentileofscore(hqm_dataframe[f'{time_period} Price Return'], hqm_dataframe.loc[row, f'{time_period} Price Return'])/100

# Print each percentile score to make sure it was calculated properly
'''
for time_period in time_periods:
    print(hqm_dataframe[f'{time_period} Return Percentile'])
'''
#Print the entire DataFrame    
hqm_dataframe
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
      <th>Ticker</th>
      <th>Price</th>
      <th>Number of Shares to Buy</th>
      <th>One-Year Price Return</th>
      <th>One-Year Return Percentile</th>
      <th>Six-Month Price Return</th>
      <th>Six-Month Return Percentile</th>
      <th>Three-Month Price Return</th>
      <th>Three-Month Return Percentile</th>
      <th>One-Month Price Return</th>
      <th>One-Month Return Percentile</th>
      <th>HQM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>122.54</td>
      <td>N/A</td>
      <td>0.433615</td>
      <td>0.877228</td>
      <td>0.359491</td>
      <td>0.714851</td>
      <td>0.166762</td>
      <td>0.592079</td>
      <td>0.041631</td>
      <td>0.79604</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.00</td>
      <td>N/A</td>
      <td>-0.445331</td>
      <td>0.0178218</td>
      <td>0.242938</td>
      <td>0.526733</td>
      <td>0.183779</td>
      <td>0.625743</td>
      <td>-0.059767</td>
      <td>0.114851</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.76</td>
      <td>N/A</td>
      <td>-0.006701</td>
      <td>0.354455</td>
      <td>0.116792</td>
      <td>0.30099</td>
      <td>0.010036</td>
      <td>0.219802</td>
      <td>0.034035</td>
      <td>0.758416</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>134.69</td>
      <td>N/A</td>
      <td>0.810656</td>
      <td>0.970297</td>
      <td>0.459919</td>
      <td>0.841584</td>
      <td>0.131126</td>
      <td>0.514851</td>
      <td>0.072090</td>
      <td>0.90297</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>108.00</td>
      <td>N/A</td>
      <td>0.278281</td>
      <td>0.756436</td>
      <td>0.106961</td>
      <td>0.279208</td>
      <td>0.237019</td>
      <td>0.738614</td>
      <td>-0.007399</td>
      <td>0.455446</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>107.53</td>
      <td>N/A</td>
      <td>0.060558</td>
      <td>0.487129</td>
      <td>0.234134</td>
      <td>0.508911</td>
      <td>0.129951</td>
      <td>0.512871</td>
      <td>-0.000292</td>
      <td>0.512871</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.26</td>
      <td>N/A</td>
      <td>0.056509</td>
      <td>0.473267</td>
      <td>0.298844</td>
      <td>0.629703</td>
      <td>0.104245</td>
      <td>0.439604</td>
      <td>0.048415</td>
      <td>0.833663</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>398.95</td>
      <td>N/A</td>
      <td>0.510499</td>
      <td>0.916832</td>
      <td>0.508562</td>
      <td>0.869307</td>
      <td>0.440450</td>
      <td>0.912871</td>
      <td>0.005573</td>
      <td>0.580198</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>43.65</td>
      <td>N/A</td>
      <td>-0.125527</td>
      <td>0.20396</td>
      <td>0.375817</td>
      <td>0.732673</td>
      <td>0.415149</td>
      <td>0.90495</td>
      <td>-0.009384</td>
      <td>0.429703</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>167.40</td>
      <td>N/A</td>
      <td>0.239934</td>
      <td>0.710891</td>
      <td>0.207671</td>
      <td>0.457426</td>
      <td>0.009394</td>
      <td>0.217822</td>
      <td>0.045042</td>
      <td>0.819802</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 12 columns</p>
</div>



## Calculating the HQM Score

We'll now calculate our `HQM Score`, which is the high-quality momentum score that we'll use to filter for stocks in this investing strategy.

The `HQM Score` will be the arithmetic mean of the 4 momentum percentile scores that we calculated in the last section.

To calculate arithmetic mean, we will use the `mean` function from Python's built-in `statistics` module.


```python
from statistics import mean

for row in hqm_dataframe.index:
    momentum_percentiles = []
    for time_period in time_periods:
        momentum_percentiles.append(hqm_dataframe.loc[row, f'{time_period} Return Percentile'])
    hqm_dataframe.loc[row, 'HQM Score'] = mean(momentum_percentiles)
```

## Selecting the 50 Best Momentum Stocks

As before, we can identify the 50 best momentum stocks in our universe by sorting the DataFrame on the `HQM Score` column and dropping all but the top 50 entries.


```python
hqm_dataframe.sort_values(by = 'HQM Score', ascending = False)
hqm_dataframe = hqm_dataframe[:51]
```

## Calculating the Number of Shares to Buy

We'll use the `portfolio_input` function that we created earlier to accept our portfolio size. Then we will use similar logic in a `for` loop to calculate the number of shares to buy for each stock in our investment universe.


```python
portfolio_input()
```

    Enter the value of your portfolio:1000000



```python
position_size = float(portfolio_size) / len(hqm_dataframe.index)
for i in range(0, len(hqm_dataframe['Ticker'])-1):
    hqm_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / hqm_dataframe['Price'][i])
hqm_dataframe
```

    C:\Users\omen\anaconda3\lib\site-packages\pandas\core\indexing.py:1765: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      isetter(loc, value)





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
      <th>Ticker</th>
      <th>Price</th>
      <th>Number of Shares to Buy</th>
      <th>One-Year Price Return</th>
      <th>One-Year Return Percentile</th>
      <th>Six-Month Price Return</th>
      <th>Six-Month Return Percentile</th>
      <th>Three-Month Price Return</th>
      <th>Three-Month Return Percentile</th>
      <th>One-Month Price Return</th>
      <th>One-Month Return Percentile</th>
      <th>HQM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>122.54</td>
      <td>160</td>
      <td>0.433615</td>
      <td>0.877228</td>
      <td>0.359491</td>
      <td>0.714851</td>
      <td>0.166762</td>
      <td>0.592079</td>
      <td>0.041631</td>
      <td>0.79604</td>
      <td>0.74505</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.00</td>
      <td>1225</td>
      <td>-0.445331</td>
      <td>0.0178218</td>
      <td>0.242938</td>
      <td>0.526733</td>
      <td>0.183779</td>
      <td>0.625743</td>
      <td>-0.059767</td>
      <td>0.114851</td>
      <td>0.321287</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.76</td>
      <td>124</td>
      <td>-0.006701</td>
      <td>0.354455</td>
      <td>0.116792</td>
      <td>0.30099</td>
      <td>0.010036</td>
      <td>0.219802</td>
      <td>0.034035</td>
      <td>0.758416</td>
      <td>0.408416</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>134.69</td>
      <td>145</td>
      <td>0.810656</td>
      <td>0.970297</td>
      <td>0.459919</td>
      <td>0.841584</td>
      <td>0.131126</td>
      <td>0.514851</td>
      <td>0.072090</td>
      <td>0.90297</td>
      <td>0.807426</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>108.00</td>
      <td>181</td>
      <td>0.278281</td>
      <td>0.756436</td>
      <td>0.106961</td>
      <td>0.279208</td>
      <td>0.237019</td>
      <td>0.738614</td>
      <td>-0.007399</td>
      <td>0.455446</td>
      <td>0.557426</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ABC</td>
      <td>100.35</td>
      <td>195</td>
      <td>0.188923</td>
      <td>0.647525</td>
      <td>-0.015866</td>
      <td>0.0970297</td>
      <td>0.019067</td>
      <td>0.243564</td>
      <td>-0.052446</td>
      <td>0.142574</td>
      <td>0.282673</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ABMD</td>
      <td>325.00</td>
      <td>60</td>
      <td>0.950220</td>
      <td>0.984158</td>
      <td>0.228242</td>
      <td>0.491089</td>
      <td>0.195365</td>
      <td>0.659406</td>
      <td>0.185162</td>
      <td>0.99604</td>
      <td>0.782673</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ABT</td>
      <td>111.61</td>
      <td>175</td>
      <td>0.309843</td>
      <td>0.79802</td>
      <td>0.215821</td>
      <td>0.471287</td>
      <td>0.021148</td>
      <td>0.251485</td>
      <td>0.024627</td>
      <td>0.69703</td>
      <td>0.554455</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ACN</td>
      <td>259.94</td>
      <td>75</td>
      <td>0.247706</td>
      <td>0.720792</td>
      <td>0.211705</td>
      <td>0.467327</td>
      <td>0.162701</td>
      <td>0.584158</td>
      <td>0.017700</td>
      <td>0.659406</td>
      <td>0.607921</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ADBE</td>
      <td>490.52</td>
      <td>39</td>
      <td>0.468160</td>
      <td>0.893069</td>
      <td>0.100594</td>
      <td>0.259406</td>
      <td>-0.001645</td>
      <td>0.182178</td>
      <td>-0.000650</td>
      <td>0.510891</td>
      <td>0.461386</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ADI</td>
      <td>151.91</td>
      <td>129</td>
      <td>0.293891</td>
      <td>0.780198</td>
      <td>0.238137</td>
      <td>0.518812</td>
      <td>0.287353</td>
      <td>0.811881</td>
      <td>0.033215</td>
      <td>0.748515</td>
      <td>0.714851</td>
    </tr>
    <tr>
      <th>11</th>
      <td>ADM</td>
      <td>52.78</td>
      <td>371</td>
      <td>0.146612</td>
      <td>0.590099</td>
      <td>0.307713</td>
      <td>0.641584</td>
      <td>0.074363</td>
      <td>0.384158</td>
      <td>0.005433</td>
      <td>0.576238</td>
      <td>0.54802</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ADP</td>
      <td>173.68</td>
      <td>112</td>
      <td>0.015760</td>
      <td>0.413861</td>
      <td>0.138892</td>
      <td>0.334653</td>
      <td>0.206260</td>
      <td>0.677228</td>
      <td>-0.031253</td>
      <td>0.247525</td>
      <td>0.418317</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ADSK</td>
      <td>312.70</td>
      <td>62</td>
      <td>0.642419</td>
      <td>0.948515</td>
      <td>0.256476</td>
      <td>0.548515</td>
      <td>0.333620</td>
      <td>0.857426</td>
      <td>0.088902</td>
      <td>0.942574</td>
      <td>0.824257</td>
    </tr>
    <tr>
      <th>14</th>
      <td>AEE</td>
      <td>77.59</td>
      <td>252</td>
      <td>0.015470</td>
      <td>0.411881</td>
      <td>0.043181</td>
      <td>0.176238</td>
      <td>-0.055551</td>
      <td>0.0772277</td>
      <td>-0.010611</td>
      <td>0.419802</td>
      <td>0.271287</td>
    </tr>
    <tr>
      <th>15</th>
      <td>AEP</td>
      <td>85.22</td>
      <td>230</td>
      <td>-0.103489</td>
      <td>0.235644</td>
      <td>0.003164</td>
      <td>0.118812</td>
      <td>-0.057985</td>
      <td>0.0712871</td>
      <td>-0.024375</td>
      <td>0.29703</td>
      <td>0.180693</td>
    </tr>
    <tr>
      <th>16</th>
      <td>AES</td>
      <td>24.51</td>
      <td>799</td>
      <td>0.262593</td>
      <td>0.736634</td>
      <td>0.687469</td>
      <td>0.954455</td>
      <td>0.307923</td>
      <td>0.831683</td>
      <td>0.142284</td>
      <td>0.980198</td>
      <td>0.875743</td>
    </tr>
    <tr>
      <th>17</th>
      <td>AFL</td>
      <td>43.42</td>
      <td>451</td>
      <td>-0.162345</td>
      <td>0.164356</td>
      <td>0.254612</td>
      <td>0.544554</td>
      <td>0.169188</td>
      <td>0.59802</td>
      <td>-0.066136</td>
      <td>0.0910891</td>
      <td>0.349505</td>
    </tr>
    <tr>
      <th>18</th>
      <td>AIG</td>
      <td>38.10</td>
      <td>514</td>
      <td>-0.243170</td>
      <td>0.0871287</td>
      <td>0.295769</td>
      <td>0.625743</td>
      <td>0.313422</td>
      <td>0.837624</td>
      <td>-0.067588</td>
      <td>0.0811881</td>
      <td>0.407921</td>
    </tr>
    <tr>
      <th>19</th>
      <td>AIV</td>
      <td>5.21</td>
      <td>3763</td>
      <td>-0.231159</td>
      <td>0.0990099</td>
      <td>0.003128</td>
      <td>0.116832</td>
      <td>0.062367</td>
      <td>0.354455</td>
      <td>0.136602</td>
      <td>0.978218</td>
      <td>0.387129</td>
    </tr>
    <tr>
      <th>20</th>
      <td>AIZ</td>
      <td>132.74</td>
      <td>147</td>
      <td>0.015155</td>
      <td>0.409901</td>
      <td>0.343707</td>
      <td>0.70495</td>
      <td>0.060794</td>
      <td>0.350495</td>
      <td>-0.003309</td>
      <td>0.487129</td>
      <td>0.488119</td>
    </tr>
    <tr>
      <th>21</th>
      <td>AJG</td>
      <td>120.86</td>
      <td>162</td>
      <td>0.286241</td>
      <td>0.770297</td>
      <td>0.238077</td>
      <td>0.516832</td>
      <td>0.125717</td>
      <td>0.50099</td>
      <td>0.030071</td>
      <td>0.724752</td>
      <td>0.628218</td>
    </tr>
    <tr>
      <th>22</th>
      <td>AKAM</td>
      <td>110.01</td>
      <td>178</td>
      <td>0.217017</td>
      <td>0.689109</td>
      <td>-0.070918</td>
      <td>0.0455446</td>
      <td>-0.034454</td>
      <td>0.124752</td>
      <td>0.011982</td>
      <td>0.631683</td>
      <td>0.372772</td>
    </tr>
    <tr>
      <th>23</th>
      <td>ALB</td>
      <td>164.47</td>
      <td>119</td>
      <td>1.329307</td>
      <td>0.99802</td>
      <td>1.103879</td>
      <td>0.994059</td>
      <td>0.729686</td>
      <td>0.984158</td>
      <td>0.175527</td>
      <td>0.994059</td>
      <td>0.992574</td>
    </tr>
    <tr>
      <th>24</th>
      <td>ALGN</td>
      <td>556.14</td>
      <td>35</td>
      <td>0.975802</td>
      <td>0.988119</td>
      <td>0.966035</td>
      <td>0.990099</td>
      <td>0.718758</td>
      <td>0.980198</td>
      <td>0.043422</td>
      <td>0.807921</td>
      <td>0.941584</td>
    </tr>
    <tr>
      <th>25</th>
      <td>ALK</td>
      <td>50.18</td>
      <td>390</td>
      <td>-0.253615</td>
      <td>0.0831683</td>
      <td>0.383479</td>
      <td>0.738614</td>
      <td>0.333661</td>
      <td>0.859406</td>
      <td>-0.060444</td>
      <td>0.110891</td>
      <td>0.44802</td>
    </tr>
    <tr>
      <th>26</th>
      <td>ALL</td>
      <td>110.51</td>
      <td>177</td>
      <td>-0.029178</td>
      <td>0.322772</td>
      <td>0.155820</td>
      <td>0.366337</td>
      <td>0.156382</td>
      <td>0.572277</td>
      <td>0.027866</td>
      <td>0.706931</td>
      <td>0.492079</td>
    </tr>
    <tr>
      <th>27</th>
      <td>ALLE</td>
      <td>115.70</td>
      <td>169</td>
      <td>-0.064054</td>
      <td>0.279208</td>
      <td>0.131904</td>
      <td>0.326733</td>
      <td>0.143996</td>
      <td>0.544554</td>
      <td>0.004511</td>
      <td>0.570297</td>
      <td>0.430198</td>
    </tr>
    <tr>
      <th>28</th>
      <td>ALXN</td>
      <td>163.63</td>
      <td>119</td>
      <td>0.489210</td>
      <td>0.90495</td>
      <td>0.402430</td>
      <td>0.768317</td>
      <td>0.381987</td>
      <td>0.893069</td>
      <td>0.308700</td>
      <td>1</td>
      <td>0.891584</td>
    </tr>
    <tr>
      <th>29</th>
      <td>AMAT</td>
      <td>91.60</td>
      <td>214</td>
      <td>0.498204</td>
      <td>0.910891</td>
      <td>0.485907</td>
      <td>0.861386</td>
      <td>0.506234</td>
      <td>0.956436</td>
      <td>0.008633</td>
      <td>0.605941</td>
      <td>0.833663</td>
    </tr>
    <tr>
      <th>30</th>
      <td>AMCR</td>
      <td>11.58</td>
      <td>1693</td>
      <td>0.142639</td>
      <td>0.582178</td>
      <td>0.103298</td>
      <td>0.265347</td>
      <td>0.040429</td>
      <td>0.30099</td>
      <td>-0.007230</td>
      <td>0.457426</td>
      <td>0.401485</td>
    </tr>
    <tr>
      <th>31</th>
      <td>AMD</td>
      <td>94.64</td>
      <td>207</td>
      <td>0.946725</td>
      <td>0.980198</td>
      <td>0.795386</td>
      <td>0.970297</td>
      <td>0.078683</td>
      <td>0.394059</td>
      <td>-0.014135</td>
      <td>0.390099</td>
      <td>0.683663</td>
    </tr>
    <tr>
      <th>32</th>
      <td>AME</td>
      <td>122.60</td>
      <td>159</td>
      <td>0.191727</td>
      <td>0.651485</td>
      <td>0.353812</td>
      <td>0.710891</td>
      <td>0.155749</td>
      <td>0.570297</td>
      <td>0.020904</td>
      <td>0.677228</td>
      <td>0.652475</td>
    </tr>
    <tr>
      <th>33</th>
      <td>AMGN</td>
      <td>230.26</td>
      <td>85</td>
      <td>-0.018702</td>
      <td>0.330693</td>
      <td>-0.109078</td>
      <td>0.0257426</td>
      <td>-0.108300</td>
      <td>0.029703</td>
      <td>-0.006484</td>
      <td>0.461386</td>
      <td>0.211881</td>
    </tr>
    <tr>
      <th>34</th>
      <td>AMP</td>
      <td>193.08</td>
      <td>101</td>
      <td>0.152911</td>
      <td>0.605941</td>
      <td>0.315546</td>
      <td>0.653465</td>
      <td>0.158116</td>
      <td>0.578218</td>
      <td>-0.049064</td>
      <td>0.162376</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>35</th>
      <td>AMT</td>
      <td>224.51</td>
      <td>87</td>
      <td>-0.016522</td>
      <td>0.334653</td>
      <td>-0.157484</td>
      <td>0.0138614</td>
      <td>-0.089169</td>
      <td>0.0376238</td>
      <td>-0.029795</td>
      <td>0.257426</td>
      <td>0.160891</td>
    </tr>
    <tr>
      <th>36</th>
      <td>AMZN</td>
      <td>3326.80</td>
      <td>5</td>
      <td>0.736774</td>
      <td>0.962376</td>
      <td>0.118215</td>
      <td>0.30495</td>
      <td>0.006213</td>
      <td>0.209901</td>
      <td>0.017924</td>
      <td>0.665347</td>
      <td>0.535644</td>
    </tr>
    <tr>
      <th>37</th>
      <td>ANET</td>
      <td>287.00</td>
      <td>68</td>
      <td>0.427676</td>
      <td>0.869307</td>
      <td>0.367418</td>
      <td>0.724752</td>
      <td>0.405379</td>
      <td>0.90297</td>
      <td>0.061243</td>
      <td>0.877228</td>
      <td>0.843564</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ANSS</td>
      <td>373.74</td>
      <td>52</td>
      <td>0.415104</td>
      <td>0.863366</td>
      <td>0.216062</td>
      <td>0.475248</td>
      <td>0.112720</td>
      <td>0.455446</td>
      <td>0.048278</td>
      <td>0.831683</td>
      <td>0.656436</td>
    </tr>
    <tr>
      <th>39</th>
      <td>ANTM</td>
      <td>329.50</td>
      <td>59</td>
      <td>0.076304</td>
      <td>0.508911</td>
      <td>0.188630</td>
      <td>0.425743</td>
      <td>0.111865</td>
      <td>0.451485</td>
      <td>-0.025423</td>
      <td>0.287129</td>
      <td>0.418317</td>
    </tr>
    <tr>
      <th>40</th>
      <td>AON</td>
      <td>206.40</td>
      <td>94</td>
      <td>0.312647</td>
      <td>0.805941</td>
      <td>0.072758</td>
      <td>0.225743</td>
      <td>-0.008433</td>
      <td>0.172277</td>
      <td>-0.035944</td>
      <td>0.221782</td>
      <td>0.356436</td>
    </tr>
    <tr>
      <th>41</th>
      <td>AOS</td>
      <td>54.88</td>
      <td>357</td>
      <td>0.178138</td>
      <td>0.637624</td>
      <td>0.160930</td>
      <td>0.378218</td>
      <td>-0.012604</td>
      <td>0.168317</td>
      <td>-0.014446</td>
      <td>0.386139</td>
      <td>0.392574</td>
    </tr>
    <tr>
      <th>42</th>
      <td>APA</td>
      <td>16.29</td>
      <td>1203</td>
      <td>-0.356016</td>
      <td>0.0415842</td>
      <td>0.228256</td>
      <td>0.493069</td>
      <td>0.671237</td>
      <td>0.974257</td>
      <td>0.068321</td>
      <td>0.889109</td>
      <td>0.599505</td>
    </tr>
    <tr>
      <th>43</th>
      <td>APD</td>
      <td>288.05</td>
      <td>68</td>
      <td>0.285386</td>
      <td>0.768317</td>
      <td>0.152965</td>
      <td>0.362376</td>
      <td>-0.056115</td>
      <td>0.0752475</td>
      <td>0.038014</td>
      <td>0.782178</td>
      <td>0.49703</td>
    </tr>
    <tr>
      <th>44</th>
      <td>APH</td>
      <td>136.85</td>
      <td>143</td>
      <td>0.223553</td>
      <td>0.69703</td>
      <td>0.384226</td>
      <td>0.740594</td>
      <td>0.182455</td>
      <td>0.621782</td>
      <td>-0.014732</td>
      <td>0.380198</td>
      <td>0.609901</td>
    </tr>
    <tr>
      <th>45</th>
      <td>APTV</td>
      <td>137.93</td>
      <td>142</td>
      <td>0.441160</td>
      <td>0.883168</td>
      <td>0.742569</td>
      <td>0.964356</td>
      <td>0.379224</td>
      <td>0.891089</td>
      <td>0.083372</td>
      <td>0.930693</td>
      <td>0.917327</td>
    </tr>
    <tr>
      <th>46</th>
      <td>ARE</td>
      <td>170.65</td>
      <td>114</td>
      <td>0.093966</td>
      <td>0.520792</td>
      <td>0.047455</td>
      <td>0.184158</td>
      <td>0.028802</td>
      <td>0.275248</td>
      <td>0.004219</td>
      <td>0.560396</td>
      <td>0.385149</td>
    </tr>
    <tr>
      <th>47</th>
      <td>ATO</td>
      <td>96.81</td>
      <td>202</td>
      <td>-0.150664</td>
      <td>0.178218</td>
      <td>-0.074058</td>
      <td>0.0415842</td>
      <td>-0.022178</td>
      <td>0.140594</td>
      <td>-0.049245</td>
      <td>0.160396</td>
      <td>0.130198</td>
    </tr>
    <tr>
      <th>48</th>
      <td>ATVI</td>
      <td>95.19</td>
      <td>205</td>
      <td>0.570001</td>
      <td>0.930693</td>
      <td>0.172482</td>
      <td>0.405941</td>
      <td>0.134769</td>
      <td>0.524752</td>
      <td>0.126449</td>
      <td>0.968317</td>
      <td>0.707426</td>
    </tr>
    <tr>
      <th>49</th>
      <td>AVB</td>
      <td>155.89</td>
      <td>125</td>
      <td>-0.233738</td>
      <td>0.0970297</td>
      <td>-0.010260</td>
      <td>0.10099</td>
      <td>-0.024659</td>
      <td>0.134653</td>
      <td>-0.101651</td>
      <td>0.0336634</td>
      <td>0.0915842</td>
    </tr>
    <tr>
      <th>50</th>
      <td>AVGO</td>
      <td>444.50</td>
      <td>N/A</td>
      <td>0.441675</td>
      <td>0.885149</td>
      <td>0.388819</td>
      <td>0.746535</td>
      <td>0.184123</td>
      <td>0.627723</td>
      <td>0.049933</td>
      <td>0.841584</td>
      <td>0.775248</td>
    </tr>
  </tbody>
</table>
</div>



## Formatting Our Excel Output

We will be using the XlsxWriter library for Python to create nicely-formatted Excel files.

XlsxWriter is an excellent package and offers tons of customization. However, the tradeoff for this is that the library can seem very complicated to new users. Accordingly, this section will be fairly long because I want to do a good job of explaining how XlsxWriter works.


```python
writer = pd.ExcelWriter('momentum_strategy.xlsx', engine='xlsxwriter')
hqm_dataframe.to_excel(writer, sheet_name='Momentum Strategy', index = False)
```

## Creating the Formats We'll Need For Our .xlsx File

You'll recall from our first project that formats include colors, fonts, and also symbols like % and $. We'll need four main formats for our Excel document:

* String format for tickers
* \$XX.XX format for stock prices
* \$XX,XXX format for market capitalization
* Integer format for the number of shares to purchase

Since we already built our formats in the last section of this course, I've included them below for you. Run this code cell before proceeding.


```python
background_color = '#0a0a23'
font_color = '#ffffff'

string_template = writer.book.add_format(
        {
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

dollar_template = writer.book.add_format(
        {
            'num_format':'$0.00',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

integer_template = writer.book.add_format(
        {
            'num_format':'0',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

percent_template = writer.book.add_format(
        {
            'num_format':'0.0%',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )
```


```python
column_formats = {
                    'A': ['Ticker', string_template],
                    'B': ['Price', dollar_template],
                    'C': ['Number of Shares to Buy', integer_template],
                    'D': ['One-Year Price Return', percent_template],
                    'E': ['One-Year Return Percentile', percent_template],
                    'F': ['Six-Month Price Return', percent_template],
                    'G': ['Six-Month Return Percentile', percent_template],
                    'H': ['Three-Month Price Return', percent_template],
                    'I': ['Three-Month Return Percentile', percent_template],
                    'J': ['One-Month Price Return', percent_template],
                    'K': ['One-Month Return Percentile', percent_template],
                    'L': ['HQM Score', integer_template]
                    }

for column in column_formats.keys():
    writer.sheets['Momentum Strategy'].set_column(f'{column}:{column}', 20, column_formats[column][1])
    writer.sheets['Momentum Strategy'].write(f'{column}1', column_formats[column][0], string_template)
```

## Saving Our Excel Output

As before, saving our Excel output is very easy:


```python
writer.save()
```

{{< img src="/images/posts/configuration/momentum_strategy.png" >}}
