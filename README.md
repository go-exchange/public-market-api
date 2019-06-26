# Public Market Data API

The main goal of a Public Market Data API is to be able to provide key information about GO.Exchange in a transparent way.

The following API calls and information are available

**General**

[GET /exchange/info - Exchange information](https://github.com/go-exchange/public-market-api/blob/master/README.md#get-exchangeinfo---exchange-information)

[GET /exchange/symbols - Available markets](https://github.com/go-exchange/public-market-api/blob/master/README.md#get-exchangesymbols---available-markets)

[GET /exchange/trades - Historical executed trades](https://github.com/go-exchange/public-market-api/blob/master/README.md#get-exchangetradessymbol---historical-executed-trades)

[GET /exchange/orders/snapshot - Current Orderbook Snapshot](https://github.com/go-exchange/public-market-api/blob/master/README.md#get-exchangeorder-booksymbol---current-order-book-snapshot)


## General

- The base endpoint is: `https://api.go.exchange/`
- All endpoints return either a JSON Object or array
- All data is returned in ascending order, oldest first, newest last
- HTTP 4XX return codes are used for malformed requests; the issue is on the sender's side.
- HTTP 5XX return codes are used for internal errors; the issue is on our side. It is important to NOT treat this as a failure operation; the execution status is UNKNOWN and could have been a success.
- For GET endpoints, parameters must be sent as a query string.
- Parameters may be sent in any order.

### GET /exchange/info - Exchange information

The /info endpoint returns information about the exchange as a whole, and is used by aggregators to display information about our exchange to users.

#### Parameters

None


#### Response

JSON Object with the following:

- `name`: The name of our exchange
- `description`: A one paragraph description in plain text
- `logo`: A URL to our logo
- `website`: A URL to our exchange
- `twitter`: A URL to our twitter profile
- `capability`: An object describing the endpoints this API implements.
    - `markets`: boolean indicating markets endpoint is implemented
    - `trades`: boolean indicating trades endpoint is implemented
    - `tradesByTimestamp`: boolean indicating trades by timestamp endpoint is implemented
    - `tradesSocket`: boolean indicating trades socket endpoint is implemented
    - `orders`: boolean indicating orders endpoint is implemented
    - `ordersSocket`: boolean indicating orders socket endpoint is implemented
    - `ordersSnapshot`: boolean indicating orders snapshot endpoint is implemented
    - `candles`: boolean indicating candles endpoint is implemented

Example:

```json
{
    "name":"GO.Exchange",
    "description":"GO.Exchange",
    "logo":"https://go.exchange/asset/logo.png",
    "twitter":"https://twitter.com/GOExchangeHQ",
    "website":"https://go.exchange",
    "result":{
    "capability":{
        "candles":false,
        "markets":true,
        "orders":false,
        "orders_snapshot":true,
        "orders_socket":false,
        "trades":true,
        "trades_by_timestamp":false,
        "trades_socket":false
    }
}
```

### GET /exchange/symbols - Available markets

The /symbols endpoint returns a list of all available markets on your exchange and is used to query other endpoints on our API

#### Parameters

None

#### Response

JSON result array of objects with the following properties:
- `amount_decimal_places`: The amount decimal places supporting by the symbol
- `base_currency_code`: The base currency of the symbol
- `min_amount`: the minimum amount for a trade.
- `name`: symbol name
- `price_decimal_places`: The price decimal places supporting by the symbol
- `quote_currency_code`: The quote currency of the symbol
- `total_decimal_places`: The total decimal places which calculated from price multiply by amount.

Example:

```json
{
   "result":[
       {
         "amount_decimal_places":4,
         "base_currency_code":"BCH",
         "min_amount":"0.0001",
         "name":"BCHBTC",
         "price_decimal_places":6,
         "quote_currency_code":"BTC",
         "total_decimal_places":10
      },
      {
         "amount_decimal_places":5,
         "base_currency_code":"BCH",
         "min_amount":"0.00001",
         "name":"BCHUSDC",
         "price_decimal_places":2,
         "quote_currency_code":"USDC",
         "total_decimal_places":7
      },
      {
         "amount_decimal_places":6,
         "base_currency_code":"BTC",
         "min_amount":"0.000001",
         "name":"BTCUSDC",
         "price_decimal_places":2,
         "quote_currency_code":"USDC",
         "total_decimal_places":8
      },
      {
         "amount_decimal_places":4,
         "base_currency_code":"ETH",
         "min_amount":"0.0002",
         "name":"ETHBTC",
         "price_decimal_places":6,
         "quote_currency_code":"BTC",
         "total_decimal_places":10
      },
      {
         "amount_decimal_places":5,
         "base_currency_code":"ETH",
         "min_amount":"0.00002",
         "name":"ETHUSDC",
         "price_decimal_places":2,
         "quote_currency_code":"USDC",
         "total_decimal_places":7
      },
      {
         "amount_decimal_places":4,
         "base_currency_code":"LTC",
         "min_amount":"0.0005",
         "name":"LTCBTC",
         "price_decimal_places":6,
         "quote_currency_code":"BTC",
         "total_decimal_places":10
      },
      {
         "amount_decimal_places":5,
         "base_currency_code":"LTC",
         "min_amount":"0.00005",
         "name":"LTCUSDC",
         "price_decimal_places":2,
         "quote_currency_code":"USDC",
         "total_decimal_places":7
      },
      {
         "amount_decimal_places":2,
         "base_currency_code":"OMG",
         "min_amount":"0.01",
         "name":"OMGBTC",
         "price_decimal_places":6,
         "quote_currency_code":"BTC",
         "total_decimal_places":8
      },
      {
         "amount_decimal_places":2,
         "base_currency_code":"OMG",
         "min_amount":"0.01",
         "name":"OMGETH",
         "price_decimal_places":6,
         "quote_currency_code":"ETH",
         "total_decimal_places":8
      },
      {
         "amount_decimal_places":2,
         "base_currency_code":"OMG",
         "min_amount":"0.01",
         "name":"OMGUSDC",
         "price_decimal_places":4,
         "quote_currency_code":"USDC",
         "total_decimal_places":6
      }
   ]
}
```

### GET /exchange/trades/:symbol - Historical executed trades

The /trades endpoint returns executed trades historically for a given market (provided via parameters). It allows outside parties to ingest all trades from our exchange for all time.

#### Parameters

- `:symbol`: **(required)** A symbol name from the /symbols endpoint
- `after`: An ID of the next page from /trades response. If none is provided, the latest trades will be returned
- `before`: An ID of the previous page from /trades response. If none is provided, the latest trades will be returned
- `limit`: the limit number of the result in /trades response. If none is provided, the default value is 20 and the maximum is 50

#### Response

JSON array of trade result object for the given symbol after (and not including) the trade ID provided, with the following properties:

- `id`: A string ID for the trade that is unique within the scope of the market
- `timestamp`: Timestamp of the trade
- `price`: The price for one unit of the base currency expressed in the quote currency as a string that is parsable to a positive number.
- `amount`: The amount of the base currency that was traded as a string that is parsable to a positive number.
- `side`: The direction of the trade [buy, sell]

Example:

```json
{
   "metadata":{
      "after":"g2wAAAABYgAfZKBq",
      "before":null,
      "limit":5
   },
   "result":[
      {
         "amount":"1.57",
         "id":"trade_01DE6MDX0B6CZA3W5FXDHGKGJY",
         "price":"0.007374437",
         "side":"sell",
         "symbol":"OMGETH",
         "timestamp":"2019-06-25T06:16:05.911015"
      },
      {
         "amount":"1.42",
         "id":"trade_01DE6MBRJ4PXTMVH6807VEK8QS",
         "price":"0.007406381",
         "side":"buy",
         "symbol":"OMGETH",
         "timestamp":"2019-06-25T06:14:55.832003"
      },
      {
         "amount":"1.85",
         "id":"trade_01DE6M9P5CRVY165D8WD5KH5BQ",
         "price":"0.007419794",
         "side":"sell",
         "symbol":"OMGETH",
         "timestamp":"2019-06-25T06:13:47.835913"
      },
      {
         "amount":"1.12",
         "id":"trade_01DE6M8G0KAXP4KM1K5FKAJJPX",
         "price":"0.007432624",
         "side":"buy",
         "symbol":"OMGETH",
         "timestamp":"2019-06-25T06:13:08.767458"
      },
      {
         "amount":"0.54",
         "id":"trade_01DE6M78R0Q38SVM19C8ZCM783",
         "price":"0.007438031",
         "side":"buy",
         "symbol":"OMGETH",
         "timestamp":"2019-06-25T06:12:28.604776"
      }
   ]
}
```

### GET /exchange/order-book/:symbol - Current Order-book Snapshot

#### Parameters

- `:symbol`: **(required)** A symbol name from the /symbols endpoint

#### Response

JSON object of all buy and sell that are currently open for the provided symbol, with the following properties:
- `buy`: a list of all open buy orders
- `sell`: as list of all open sell orders
- `timestamp`: the timestamp this snapshot

Each order is an object with the following entries:
- `price`: The price for one unit of the base currency expressed in the quote currency as a JSON number
- `amount`: The amount of the base currency available at this price point as a JSON number
- `symbol`: symbol name
- `total`: the total calculated by the price multiply by the amount of the order

Example

```json
{
   "result":{
      "buy":[
         {        
            "amount":"35.54",
            "price":"2.2484",
            "symbol":"OMGUSDC",
            "total":"79.908136"
         },
         {

            "amount":"85.94",
            "price":"2.2464",
            "symbol":"OMGUSDC",
            "total":"193.055616"
         }
      ],
      "sell":[
         {
            "amount":"122.54",
            "price":"2.3733",
            "symbol":"OMGUSDC",
            "total":"290.824182"
         },
         {
            "amount":"72.91",
            "price":"2.3756",
            "symbol":"OMGUSDC",
            "total":"173.204996"
         }
      ]
    }
}
```
