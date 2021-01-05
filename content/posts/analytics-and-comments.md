---
title: "Algorithmic-Trading-Python"
date: 2020-12-09T06:00:23+06:00
hero: /images/posts/writing-posts/analytics.svg
description: Adding analytics and disquss comment in hugo theme Toha
menu:
  sidebar:
    name: Algorithmic-Trading-Python
    identifier: Algorithmic-trading-python
    weight: 1
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
```python
stocks = pd.read_csv('sp_500_stocks.csv')
```

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

## Parsing Our API Call



```python
data['latestPrice']
data['marketCap']
```

## Adding Stocks Data to a Pandas DataFrame



```python
my_columns = ['Ticker', 'Price','Market Capitalization', 'Number Of Shares to Buy']
final_dataframe = pd.DataFrame(columns = my_columns)
final_dataframe
```


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

## Calculating the Number of Shares to Buy



```python
portfolio_size = input("Enter the value of your portfolio:")

try:
    val = float(portfolio_size)
except ValueError:
    print("That's not a number! \n Try again:")
    portfolio_size = input("Enter the value of your portfolio:")
```


```python
position_size = float(portfolio_size) / len(final_dataframe.index)
for i in range(0, len(final_dataframe['Ticker'])-1):
    final_dataframe.loc[i, 'Number Of Shares to Buy'] = math.floor(position_size / final_dataframe['Price'][i])
final_dataframe
```

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
