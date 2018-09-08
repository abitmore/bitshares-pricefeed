# Price Feed Script for BitShares

## Installation 

Ubuntu 16.04 LTS

Start the installation

```
cd ~
pip3 install bitshares-pricefeed --user
```

Create config.yml
```
bitshares-pricefeed create
```

Add a feed producer name to the config.yml file just created
```
vim config.yml
# The producer name(s)
producer: your_witness_name
```

Enter Credentials

```
bitshares-pricefeed addkey
```

You will need to enter your cli wallet encryption passphrase. If you
don't have a pybitshares wallet, yet, one will be created:

```
Wallet Encryption Passphrase:
Repeat for confirmation:
```

You will need to enter your Private Key (Active key) here. Hit enter the second time it asks you.

```
Private Key (wif) [Enter to quit]:
```

Manually run the feed update

```
bitshares-pricefeed update
```

Create a place for a logfile

```
sudo touch /var/log/bitshare-pricefeed.log
sudo chown ubuntu /var/log/bitshare-pricefeed.log
```

Add to cron, where PASSWD is your Wallet Encryption Passphrase. This logic will send stdin and sterr to the logfile.

```
$ crontab -e

SHELL=/bin/bash
PATH=/home/ubuntu/bin:/home/ubuntu/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
UNLOCK="PASSWD"

0,15,30,45 * * * * bitshares-pricefeed --configfile /home/ubuntu/config.yml --skip-critical --no-confirm-warning update >> /var/log/bitshares-pricefeed.log 2>&1
```

Alternatively build and use the Docker image:

```
docker build -t bitshares-pricefeed .
docker run -v /path/to/config:/config -v /path/to/wallet:/root/.local/share/bitshares -e UNLOCK=PASS bitshares-pricefeed
```

## Help

```
# bitshares-pricefeed --help
Usage: bitshares-pricefeed [OPTIONS] COMMAND [ARGS]...

Options:
  --configfile TEXT
  --dry-run                       Only compute prices and print result, no
                                  publication.
  --confirm-warning / --no-confirm-warning
                                  Need for manual confirmation of warnings
  --skip-critical / --no-skip-critical
                                  Skip critical feeds
  --node <wss://host:port>        Node to connect to
  --help                          Show this message and exit.

Commands:
  addkey  Add a private key to the wallet
  create  Create config file
  update  Update price feed for assets
```

## Sources

The following data sources are currently available:

Name | Status | Assets type | API Key | Description
 --- | ---    | ---         | ---     |   ---
 AEX |  OK    |   Crypto    | No      | last and volume (in quote currency) from CEX ticker api with 15 sec delay 
AlphaVantage | OK | FIAT, Stocks, BTC | Yes | last from unknown source for currencies and from iex for stocks. volume only for stocks (in nb of shares).
Big.One | OK | Crypto | Yes | bid/ask average and volume (in quote currency) from bulk CEX ticker API.
Binance | OK | Crypto | No | last and volume (in quote currency) from CEX ticker api
BitcoinAverage | KO | Crypto | No | used API is deprecated not maintained anymore, need to be upgraded to ApiV2
Bitcoin Venezuela | OK | Crypto | No | ticker from api with 15 minutes delay, no volume
BitsharesFeed | OK | Crypto (MPA) | No | current feed price in Bitshares DEX, no volume.
Bitstamp | OK | Crypto | No | last and volume (in quote currency) from CEX ticker api
Bittrex | OK | Crypto | No | last and volume (in quote currency) from summary api (bulk)
Coincap | OK | ALTCAP & ALTCAP.X | No | use provided market cap, no volume
CoinEgg | OK | Crypto |No | last and volume (in quote currency) from CEX ticker api
Coinmarketcap | Warn | Crypto | No | volume weighted average of all prices reported at each market, volume in USD, 5 minutes delay (see https://coinmarketcap.com/faq/). V1 API will be closed December 4th, 2018. 
CoinmarketcapPro | OK | Crypto | Yes | volume weighted average of all prices reported at each market, volume in quote, 1 minutes delay. Use v2 api.
Currencylayer | OK | FIAT, BTC | Yes | ticker from api, only USD as base and hourly updated with free subscription, no volume info. From various source (https://currencylayer.com/faq)
CoinTiger | OK | Crypto | No | last and volume (in quote currency) from summary api (bulk)
Fixer | OK | FIAT | Yes |  Very similar to CurrencyLayer, ticker from api, daily from European Central Bank, only EUR with free subscription, no volume info.
Graphene | OK | Crypto, FIAT, Stocks | No | last and volume (in quote currency) from Bitshares DEX in realtime
Huobi | OK | Crypto | No | close price and volume (in quote currency) from CEX API in realtime
IEX  | OK | Stocks | No | last ("IEX real time price", "15 minute delayed price", "Close" or "Previous close") and volume. 
IndoDax | OK | Crypto | No | last and volume (in quote currency) from CEX ticker API.
LBank | OK | Crypto | No | last and volume (in quote currency) from CEX API in realtime
MagicWallet | OK | BITCNY/CNY | Yes | BITCNY/CNY ratio from deposti/withdraw on MagicWallet.
OkCoin  | OK | Crypto | No | last and volume (in quote currency) from CEX API in realtime
OpenExchangeRates | OK | FIAT, BTC | Yes | ticker from api, only USD as base and hourly updated with free subscription, no volume info. From unknown sources except Bitcoin wich is from CoinDesk (https://openexchangerates.org/faq#sources)
Poloniex | OK | Crypto | No | last and volume (in quote currency) from CEX API in realtime
Quantl | OK | Commodities | Yes | daily price from London Bullion Market Association (LBMA), no volume
RobinHood | OK | Stocks | No | last, no volume, from unknown source in real time
WorldCoinIndex | OK | Crypto | Yes| volume weighted price, sum of market volume.
ZB | OK | Crypto | No |last and volume (in quote currency) from CEX API in realtime

### Special sources:

#### Manual

A manual source is available to inject some manually set / constand data to the source feed.
This could be usefull for:
  -  testing to avoid connection to a third party datasource.
  -  inject a static rate like BitUSD/USD 

Example:
```
exchanges:
  manual:
    klass: Manual
    feed:
      USD:
        BTS:
          price: 42
          volume: 1
```

#### Composite

A composite source could be used to group multiple sources together in order to aggregate them using a specific fomula.

Aggregation types formulas could be:
  - `min`: select the minimum value for each pairs
  - `max`: select the maximum value for each pairs
  - `mean`: compute the mean price of all the pairs and sum the volume.
  - `weighted_mean`: compute the volume weighted mean price of all the pairs and sum the volume.
  - `median`: compute the median price of all the pairs and sum the volume.
  - `first_valid`: select the first pairs where the source match an ordered list of sources.

The main use cases is if you want to retrieve a pair like BTC/USD from multiples sources (worldcoinindex, bitcoinaverage, coinmarketcap), but you want to use only one of the value in all your computations. The same apply for FIAT exchange rates from (fixer, currencylayer, openexchangerate, ...).

See example configuration in `examples/composite.yaml`

#### Algorithm Based Assets

Some assets use a computed formula as price like HERO or HERTZ. 
Those are implemented with a source class that return the computed value, usually with USD as quote.

## Development

To run tests you need get API keys for the providers, and register them as environment variables:

```
export QUANDL_APIKEY=
export OPENEXCHANGERATE_APIKEY=
export FIXER_APIKEY=
export CURRENCYLAYER_APIKEY=
export ALPHAVANTAGE_APIKEY=
export WORLDCOININDEX_APIKEY= 
export MAGICWALLET_APIKEY=
export COINMARKETCAP_APIKEY=
```

To run all tests use:  `pytest`.

To run a specific test: `pytest -k bitcoinvenezuela`.

# IMPORTANT NOTE

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.
