---
title: "Equal Weight S&P 500"
date: 2020-12-09T06:00:23+06:00
hero: /images/posts/writing-posts/analytics.svg
menu:
  sidebar:
    name: Equal Weight S&P 500
    identifier: Equal-Weight S&P 500-algorithmic-trading-python
    parent: Algorithmic-Trading-Python
    weight: 110
---
# Equal-Weight S&P 500 Index Fund

#### Library Imports


```python
import numpy as np #The Numpy numerical computing library
import pandas as pd #The Pandas data science library
import requests #The requests library for HTTP requests in Python
import xlsxwriter #The XlsxWriter libarary for
import math #The Python math module
```

## Importing Our List of Stocks

The next thing we need to do is import the constituents of the S&P 500.

These constituents change over time, so in an ideal world you would connect directly to the index provider (Standard & Poor's) and pull their real-time constituents on a regular basis.

Paying for access to the index provider's API is outside of the scope of this course.

There's a static version of the S&P 500 constituents available here. [Click this link to download them now](http://nickmccullum.com/algorithmic-trading-python/sp_500_stocks.csv). Move this file into the `starter-files` folder so it can be accessed by other files in that directory.

Now it's time to import these stocks to our Jupyter Notebook file.


```python
stocks = pd.read_csv('sp_500_stocks.csv')
stocks
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 1 columns</p>
</div>



## Acquiring an API Token



```python
from secrets import IEX_CLOUD_API_TOKEN

```

## Making Our First API Call

Now it's time to structure our API calls to IEX cloud.

We need the following information from the API:

* Market capitalization for each stock
* Price of each stock




```python
symbol='AAPL'
api_url = f'https://sandbox.iexapis.com/stable/stock/{symbol}/quote?token={IEX_CLOUD_API_TOKEN}'
data = requests.get(api_url).json()
data
```




    {'symbol': 'AAPL',
     'companyName': 'Apple Inc',
     'primaryExchange': 'AETRABGOTSD)ECLKAQGEL N /(NASLMS ',
     'calculationPrice': 'close',
     'open': 132.82,
     'openTime': 1619212250569,
     'openSource': 'icaifofl',
     'close': 135.21,
     'closeTime': 1686759691314,
     'closeSource': 'coalfifi',
     'high': 133.43,
     'highTime': 1653195548544,
     'highSource': 'e dtep layin de5mre1ciu',
     'low': 134.63,
     'lowTime': 1679936896691,
     'lowSource': 'id lare de5einc1pu meyt',
     'latestPrice': 132.01,
     'latestSource': 'Close',
     'latestTime': 'January 5, 2021',
     'latestUpdate': 1653603064605,
     'latestVolume': 101132352,
     'iexRealtimePrice': 133.565,
     'iexRealtimeSize': 341,
     'iexLastUpdated': 1638679179531,
     'delayedPrice': 136.17,
     'delayedPriceTime': 1664004810632,
     'oddLotDelayedPrice': 134.46,
     'oddLotDelayedPriceTime': 1631059082434,
     'extendedPrice': 134.02,
     'extendedChange': -0.38,
     'extendedChangePercent': -0.00292,
     'extendedPriceTime': 1672184472704,
     'previousClose': 131.89,
     'previousVolume': 148412217,
     'change': 1.6,
     'changePercent': 0.01264,
     'volume': 102349580,
     'iexMarketPercent': 0.008461502558137022,
     'iexVolume': 842702,
     'avgTotalVolume': 114719753,
     'iexBidPrice': 0,
     'iexBidSize': 0,
     'iexAskPrice': 0,
     'iexAskSize': 0,
     'iexOpen': 133.816,
     'iexOpenTime': 1618089489908,
     'iexClose': 132.746,
     'iexCloseTime': 1657716086594,
     'marketCap': 2297206109374,
     'peRatio': 41.85,
     'week52High': 142.59,
     'week52Low': 57.49,
     'ytdChange': -0.000305520135732213,
     'lastTradeTime': 1652545062676,
     'isUSMarketOpen': False}



## Parsing Our API Call



```python
data['latestPrice']
data['marketCap']
```




    2297206109374



## Adding Stocks Data to a Pandas DataFrame



```python
my_columns = ['Ticker', 'Price','Market Capitalization', 'Number Of Shares to Buy']
final_dataframe = pd.DataFrame(columns = my_columns)
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
      <th>Market Capitalization</th>
      <th>Number Of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
final_dataframe = final_dataframe.append(
                                        pd.Series(['AAPL',
                                                   data['latestPrice'],
                                                   data['marketCap'],
                                                   'N/A'],
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
      <th>Market Capitalization</th>
      <th>Number Of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AAPL</td>
      <td>132.01</td>
      <td>2297206109374</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
</div>



## Looping Through The Tickers in Our List of Stocks

Using the same logic that we outlined above, we can pull data for all S&P 500 stocks and store their data in the DataFrame using a `for` loop.


```python
final_dataframe = pd.DataFrame(columns = my_columns)
for symbol in stocks['Ticker']:
    api_url = f'https://sandbox.iexapis.com/stable/stock/{symbol}/quote?token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(api_url).json()
    final_dataframe = final_dataframe.append(
                                        pd.Series([symbol,
                                                   data['latestPrice'],
                                                   data['marketCap'],
                                                   'N/A'],
                                                  index = my_columns),
                                        ignore_index = True)

```


```python
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
      <th>Market Capitalization</th>
      <th>Number Of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>123.28</td>
      <td>37252732674</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>15.55</td>
      <td>9444030484</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>161.62</td>
      <td>10790697543</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>132.44</td>
      <td>2281521989223</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>108.10</td>
      <td>192629273419</td>
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
      <td>106.14</td>
      <td>31987831072</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>162.44</td>
      <td>32929036367</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>393.44</td>
      <td>21275127218</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>45.16</td>
      <td>7195078845</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>166.06</td>
      <td>81283122681</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



## Using Batch API Calls to Improve Performance



```python
# Function sourced from
# https://stackoverflow.com/questions/312443/how-do-you-split-a-list-into-evenly-sized-chunks
def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]
```


```python
symbol_groups = list(chunks(stocks['Ticker'], 100))
symbol_strings = []
for i in range(0, len(symbol_groups)):
    symbol_strings.append(','.join(symbol_groups[i]))
#     print(symbol_strings[i])

final_dataframe = pd.DataFrame(columns = my_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        final_dataframe = final_dataframe.append(
                                        pd.Series([symbol,
                                                   data[symbol]['quote']['latestPrice'],
                                                   data[symbol]['quote']['marketCap'],
                                                   'N/A'],
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
      <th>Market Capitalization</th>
      <th>Number Of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>123.82</td>
      <td>37772880830</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.11</td>
      <td>9522009635</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>159.04</td>
      <td>10802577508</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>137.27</td>
      <td>2303401190430</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>110.10</td>
      <td>195620691288</td>
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
      <td>109.95</td>
      <td>32658071664</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>161.97</td>
      <td>33870074736</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>381.11</td>
      <td>21025179119</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>44.25</td>
      <td>7267656181</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>166.87</td>
      <td>81448084294</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



## Calculating the Number of Shares to Buy



```python
portfolio_size = input("Enter the value of your portfolio:")

try:
    val = float(portfolio_size)
except ValueError:
    print("That's not a number! \n Try again:")
    portfolio_size = input("Enter the value of your portfolio:")
```

    Enter the value of your portfolio:1000000



```python
position_size = float(portfolio_size) / len(final_dataframe.index)
for i in range(0, len(final_dataframe['Ticker'])-1):
    final_dataframe.loc[i, 'Number Of Shares to Buy'] = math.floor(position_size / final_dataframe['Price'][i])
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
      <th>Market Capitalization</th>
      <th>Number Of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>123.82</td>
      <td>37772880830</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>16.11</td>
      <td>9522009635</td>
      <td>122</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>159.04</td>
      <td>10802577508</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>137.27</td>
      <td>2303401190430</td>
      <td>14</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>110.10</td>
      <td>195620691288</td>
      <td>17</td>
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
      <td>109.95</td>
      <td>32658071664</td>
      <td>18</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>161.97</td>
      <td>33870074736</td>
      <td>12</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>381.11</td>
      <td>21025179119</td>
      <td>5</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>44.25</td>
      <td>7267656181</td>
      <td>44</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>166.87</td>
      <td>81448084294</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



### Initializing our XlsxWriter Object


```python
writer = pd.ExcelWriter('recommended_trades.xlsx', engine='xlsxwriter')
final_dataframe.to_excel(writer, sheet_name='Recommended Trades', index = False)
```

### Creating the Formats We'll Need For Our `.xlsx` File

Formats include colors, fonts, and also symbols like `%` and `$`. We'll need four main formats for our Excel document:
* String format for tickers
* \\$XX.XX format for stock prices
* \\$XX,XXX format for market capitalization
* Integer format for the number of shares to purchase


```python
background_color = '#0a0a23'
font_color = '#ffffff'

string_format = writer.book.add_format(
        {
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

dollar_format = writer.book.add_format(
        {
            'num_format':'$0.00',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

integer_format = writer.book.add_format(
        {
            'num_format':'0',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )
```

### Applying the Formats to the Columns of Our `.xlsx` File

We can use the `set_column` method applied to the `writer.sheets['Recommended Trades']` object to apply formats to specific columns of our spreadsheets.

Here's an example:

```python
writer.sheets['Recommended Trades'].set_column('B:B', #This tells the method to apply the format to column B
                     18, #This tells the method to apply a column width of 18 pixels
                     string_format #This applies the format 'string_format' to the column
                    )
```


```python
# writer.sheets['Recommended Trades'].write('A1', 'Ticker', string_format)
# writer.sheets['Recommended Trades'].write('B1', 'Price', string_format)
# writer.sheets['Recommended Trades'].write('C1', 'Market Capitalization', string_format)
# writer.sheets['Recommended Trades'].write('D1', 'Number Of Shares to Buy', string_format)
# writer.sheets['Recommended Trades'].set_column('A:A', 20, string_format)
# writer.sheets['Recommended Trades'].set_column('B:B', 20, dollar_format)
# writer.sheets['Recommended Trades'].set_column('C:C', 20, dollar_format)
# writer.sheets['Recommended Trades'].set_column('D:D', 20, integer_format)

```

This code works too.

Let's simplify this by putting it in 2 loops:


```python
column_formats = {
                    'A': ['Ticker', string_format],
                    'B': ['Price', dollar_format],
                    'C': ['Market Capitalization', dollar_format],
                    'D': ['Number of Shares to Buy', integer_format]
                    }

for column in column_formats.keys():
    writer.sheets['Recommended Trades'].set_column(f'{column}:{column}', 20, column_formats[column][1])
    writer.sheets['Recommended Trades'].write(f'{column}1', column_formats[column][0], string_format)
```

## Saving Our Excel Output

Saving our Excel file is very easy:


```python
writer.save()
```
{{< img src="/images/posts/configuration/Recomended_Trades.png" >}}
