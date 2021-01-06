---
title: "Value Strategy S&P 500"
date: 2020-12-09T06:00:23+06:00
hero: /images/posts/writing-posts/analytics.svg
menu:
  sidebar:
    name: Value Strategy S&P 500
    identifier: Value Strategy S&P 500-algorithmic-trading-python
    parent: Algorithmic-Trading-Python
    weight: 112
---
# Quantitative Value Strategy
"Value investing" means investing in the stocks that are cheapest relative to common measures of business value (like earnings or assets).

For this project, we're going to build an investing strategy that selects the 50 stocks with the best value metrics. From there, we will calculate recommended trades for an equal-weight portfolio of these 50 stocks.

## Library Imports
The first thing we need to do is import the open-source software libraries that we'll be using in this tutorial.


```python
import numpy as np #The Numpy numerical computing library
import pandas as pd #The Pandas data science library
import requests #The requests library for HTTP requests in Python
import xlsxwriter #The XlsxWriter libarary for
import math #The Python math module
from scipy import stats #The SciPy stats module
```

## Importing Our List of Stocks & API Token
As before, we'll need to import our list of stocks and our API token before proceeding. Make sure the .csv file is still in your working directory and import it with the following command:


```python
stocks = pd.read_csv('sp_500_stocks.csv')
from secrets import IEX_CLOUD_API_TOKEN
```

## Making Our First API Call
It's now time to make the first version of our value screener!

We'll start by building a simple value screener that ranks securities based on a single metric (the price-to-earnings ratio).


```python
symbol = 'AAPL'
api_url = f'https://sandbox.iexapis.com/stable/stock/{symbol}/quote?token={IEX_CLOUD_API_TOKEN}'
data = requests.get(api_url).json()
data
```




    {'symbol': 'AAPL',
     'companyName': 'Apple Inc',
     'primaryExchange': 'T/ARMSAG ASNLOEKL(E ELGNT)A DSCQB',
     'calculationPrice': 'close',
     'open': 132.65,
     'openTime': 1610298510956,
     'openSource': 'aicloiff',
     'close': 136.47,
     'closeTime': 1680709414431,
     'closeSource': 'cffioila',
     'high': 134.01,
     'highTime': 1619000127163,
     'highSource': 'epietry niua m1cedel5 d',
     'low': 130.71,
     'lowTime': 1644376501489,
     'lowSource': 'rdmi dut ee1ceypae5 lni',
     'latestPrice': 133.06,
     'latestSource': 'Close',
     'latestTime': 'January 5, 2021',
     'latestUpdate': 1623069866513,
     'latestVolume': 98221115,
     'iexRealtimePrice': 134.585,
     'iexRealtimeSize': 335,
     'iexLastUpdated': 1632421031376,
     'delayedPrice': 136.63,
     'delayedPriceTime': 1654713135143,
     'oddLotDelayedPrice': 133.17,
     'oddLotDelayedPriceTime': 1659668562621,
     'extendedPrice': 134.95,
     'extendedChange': -0.37,
     'extendedChangePercent': -0.00283,
     'extendedPriceTime': 1654536738276,
     'previousClose': 131.94,
     'previousVolume': 148445311,
     'change': 1.6,
     'changePercent': 0.0124,
     'volume': 101834477,
     'iexMarketPercent': 0.008408574643892091,
     'iexVolume': 827866,
     'avgTotalVolume': 113110352,
     'iexBidPrice': 0,
     'iexBidSize': 0,
     'iexAskPrice': 0,
     'iexAskSize': 0,
     'iexOpen': 131.31,
     'iexOpenTime': 1615013456994,
     'iexClose': 135.71,
     'iexCloseTime': 1646121499290,
     'marketCap': 2263402362800,
     'peRatio': 41.92,
     'week52High': 143.4,
     'week52Low': 57.1,
     'ytdChange': -0.000305933324587714,
     'lastTradeTime': 1676142605530,
     'isUSMarketOpen': False}



## Parsing Our API Call
This API call has the metric we need - the price-to-earnings ratio.

Here is an example of how to parse the metric from our API call:


```python
pe_ratio = data['peRatio']
pe_ratio
```




    41.92



## Executing A Batch API Call & Building Our DataFrame

Just like in our first project, it's now time to execute several batch API calls and add the information we need to our DataFrame.

We'll start by running the following code cell, which contains some code we already built last time that we can re-use for this project. More specifically, it contains a function called chunks that we can use to divide our list of securities into groups of 100.


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

my_columns = ['Ticker', 'Price', 'Price-to-Earnings Ratio', 'Number of Shares to Buy']
```

Now we need to create a blank DataFrame and add our data to the data frame one-by-one.


```python
final_dataframe = pd.DataFrame(columns = my_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        final_dataframe = final_dataframe.append(
                                        pd.Series([symbol,
                                                   data[symbol]['quote']['latestPrice'],
                                                   data[symbol]['quote']['peRatio'],
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
      <th>Price-to-Earnings Ratio</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>121.51</td>
      <td>54</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>15.87</td>
      <td>-1.1</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>158.03</td>
      <td>23.72</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>134.02</td>
      <td>40.81</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>109.90</td>
      <td>24.03</td>
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
      <td>106.74</td>
      <td>32.13</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.88</td>
      <td>-217.5</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>387.65</td>
      <td>44.73</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>44.23</td>
      <td>19.04</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>165.52</td>
      <td>51.32</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



## Removing Glamour Stocks

The opposite of a "value stock" is a "glamour stock".

Since the goal of this strategy is to identify the 50 best value stocks from our universe, our next step is to remove glamour stocks from the DataFrame.

We'll sort the DataFrame by the stocks' price-to-earnings ratio, and drop all stocks outside the top 50.


```python
final_dataframe.sort_values('Price-to-Earnings Ratio', inplace = True)
final_dataframe = final_dataframe[final_dataframe['Price-to-Earnings Ratio'] > 0]
final_dataframe = final_dataframe[:50]
final_dataframe.reset_index(inplace = True)
final_dataframe.drop('index', axis=1, inplace = True)
```

## Calculating the Number of Shares to Buy
We now need to calculate the number of shares we need to buy.

To do this, we will use the `portfolio_input` function that we created in our momentum project.

I have included this function below.


```python
def portfolio_input():
    global portfolio_size
    portfolio_size = input("Enter the value of your portfolio:")

    try:
        val = float(portfolio_size)
    except ValueError:
        print("That's not a number! \n Try again:")
        portfolio_size = input("Enter the value of your portfolio:")
```

Use the `portfolio_input` function to accept a `portfolio_size` variable from the user of this script.


```python
portfolio_input()
```

    Enter the value of your portfolio:1000000


You can now use the global `portfolio_size` variable to calculate the number of shares that our strategy should purchase.


```python
position_size = float(portfolio_size) / len(final_dataframe.index)
for i in range(0, len(final_dataframe['Ticker'])):
    final_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / final_dataframe['Price'][i])
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
      <th>Price-to-Earnings Ratio</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NRG</td>
      <td>37.810</td>
      <td>2.27</td>
      <td>528</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NLOK</td>
      <td>21.130</td>
      <td>4.14</td>
      <td>946</td>
    </tr>
    <tr>
      <th>2</th>
      <td>UNM</td>
      <td>23.200</td>
      <td>4.91</td>
      <td>862</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AIV</td>
      <td>5.220</td>
      <td>5</td>
      <td>3831</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BIO</td>
      <td>606.400</td>
      <td>5.04</td>
      <td>32</td>
    </tr>
    <tr>
      <th>5</th>
      <td>KIM</td>
      <td>14.770</td>
      <td>6.53</td>
      <td>1354</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ALL</td>
      <td>108.640</td>
      <td>7.72</td>
      <td>184</td>
    </tr>
    <tr>
      <th>7</th>
      <td>MET</td>
      <td>48.040</td>
      <td>7.95</td>
      <td>416</td>
    </tr>
    <tr>
      <th>8</th>
      <td>EBAY</td>
      <td>53.580</td>
      <td>8.25</td>
      <td>373</td>
    </tr>
    <tr>
      <th>9</th>
      <td>CPB</td>
      <td>51.100</td>
      <td>8.4</td>
      <td>391</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BIIB</td>
      <td>249.510</td>
      <td>8.48</td>
      <td>80</td>
    </tr>
    <tr>
      <th>11</th>
      <td>KR</td>
      <td>33.100</td>
      <td>8.64</td>
      <td>604</td>
    </tr>
    <tr>
      <th>12</th>
      <td>PHM</td>
      <td>42.390</td>
      <td>8.84</td>
      <td>471</td>
    </tr>
    <tr>
      <th>13</th>
      <td>SRE</td>
      <td>126.290</td>
      <td>8.84</td>
      <td>158</td>
    </tr>
    <tr>
      <th>14</th>
      <td>PNC</td>
      <td>155.450</td>
      <td>9.16</td>
      <td>128</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BK</td>
      <td>43.550</td>
      <td>9.58</td>
      <td>459</td>
    </tr>
    <tr>
      <th>16</th>
      <td>NEE</td>
      <td>77.780</td>
      <td>9.67</td>
      <td>257</td>
    </tr>
    <tr>
      <th>17</th>
      <td>CTL</td>
      <td>11.000</td>
      <td>9.93</td>
      <td>1818</td>
    </tr>
    <tr>
      <th>18</th>
      <td>HBI</td>
      <td>14.400</td>
      <td>10</td>
      <td>1388</td>
    </tr>
    <tr>
      <th>19</th>
      <td>INTC</td>
      <td>51.560</td>
      <td>10</td>
      <td>387</td>
    </tr>
    <tr>
      <th>20</th>
      <td>HIG</td>
      <td>49.030</td>
      <td>10.09</td>
      <td>407</td>
    </tr>
    <tr>
      <th>21</th>
      <td>MAS</td>
      <td>54.800</td>
      <td>10.1</td>
      <td>364</td>
    </tr>
    <tr>
      <th>22</th>
      <td>MGM</td>
      <td>30.510</td>
      <td>10.35</td>
      <td>655</td>
    </tr>
    <tr>
      <th>23</th>
      <td>LEN</td>
      <td>76.140</td>
      <td>10.5</td>
      <td>262</td>
    </tr>
    <tr>
      <th>24</th>
      <td>DHI</td>
      <td>68.300</td>
      <td>10.79</td>
      <td>292</td>
    </tr>
    <tr>
      <th>25</th>
      <td>AFL</td>
      <td>44.710</td>
      <td>10.95</td>
      <td>447</td>
    </tr>
    <tr>
      <th>26</th>
      <td>PFG</td>
      <td>50.790</td>
      <td>11.17</td>
      <td>393</td>
    </tr>
    <tr>
      <th>27</th>
      <td>TSN</td>
      <td>66.040</td>
      <td>11.23</td>
      <td>302</td>
    </tr>
    <tr>
      <th>28</th>
      <td>FOXA</td>
      <td>29.959</td>
      <td>11.4</td>
      <td>667</td>
    </tr>
    <tr>
      <th>29</th>
      <td>PBCT</td>
      <td>13.110</td>
      <td>11.4</td>
      <td>1525</td>
    </tr>
    <tr>
      <th>30</th>
      <td>VIAC</td>
      <td>37.550</td>
      <td>11.66</td>
      <td>532</td>
    </tr>
    <tr>
      <th>31</th>
      <td>PGR</td>
      <td>101.290</td>
      <td>11.67</td>
      <td>197</td>
    </tr>
    <tr>
      <th>32</th>
      <td>CVS</td>
      <td>71.300</td>
      <td>11.72</td>
      <td>280</td>
    </tr>
    <tr>
      <th>33</th>
      <td>STT</td>
      <td>74.550</td>
      <td>11.78</td>
      <td>268</td>
    </tr>
    <tr>
      <th>34</th>
      <td>MS</td>
      <td>70.280</td>
      <td>11.96</td>
      <td>284</td>
    </tr>
    <tr>
      <th>35</th>
      <td>PPL</td>
      <td>28.450</td>
      <td>12.08</td>
      <td>702</td>
    </tr>
    <tr>
      <th>36</th>
      <td>C</td>
      <td>62.400</td>
      <td>12.32</td>
      <td>320</td>
    </tr>
    <tr>
      <th>37</th>
      <td>LNC</td>
      <td>50.070</td>
      <td>12.66</td>
      <td>399</td>
    </tr>
    <tr>
      <th>38</th>
      <td>DISH</td>
      <td>32.270</td>
      <td>13.19</td>
      <td>619</td>
    </tr>
    <tr>
      <th>39</th>
      <td>MTB</td>
      <td>129.990</td>
      <td>13.2</td>
      <td>153</td>
    </tr>
    <tr>
      <th>40</th>
      <td>HII</td>
      <td>171.070</td>
      <td>13.23</td>
      <td>116</td>
    </tr>
    <tr>
      <th>41</th>
      <td>VZ</td>
      <td>60.480</td>
      <td>13.32</td>
      <td>330</td>
    </tr>
    <tr>
      <th>42</th>
      <td>ABC</td>
      <td>99.370</td>
      <td>13.43</td>
      <td>201</td>
    </tr>
    <tr>
      <th>43</th>
      <td>BSX</td>
      <td>37.250</td>
      <td>13.44</td>
      <td>536</td>
    </tr>
    <tr>
      <th>44</th>
      <td>WHR</td>
      <td>180.260</td>
      <td>13.47</td>
      <td>110</td>
    </tr>
    <tr>
      <th>45</th>
      <td>NEM</td>
      <td>65.980</td>
      <td>13.49</td>
      <td>303</td>
    </tr>
    <tr>
      <th>46</th>
      <td>AMP</td>
      <td>193.250</td>
      <td>13.66</td>
      <td>103</td>
    </tr>
    <tr>
      <th>47</th>
      <td>HUM</td>
      <td>431.160</td>
      <td>13.72</td>
      <td>46</td>
    </tr>
    <tr>
      <th>48</th>
      <td>RE</td>
      <td>227.470</td>
      <td>13.74</td>
      <td>87</td>
    </tr>
    <tr>
      <th>49</th>
      <td>HPQ</td>
      <td>25.550</td>
      <td>13.86</td>
      <td>782</td>
    </tr>
  </tbody>
</table>
</div>



## Building a Better (and More Realistic) Value Strategy
Every valuation metric has certain flaws.

For example, the price-to-earnings ratio doesn't work well with stocks with negative earnings.

Similarly, stocks that buyback their own shares are difficult to value using the price-to-book ratio.

Investors typically use a `composite` basket of valuation metrics to build robust quantitative value strategies. In this section, we will filter for stocks with the lowest percentiles on the following metrics:

* Price-to-earnings ratio
* Price-to-book ratio
* Price-to-sales ratio
* Enterprise Value divided by Earnings Before Interest, Taxes, Depreciation, and Amortization (EV/EBITDA)
* Enterprise Value divided by Gross Profit (EV/GP)

Some of these metrics aren't provided directly by the IEX Cloud API, and must be computed after pulling raw data. We'll start by calculating each data point from scratch.


```python
symbol = 'AAPL'
batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=advanced-stats,quote&symbols={symbol}&token={IEX_CLOUD_API_TOKEN}'
data = requests.get(batch_api_call_url).json()

# P/E Ratio
pe_ratio = data[symbol]['quote']['peRatio']

# P/B Ratio
pb_ratio = data[symbol]['advanced-stats']['priceToBook']

#P/S Ratio
ps_ratio = data[symbol]['advanced-stats']['priceToSales']

# EV/EBITDA
enterprise_value = data[symbol]['advanced-stats']['enterpriseValue']
ebitda = data[symbol]['advanced-stats']['EBITDA']
ev_to_ebitda = enterprise_value/ebitda

# EV/GP
gross_profit = data[symbol]['advanced-stats']['grossProfit']
ev_to_gross_profit = enterprise_value/gross_profit
```

Let's move on to building our DataFrame. You'll notice that I use the abbreviation `rv` often. It stands for `robust value`, which is what we'll call this sophisticated strategy moving forward.


```python
rv_columns = [
    'Ticker',
    'Price',
    'Number of Shares to Buy',
    'Price-to-Earnings Ratio',
    'PE Percentile',
    'Price-to-Book Ratio',
    'PB Percentile',
    'Price-to-Sales Ratio',
    'PS Percentile',
    'EV/EBITDA',
    'EV/EBITDA Percentile',
    'EV/GP',
    'EV/GP Percentile',
    'RV Score'
]

rv_dataframe = pd.DataFrame(columns = rv_columns)

for symbol_string in symbol_strings:
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch?symbols={symbol_string}&types=quote,advanced-stats&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        enterprise_value = data[symbol]['advanced-stats']['enterpriseValue']
        ebitda = data[symbol]['advanced-stats']['EBITDA']
        gross_profit = data[symbol]['advanced-stats']['grossProfit']

        try:
            ev_to_ebitda = enterprise_value/ebitda
        except TypeError:
            ev_to_ebitda = np.NaN

        try:
            ev_to_gross_profit = enterprise_value/gross_profit
        except TypeError:
            ev_to_gross_profit = np.NaN

        rv_dataframe = rv_dataframe.append(
            pd.Series([
                symbol,
                data[symbol]['quote']['latestPrice'],
                'N/A',
                data[symbol]['quote']['peRatio'],
                'N/A',
                data[symbol]['advanced-stats']['priceToBook'],
                'N/A',
                data[symbol]['advanced-stats']['priceToSales'],
                'N/A',
                ev_to_ebitda,
                'N/A',
                ev_to_gross_profit,
                'N/A',
                'N/A'
        ],
        index = rv_columns),
            ignore_index = True
        )
```

## Dealing With Missing Data in Our DataFrame

Our DataFrame contains some missing data because all of the metrics we require are not available through the API we're using.

You can use pandas' `isnull` method to identify missing data:


```python
rv_dataframe[rv_dataframe.isnull().any(axis=1)]
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
      <th>Price-to-Earnings Ratio</th>
      <th>PE Percentile</th>
      <th>Price-to-Book Ratio</th>
      <th>PB Percentile</th>
      <th>Price-to-Sales Ratio</th>
      <th>PS Percentile</th>
      <th>EV/EBITDA</th>
      <th>EV/EBITDA Percentile</th>
      <th>EV/GP</th>
      <th>EV/GP Percentile</th>
      <th>RV Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>40</th>
      <td>AON</td>
      <td>214.500</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>71</th>
      <td>BRK.B</td>
      <td>227.970</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>118</th>
      <td>CTL</td>
      <td>11.000</td>
      <td>N/A</td>
      <td>10.14</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>136</th>
      <td>DISCK</td>
      <td>27.230</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>165</th>
      <td>ETFC</td>
      <td>51.140</td>
      <td>N/A</td>
      <td>14.39</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>168</th>
      <td>EVRG</td>
      <td>53.890</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>190</th>
      <td>FOX</td>
      <td>29.867</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>192</th>
      <td>FRC</td>
      <td>155.700</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>194</th>
      <td>FTI</td>
      <td>10.710</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>204</th>
      <td>GOOG</td>
      <td>1781.780</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>326</th>
      <td>MYL</td>
      <td>15.972</td>
      <td>N/A</td>
      <td>31.42</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>327</th>
      <td>NBL</td>
      <td>8.740</td>
      <td>N/A</td>
      <td>-0.75</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>348</th>
      <td>NWS</td>
      <td>18.510</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>363</th>
      <td>PEG</td>
      <td>58.020</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>442</th>
      <td>TROW</td>
      <td>150.740</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>452</th>
      <td>UA</td>
      <td>15.620</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>454</th>
      <td>UAL</td>
      <td>45.300</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>498</th>
      <td>XRX</td>
      <td>22.990</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>None</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>NaN</td>
      <td>N/A</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
</div>



Dealing with missing data is an important topic in data science.

There are two main approaches:

* Drop missing data from the data set (pandas' `dropna` method is useful here)
* Replace missing data with a new value (pandas' `fillna` method is useful here)

In this tutorial, we will replace missing data with the average non-`NaN` data point from that column.

Here is the code to do this:


```python
for column in ['Price-to-Earnings Ratio', 'Price-to-Book Ratio','Price-to-Sales Ratio',  'EV/EBITDA','EV/GP']:
    rv_dataframe[column].fillna(rv_dataframe[column].mean(), inplace = True)
```

Now, if we run the statement from earlier to print rows that contain missing data, nothing should be returned:


```python
rv_dataframe[rv_dataframe.isnull().any(axis=1)]
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
      <th>Price-to-Earnings Ratio</th>
      <th>PE Percentile</th>
      <th>Price-to-Book Ratio</th>
      <th>PB Percentile</th>
      <th>Price-to-Sales Ratio</th>
      <th>PS Percentile</th>
      <th>EV/EBITDA</th>
      <th>EV/EBITDA Percentile</th>
      <th>EV/GP</th>
      <th>EV/GP Percentile</th>
      <th>RV Score</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



## Calculating Value Percentiles

We now need to calculate value score percentiles for every stock in the universe. More specifically, we need to calculate percentile scores for the following metrics for every stock:

* Price-to-earnings ratio
* Price-to-book ratio
* Price-to-sales ratio
* EV/EBITDA
* EV/GP

Here's how we'll do this:


```python
metrics = {
            'Price-to-Earnings Ratio': 'PE Percentile',
            'Price-to-Book Ratio':'PB Percentile',
            'Price-to-Sales Ratio': 'PS Percentile',
            'EV/EBITDA':'EV/EBITDA Percentile',
            'EV/GP':'EV/GP Percentile'
}

for row in rv_dataframe.index:
    for metric in metrics.keys():
        rv_dataframe.loc[row, metrics[metric]] = stats.percentileofscore(rv_dataframe[metric], rv_dataframe.loc[row, metric])/100

# Print each percentile score to make sure it was calculated properly
for metric in metrics.values():
    print(rv_dataframe[metric])

#Print the entire DataFrame    
rv_dataframe
```

    0      0.859406
    1      0.144554
    2      0.453465
    3      0.746535
    4      0.455446
             ...   
    500    0.642574
    501    0.019802
    502    0.774257
    503    0.360396
    504    0.829703
    Name: PE Percentile, Length: 505, dtype: object
    0       0.732673
    1      0.0455446
    2       0.409901
    3       0.962376
    4       0.849505
             ...    
    500    0.0415842
    501     0.382178
    502     0.829703
    503      0.10396
    504     0.944554
    Name: PB Percentile, Length: 505, dtype: object
    0      0.809901
    1      0.029703
    2      0.161386
    3      0.851485
    4      0.671287
             ...   
    500    0.752475
    501    0.679208
    502    0.677228
    503     0.40198
    504    0.936634
    Name: PS Percentile, Length: 505, dtype: object
    0       0.843564
    1      0.0277228
    2       0.269307
    3        0.80198
    4       0.489109
             ...    
    500     0.722772
    501     0.778218
    502     0.819802
    503     0.235644
    504     0.853465
    Name: EV/EBITDA Percentile, Length: 505, dtype: object
    0       0.813861
    1      0.0574257
    2       0.130693
    3       0.942574
    4       0.643564
             ...    
    500     0.681188
    501     0.518812
    502     0.724752
    503     0.138614
    504     0.924752
    Name: EV/GP Percentile, Length: 505, dtype: object





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
      <th>Price-to-Earnings Ratio</th>
      <th>PE Percentile</th>
      <th>Price-to-Book Ratio</th>
      <th>PB Percentile</th>
      <th>Price-to-Sales Ratio</th>
      <th>PS Percentile</th>
      <th>EV/EBITDA</th>
      <th>EV/EBITDA Percentile</th>
      <th>EV/GP</th>
      <th>EV/GP Percentile</th>
      <th>RV Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>125.29</td>
      <td>N/A</td>
      <td>54.00</td>
      <td>0.859406</td>
      <td>7.77</td>
      <td>0.732673</td>
      <td>7.1200</td>
      <td>0.809901</td>
      <td>33.527582</td>
      <td>0.843564</td>
      <td>13.473834</td>
      <td>0.813861</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>15.56</td>
      <td>N/A</td>
      <td>-1.10</td>
      <td>0.144554</td>
      <td>-1.73</td>
      <td>0.0455446</td>
      <td>0.3912</td>
      <td>0.029703</td>
      <td>-6.076622</td>
      <td>0.0277228</td>
      <td>1.435641</td>
      <td>0.0574257</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.65</td>
      <td>N/A</td>
      <td>23.53</td>
      <td>0.453465</td>
      <td>2.96</td>
      <td>0.409901</td>
      <td>1.1100</td>
      <td>0.161386</td>
      <td>11.039143</td>
      <td>0.269307</td>
      <td>2.559108</td>
      <td>0.130693</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>137.01</td>
      <td>N/A</td>
      <td>41.41</td>
      <td>0.746535</td>
      <td>34.35</td>
      <td>0.962376</td>
      <td>8.3700</td>
      <td>0.851485</td>
      <td>28.413277</td>
      <td>0.80198</td>
      <td>20.964473</td>
      <td>0.942574</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>109.40</td>
      <td>N/A</td>
      <td>23.65</td>
      <td>0.455446</td>
      <td>12.50</td>
      <td>0.849505</td>
      <td>4.7100</td>
      <td>0.671287</td>
      <td>15.907432</td>
      <td>0.489109</td>
      <td>9.424546</td>
      <td>0.643564</td>
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
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>109.24</td>
      <td>N/A</td>
      <td>32.57</td>
      <td>0.642574</td>
      <td>-4.11</td>
      <td>0.0415842</td>
      <td>5.8800</td>
      <td>0.752475</td>
      <td>23.289025</td>
      <td>0.722772</td>
      <td>10.093876</td>
      <td>0.681188</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.94</td>
      <td>N/A</td>
      <td>-216.40</td>
      <td>0.019802</td>
      <td>2.75</td>
      <td>0.382178</td>
      <td>4.7700</td>
      <td>0.679208</td>
      <td>26.435894</td>
      <td>0.778218</td>
      <td>7.943749</td>
      <td>0.518812</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>385.16</td>
      <td>N/A</td>
      <td>44.17</td>
      <td>0.774257</td>
      <td>10.91</td>
      <td>0.829703</td>
      <td>4.7600</td>
      <td>0.677228</td>
      <td>29.585640</td>
      <td>0.819802</td>
      <td>11.387431</td>
      <td>0.724752</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>45.75</td>
      <td>N/A</td>
      <td>18.69</td>
      <td>0.360396</td>
      <td>1.04</td>
      <td>0.10396</td>
      <td>2.4500</td>
      <td>0.40198</td>
      <td>10.501364</td>
      <td>0.235644</td>
      <td>2.687455</td>
      <td>0.138614</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>168.09</td>
      <td>N/A</td>
      <td>50.46</td>
      <td>0.829703</td>
      <td>27.04</td>
      <td>0.944554</td>
      <td>12.8900</td>
      <td>0.936634</td>
      <td>35.640199</td>
      <td>0.853465</td>
      <td>18.828561</td>
      <td>0.924752</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 14 columns</p>
</div>



## Calculating the RV Score
We'll now calculate our RV Score (which stands for Robust Value), which is the value score that we'll use to filter for stocks in this investing strategy.

The RV Score will be the arithmetic mean of the 4 percentile scores that we calculated in the last section.

To calculate arithmetic mean, we will use the mean function from Python's built-in statistics module.


```python
from statistics import mean

for row in rv_dataframe.index:
    value_percentiles = []
    for metric in metrics.keys():
        value_percentiles.append(rv_dataframe.loc[row, metrics[metric]])
    rv_dataframe.loc[row, 'RV Score'] = mean(value_percentiles)

rv_dataframe
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
      <th>Price-to-Earnings Ratio</th>
      <th>PE Percentile</th>
      <th>Price-to-Book Ratio</th>
      <th>PB Percentile</th>
      <th>Price-to-Sales Ratio</th>
      <th>PS Percentile</th>
      <th>EV/EBITDA</th>
      <th>EV/EBITDA Percentile</th>
      <th>EV/GP</th>
      <th>EV/GP Percentile</th>
      <th>RV Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>125.29</td>
      <td>N/A</td>
      <td>54.00</td>
      <td>0.859406</td>
      <td>7.77</td>
      <td>0.732673</td>
      <td>7.1200</td>
      <td>0.809901</td>
      <td>33.527582</td>
      <td>0.843564</td>
      <td>13.473834</td>
      <td>0.813861</td>
      <td>0.811881</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>15.56</td>
      <td>N/A</td>
      <td>-1.10</td>
      <td>0.144554</td>
      <td>-1.73</td>
      <td>0.0455446</td>
      <td>0.3912</td>
      <td>0.029703</td>
      <td>-6.076622</td>
      <td>0.0277228</td>
      <td>1.435641</td>
      <td>0.0574257</td>
      <td>0.0609901</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.65</td>
      <td>N/A</td>
      <td>23.53</td>
      <td>0.453465</td>
      <td>2.96</td>
      <td>0.409901</td>
      <td>1.1100</td>
      <td>0.161386</td>
      <td>11.039143</td>
      <td>0.269307</td>
      <td>2.559108</td>
      <td>0.130693</td>
      <td>0.28495</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>137.01</td>
      <td>N/A</td>
      <td>41.41</td>
      <td>0.746535</td>
      <td>34.35</td>
      <td>0.962376</td>
      <td>8.3700</td>
      <td>0.851485</td>
      <td>28.413277</td>
      <td>0.80198</td>
      <td>20.964473</td>
      <td>0.942574</td>
      <td>0.86099</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>109.40</td>
      <td>N/A</td>
      <td>23.65</td>
      <td>0.455446</td>
      <td>12.50</td>
      <td>0.849505</td>
      <td>4.7100</td>
      <td>0.671287</td>
      <td>15.907432</td>
      <td>0.489109</td>
      <td>9.424546</td>
      <td>0.643564</td>
      <td>0.621782</td>
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
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>109.24</td>
      <td>N/A</td>
      <td>32.57</td>
      <td>0.642574</td>
      <td>-4.11</td>
      <td>0.0415842</td>
      <td>5.8800</td>
      <td>0.752475</td>
      <td>23.289025</td>
      <td>0.722772</td>
      <td>10.093876</td>
      <td>0.681188</td>
      <td>0.568119</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>159.94</td>
      <td>N/A</td>
      <td>-216.40</td>
      <td>0.019802</td>
      <td>2.75</td>
      <td>0.382178</td>
      <td>4.7700</td>
      <td>0.679208</td>
      <td>26.435894</td>
      <td>0.778218</td>
      <td>7.943749</td>
      <td>0.518812</td>
      <td>0.475644</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>385.16</td>
      <td>N/A</td>
      <td>44.17</td>
      <td>0.774257</td>
      <td>10.91</td>
      <td>0.829703</td>
      <td>4.7600</td>
      <td>0.677228</td>
      <td>29.585640</td>
      <td>0.819802</td>
      <td>11.387431</td>
      <td>0.724752</td>
      <td>0.765149</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>45.75</td>
      <td>N/A</td>
      <td>18.69</td>
      <td>0.360396</td>
      <td>1.04</td>
      <td>0.10396</td>
      <td>2.4500</td>
      <td>0.40198</td>
      <td>10.501364</td>
      <td>0.235644</td>
      <td>2.687455</td>
      <td>0.138614</td>
      <td>0.248119</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>168.09</td>
      <td>N/A</td>
      <td>50.46</td>
      <td>0.829703</td>
      <td>27.04</td>
      <td>0.944554</td>
      <td>12.8900</td>
      <td>0.936634</td>
      <td>35.640199</td>
      <td>0.853465</td>
      <td>18.828561</td>
      <td>0.924752</td>
      <td>0.897822</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 14 columns</p>
</div>



## Selecting the 50 Best Value Stocks¶

As before, we can identify the 50 best value stocks in our universe by sorting the DataFrame on the RV Score column and dropping all but the top 50 entries.


```python
rv_dataframe.sort_values(by = 'RV Score', inplace = True)
rv_dataframe = rv_dataframe[:50]
rv_dataframe.reset_index(drop = True, inplace = True)
```

## Calculating the Number of Shares to Buy
We'll use the `portfolio_input` function that we created earlier to accept our portfolio size. Then we will use similar logic in a for loop to calculate the number of shares to buy for each stock in our investment universe.


```python
portfolio_input()
```

    Enter the value of your portfolio:1000000



```python
position_size = float(portfolio_size) / len(rv_dataframe.index)
for i in range(0, len(rv_dataframe['Ticker'])-1):
    rv_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / rv_dataframe['Price'][i])
rv_dataframe
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
      <th>Price-to-Earnings Ratio</th>
      <th>PE Percentile</th>
      <th>Price-to-Book Ratio</th>
      <th>PB Percentile</th>
      <th>Price-to-Sales Ratio</th>
      <th>PS Percentile</th>
      <th>EV/EBITDA</th>
      <th>EV/EBITDA Percentile</th>
      <th>EV/GP</th>
      <th>EV/GP Percentile</th>
      <th>RV Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>UNM</td>
      <td>23.08</td>
      <td>866</td>
      <td>4.88</td>
      <td>0.154455</td>
      <td>0.4228</td>
      <td>0.049505</td>
      <td>0.3837</td>
      <td>0.0277228</td>
      <td>2.427957</td>
      <td>0.0316832</td>
      <td>0.378642</td>
      <td>0.0039604</td>
      <td>0.0534653</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AIG</td>
      <td>38.24</td>
      <td>523</td>
      <td>-6.75</td>
      <td>0.106931</td>
      <td>0.5161</td>
      <td>0.0554455</td>
      <td>0.7362</td>
      <td>0.0633663</td>
      <td>4.176675</td>
      <td>0.0415842</td>
      <td>0.683633</td>
      <td>0.0138614</td>
      <td>0.0562376</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAL</td>
      <td>15.56</td>
      <td>1285</td>
      <td>-1.10</td>
      <td>0.144554</td>
      <td>-1.7300</td>
      <td>0.0455446</td>
      <td>0.3912</td>
      <td>0.029703</td>
      <td>-6.076622</td>
      <td>0.0277228</td>
      <td>1.435641</td>
      <td>0.0574257</td>
      <td>0.0609901</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MET</td>
      <td>48.92</td>
      <td>408</td>
      <td>8.03</td>
      <td>0.164356</td>
      <td>0.5959</td>
      <td>0.0613861</td>
      <td>0.6400</td>
      <td>0.0554455</td>
      <td>4.168950</td>
      <td>0.039604</td>
      <td>0.612110</td>
      <td>0.0118812</td>
      <td>0.0665347</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HFC</td>
      <td>26.22</td>
      <td>762</td>
      <td>-27.30</td>
      <td>0.0673267</td>
      <td>0.8000</td>
      <td>0.0772277</td>
      <td>0.2960</td>
      <td>0.0138614</td>
      <td>6.242916</td>
      <td>0.0613861</td>
      <td>2.213951</td>
      <td>0.114851</td>
      <td>0.0669307</td>
    </tr>
    <tr>
      <th>5</th>
      <td>F</td>
      <td>8.98</td>
      <td>2227</td>
      <td>-16.33</td>
      <td>0.0831683</td>
      <td>1.1000</td>
      <td>0.117822</td>
      <td>0.2633</td>
      <td>0.0118812</td>
      <td>4.502256</td>
      <td>0.0435644</td>
      <td>2.032083</td>
      <td>0.10099</td>
      <td>0.0714851</td>
    </tr>
    <tr>
      <th>6</th>
      <td>HIG</td>
      <td>49.80</td>
      <td>401</td>
      <td>10.28</td>
      <td>0.192079</td>
      <td>1.0030</td>
      <td>0.0970297</td>
      <td>0.8624</td>
      <td>0.0910891</td>
      <td>4.069466</td>
      <td>0.0376238</td>
      <td>0.821334</td>
      <td>0.019802</td>
      <td>0.0875248</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ALL</td>
      <td>112.19</td>
      <td>178</td>
      <td>7.72</td>
      <td>0.162376</td>
      <td>1.3300</td>
      <td>0.175248</td>
      <td>0.7606</td>
      <td>0.0693069</td>
      <td>2.434020</td>
      <td>0.0336634</td>
      <td>0.722561</td>
      <td>0.0158416</td>
      <td>0.0912871</td>
    </tr>
    <tr>
      <th>8</th>
      <td>BA</td>
      <td>221.09</td>
      <td>90</td>
      <td>-27.81</td>
      <td>0.0653465</td>
      <td>-10.5900</td>
      <td>0.0316832</td>
      <td>2.0200</td>
      <td>0.343564</td>
      <td>-28.805545</td>
      <td>0.0158416</td>
      <td>-205.924417</td>
      <td>0.0019802</td>
      <td>0.0916832</td>
    </tr>
    <tr>
      <th>9</th>
      <td>KSS</td>
      <td>39.63</td>
      <td>504</td>
      <td>-58.83</td>
      <td>0.049505</td>
      <td>1.2900</td>
      <td>0.166337</td>
      <td>0.3700</td>
      <td>0.0247525</td>
      <td>9.657999</td>
      <td>0.188119</td>
      <td>1.435561</td>
      <td>0.0554455</td>
      <td>0.0968317</td>
    </tr>
    <tr>
      <th>10</th>
      <td>LNC</td>
      <td>49.18</td>
      <td>406</td>
      <td>12.54</td>
      <td>0.223762</td>
      <td>0.4495</td>
      <td>0.0514851</td>
      <td>0.5637</td>
      <td>0.0475248</td>
      <td>8.958868</td>
      <td>0.158416</td>
      <td>0.525545</td>
      <td>0.00792079</td>
      <td>0.0978218</td>
    </tr>
    <tr>
      <th>11</th>
      <td>PFG</td>
      <td>50.92</td>
      <td>392</td>
      <td>11.50</td>
      <td>0.209901</td>
      <td>0.8602</td>
      <td>0.0851485</td>
      <td>0.9001</td>
      <td>0.0990099</td>
      <td>7.328522</td>
      <td>0.0891089</td>
      <td>0.908096</td>
      <td>0.029703</td>
      <td>0.102574</td>
    </tr>
    <tr>
      <th>12</th>
      <td>VIAC</td>
      <td>38.00</td>
      <td>526</td>
      <td>11.45</td>
      <td>0.207921</td>
      <td>1.5700</td>
      <td>0.206931</td>
      <td>0.6624</td>
      <td>0.0574257</td>
      <td>6.096216</td>
      <td>0.0594059</td>
      <td>1.125661</td>
      <td>0.0356436</td>
      <td>0.113465</td>
    </tr>
    <tr>
      <th>13</th>
      <td>LB</td>
      <td>42.03</td>
      <td>475</td>
      <td>-54.40</td>
      <td>0.0514851</td>
      <td>-7.4700</td>
      <td>0.0356436</td>
      <td>0.9583</td>
      <td>0.116832</td>
      <td>9.088248</td>
      <td>0.164356</td>
      <td>3.543749</td>
      <td>0.205941</td>
      <td>0.114851</td>
    </tr>
    <tr>
      <th>14</th>
      <td>L</td>
      <td>44.93</td>
      <td>445</td>
      <td>-11.87</td>
      <td>0.0950495</td>
      <td>0.7392</td>
      <td>0.0653465</td>
      <td>0.9179</td>
      <td>0.10297</td>
      <td>11.427712</td>
      <td>0.293069</td>
      <td>0.893883</td>
      <td>0.0257426</td>
      <td>0.116436</td>
    </tr>
    <tr>
      <th>15</th>
      <td>CVS</td>
      <td>71.62</td>
      <td>279</td>
      <td>12.03</td>
      <td>0.215842</td>
      <td>1.3500</td>
      <td>0.179208</td>
      <td>0.3612</td>
      <td>0.0217822</td>
      <td>7.971860</td>
      <td>0.118812</td>
      <td>1.464570</td>
      <td>0.0594059</td>
      <td>0.11901</td>
    </tr>
    <tr>
      <th>16</th>
      <td>HPQ</td>
      <td>24.79</td>
      <td>806</td>
      <td>13.90</td>
      <td>0.253465</td>
      <td>-16.6400</td>
      <td>0.0237624</td>
      <td>0.5601</td>
      <td>0.0455446</td>
      <td>8.264951</td>
      <td>0.128713</td>
      <td>3.085663</td>
      <td>0.178218</td>
      <td>0.125941</td>
    </tr>
    <tr>
      <th>17</th>
      <td>MRO</td>
      <td>7.64</td>
      <td>2617</td>
      <td>-9.30</td>
      <td>0.0990099</td>
      <td>0.5514</td>
      <td>0.0574257</td>
      <td>1.5200</td>
      <td>0.249505</td>
      <td>4.968392</td>
      <td>0.0475248</td>
      <td>3.121976</td>
      <td>0.180198</td>
      <td>0.126733</td>
    </tr>
    <tr>
      <th>18</th>
      <td>APA</td>
      <td>16.49</td>
      <td>1212</td>
      <td>-0.77</td>
      <td>0.146535</td>
      <td>-3.8300</td>
      <td>0.0435644</td>
      <td>1.2200</td>
      <td>0.182178</td>
      <td>7.927769</td>
      <td>0.116832</td>
      <td>2.800306</td>
      <td>0.146535</td>
      <td>0.127129</td>
    </tr>
    <tr>
      <th>19</th>
      <td>KR</td>
      <td>32.10</td>
      <td>623</td>
      <td>8.74</td>
      <td>0.172277</td>
      <td>2.5100</td>
      <td>0.362376</td>
      <td>0.1891</td>
      <td>0.00792079</td>
      <td>5.654425</td>
      <td>0.049505</td>
      <td>1.180783</td>
      <td>0.0455446</td>
      <td>0.127525</td>
    </tr>
    <tr>
      <th>20</th>
      <td>AFL</td>
      <td>44.57</td>
      <td>448</td>
      <td>10.83</td>
      <td>0.2</td>
      <td>1.0500</td>
      <td>0.106931</td>
      <td>1.4500</td>
      <td>0.228713</td>
      <td>5.955477</td>
      <td>0.0534653</td>
      <td>1.407816</td>
      <td>0.0534653</td>
      <td>0.128515</td>
    </tr>
    <tr>
      <th>21</th>
      <td>DXC</td>
      <td>28.01</td>
      <td>714</td>
      <td>-1.80</td>
      <td>0.135644</td>
      <td>1.6100</td>
      <td>0.220792</td>
      <td>0.3700</td>
      <td>0.0247525</td>
      <td>6.870793</td>
      <td>0.0811881</td>
      <td>3.285088</td>
      <td>0.190099</td>
      <td>0.130495</td>
    </tr>
    <tr>
      <th>22</th>
      <td>PRU</td>
      <td>78.63</td>
      <td>254</td>
      <td>-191.17</td>
      <td>0.0217822</td>
      <td>0.4645</td>
      <td>0.0534653</td>
      <td>0.5220</td>
      <td>0.039604</td>
      <td>17.786351</td>
      <td>0.544554</td>
      <td>0.498539</td>
      <td>0.00594059</td>
      <td>0.133069</td>
    </tr>
    <tr>
      <th>23</th>
      <td>TAP</td>
      <td>49.00</td>
      <td>408</td>
      <td>-65.25</td>
      <td>0.0435644</td>
      <td>0.8147</td>
      <td>0.0792079</td>
      <td>1.0700</td>
      <td>0.152475</td>
      <td>7.783166</td>
      <td>0.106931</td>
      <td>4.776156</td>
      <td>0.285149</td>
      <td>0.133465</td>
    </tr>
    <tr>
      <th>24</th>
      <td>HRB</td>
      <td>16.44</td>
      <td>1216</td>
      <td>15.43</td>
      <td>0.285149</td>
      <td>-10.6900</td>
      <td>0.0277228</td>
      <td>0.9648</td>
      <td>0.120792</td>
      <td>5.928566</td>
      <td>0.0514851</td>
      <td>3.271309</td>
      <td>0.188119</td>
      <td>0.134653</td>
    </tr>
    <tr>
      <th>25</th>
      <td>TRV</td>
      <td>140.36</td>
      <td>142</td>
      <td>15.65</td>
      <td>0.291089</td>
      <td>1.2300</td>
      <td>0.148515</td>
      <td>1.0800</td>
      <td>0.155446</td>
      <td>4.672264</td>
      <td>0.0455446</td>
      <td>1.076357</td>
      <td>0.0336634</td>
      <td>0.134851</td>
    </tr>
    <tr>
      <th>26</th>
      <td>GPS</td>
      <td>19.93</td>
      <td>1003</td>
      <td>-6.91</td>
      <td>0.10495</td>
      <td>3.1600</td>
      <td>0.438614</td>
      <td>0.5511</td>
      <td>0.0435644</td>
      <td>-10.046970</td>
      <td>0.0257426</td>
      <td>1.569539</td>
      <td>0.0633663</td>
      <td>0.135248</td>
    </tr>
    <tr>
      <th>27</th>
      <td>C</td>
      <td>63.60</td>
      <td>314</td>
      <td>12.38</td>
      <td>0.221782</td>
      <td>0.7647</td>
      <td>0.0673267</td>
      <td>1.4200</td>
      <td>0.221782</td>
      <td>7.303864</td>
      <td>0.0871287</td>
      <td>1.737499</td>
      <td>0.0811881</td>
      <td>0.135842</td>
    </tr>
    <tr>
      <th>28</th>
      <td>VLO</td>
      <td>58.01</td>
      <td>344</td>
      <td>-2124.50</td>
      <td>0.0019802</td>
      <td>1.2200</td>
      <td>0.144554</td>
      <td>0.3125</td>
      <td>0.0158416</td>
      <td>11.381653</td>
      <td>0.289109</td>
      <td>4.149409</td>
      <td>0.241584</td>
      <td>0.138614</td>
    </tr>
    <tr>
      <th>29</th>
      <td>AIZ</td>
      <td>133.32</td>
      <td>150</td>
      <td>19.73</td>
      <td>0.394059</td>
      <td>1.3500</td>
      <td>0.179208</td>
      <td>0.7706</td>
      <td>0.0732673</td>
      <td>1.753216</td>
      <td>0.029703</td>
      <td>0.733802</td>
      <td>0.0178218</td>
      <td>0.138812</td>
    </tr>
    <tr>
      <th>30</th>
      <td>OXY</td>
      <td>19.54</td>
      <td>1023</td>
      <td>-1.16</td>
      <td>0.142574</td>
      <td>1.8100</td>
      <td>0.263366</td>
      <td>0.9227</td>
      <td>0.10495</td>
      <td>6.365344</td>
      <td>0.0653465</td>
      <td>2.851170</td>
      <td>0.150495</td>
      <td>0.145347</td>
    </tr>
    <tr>
      <th>31</th>
      <td>NWSA</td>
      <td>18.48</td>
      <td>1082</td>
      <td>-11.11</td>
      <td>0.0970297</td>
      <td>1.4500</td>
      <td>0.190099</td>
      <td>1.2500</td>
      <td>0.19505</td>
      <td>10.272083</td>
      <td>0.213861</td>
      <td>1.178007</td>
      <td>0.0435644</td>
      <td>0.147921</td>
    </tr>
    <tr>
      <th>32</th>
      <td>WRK</td>
      <td>44.05</td>
      <td>454</td>
      <td>-16.50</td>
      <td>0.0811881</td>
      <td>1.1200</td>
      <td>0.124752</td>
      <td>0.6646</td>
      <td>0.0594059</td>
      <td>7.551311</td>
      <td>0.0970297</td>
      <td>6.394482</td>
      <td>0.388119</td>
      <td>0.150099</td>
    </tr>
    <tr>
      <th>33</th>
      <td>GM</td>
      <td>42.27</td>
      <td>473</td>
      <td>19.35</td>
      <td>0.384158</td>
      <td>1.3900</td>
      <td>0.183168</td>
      <td>0.5210</td>
      <td>0.0376238</td>
      <td>3.836868</td>
      <td>0.0356436</td>
      <td>2.280291</td>
      <td>0.116832</td>
      <td>0.151485</td>
    </tr>
    <tr>
      <th>34</th>
      <td>HCA</td>
      <td>167.26</td>
      <td>119</td>
      <td>17.22</td>
      <td>0.336634</td>
      <td>-31.7700</td>
      <td>0.0118812</td>
      <td>1.1300</td>
      <td>0.166337</td>
      <td>9.319009</td>
      <td>0.172277</td>
      <td>1.666852</td>
      <td>0.0732673</td>
      <td>0.152079</td>
    </tr>
    <tr>
      <th>35</th>
      <td>BK</td>
      <td>43.50</td>
      <td>459</td>
      <td>9.48</td>
      <td>0.180198</td>
      <td>0.9549</td>
      <td>0.0891089</td>
      <td>2.0000</td>
      <td>0.335644</td>
      <td>6.543070</td>
      <td>0.0693069</td>
      <td>2.020461</td>
      <td>0.0990099</td>
      <td>0.154653</td>
    </tr>
    <tr>
      <th>36</th>
      <td>AIV</td>
      <td>5.06</td>
      <td>3952</td>
      <td>5.01</td>
      <td>0.156436</td>
      <td>0.3850</td>
      <td>0.0475248</td>
      <td>0.9104</td>
      <td>0.10099</td>
      <td>9.208645</td>
      <td>0.168317</td>
      <td>5.652447</td>
      <td>0.352475</td>
      <td>0.165149</td>
    </tr>
    <tr>
      <th>37</th>
      <td>UHS</td>
      <td>137.50</td>
      <td>145</td>
      <td>13.76</td>
      <td>0.247525</td>
      <td>1.9700</td>
      <td>0.281188</td>
      <td>1.0300</td>
      <td>0.142574</td>
      <td>7.827764</td>
      <td>0.110891</td>
      <td>1.221659</td>
      <td>0.0475248</td>
      <td>0.165941</td>
    </tr>
    <tr>
      <th>38</th>
      <td>CVX</td>
      <td>90.00</td>
      <td>222</td>
      <td>-19.46</td>
      <td>0.0732673</td>
      <td>1.2600</td>
      <td>0.156436</td>
      <td>1.4300</td>
      <td>0.224752</td>
      <td>9.918509</td>
      <td>0.20198</td>
      <td>3.781557</td>
      <td>0.215842</td>
      <td>0.174455</td>
    </tr>
    <tr>
      <th>39</th>
      <td>RE</td>
      <td>231.07</td>
      <td>86</td>
      <td>13.81</td>
      <td>0.249505</td>
      <td>0.9446</td>
      <td>0.0871287</td>
      <td>1.0239</td>
      <td>0.136634</td>
      <td>13.081357</td>
      <td>0.370297</td>
      <td>0.998505</td>
      <td>0.0316832</td>
      <td>0.17505</td>
    </tr>
    <tr>
      <th>40</th>
      <td>EOG</td>
      <td>54.65</td>
      <td>365</td>
      <td>-104.86</td>
      <td>0.0277228</td>
      <td>1.5800</td>
      <td>0.209901</td>
      <td>2.5800</td>
      <td>0.429703</td>
      <td>6.262858</td>
      <td>0.0633663</td>
      <td>2.762529</td>
      <td>0.144554</td>
      <td>0.17505</td>
    </tr>
    <tr>
      <th>41</th>
      <td>COTY</td>
      <td>7.21</td>
      <td>2773</td>
      <td>-6.70</td>
      <td>0.108911</td>
      <td>1.2900</td>
      <td>0.166337</td>
      <td>1.3600</td>
      <td>0.212871</td>
      <td>-29.184145</td>
      <td>0.0138614</td>
      <td>6.144947</td>
      <td>0.376238</td>
      <td>0.175644</td>
    </tr>
    <tr>
      <th>42</th>
      <td>CCL</td>
      <td>20.90</td>
      <td>956</td>
      <td>-1.98</td>
      <td>0.132673</td>
      <td>1.1800</td>
      <td>0.132673</td>
      <td>2.3000</td>
      <td>0.386139</td>
      <td>-16.597463</td>
      <td>0.0178218</td>
      <td>3.675844</td>
      <td>0.213861</td>
      <td>0.176634</td>
    </tr>
    <tr>
      <th>43</th>
      <td>IVZ</td>
      <td>18.11</td>
      <td>1104</td>
      <td>16.41</td>
      <td>0.318812</td>
      <td>0.5909</td>
      <td>0.0594059</td>
      <td>1.3100</td>
      <td>0.206931</td>
      <td>9.479119</td>
      <td>0.180198</td>
      <td>2.490689</td>
      <td>0.122772</td>
      <td>0.177624</td>
    </tr>
    <tr>
      <th>44</th>
      <td>CNC</td>
      <td>61.39</td>
      <td>325</td>
      <td>16.60</td>
      <td>0.324752</td>
      <td>1.4300</td>
      <td>0.187129</td>
      <td>0.3532</td>
      <td>0.019802</td>
      <td>12.819741</td>
      <td>0.352475</td>
      <td>0.540062</td>
      <td>0.00990099</td>
      <td>0.178812</td>
    </tr>
    <tr>
      <th>45</th>
      <td>PHM</td>
      <td>41.74</td>
      <td>479</td>
      <td>8.85</td>
      <td>0.176238</td>
      <td>1.7900</td>
      <td>0.256436</td>
      <td>1.0100</td>
      <td>0.129703</td>
      <td>6.945471</td>
      <td>0.0831683</td>
      <td>4.434636</td>
      <td>0.255446</td>
      <td>0.180198</td>
    </tr>
    <tr>
      <th>46</th>
      <td>FANG</td>
      <td>54.24</td>
      <td>368</td>
      <td>-3.10</td>
      <td>0.120792</td>
      <td>0.7939</td>
      <td>0.0732673</td>
      <td>2.5200</td>
      <td>0.416832</td>
      <td>5.989763</td>
      <td>0.0554455</td>
      <td>4.114673</td>
      <td>0.239604</td>
      <td>0.181188</td>
    </tr>
    <tr>
      <th>47</th>
      <td>TSN</td>
      <td>64.10</td>
      <td>312</td>
      <td>11.12</td>
      <td>0.20198</td>
      <td>1.5500</td>
      <td>0.20297</td>
      <td>0.5490</td>
      <td>0.0415842</td>
      <td>7.573876</td>
      <td>0.0990099</td>
      <td>5.921980</td>
      <td>0.364356</td>
      <td>0.18198</td>
    </tr>
    <tr>
      <th>48</th>
      <td>PBCT</td>
      <td>12.99</td>
      <td>1539</td>
      <td>11.30</td>
      <td>0.20396</td>
      <td>0.7348</td>
      <td>0.0633663</td>
      <td>2.4000</td>
      <td>0.39604</td>
      <td>8.476318</td>
      <td>0.136634</td>
      <td>2.543873</td>
      <td>0.128713</td>
      <td>0.185743</td>
    </tr>
    <tr>
      <th>49</th>
      <td>STT</td>
      <td>75.17</td>
      <td>N/A</td>
      <td>12.16</td>
      <td>0.217822</td>
      <td>1.1600</td>
      <td>0.130693</td>
      <td>2.1500</td>
      <td>0.366337</td>
      <td>8.065780</td>
      <td>0.120792</td>
      <td>2.061982</td>
      <td>0.10297</td>
      <td>0.187723</td>
    </tr>
  </tbody>
</table>
</div>



## Formatting Our Excel Output

We will be using the XlsxWriter library for Python to create nicely-formatted Excel files.

XlsxWriter is an excellent package and offers tons of customization. However, the tradeoff for this is that the library can seem very complicated to new users. Accordingly, this section will be fairly long because I want to do a good job of explaining how XlsxWriter works.


```python
writer = pd.ExcelWriter('value_strategy.xlsx', engine='xlsxwriter')
rv_dataframe.to_excel(writer, sheet_name='Value Strategy', index = False)
```

## Creating the Formats We'll Need For Our .xlsx File
You'll recall from our first project that formats include colors, fonts, and also symbols like % and $. We'll need four main formats for our Excel document:

* String format for tickers
* \$XX.XX format for stock prices
* \$XX,XXX format for market capitalization
* Integer format for the number of shares to purchase
* Float formats with 1 decimal for each valuation metric

Since we already built some formats in past sections of this course, I've included them below for you. Run this code cell before proceeding.


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

float_template = writer.book.add_format(
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
                    'D': ['Price-to-Earnings Ratio', float_template],
                    'E': ['PE Percentile', percent_template],
                    'F': ['Price-to-Book Ratio', float_template],
                    'G': ['PB Percentile',percent_template],
                    'H': ['Price-to-Sales Ratio', float_template],
                    'I': ['PS Percentile', percent_template],
                    'J': ['EV/EBITDA', float_template],
                    'K': ['EV/EBITDA Percentile', percent_template],
                    'L': ['EV/GP', float_template],
                    'M': ['EV/GP Percentile', percent_template],
                    'N': ['RV Score', percent_template]
                 }

for column in column_formats.keys():
    writer.sheets['Value Strategy'].set_column(f'{column}:{column}', 25, column_formats[column][1])
    writer.sheets['Value Strategy'].write(f'{column}1', column_formats[column][0], column_formats[column][1])
```

## Saving Our Excel Output


```python
writer.save()
```


{{< img src="/images/posts/configuration/value_strategy.png" >}}
