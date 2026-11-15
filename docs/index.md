# SafeSMM API v2 Reference

This is the request and response reference for the [SafeSMM](https://safesmm.net) SMM panel API v2. Developers, resellers and agencies use it to connect a website, Telegram bot, app or their own panel to SafeSMM services.

The official documentation page on our website is [safesmm.net/api/docs](https://safesmm.net/api/docs). A step by step developer guide is also available on [GitBook](https://safesmm.gitbook.io/safesmm).

## Basics

| Item | Value |
| --- | --- |
| Endpoint | `https://safesmm.net/api/v2` |
| HTTP method | POST (form fields) |
| Authentication | `key` field with your API key |
| Response | JSON |
| Actions | `services`, `add`, `status`, `refill`, `cancel`, `balance` |

To get an API key, [create a free account](https://safesmm.net/signup) and open the Account page. Keep the key private and never put it in front-end JavaScript.

Always send requests to the exact endpoint above, with https and without www. A redirect can drop the POST body, and the panel then answers with an API key error.

## balance

Returns the balance of your account.

Fields: `key`, `action=balance`

~~~bash
curl -X POST https://safesmm.net/api/v2 \
  -d "key=YOUR_API_KEY" \
  -d "action=balance"
~~~

~~~json
{
  "status": "success",
  "balance": "100.84292",
  "currency": "USD"
}
~~~

## services

Returns every service with its ID, name, category, rate per 1,000, minimum and maximum, and whether refill and drip-feed are supported.

Fields: `key`, `action=services`

~~~json
[
  {
    "service": 1,
    "name": "Instagram Followers",
    "category": "Instagram",
    "type": "default",
    "rate": "0.90",
    "min": "50",
    "max": "10000",
    "refill": true,
    "dripfeed": true
  }
]
~~~

Service IDs, rates and limits can change, so refresh this list once a day and check the quantity against `min` and `max` before you place an order.
## add

Places a new order and returns the order ID.

Fields: `key`, `action=add`, `service`, `link`, `quantity`

Optional drip-feed fields: `runs` (number of deliveries) and `interval` (minutes between deliveries). With drip-feed, `quantity` is the amount per run, not the total.

~~~json
{
  "status": "success",
  "order": 23501
}
~~~

Save the `order` value right away. It is the ID you need for status, refill and cancel.

## status

Returns the status of one order (`order`) or several orders (`orders`, IDs separated by commas).

~~~json
{
  "order": 23501,
  "status": "Completed",
  "charge": "0.0360",
  "start_count": "1250",
  "remains": "0",
  "currency": "USD"
}
~~~

Multiple orders in one request:

~~~json
{
  "23501": {
    "charge": "0.2750",
    "start_count": "3572",
    "status": "Partial",
    "remains": "157",
    "currency": "USD"
  },
  "23502": {
    "error": "Incorrect order ID"
  }
}
~~~

Status values you can receive include Pending, In progress, Completed, Partial and Canceled. Checking many orders in one request is faster than one request per order.

## refill

Requests a refill for an order, on services that support refill.

Fields: `key`, `action=refill`, `order`

~~~json
{
  "refill": "1"
}
~~~

## cancel

Requests cancellation, on services that allow it.

Fields: `key`, `action=cancel`, `order`

~~~json
{
  "cancel": "1"
}
~~~
## Errors

Errors are returned inside the JSON as an `error` field, often together with HTTP status 200. Check for `error` in every response before you treat a request as successful.

| Error | Usual cause |
| --- | --- |
| Invalid API key | Key copied with a space, an old key, or the request went to http or www |
| Incorrect order ID | Your own internal order number was sent instead of the panel order ID |
| Not enough funds | The balance does not cover the charge |
| Service or quantity error | Old service ID, or quantity outside min and max |

More causes and fixes: [SMM panel API errors guide](https://safesmm.net/blog/smm-panel-api-errors)

## Code examples

Python, using the official client from PyPI:

~~~python
# pip install safesmm
from safesmm import SafeSMM

api = SafeSMM("YOUR_API_KEY")
print(api.balance())

order = api.add_order(service=1, link="https://example.com/post", quantity=100)
print(api.status(order["order"]))
~~~

PHP with cURL:

~~~php
<?php
$ch = curl_init('https://safesmm.net/api/v2');
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query([
    'key'    => 'YOUR_API_KEY',
    'action' => 'balance',
]));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$result = json_decode(curl_exec($ch), true);
curl_close($ch);
print_r($result);
~~~

More code: [PyPI package](https://pypi.org/project/safesmm/) and [PHP and Python examples on GitHub](https://github.com/safesmmnet/safesmm-api-examples). Tutorials on our blog: [SMM panel API guide in PHP and Python](https://safesmm.net/blog/smm-panel-api-guide) and [connect the API to WordPress and WooCommerce](https://safesmm.net/blog/connect-smm-panel-api-wordpress-woocommerce).

## Balance and payments

The API uses the same balance as your SafeSMM account. Funds can be added with USDT (TRC20 or BEP20), Binance Pay, Cryptomus and Perfect Money, and crypto deposits are credited automatically. Details: [SMM panel with crypto payments](https://safesmm.net/smm-panel-crypto)

Resellers and agencies with high volume can see wholesale pricing and bulk discounts on the [SMM reseller panel page](https://safesmm.net/smm-reseller-panel).

## Support

Telegram: @SafeSMM_Admin. Email: safesmmnet@gmail.com. Website: [safesmm.net](https://safesmm.net)

This reference is maintained by the SafeSMM team.
