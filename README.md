
# API DOCUMENTATION FOR TOKENOMY EXCHANGE

## PUBLIC API
It doesn't need an API key to call Public API. You can call a simple GET request or open it directly from the browser.

Market Last Prices:  https://exchange.tokenomy.com/api/prices
All Pairs Summary:  https://exchange.tokenomy.com/api/summaries
Ticker TEN/BTC:  https://exchange.tokenomy.com/api/ten_btc/ticker
Trades TEN/BTC:  https://exchange.tokenomy.com/api/ten_btc/trades
Depth TEN/BTC:  https://exchange.tokenomy.com/api/ten_btc/depth

## PRIVATE API

To use Private API first you need to obtain your API credentials by logging into your Tokenomy.com account and open https://exchange.tokenomy.com/trade_api. These credentials contain "API Key" and "Secret Key". Please keep these credentials safe.

There are 3 different permissions that can be applied to API Key: view, trade and withdraw.  

| Permission  | Allowed Methods |
| ------------- | ------------- |
| view  | getInfo, transHistory, tradeHistory, openOrders, orderHistory, getOrder |
| trade | trade, cancelOrder  |
| withdraw | withdrawCoin  |

### Authentication
Authorization is done by sending data via HTTP header with the following variable:

Key — API key. Example: 14B0C9D6-UG71XOID-SH4IB5VQ-A9LK3YVZ-4HFR8X2T
Sign — POST data (?param=val\&param1=val1) encrypted with method HMAC-SHA512 using secret key

Send request to this URL: https://exchange.tokenomy.com/tapi

All requests must be sent with POST.

For each request you have to include these variables to make the call valid: *nonce* and *method*. 

*nonce* :  
    An increment integer. For example if the last request's nonce is 1000, the next request should be 1001 or a larger number. To learn more about nonce http://en.wikipedia.org/wiki/Cryptographic_nonce  

*method* :  
	Specify the method you want to call. 

To get the better picture on how to pass the authentication, you can check PHP function on the last page of this documentation.

### Responses

All responses are returned with JSON format. See example below.

Response format if a request is successfully executed:

```
{
	"success":1,
	"return":{
		/*here is some returned data*/
	}
}
```

Response format if a request returns an error:

```
{
	"success":0,
	"error":"some error message"
}
```

### API Methods
#### getInfo
To query user balances, deposit addresses and server's timestamp.

Parameter: none

Response example:  
```json
{
  "success": 1,
  "return": {
    "balance": {
      "btc": "0.14240799",
      "eth": "997.26227700",
      "ten": "4655.64648889",
      ... /* other coins */
    },
    "balance_hold": { /* pending order or pending withdraw */
      "btc": "19.85658069",
      "eth": "0.00000000",
      "ten": "13.00000000",
      ... /* other coins */
    },
    "address": {
      "btc": "1R2E8QhztN4zDAwSWU5SCdSvfAHpACrNT",
      "eth": "0x64cd8a8f84c90998398256f454e6e745ccc29b8f",
      "ten": "0x64cd8a8f84c90998398256f454e6e745ccc29b8f",
      ... /* other coins */
    },
    "user_id": "1",
    "name": "Jony Kesiangan",
    "email": "jony@gmail.com",
    "server_time": 1525018450
  }
}
```

#### transHistory
To get history of deposits and withdrawals of all assets.

Parameter: none

Response example:
```json
{
   "success":1,
   "return":{
      "withdraw":{
         "btc":[
            {
               "status":"success",
               "btc":"1.5",
               "fee":"0.0005",
               "amount":"1.4995",
               "submit_time":"1525018450",
               "success_time":"1525018950"
            }
         ],
         "ltc":[
         ],
         ... /* other coins */
      },
      "deposit":{
         "btc":[
            {
               "status":"success",
               "btc":"2",
               "amount":"2",
               "success_time":"1525018450"
            }
         ],
         "ltc":[
         ],
         ... /* other coins */
      }
   }
}
```

#### trade

To create trade orders

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| pair | Yes | market pair | ten_btc, eth_btc, ... | -  |
| type | Yes | buy or sell | buy / sell | - |
| price | Yes (limit order) | order price | numerical | - |
| btc | Yes (buy market order) | amount of btc to trade | numerical | - |
| ten, eth, etc.. | Yes (buy limit order, buy limit order, sell limit order) | amount of coin to trade | numerical | - |

Response example:

```json
{
  "is_error": false,
  "success": 1,
  "sold_ten": "0.00000000", /* coin that sold immediately (taker) */
  "receive_btc": "0.00000000", /* btc that bought immediately (taker) */
  "remain_ten": "1.00000000", /* coin that remain as limit order */
  "order_id": 168,
  "balance": {
    "btc": "0.14240799",
    "eth": "997.26227700",
    "ten": "4654.64648889",
    "frozen_btc": "19.85658069", /* pending order or pending withdraw */
    "frozen_eth": "0.00000000", /* pending order or pending withdraw */
    "frozen_ten": "14.00000000" /* pending order or pending withdraw */
  }
}
```

#### tradeHistory

To get history of executed trades.

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| count | No | number to display | numerical | 1000 |
| from_id | No | first ID | numerical | 0 |
| end_id | No | END ID | numerical | ∞ |
| order | No | sort order | asc / desc | desc |
| since | No | start time | | UNIX time |
| end | No | end time | | UNIX time |
| pair | Yes | market pair | ten_btc, eth_btc, ... | ten_btc |

Response example:
```json
{
  "success": 1,
  "return": {
    "trades": [
      {
        "trade_id": "142",
        "order_id": "167",
        "type": "sell",
        "ten": "3578.35108959",
        "price": "0.00004055",
        "trade_time": "1525451798"
      },
      {
        "trade_id": "1",
        "order_id": "11",
        "type": "buy",
        "ten": "3578.35108959",
        "price": "0.00003098",
        "trade_time": "1524658788"
      }
    ]
  }
}
```

#### openOrders

To get the list of open orders.

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| pair | No | market pair | ten_btc, eth_btc, ... | - |

Response example (if pair is set):
```json
{
  "success": 1,
  "return": {
    "orders": {
      "ten_btc": [
        {
          "order_id": "83",
          "submit_time": "1525013128",
          "price": "0.00003042",
          "type": "buy",
          "order_btc": "0.00003042", /* total order */
          "remain_btc": "0.00003042" /* amount remains pending */
        },
        {
          "order_id": "96",
          "submit_time": "1525013174",
          "price": "0.00003057",
          "type": "sell",
          "order_ten": "1.00000000", /* total order */
          "remain_ten": "0.50000000" /* amount remains pending */
        }
      ]
    }
  }
}
```

Response example (if pair is not set):
```json
{
  "success": 1,
  "return": {
    "orders": {
      "ten_btc": [
        {
          "order_id": "96",
          "submit_time": "1525013174",
          "price": "0.00003057",
          "type": "sell",
          "order_ten": "1.00000000", /* total order */
          "remain_ten": "0.50000000" /* amount remains pending */
        }
      ],
      "eth_btc": [
        {
          "order_id": "12345",
          "submit_time": "1525013174",
          "price": "0.08265",
          "type": "buy",
          "order_btc": "1.00000000",
          "remain_btc": "1.00000000"
        }
      ]
    }
  }
}
```
#### orderHistory

To get the list of closed orders.

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| pair | Yes | market pair | ten_btc, eth_btc, ... |
| count | no | | integer | 100 |
| from | no | | integer | 0 |

Response example:
```json
{
  "success": 1,
  "return": {
    "orders": [
      {
        "order_id": "166",
        "type": "sell",
        "price": "0.00003055",
        "submit_time": "1525451792",
        "finish_time": "1525451798",
        "method": "limit",
        "status": "filled",
        "order_ten": "3578.35108959",
        "remain_ten": "0.00000000"
      },
      {
        "order_id": "165",
        "type": "sell",
        "price": "0.00003055",
        "submit_time": "1525451782",
        "finish_time": "1525451789",
        "method": "limit",
        "status": "cancelled",
        "order_ten": "3578.35108959",
        "remain_ten": "3578.35108959"
      },
      {
        "order_id": "163",
        "type": "buy",
        "price": "0.00003045",
        "submit_time": "1525451475",
        "finish_time": "1525451791",
        "method": "limit",
        "status": "cancelled",
        "order_btc": "80.80753654",
        "remain_btc": "80.69855599"
      }
    ]
  }
}
```

#### getOrder

To get the details of a specific order.

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| pair | Yes | market pair | ten_btc, eth_btc, ... | - |
| order_id | Yes | Order ID | integer | - |

Response example:
```json
{
  "success": 1,
  "return": {
    "order": {
      "order_id": "166",
      "method": "limit",
      "type": "sell",
      "price": "0.00003055",
      "order_ten": "3578.35108959",
      "remain_ten": "0.00000000",
      "submit_time": "1525451792",
      "finish_time": "1525451798",
      "status": "filled"
    }
  }
}
```

#### cancelOrder

This method is for canceling an existing open order.

Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| pair | Yes | market pair | ten_btc, eth_btc, ... | - |
| order_id | Yes | Order ID | numerical | - |
| type | Yes | transaction type (buy or sell) | buy / sell | - |

Response example:
```json
{
  "success": 1,
  "return": {
    "order_id": "167",
    "type": "buy",
    "pair": "ten_btc",
    "balance": {
      "btc": "80.80756699",
      "eth": "997.26227700",
      "ten": "4655.64648889",
      "frozen_btc": "29.19142169",
      "frozen_eth": "0.00000000",
      "frozen_ten": "13.00000000"
    }
  }
}
```

#### withdrawCoin

This method is for withdrawing assets.

To be able to use this method you need to enable *withdraw* permission when you generate the API Key. Otherwise you will get “No permission” error.

You also need to prepare a *Callback URL*. *Callback URL* is a URL that our system will call to verify your withdrawal requests, you can consider it like a 2FA verification. Various parameters will be sent to the *Callback URL*. From your server side, check the parameters if the request is valid. If all data is correct, print out a string “ok”  (without quotes). If we don't receive “ok” (without quotes) response, we will reject the request. 

Callback call will be sent through a POST request, with 5 seconds connection timeout.

Request Parameter:

| Parameter  | Required? | Description | Value | Default |
| ------------- | ------------- |  ------------- |  ------------- |  ------------- |
| currency | Yes | Asset to withdraw | *ten, btc, eth, ...* | - |
| withdraw_address | Yes | Receiver address | a valid address | - |
| withdraw_amount | Yes | Amount to send | numerical | - |
| withdraw_memo | No | Memo to be sent to the receiver, if supported by the asset platform. Exchanges use this memo for accepting deposits for certain assets. Example: Destination Tag (for Ripple), Message (for NXT), Memo (for BitShares) | a valid memo/message/destination tag | - |
| request_id | Yes | Custom string you need to provide to identify each withdrawal request. request_id will be passed to callback call so your system can identify the request. | alphanumeric, max length 255 | - |

Response example:
```json
{
  "success": 1,
  "status": "approved",
  "withdraw_currency": "btc",
  "withdraw_address": "1R2E8QhztN4zDAwSWU5SCdSvfAHpACrNT",
  "withdraw_amount": "1.00000000",
  "fee": "0.00050000",
  "amount_after_fee": "0.99950000",
  "submit_time": "1509469200",
  "withdraw_id": "btc-12345", /* id string */
  "txid": "",
  "withdraw_memo": "" /* destination tag/message/memo */
}
```

Callback parameter:

| Parameter  | Description | 
| ------------- | ------------- |
| request_id | request_id from your request |
| withdraw_currency | currency from your request |
| withdraw_address | withdraw_address from your request |
| withdraw_amount | withdraw_amount from your request |
| withdraw_memo | withdraw memo from your request (if any) |
| requester_ip | ip address of the request |
| request_date | time the request submitted |

### PHP Function
Below is example of a PHP function to call Tokenomy Exchange private API. You can refer to the function to write code in different languages.
```php
<?php

define('TOKENOMY_KEY', ''); //define your api key
define('TOKENOMY_SECRET', ''); //define your api secret

function tokenomy_query($method, array $req = array()) {
        // API settings
        $key = TOKENOMY_KEY; // your API-key
        $secret = TOKENOMY_SECRET; // your Secret-key
 
        $req['method'] = $method;
        $req['nonce'] = time();
       
        // generate the POST data string
        $post_data = http_build_query($req, '', '&');
 
        $sign = hash_hmac('sha512', $post_data, $secret);
 
        // generate the extra headers
        $headers = array(
			'Sign: '.$sign,
			'Key: '.$key,
        );
 
        // our curl handle (initialize if required)
        static $ch = null;
        if (is_null($ch)) {
                $ch = curl_init();
                curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
                curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/4.0 (compatible; TOKENOMY PHP client; '.php_uname('s').'; PHP/'.phpversion().')');
        }
        curl_setopt($ch, CURLOPT_URL, 'https://exchange.tokenomy.com/tapi/');
        curl_setopt($ch, CURLOPT_POSTFIELDS, $post_data);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
  
        // run the query
        $res = curl_exec($ch);
        if ($res === false) throw new Exception('Could not get reply: '.curl_error($ch));
        $dec = json_decode($res, true);
        if (!$dec) throw new Exception('Invalid data received, please make sure connection is working and requested API exists: '.$res);
        
        curl_close($ch);
        $ch = null;
        return $dec;
}

$result = tokenomy_query('getInfo');

print_r($result); //print out the result
```

#### Troubleshooting

**I'm getting 403 Unauthorized or 403 Forbidden error.**  
Our firewall is often too aggressive and it creates false positive for genuine requests. Please send your IP addresses to our customer service so we can manually unblock your IPs.

**I'm getting “Too Many Requests” error.**  
To prevent abusive requests, we limit API call to 180 requests per minute.

**I'm getting "Invalid nonce" error after migration to production server.**  
If you use timestamp for your nonce, your production server time setting might doesn't match with your development server. To prevent this you can add 86400 to the nonce. To reset nonce, disable and create a new API Key.
