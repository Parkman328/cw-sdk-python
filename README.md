# Cryptowatch Python SDK

The Cryptowatch Python library provides a convenient access to the [Cryptowatch API](https://docs.cryptowat.ch/home/) from applications written in the Python language.

It includes the following features:
 * Auto-serialization of API responses into Python objects
 * API credentials automatically read from your `~/.cw/credentials.yml` config file
 * Custom exceptions for API-specific issues (e.g.: Requests Allowance)
 * Smart back-off retries in case of API connectivity loss

## Example

Showing all Kraken markets that already gained at least 5% over the current weekly candle.

```python
import cryptowatch as cw
from datetime import datetime, timedelta

# Get all Kraken markets
kraken = cw.markets.list("kraken")

# For each Kraken market...
for market in kraken.markets:

    # Forge current market ticker, like KRAKEN:BTCUSD
    ticker = "{}:{}".format(market.exchange, market.pair).upper()
    # Request weekly candles for that market
    candles = cw.markets.get(ticker, ohlc=True, periods=["1w"])

    # Each candle is a list of [close_timestamp, open, high, low, close, volume, volume_quote]
    # Get close_timestamp, open and close from the most recent weekly candle
    close_ts, wkly_open, wkly_close = (
        candles.of_1w[-1][0],
        candles.of_1w[-1][1],
        candles.of_1w[-1][4],
    )

    # Compute market performance, skip if open was 0
    if wkly_open == 0:
        continue
    perf = (wkly_open - wkly_close) * 100 / wkly_open

    # If the market performance was 5% or more, print it
    if perf >= 5:
        open_ts = datetime.utcfromtimestamp(close_ts) - timedelta(days=7)
        print("{} gained {:.2f}% since {}".format(ticker, perf, open_ts))
```


## Installation
```
pip install cryptowatch-sdk
```

### Requirements

* python v3.7+
* requests v0.8.8+
* marshmallow v3.2.2+
* pyyaml v5.1.2+

## API Crendential

Using a credential file will allow you to authenticate your requests and grant you the API access of your account tier (Free, Basic or Pro).

### Setup your credential file

1. Generate an Cryptowatch API key from [your account](https://cryptowat.ch/account/api-access)
2. Create your credential file on your machine by running in order:

    2.1 `mkdir $HOME/.cw`

    2.2 `echo "apikey: 123" > $HOME/.cw/credentials.yml` (where `123` is your 20 digits **public key**)

3. Verify with `cat $HOME/.cw/credentials.yml` that you see something like below (`123` being your public key):

```
apikey: 123
```

The SDK will read your public key as soon as `import cryptowatch` is ran in your script.


## Usage

### REST API

```python
import cryptowatch as cw

# Assets
cw.assets.list()
cw.assets.get("BTC")

# Exchanges
cw.exchanges.list()
cw.exchanges.get("KRAKEN")

# Instruments
cw.instruments.list()
cw.instruments.get("BTCUSD")

# Markets
cw.markets.list() # Returns list of all markets on all exchanges
cw.markets.list("BINANCE") # Returns all markets on Binance

# Returns market summary (last, high, low, change, volume)
cw.markets.get("KRAKEN:BTCUSD")
# Return market candlestick info (open, high, low, close, volume) on some timeframes
cw.markets.get("KRAKEN:BTCUSD", ohlc=True, periods=["4h", "1h", "1d"])

# Returns market last trades
cw.markets.get("KRAKEN:BTCUSD", trades=True)

# Return market current orderbook
cw.markets.get("KRAKEN:BTCUSD", orderbook=True)
# Return market current orderbook liquidity
cw.markets.get("KRAKEN:BTCUSD", liquidity=True)
```

### Logging

Logging can be enabled through Python's `logging` module:

```python
import logging

logging.basicConfig()
logging.getLogger("cryptowatch").setLevel(logging.DEBUG)
```

### CLI

The library exposes a simple utility, named `cryptowatch`, to return last market prices.


#### By default it returns Kraken's BTCUSD market

```
> cryptowatch
7425.0
```

#### Add another Kraken market to return this market last price

```
> cryptowatch btceur
6758.1
```

#### You can also specify your own exchange

```
> cryptowatch binance:ethbtc
0.020359
```

When the market doesn't exist a return code of `1` will be set (`0` otherwise):

```
> cryptowatch binance:nosuchmarketusd
> echo $?
1
```



## Testing

Unit tests are under the [tests](https://github.com/cryptowatch/cw-sdk-python/tree/master/tests) folder and use `pytest`, run them all with:

```
make test
```

Integration tests sending real HTTP requests to the Cryptowatch API can be run with:

```
make test-http-real
```

## Development

Testing and developement dependencies are in the [requirements.txt](https://github.com/cryptowatch/cw-sdk-python/tree/master/requirements.txt) file, install them with:

```
pip install -r requirements.txt
```

The code base use the [Black](https://black.readthedocs.io/en/stable/) linter, run it with:

```
make lint
```

## License

[BSD-2-Clause](https://github.com/cryptowatch/cw-sdk-python/tree/master/LICENSE)
