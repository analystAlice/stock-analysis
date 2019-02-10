# Stock Analysis
Package for making elements of technical analysis of a stock easier. This package is meant to be a starting point for you to develop your own. As such, all the instructions for installing/setup will be assuming you will continue to develop on your end.

## Setup
```
# should install requirements.txt packages
pip install -e stock_analysis # path to top level where setup.py is

# if not, install them explicitly
pip install -r requirements.txt
```

## Usage
This section will show some of the functionality of each class; however, it is by no means exhaustive.

### Getting data
```
from stock_analysis import StockReader

reader = StockReader('2017-01-01', '2018-12-31')

# get bitcoin data
bitcoin = reader.get_bitcoin_data()

# get faang data
fb, aapl, amzn, nflx, goog = (
    reader.get_ticker_data(ticker) \
    for ticker in ['FB', 'AAPL', 'AMZN', 'NFLX', 'GOOG']
)

# get S&P 500 data
sp = reader.get_index_data()
```

### Grouping data
```
from stock_analysis.utils import group_stocks, describe_group

faang = group_stocks(
    {
        'Facebook' : fb,
        'Apple' : aapl,
        'Amazon' : amzn,
        'Netflix' : nflx,
        'Google' : goog
    }
)

# describe the group
describe_group(faang)
```

### Visualizing data
Be sure to check out the other methods here for different plot types, reference lines, shaded regions, and more!

#### Single asset
Evolution over time:
```
import matplotlib.pyplot as plt
from stock_analysis import StockVisualizer

netflix_viz = StockVisualizer(nflx)

ax = netflix_viz.evolution_over_time(
    'close',
    figsize=(10, 4),
    legend=False,
    title='Netflix closing price over time'
)
netflix_viz.add_reference_line(
    ax,
    x=nflx.high.idxmax(),
    color='k',
    linestyle=':',
    label=f'highest value ({nflx.high.idxmax():%b %d})',
    alpha=0.5
)
plt.show()
```

<img src="images/netflix_line_graph.png?raw=true" align="center" width="600" alt="line graph with reference line">

After hours trades:
```
netflix_viz.after_hours_trades()
plt.show()
```

<img src="images/netflix_after_hours_trades.png?raw=true" align="center" width="800" alt="after hours trades graph">

*Note: run `help()` on the `StockVisualizer` for more visualizations*

#### Asset groups
Correlation heatmap:
```
from stock_analysis import AssetGroupVisualizer

faang_viz = AssetGroupVisualizer(faang)
faang_viz.heatmap(True)
```

<img src="images/faang_heatmap.png?raw=true" align="center" width="450" alt="correlation heatmap">

*Note: run `help()` on the `AssetGroupVisualizer` for more visualizations. This object has many of the visualizations of the `StockVisualizer`.*

### Analyzing data
Below are a few of the metrics you can calculate.

#### Single asset
```
from stock_analysis import StockAnalyzer

nflx_analyzer = stock_analysis.StockAnalyzer(nflx)
nflx_analyzer.annualized_volatility()
```

#### Asset group
Methods of the `StockAnalyzer` can be accessed by name with the `AssetGroupAnalyzer`'s `analyze()` method.
```
from stock_analysis import AssetGroupAnalyzer

faang_analyzer = AssetGroupAnalyzer(faang)
faang_analyzer.analyze('annualized_volatility')

faang_analyzer.analyze('beta')
```

### Modeling
```
from stock_analysis import StockModeler
```

#### Time series decomposition
```
decomposition = StockModeler.decompose(nflx, 20)
fig = decomposition.plot()
plt.show()
```

<img src="images/nflx_ts_decomposition.png?raw=true" align="center" width="450" alt="time series decomposition">

#### ARIMA
Build the model:
```
arima_model = StockModeler.arima(nflx, 10, 1, 5)
```

Check the residuals:
```
StockModeler.plot_residuals(arima_model)
plt.show()
```

<img src="images/arima_residuals.png?raw=true" align="center" width="650" alt="ARIMA residuals">

Graph the predictions:
```
arima_ax = StockModeler.arima_predictions(
    arima_model, start=start, end=end,
    df=nflx, ax=axes[0], title='ARIMA'
)
plt.show()
```

<img src="images/arima_predictions.png?raw=true" align="center" width="450" alt="ARIMA predictions">

#### Linear regression
Build the model:
```
X, Y, lm = StockModeler.regression(nflx)
```

Check the residuals:
```
StockModeler.plot_residuals(lm)
plt.show()
```

<img src="images/lm_residuals.png?raw=true" align="center" width="650" alt="linear regression residuals">

Graph the predictions:
```
linear_reg = StockModeler.regression_predictions(
    lm, start=start, end=end,
    df=nflx, ax=axes[1], title='Linear Regression'
)
plt.show()
```

<img src="images/lm_predictions.png?raw=true" align="center" width="450" alt="linear regression predictions">
