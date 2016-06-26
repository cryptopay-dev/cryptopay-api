This document describes the API calls available as part of the Cryptopay Payment Gateway v1 API.

If your language of choice is Ruby we recommend using the [Cryptopay gem](https://github.com/cryptopay-dev/cryptopay-ruby) instead of writing your own client.

## Table of contents

- [**General API description**](#general-api-description)
<p></p>
  - [Authentication](#authentication)
  - [Base URL](#base-url)
  - [Formats and required HTTP request headers](#formats-and-required-https-request-headers)
  - [Rate-limiting](#rate-limiting)
  - [Pagination](#pagination)
    - [HTTP response header](#https-response-header)
    - [Controlling pagination](#controlling-pagination)
  - [Error handling](#error-handling)
  - [Successful calls](#successful-calls)
<p></p>
- [**API Calls**](#api-calls)
<p></p>
  - [Account](#account)
     - [Get Cryptopay rate](#get-cryptopay-rate)
     - [Get Cryptopay ASK rate](#get-cryptopay-ask-rate)
     - [Get balances (A)](#get-balances-a)
  - [Invoices](#invoices)
     - [Create an invoice (A)](#create-an-invoice-a)
     - [View an invoice (A)](#view-an-invoice-a)
     - [Requote an invoice (A)](#requote-an-invoice-a)
     - [List invoices (A,P)](#list-invoices-ap)
  - [Payment buttons](#payment-buttons)
     - [Create button (A)](#create-button-a)
     - [Buttons customizing](#buttons-customizing)
     - [Button events handle](#button-events-handling)
  - [Payment iframes](#payment-iframes)
     - [Create an iframe (A)](#create-an-iframe-a)
     - [iFrame customizing](#iframes-customizing)
     - [iFrame events handle](#iframes-events-handling)
  - [Hosted pages](#hosted-pages)
     - [Create hosted page (A)](#create-hosted-page-a)]
     - [Hosted page events handling](#hosted-page-events-handling)
  - [Cryptopay payment callbacks](#cryptopay-payment-callbacks)
     - [Callback validation](#callback-validation)

<p></p>
- [**Appendix**](#appendix)
   - [Codes and types tables](#codes-and-types-tables)
      - [Currencies](#currencies)
      - [Invoice statuses](#invoice-statuses)


# General API description

## Authentication

Calls that require authentication are marked with "A" in the call description title.

You can authenticate with Cryptopay by providing your API key in the HTTPS query. If your API key get compromised, you can easily reset it in [Settings - API](https://cryptopay.me/settings) tab.

All API queries must be made over HTTPS, and plain HTTP will be refused. You must include your API key in all requests.

A complete URL with API key authentication would look like this `https://cryptopay.me/api/v1/invoice?api_key=ff714265c70908645bab010c0f3bfb65`

## Base URL

The base URL for all calls is `https://cryptopay.me/api/v1/`. A complete URL would look like this `https://cryptopay.me/api/v1/invoice/3a7bc1b2-9b7e-4dc3-9ffc-b3c08962ff4d`.

## Formats and required HTTPS request headers

Cryptopay API will respond with JSON payload. It expects parameters to be passed in JSON with the correct `Content-Type: application/json` being set.

## Rate-limiting

API calls are rate-limited by IP to 6000 calls per hour.

## Pagination

Some API calls returning collections may be paginated. In this case the call description title mentions it.

Calls that return paginated data are marked with "P" in the call description title.

### HTTPS response header

Calls that return paginated collections will add a `Pagination` HTTP header to the response. It will contain a pagination meta-data JSON object.

**Pagination header example**
```javascript
    {

      // Total number of items in the collection
      "total":3,

      // Current page number
      "page":1,

      // Total amount of available pages
      "total_pages":1,

    }
```
### Controlling pagination

Optional pagination parameters may be passed in the request URI in order to control the way the collection gets paginated. If parameters are incorrect — HTTP 400 Bad request status will be returned along with an empty body.

| Parameter  | Default |	Acceptable values      |
|-----------|---------|--------------------------------|
| page      | 1       | Positive integer > 0            |
| per_page  | 20      | Positive integer >= 1  |
| from      | optional | Unix timestamp created_at > {from} |
| to        | optional | Unix timestamp created_at < {to} |



## Datetime formats

Datetime values will be returned as Unix timestamps, for example: `1392211599`

## Error handling

Whenever an error is encountered, Cryptopay response will include:

 * An HTTP 422 status (Unprocessable entity) or HTTP 400 (Bad request), HTTP 401 (Unauthorized)
 * A JSON array of localised error messages as body

## Successful calls

If API call was successful, the platform will respond with :

 * HTTP 200 status (OK) or HTTP 201 (Created),
 * JSON representation of the entity being created or updated if relevant


# API Calls

## Account

### Get Cryptopay rate

This call will return current Cryptopay exchange rate

**Request path :** `/api/v1/rates`

**Request method :** `GET`

**Request parameters**

**Response**

A JSON object with the following attributes is returned :

**Example request :** `GET /api/v1/rates`

**Example response :**

```javascript
{
  "USD": 581.95,
  "EUR": 421.58,
  "GBP": 339.7
}
```

### Get Cryptopay ASK rate

This call will return current Cryptopay ASK exchange rate

**Request path :** `/api/v1/ask_rates`

**Request method :** `GET`

**Request parameters**

**Response**

A JSON object with the following attributes is returned :

**Example request :** `GET /api/v1/ask_rates`

**Example response :**

```javascript
{
  "USD": 591.95,
  "EUR": 431.58,
  "GBP": 359.7
}
```

### Get balance (A)

This call will return all the balances of Cryptopay account .

**Request path :** `/api/v1/balance`

**Request method :** `GET`

**Request parameters**

| Name                | Type    | Description                                                                  |
|---------------------|---------|------------------------------------------------------------------------------|
| api_key             | String  | Your Cryptopay api key                                                       |

**Response**

A JSON object with the following attributes is returned :


**Example request :** `GET /api/v1/balance`

**Example response :**
```javascript
{
  "GBP":"0.0",
  "EUR":"21.23",
  "USD":"21.30"
  //"BTC" : "0.00" will be returned if Split setting was activated for the account
}
```


## Invoices

Invoices are requests for payment. They can be expressed either in BTC, GBP, EUR or USD. They all get a unique Bitcoin payment address assigned and a Bitcoin amount calculated from the requested currency amount.

Payment can be made by sending the `btc_price` amount of Bitcoins to the `btc_address` address. The invoice payment will trigger a `POST`to the `callback_url`.

### Create an invoice (A)

This call creates an invoice.

**Request path :** `/api/v1/invoices`

**Request method :** `POST`

**Request parameters**

| Name                  | Type    | Description                                                                  |
|-----------------------|---------|------------------------------------------------------------------------------|
| api_key               | String  | Your Cryptopay API key                                                       |
| price                 | Decimal | Requested amount to be credited upon payment                                 |
| currency              | String  | Currency in which the amount is expressed, default is EUR  _(optional)_      |
| description           | String  | Invoice description _(optional)_                                             |
| id                    | String  | Merchant assigned order ID _(optional)_                                      |
| name                  | String  | Merchant assigned name _(optional)_                                          |
| callback\_url         | String  | URL to which a callback should be sent when the invoice is paid _(optional)_ |
| success\_redirect\_url| String  | URL to redirect customer after payment completes  _(optional)_      |
| confirmations_count   | Decimal | Required number of confirmations _(optional)_ |
| callback_params       | Hash    | Additional parameters to include in callback _(optional)_ |
**Response**

An invoice JSON object is returned.

| Name                  | Type     | Description                                                     |
|-----------------------|----------|-----------------------------------------------------------------|
| uuid                  | UUID     | Invoice identifier                                              |
| description           | String   | Invoice description                                             |
| status                | String   | Invoice status (Invoice state _(see appendix)_                  |
| btc\_price            | Decimal  | Amount payable, expressed in BTC                                |
| btc\_address          | String   | Bitcoin payment address                                         |
| short\_id             | String   | Cryptopay's short id for invoice                                |
| callback_params       | String   | Additional parameters to include in callback                    |
| confirmation_count    | Decimal  | Required number of confirmations before callback is fired       |
| id                    | String   | Merchant assigned order ID                                      |
| name                  | String   | Merchant assigned name _(optional)_                             |
| price                 | Decimal  | Requested amount to be credited upon payment                    |
| currency              | String   | Currency in which the amount is expressed                       |
| created\_at           | Timestamp| Creation timestamp                                              |
| valid\_till           | Timestamp| Expiration timestamp                                            |
| url                   | String   | The URL at which this invoice is publicly visible and payable   |
| bitcoin_uri           | String   | The URI to create a QR-code with payment information            |
| callback\_url         | String   | URL to which a callback should be made when the invoice is paid |
| success\_redirect\_url| String   | URL to redirect customer after payment completes                |
| txid                  | Array    | An array with Bitcoin transaction identifiers                   |

**Example request :** `POST /api/v1/invoices`
```javascript
{
    "api_key":"0e3434768dc029f0695d3dd12basd6205f",
    "confirmations_count":"4",
    "id":"Order #123",
    "name":"An extra name",
    "price":"10.12",
    "currency":"GBP",
    "description":"Here goes a sample description",
    "callback_url":"http://example.com/cryptopay/callback",
    "success_redirect_url":"http://example.com/thankyou.html",
    "callback_params":[
    {
        "order_id":"Acme 123",
        "customer_name":"Dmitry"
    }
    ]
}
```
**Example response :**
```javascript
{
   "uuid":"88b47b20-6227-4161-b0e0-44e06bef2b67",
   "description":"Here goes a sample description",
   "status":"pending",
   "btc_price":"0.0299",
   "btc_address":"13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3",
   "short_id":"88B47B20",
   "callback_params":[
      {
         "order_id":"Acme 123",
         "customer_name":"Dmitry"
      }
   ],
   "confirmations_count":4,
   "id":"Order #123",
   "name":"An extra name",
   "price":10.12,
   "currency":"GBP",
   "created_at":1393072242,
   "valid_till":1393072842,
   "callback_url":"http://example.com/cryptopay/callback",
   "success_redirect_url":"http://example.com/thankyou.html",
   "url":"http://cryptopay.me/orders/88b47b20-6227-4161-b0e0-44e06bef2b67/d",
   "bitcoin_uri":"bitcoin:13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3?amount=0.0299&label=Order+%23123",
   "txid":[]
}
```

### View an invoice (A)

This call returns the particular invoice.

**Request path :** `/api/v1/invoices/#{uuid}`

**Request method :** `GET`

**Request parameters**

| Name 		| Type 	  | Description      	     |
|---------|---------|------------------------|
| api_key | String  | Your Cryptopay API key |
| uuid 		| UUID    | Invoice identifier     |


**Response**

A JSON object with the following parameters is returned.

| Name                  | Type     | Description                                                     |
|-----------------------|----------|-----------------------------------------------------------------|
| uuid                  | UUID     | Invoice identifier                                              |
| description           | String   | Invoice description                                             |
| status                | String   | Invoice status (Invoice state _(see appendix)_                  |
| btc\_price            | Decimal  | Amount payable, expressed in BTC                                |
| btc\_address          | String   | Bitcoin payment address                                         |
| short\_id             | String   | Cryptopay's short id for invoice                                |
| callback_params       | String   | Additional parameters to include in callback                    |
| confirmation_count    | Decimal  | Required number of confirmations before callback is fired       |
| id                    | String   | Merchant assigned order ID                                      |
| name                  | String   | Merchant assigned name _(optional)_                             |
| price                 | Decimal  | Requested amount to be credited upon payment                    |
| currency              | String   | Currency in which the amount is expressed                       |
| created\_at           | Timestamp| Creation timestamp                                              |
| valid\_till           | Timestamp| Expiration timestamp                                            |
| url                   | String   | The URL at which this invoice is publicly visible and payable   |
| bitcoin_uri           | String   | The URI to create a QR-code with payment information            |
| callback\_url         | String   | URL to which a callback should be made when the invoice is paid |
| success\_redirect\_url| String   | URL to redirect customer after payment completes                |
| txid                  | Array    | An array with Bitcoin transaction identifiers                   |

**Example request :** `GET /api/v1/invoices/70c7936b-f8ce-443a-8338-3762de0a1e92`

**Example response :**

```javascript
{
   "uuid":"88b47b20-6227-4161-b0e0-44e06bef2b67",
   "description":"Here goes a sample description",
   "status":"pending",
   "btc_price":"0.0299",
   "btc_address":"13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3",
   "short_id":"88B47B20",
   "callback_params":[
      {
         "order_id":"Acme 123",
         "customer_name":"Dmitry"
      }
   ],
   "confirmations_count":4,
   "id":"Order #123",
   "name":"An extra name",
   "price":10.12,
   "currency":"GBP",
   "created_at":1393072242,
   "valid_till":1393072842,
   "callback_url":"http://example.com/cryptopay/callback",
   "success_redirect_url":"http://example.com/thankyou.html",
   "url":"http://cryptopay.me/orders/88b47b20-6227-4161-b0e0-44e06bef2b67/d",
   "bitcoin_uri":"bitcoin:13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3?amount=0.0299&label=Order+%23123",
   "txid":[]
}
```


### Requote an invoice (A)

Cryptopay's invoice is valid for 10 minutes and will expire after that. This call is used to requote the invoice — create a new invoice with exactly the same parameters.


**Request path :** `/api/v1/invoices/#{uuid}`

**Request method :** `PUT`

**Request parameters**

| Name 		| Type 	  | Description      	     |
|---------|---------|------------------------|
| api_key | String  | Your Cryptopay API key |
| uuid 		| UUID    | Invoice identifier     |


**Response**

A JSON object of the new invoice with the following parameters is returned.

| Name                  | Type     | Description                                                     |
|-----------------------|----------|-----------------------------------------------------------------|
| uuid                  | UUID     | Invoice identifier                                              |
| description           | String   | Invoice description                                             |
| status                | String   | Invoice status (Invoice state _(see appendix(#appendix))_                  |
| btc\_price            | Decimal  | Amount payable, expressed in BTC                                |
| btc\_address          | String   | Bitcoin payment address                                         |
| short\_id             | String   | Cryptopay's short id for invoice                                |
| callback_params       | String   | Additional parameters to include in callback                    |
| confirmation_count    | Decimal  | Required number of confirmations before callback is fired       |
| id                    | String   | Merchant assigned order ID                                      |
| name                  | String   | Merchant assigned name _(optional)_                             |
| price                 | Decimal  | Requested amount to be credited upon payment                    |
| currency              | String   | Currency in which the amount is expressed                       |
| created\_at           | Timestamp| Creation timestamp                                              |
| valid\_till           | Timestamp| Expiration timestamp                                            |
| url                   | String   | The URL at which this invoice is publicly visible and payable   |
| bitcoin_uri           | String   | The URI to create a QR-code with payment information            |
| callback\_url         | String   | URL to which a callback should be made when the invoice is paid |
| success\_redirect\_url| String   | URL to redirect customer after payment completes                |
| txid                  | Array    | An array with Bitcoin transaction identifiers                   |

**Example request :** `PUT /api/v1/invoices/d2b18f9a-587e-4c8c-b6d2-f3b03e778c02`

**Example response :**

```javascript
{
   "uuid":"88b47b20-6227-4161-b0e0-44e06bef2b67",
   "description":"Here goes a sample description",
   "status":"pending",
   "btc_price":"0.0299",
   "btc_address":"13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3",
   "short_id":"88B47B20",
   "callback_params":[
      {
         "order_id":"Acme 123",
         "customer_name":"Dmitry"
      }
   ],
   "confirmations_count":4,
   "id":"Order #123",
   "name":"An extra name",
   "price":10.12,
   "currency":"GBP",
   "created_at":1393072242,
   "valid_till":1393072842,
   "callback_url":"http://example.com/cryptopay/callback",
   "success_redirect_url":"http://example.com/thankyou.html",
   "url":"http://cryptopay.me/orders/88b47b20-6227-4161-b0e0-44e06bef2b67/d",
   "bitcoin_uri":"bitcoin:13DuuapeP9t8HRU5jD6sdgQVMioUA6P7m3?amount=0.0299&label=Order+%23123",
   "txid":[]
}
```


### List invoices (A,P)

This call will return a paginated list of invoices for the client account.

**Request path :** `/api/v1/invoices`

**Request method :** `GET`

**Request parameters**

| Name 		| Type 	  | Description      	     |
|---------|---------|------------------------|
| api_key | String  | Your Cryptopay api key |

**Response**

A JSON array of invoice objects is returned.

**Example request :** `GET /api/v1/invoices`

**Example response :**
```javascript
    {
   "total":14,
   "page":5,
   "total_pages":5,
   "invoices":[
      {
         "uuid":"bb3a00dd-1a4f-4289-a813-36352b74bd4b",
         "description":null,
         "status":"timeout",
         "btc_price":"0.0297",
         "btc_address":"19gC6QqWjzTnUqW422wcLLb5ocQZEZyboR",
         "short_id":"BB3A00DD",
         "callback_params":null,
         "confirmations_count":1,
         "id":null,
         "price":10.0,
         "currency":"GBP",
         "created_at":1392992528,
         "valid_till":1392993128,
         "url":"http://cryptopay.me/orders/bb3a00dd-1a4f-4289-a813-36352b74bd4b/d",
         "bitcoin_uri":"bitcoin:19gC6QqWjzTnUqW422wcLLb5ocQZEZyboR?amount=0.0297",
         "callback_url":null,
         "success_redirect_url":null,
         "txid":[]
      },
      {
         "uuid":"102a688f-1d57-40de-ad4b-279705c95e12",
         "description":null,
         "status":"timeout",
         "btc_price":"0.0241",
         "btc_address":"13AMbh6dyt67PU38L2me9Nu4yggQPoxZ9J",
         "short_id":"102A688F",
         "callback_params":null,
         "confirmations_count":1,
         "id":null,
         "price":10.0,
         "currency":"EUR",
         "created_at":1392992515,
         "valid_till":1392993114,
         "url":"http://cryptopay.me/orders/102a688f-1d57-40de-ad4b-279705c95e12/d",
         "bitcoin_uri":"bitcoin:13AMbh6dyt67PU38L2me9Nu4yggQPoxZ9J?amount=0.0241",
         "callback_url":null,
         "success_redirect_url":null,
         "txid":[]
      }
   ]
}
```

## Payment buttons

Payment buttons can be used to accept bitcoin for an individual item or to integrate with your existing shopping cart solution. For example, you could create a new payment button for each shopping cart on your website, setting the total and order number in the button at checkout.

### Create button (A)

Resource that creates a payment button token to accept bitcoin on your website.

**Request path :** `/api/v1/buttons`

**Request method :** `POST`

**Request parameters**

| Name                | Type    | Description                                                                  |
|---------------------|---------|------------------------------------------------------------------------------|
| id                  | String  | Merchant assigned order ID                                                   |
| price               | Decimal | Requested amount to be credited upon payment                                 |
| currency            | String  | Currency in which the amount is expressed, default is EUR _(optional)_       |
| name                | String  | Order-related name _(optional)_                                              |

**Response**

A JSON object with the following parameters is returned.

**Example request :** `POST /api/v1/buttons`
```javascript  
{
  "id": "FR2348FD",
  "price": 10.3,
  "currency": "EUR",
  "name": "test"
}
```

**Example response :**
```javascript
{
  "token": "LyJ7ryzQpE6HupB7xx8R",
  "name": "test",
  "id": "FR2348FD",
  "price": 10.3,
  "currency": "EUR",
  "created_at": 1393013086,
  "callback_url": "http://yourawesomesite.com/cryptopay_handler"
}
```

Sample embed code

The button API will return a `token` parameter, which you can use to generate the embed HTML (described below).

```html
    <a href="#" class="cryptopayButton cpaGrey" data-cryptopay-token="7TWB2jAKgUntYGTjthkf"></a>
    <script src="https://cryptopay.me/assets/button.js"></script>
```

It's worth noting that generating multiple buttons this way is only necessary if you want to set prices dynamically. If you only have a few items with the same price you can generate buttons manually and modify the `data-cryptopay-token` attribute to change which value is returned in the callback.


### Buttons customizing

You can customise the button however you like. You need to make sure to include `<script src="https://cryptopay.me/assets/button.js"></script>` and that your element has `data-cryptopay-token` parameter with `token`.


### Button events handling
The best way to track payments is to use Cryptopay's [callback](#payment-callbacks), whick is fired when payment is confirmed by Bitcoin network.

If you would like to implement custom event tracking logic, you can track a `cryptopay-invoice` javascript event in `window` context.

```javascript
<script type="text/javascript">
window.addEventListener('cryptopay-invoice', function(event){
  console.log(event.detail);
}, false);
</script>
```
This can be used, for example, to redirect the user to a 'confirmation' page where you wait for the callback to reach your site. You can obtain invoice parameters and it's status from `event.detail` object.

It's important to note that the this event does not guarantee a payment has arrived successfully (any user could trigger this javascript event on the page). It is just a convenient way to move to the next step in your checkout. You should always verify the payment was received via the callback to your site

## Payment iframes

Payment ifames can be used to accept bitcoin for an individual item without customers being redirected to an external site. For example, you could create a new ifame for each shopping cart on your website, setting the total and order number in the iframe at checkout.

### Create an iframe (A)

Resource that creates a payment button token to accept bitcoin on your website.

**Request path :** `/api/v1/iframes`

**Request method :** `POST`

**Request parameters**

| Name                | Type    | Description                                                                  |
|---------------------|---------|------------------------------------------------------------------------------|
| id                  | String  | Merchant assigned order ID                                                   |
| price               | Decimal | Requested amount to be credited upon payment                                 |
| currency            | String  | Currency in which the amount is expressed, default is EUR _(optional)_       |
| name                | String  | Order-related name _(optional)_                                              |

**Response**

A JSON object with the following parameters is returned.

**Example request :** `POST /api/v1/iframes`
```javascript  
{
  "id": "FR2348FD",
  "price": 10.3,
  "currency": "EUR",
  "name": "test"
}
```

**Example response :**
```javascript
{
  "token": "LyJ7ryzQpE6HupB7xx8R",
  "name": "test",
  "id": "FR2348FD",
  "price": 10.3,
  "currency": "EUR",
  "created_at": 1393013086,
  "callback_url": "http://yourawesomesite.com/cryptopay_handler"
}
```

Sample embed code

The iframe API endpoint will return a `token` parameter, which you can use to generate the embed HTML (described below).

```html
      <div style="max-width: 605px">
      <iframe width="100%" height="300px" scrolling="no" frameborder="no" allowtransparency="true" src="https://cryptopay.me/iframes/TOKEN">
      </iframe></div>
```

### iFrames customizing

You can customise the iframe however you like. For example, you can add a <border>, <box-shadow> or any other styling.


### iFrames events handling
The best way to track payments is to use Cryptopay's [callback](#payment-callbacks), whick is fired when payment is confirmed by Bitcoin network.

If you would like to implement custom event tracking logic, you can track a `cryptopay-invoice` javascript event in `window` context.

```javascript
<script type="text/javascript">
window.addEventListener('cryptopay-invoice', function(event){
  console.log(event.detail);
}, false);
</script>
```
This can be used, for example, to redirect the user to a 'confirmation' page where you wait for the callback to reach your site. You can obtain invoice parameters and it's status from `event.detail` object.

It's important to note that the this event does not guarantee a payment has arrived successfully (any user could trigger this javascript event on the page). It is just a convenient way to move to the next step in your checkout. You should always verify the payment via the callback to your site


## Hosted page

Hosted pages can be used to accept bitcoin for an individual item or to integrate with your existing shopping cart solution. For example, you could create a new hosted page for each shopping cart on your website, setting the total and order number in the button at checkout. Your customer will be redirected to Cryptopay's page in order to complete payment.


### Create hosted page (A)

This call creates a hosted page token

**Request path :** `/api/v1/hosted`

**Request method :** `POST`

**Request parameters**

| Name                 | Type    | Description                                                                  |
|----------------------|---------|------------------------------------------------------------------------------|
| items                | Array   | List of items attributes (see description below)                             |
| currency             | String  | Currency in which the amount is expressed, default is EUR  _(optional)_       |
| id                   | String  | Merchant assigned order ID _(optional)_                                      |
| collect_email        | Boolean | Collect customer's email address during checkout _(optional)_   |
| collect_phone        | Boolean | Collect customer's phone number during checkout _(optional)_   |
| collect_name         | Boolean | Collect customer's full name during checkout  _(optional)_    |
| collect_address      | Boolean | Collect customer's address during checkout     _(optional)_   |
| form                 | Array   | Requested fields                            _(optional)_                    |
| change_quantity      | Boolean | ALlow customer to change quantity of items, default is true  _(optional)_   |
| callback\_url        | String  | URL to which a callback should be made when the invoice is paid _(optional)_  |
| success\_redirect\_url       | String  | URL to redirect customer after payment completes  _(optional)_       |


#### Hosted page item attribute
| Name                 | Type    | Description                                                                  |
|----------------------|---------|------------------------------------------------------------------------------|
| name                 | String  | Currency in which the amount is expressed                                    |
| price                | Decimal | Price                                                                        |
| quantity             | Integer | Set the quantity per checkout (if change_quantity set to true)               |
| description          | String  | Description of an item _(optional)_  |
| vat\_rate            | Decimal | Vat rate    _(optional)_                                               |


**Response**

An invoice JSON object is returned.

**Example request :** `POST /api/v1/hosted`
```javascript  
{
   "id":"Awesome invoice 120",
   "currency":"GBP",
   "collect_email":true,
   "collect_phone":true,
   "collect_name":true,
   "collect_address":true,
   "success_redirect_url":"http://example.com/success.html",
   "callback_url":"http://requestb.in/13lffoc1",
   "form":["Date of Birth:"],
   "items":[
      {
         "name":"Test item 1",
         "description":"Test item 1 description",
         "quantity":1,
         "vat_rate":13,
         "price":55
      },
      {
         "name":"Test item 2",
         "description":"Test item 2 description",
         "quantity":1,
         "vat_rate":13,
         "price":66
      }
   ]
}
```

**Example response :**
```javascript
{
   "token":"ys4UoFQfczySKY7VwS17",
   "success_redirect_url":"http://example.com/success.html",
   "callback_url":"http://requestb.in/13lffoc1",
   "form":[
      "Date of Birth:"
   ],
   "collect_email":true,
   "collect_name":true,
   "collect_address":true,
   "collect_phone":true,
   "change_quantity":true,
   "id":"Awesome invoice 120",
   "price":136.73,
   "currency":"GBP",
   "created_at":1393432965,
   "url":"http://cryptopay.me/hosted/ys4UoFQfczySKY7VwS17",
   "items":[
      {
         "name":"Test item 1",
         "description":"Test item 1 description",
         "quantity":1.0,
         "vat_rate":13.0,
         "price":55.0,
         "currency":"GBP"
      },
      {
         "name":"Test item 2",
         "description":"Test item 2 description",
         "quantity":1.0,
         "vat_rate":13.0,
         "price":66.0,
         "currency":"GBP"
      }
   ]
}
```


### Allow customers to change quanity of items at the hosted checkout.

Using hosted checkout facility you can invoice for several items at once. If you want your customers to be able to choose the quantity of items they would like to purchase, you need to pass `change_quantity` parameter with value `true`

### Collect user info

You might want to collect information from your customers during the checkout process, it might be email adderess, phone number, shipping address etc. We provide several preset for you convenience:

|     Name        |  Type    | Default | Description |
|-----------------|----------|---------|--------------------------------------------------|
| collect_email   | Boolean  |  True   | Collect customer's email address during checkout |
| collect_phone   | Boolean  |  True   | Collect customer's phone number during checkout  |
| collect_name    | Boolean  |  True   | Collect customer's full name during checkout     |
| collect_address | Boolean  |  True   | Collect customer's address during checkout       |


In order to collect custom information you can use `form` parameter, for example `form = ['Date of Birth:', 'Tax ID']`

### Hosted page events handling

The best way to track payments is to use Cryptopay's [callback](#payment-callbacks), whick is fired when payment is confirmed by Bitcoin network.

## Cryptopay payment callbacks

When Cryptopay receives the payment, we will perform a HTTPS POST to the URL given as `callback_url`. The Content-Type for the request will be `application/json`.

The parameters sent are the same as the ones returned by a [view invoice](#view-an-invoice-a) with an extra `validation_hash` parameter.

Cryptopay IPN is expecting to get a 200 OK response from you. If it doesn't get 200 OK — it will keep posting callbacks on the following schedule.

Cryptopay callback is scheduled  in 5 seconds + N ** 4, where N is the number of retries.

With the default of 25 attempts, the last retry will be 20 days later, with the last interval being almost 100 hours.

**Callback handling example, PHP**

```PHP  
// Retrieving the data
$json = file_get_contents('php://input');

// Parsing the Json..
$callback = json_decode($json, true);

// You can now fetch $callback['btc_price'] for example

// HTTP response
header('HTTP/1.1 200 OK');
```

### Callback validation

A `validation_hash` parameter is added to all callback requests. Its purpose is to authenticate the call from Cryptopay to the callback URL. This signature **must** be properly checked by the receiving server in order to ensure that the request is legitimate and hasn't been tampered with.

The signature is computed by concatenating the Merchant API Key with an invoice UUID, price in cents (multiply of callback `price` by 100) and currency ISO code and applying a SHA1 hash function to it. You must use the following pattern: "#{merchant.api\_key}\_#{uuid}\_#{price\_cents}#{currency}"

**Example signed callback request :**

In this example the client API key is `76b7c5d75bececcef0b44f01275d1357`.
```javascript
{
   "uuid":"248e5bb8-486c-457b-a2a3-59474baded6e",
   "price_currency":"GBP",
   "description":null,
   "status":"pending",
   "btc_price":"0.0343",
   "btc_address":"1RM2o9chCxS7j7sWbtB6RR8Raa1C66evs",
   "callback_params":null,
   "id":null,
   "name":null,
   "price":"10.0",
   "currency":"GBP",
   "created_at":1393334945,
   "valid_till":1393335545,
   "validation_hash":"715d7f713372e91765078d607416b69b1d6a8795",
   "callback_url":"http://requestb.in/13lffoc1",
   "success_redirect_url":"",
   "url":"http://cryptopay.me/orders/248e5bb8-486c-457b-a2a3-59474baded6e/d",
   "bitcoin_uri":"bitcoin:1RM2o9chCxS7j7sWbtB6RR8Raa1C66evs?amount=0.0343"
}
```

Signature will be computed as: `76b7c5d75bececcef0b44f01275d1357_248e5bb8-486c-457b-a2a3-59474baded6e_1000GBP`

SHA1 hash: `715d7f713372e91765078d607416b69b1d6a8795`

This signature can be easily by doing `Digest::SHA1.hexdigest(data)` (Ruby) or `hash('sha1', $data);` (PHP) where `data` is the `#{merchant.api_key}_#{uuid}_#{price_cents}#{price_currency}`




# Appendix

## Codes and types tables

### Currencies

The following currencies are available :

| Symbol | Currency       |
|--------|----------------|
| BTC    | Bitcoin        |
| EUR    | Euro           |
| GBP    | Pound Sterling |
| USD    | US Dollar      |


### Invoice statuses

| State          | Description                                           |
|----------------|-------------------------------------------------------|
| Pending        | Invoice has been created                              |
| Paid           | Payment received by Cryptopay, but is not confirmed by Bitcoin network yet                  |
| Partpaid       | Payment received is less than the amount requested    |
| Confirmed      | Invoice has been confirmed and credited into account |
| Timeout        | Invoice has expired                               |
