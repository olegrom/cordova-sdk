# BitPay SDK Cordova Plugin

# Getting Started

This is a plugin for Cordova/PhoneGap and provides several modules that can be included in your application. If you do not already have an account at BitPay, please go to https://test.bitpay.com/start to signup for development account, or for production at https://bitpay.com/start. After registration, depending on the kind of application you are building you may need to go through a pairing process for some of the API capabilities. 

Install the SDK plugin:

```
$ cordova plugin add https://github.com/bitpay/cordova-sdk.git

```

## Generate A Point-of-Sale Token to Distribute

**Use case:** Generating and tracking invoices in a distributed application.

Login to https://test.bitpay.com and navigate to *[My Account > API Tokens](https://test.bitpay.com/api-tokens)*. Click on the *Add New Token* and make a token with the *Point-of-Sale* capability for multiple clients. You can then include this token with your application.

## Invoices

We can easily create invoices, add behavior for payment events and open a bitcoin wallet in Android.

```javascript

document.addEventListener("deviceready", function(){

    var Bitpay = cordova.require('com.bitpay.sdk.cordova.Bitpay');

    var bitpay = new Bitpay({
        host: 'test.bitpay.com',
        port: 443,
        token: '70163c90f...' // as retrieved from above, point-of-sale capability
    });

    // Create Invoice
    bitpay.createInvoice({
        price: 314.15,
        currency: 'USD'
    }, function(error, invoice){
      if (error) throw error;

      invoice.on('payment', function(e){
        var invoice = e.detail;
        //do something on payment
      })

      // use this to open a bitcoin wallet
      invoice.openWallet();

      // generate a qrcode
      invoice.getQrCode({format: 'BIP72'}, function(elm){
        // do something with the qrcode elm
      });

    });

    // Get Invoice
    bitpay.getInvoice({
        id: 'RyNzmZEbGwACpmNg8X6jGN'
    }, function(error, invoice){
      if (error) throw error;

      // do something with the invoice

    });

}

```

For additional capabilities with invoices please refer to the [`Merchant.findInvoices`](#findinvoices) section below.

## Using the API Client

To make direct API calls, you can instantiate a BitPay client as follows:

```javascript
    var Client = cordova.require('com.bitpay.sdk.cordova.RPCClient');

    var pos = new Client({
        host: 'test.bitpay.com',
        port: 443,
        token: '70163c90f...' // any type of token, point-of-sale in the example below
    });

```

Now your app is ready to make API calls:

```javascript
    // Create Invoice
    pos.call('createInvoice', {
        price: 314.15,
        currency: 'USD'
    }, function(error, data){
      if (error) throw error;

      // log the output
      console.log(data);

    });
```

The above would output:

```javascript
{
  url: 'https://test.bitpay.com/invoice?id=RyNzmZEbGwACpmNg8X6jGN',
  status: 'new',
  btcPrice: '0.7437',
  btcDue: '0.7437',
  price: 314.15,
  currency: 'USD',
  exRates: { USD: 422.4228332222813 },
  invoiceTime: 1411602264553,
  expirationTime: 1411603164553,
  currentTime: 1411602264571,
  id: 'RyNzmZEbGwACpmNg8X6jGN',
  btcPaid: '0.0000',
  rate: 422.42,
  exceptionStatus: false,
  transactions: [],
  paymentUrls: {
    BIP21: 'bitcoin:mwA98U1rwyeVfJVd1CeTP4RxRYPPoZBeBm?amount=0.7437',
    BIP72: 'bitcoin:mwA98U1rwyeVfJVd1CeTP4RxRYPPoZBeBm?amount=0.7437&r=https://test.bitpay.com/i/RyNzmZEbGwACpmNg8X6jGN',
    BIP72b: 'bitcoin:?r=https://test.bitpay.com/i/RyNzmZEbGwACpmNg8X6jGN',
    BIP73: 'https://test.bitpay.com/i/RyNzmZEbGwACpmNg8X6jGN'
  },
  token: 'A1fV8rRzJnbdLb61stsEnEFdfWZShzfGnbsz2J3BdkJh2XaVdFxzfGaBQDdVS2sn9M'
}
```

**Note**: The `paymentUrls` are *temporary* and will change and not be available after 15 minutes when the invoice expires, or the invoice has been paid.

Using other Capabilities:

```javascript
    // To use public capabilities, do not pass a token
    var public = new BitpayRPCClient({
        host: 'test.bitpay.com',
        port: 443
    });

    // Track invoice state
    public.call('getInvoice', {
      id: 'RyNzmZEbGwACpmNg8X6jGN',
    }, function(error, data){
       if ( error ) throw error;
       // do something with the invoice response
    });

    // Get the current exchange rates
    public.call('getRates', null, function(error, data){
       if ( error ) throw error;
       // do something with the rates response
    });

```

## Sample Applications

- Music store app using the SDK [here](https://github.com/bitpay/sample-cordova-skd.git).

## Using the Command Line Tool

To explore more of the API, we've included a command line tool to make API calls. Any API call that you can make from the CLI you can make in your Cordova application. To get started, install the dependencies and navigate to the plugin bin directory. Configuration files will be stored in `.bitpay` directory of this plugin.

```
$ npm install

$ cd plugins/com.bitpay.sdk.cordova/bin

$ ./bitpay.js pair -S test -F merchant

```

The `-S` option is the name of the server, it can be `test` or `live`. The `-F` option is the name of the capability that you want to use. If you have not already configured a `Client ID` it will prompt you to save one. Once complete you should receive a response with a `pairingCode` that you can then approve at *My Account > API Tokens* and enter the `pairingCode`. Once completed you should be able to issue API calls.

Create an invoice:

```
$ ./bitpay.js call -S test -F merchant -M createInvoice -P '{"price": 100.00, "currency": "USD"}'
```

Create tokens for application distribution:

```
$ ./bitpay.js call -S test -F merchant -M createPublicPOSToken
```

## Available Modules

This plugin provides several modules that can be included in your application. 

### API Requests
There is a client, `lib/rpc-client.js`, that handles signing and determining the parameters of the request that will work with several request adapters. There are two adapters included, one that uses XHR, `lib/request-xhr.js`, for use in a browser, and another that uses the node https module, `request-node-https.js`. New adapters can be added so long as they respond and handle the same params and return the response. 
### API Configuration
There is a configuration module, `lib/config.js`, for saving Client IDs and API Tokens that will work with several storage adapters. Two have been included, one that uses Localstorage, `lib/json-localstorage.js` and another that uses the node fs module, `lib/json-node-filesystem.js`. New adapters can be written so lon as they handle the same params and return the same response.

### Invoices
There is a few modules included, `lib/invoice.js` and `lib/bitpay.js` that are cordova specific, and include features for creating, retrieving and tracking the state of an invoice. In addition to opening a wallet and generating QR codes.

# BitPay API Calls

## Public

To use the public capabilities do not include a token or identity *(sign the API call)*.

### createToken

Creates or claims an access token. If you supply an `id` and `facade` you will receive a `pairingCode` that you can either give to an account owner to approve yourself by authenticating at the *My Account -> API Tokens*. If you already have a `pairingCode`, you can pass both an `id` and `pairingCode` to link your Client ID to the token.

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
id            | Client ID                  | true
pairingCode   | string                     |
facade        | string *(merchant, pos)*   |
label         | string                     |

**Note**: The label is alphanumeric with spaces, underscores and dashes. A Client ID is based on the [bitcoin identity protocol](https://en.bitcoin.it/wiki/Identity_protocol_v1).

Example Params:

```javascript
{
  "id": "Tf4iyFq4hgEf3iVHkeihR9DKPVpqEWF5crd",
  "facade": "merchant"
}
```

Example Response:

```javascript
[
  {
    token: '4bdD1muDaxBydeYHSdNagEy1w44FJftrfftmJ5pLfcXU',
    facade: 'merchant',
    dateCreated: 1411506027126,
    pairingExpiration: 1411592427126,
    pairingCode: 'ww0MuhY',
    policies: [
      {
        policy: 'id',
        method: 'inactive',
        params: [ 'Tf4iyFq4hgEf3iVHkeihR9DKPVpqEWF5crd' ]
      }
    ]
 }
]
```

### getInvoice

Retrieves the specified invoice.

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
invoiceId     | BitPay Invoice ID (string) | true



Example Params:

```javascript

{
  "invoiceId": "MKBena5VPtX1SVwtirJYRa"
}

```

Example Response:

```javascript

{
  url: 'https://sample.com/invoice?id=MKBena5VPtX1SVwtirJYRa',
  status: 'new',
  btcPrice: '0.2260',
  btcDue: '0.2260',
  price: 100,
  currency: 'USD',
  exRates: {
    USD: 442.55566469557
  },
  invoiceTime: 1411506491781,
  expirationTime: 1411507391781,
  currentTime: 1411506524704,
  id: 'MKBena5VPtX1SVwtirJYRa',
  btcPaid: '0.0000',
  rate: 442.56,
  exceptionStatus: false,
  paymentUrls: {
    BIP21: 'bitcoin:n4qa4vHkFG9EK7kB1izSVk1kqK89oVC97o?amount=0.2260',
    BIP72: 'bitcoin:n4qa4vHkFG9EK7kB1izSVk1kqK89oVC97o?amount=0.2260&r=https://sample.com/i/MKBena5VPtX1SVwtirJYRa',
    BIP72b: 'bitcoin:?r=https://sample.com/i/MKBena5VPtX1SVwtirJYRa',
    BIP73: 'https://sample.com/i/MKBena5VPtX1SVwtirJYRa'
  },
  token: 'A8gT6iTyQFWa6tYTAccJfAEJMaJaNEscrrxNzx16yJaHTwNq2kVqSTYvWyngmUp61X'
}

```

**Note**: The `paymentUrls` are *temporary* and will change and not be available after 15 minutes when the invoice expires, or the invoice has been paid.

### getInvoiceBusToken

Will retrieve a token to listen using [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) to payment events for invoices. The token returned will represent a combination of *capabilities* (listening to an event) and *resource* (the invoice).

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
invoiceId     | BitPay Invoice ID (string) | true


Example Params:

```javascript

{
  "invoiceId": "MKBena5VPtX1SVwtirJYRa"
}

```

Example Response:

```javascript
{
  url: 'https://sample.com/events',
  token: '5qkeDrNot4VAUABq6BgdBe6nnq3PPmYiwVhC97eeEupUFao5Vze6NnNxqDXbktB8qy',
  events: [ 'payment' ],
  actions: [ 'subscribe', 'unsubscribe' ]
}
```

You can then use these values to subscribe to a payment event.

Example:

```javascript

var bus = new EventSource( "https://.../events?token=5qkeD..&action=subscribe&events[]=payment" );

bus.onerror = function(e){
  // do something on error
}

bus.addEventListener('connect', function(e){
  // do something on the connect event
});

bus.addEventListener('statechange', function(e){

  var invoice = JSON.parse(e.data);
  // do something with the invoice

});

```

### getRateForCurrency

Retrieves the exchange rate for the given currency.

Example Params:

```javascript
{
  "currency": "USD"
}

```

Example Response:

```javascript
{
  code: 'USD',
  name: 'US Dollar',
  rate: 437.02
}
```

### getRates

Retrieves the list of exchange rates.

Example Response:

```javascript

[
  {
    code: 'USD',
    name: 'US Dollar',
    rate: 437.29 },
  {
    code: 'EUR',
    name: 'Eurozone Euro',
    rate: 340.0488 },
    name: 'Zambian Kwacha',
    rate: 2705.8471 },
  {
    code: 'ZWL',
    name: 'Zimbabwean Dollar',
    rate: 140952.9499
  }
]

```

### getCurrencies

Retrieves the list of supported currencies.

Example Response:

```javascript

[
  {
    code: 'BTC',
    symbol: '฿',
    precision: 4,
    exchangePctFee: 200,
    payoutEnabled: true,
    name: 'Bitcoin',
    plural: 'Bitcoin',
    alts: 'btc',
    payoutFields: [ 'bitcoinAddress' ]
  },
  { code: 'ZWL',
    symbol: 'Z$',
    precision: 2,
    exchangePctFee: 200,
    payoutEnabled: false,
    name: 'Zimbabwean Dollar',
    plural: 'Zimbabwean Dollar',
    alts: '',
    payoutFields: []
  }
]

```

### createAccount

Creates an application for a new merchant account.

Parameters:

Name          | Type          | Required
------------- | ------------- | -------------
users         | array         | true
orgs          | array         | true

Example Params:

```javascript
{
  "users": [{
    "email": "satoshi@sample.com",
    "firstName": "Satoshi",
    "lastName": "Nakamoto",
    "phone": "4041235678",
    "agreedToTOSandPP": "true" // Do you agree to the Terms of Service
  }],
  "orgs": [{
    "name": "Satoshi Widgets",
    "address1": "123 Peachtree St",
    "address2": "Suite 200",
    "city": "Atlanta",
    "state": "GA",
    "zip": "30303",
    "country": "USA",
    "isNonProfit": false, // optional
    "usTaxId": "taxidhere", // optional
    "cartPos": "WooCommerce" //optional
    "affiliateOid": "affiliateoidhere" // optional
    "industry": "Accounting",
  }]
}
```

Example Response:

```javascript
{
  verificationCode: '0.1406255280598998',
  requiredTier: 0,
  accountName: 'Satoshi Widgets',
  contactEmail: 'satoshi@sample.com',
  contactFirstName: 'Satoshi',
  contactLastName: 'Nakamoto',
  contactPhone: '4041235678',
  profileAddress1: '123 Peachtree St',
  profileAddress2: 'Suite 200',
  profileCity: 'Atlanta',
  profileState: 'GA',
  profilePostalCode: '30303',
  profileCountry: 'USA',
  profileIndustry: 'Accounting',
  profileAgreedToTOSandPP: true,
  updated: '2014-09-23T20:31:09.759Z',
  requiresReview: false,
  activated: false,
  verified: false
}
```

## Point-of-Sale

To use the point-of-sale capabilities, you'll need to generate a token. These tokens can either have individual client restrictions, so that only allowed clients can use these capabilities. In the use case that you're developing a mobile application, you can generate a token with these capabilities that can be used by many clients and can be included with your application. More information below at `Merchant.createPublicPOSToken`.

### createInvoice

Will create an invoice

Parameters:

Name          | Type           | Required
------------- | -------------  | -------------
price         | number         | true
currency      | currency code  | true

Example Params:

```javascript
{
  "price": 100.00,
  "currency": "USD"
}
```

Example Response:

```javascript
{
  url: 'https://sample.com/invoice?id=MKBena5VPtX1SVwtirJYRa',
  status: 'new',
  btcPrice: '0.2260',
  btcDue: '0.2260',
  price: 100,
  currency: 'USD',
  exRates: { USD: 442.55566469557 },
  invoiceTime: 1411506491781,
  expirationTime: 1411507391781,
  currentTime: 1411506491798,
  id: 'MKBena5VPtX1SVwtirJYRa',
  btcPaid: '0.0000',
  rate: 442.56,
  exceptionStatus: false,
  transactions: [],
  paymentUrls: {
    BIP21: 'bitcoin:n4qa4vHkFG9EK7kB1izSVk1kqK89oVC97o?amount=0.2260',
    BIP72: 'bitcoin:n4qa4vHkFG9EK7kB1izSVk1kqK89oVC97o?amount=0.2260&r=https://sample.com/i/MKBena5VPtX1SVwtirJYRa',
    BIP72b: 'bitcoin:?r=https://sample.com/i/MKBena5VPtX1SVwtirJYRa',
    BIP73: 'https://sample.com/i/MKBena5VPtX1SVwtirJYRa'
  },
  token: '8Esi7g1utRuS8USZqrATQechZX9jkmHinQM4rpD76izieEJ8n3AsBczz5EJ24jiiRp'
}
```

**Note**: The `paymentUrls` are *temporary* and will change and not be available after 15 minutes when the invoice expires, or the invoice has been paid.


### getInvoiceSettings

This will return the merchant information and settings for creating invoices.

Example Response:

```javascript
{
  account: '5421f0b9edfb002433004520',
  additionalCurrencies: [ 'USD','EUR','GBP','JPY' ],
  defaultCurrency: 'USD',
  notificationEmail: 'satoshi@sample.com',
  notificationURL: 'https://sample.com/ipn',
  merchantName: 'Satoshis Widgets'
}
```

## Merchant

To use the merchant capabilities on a merchant resource, you'll need to generate a token. With public capabilities you can create a token and specify `merchant` as the `facade`, and then use the return `pairingCode` to add the token to your account at *My Account -> API Tokens*, or give this `pairingCode` to an merchant organization administrator to add and approve.

### findInvoices

Retrieves invoices for the calling merchant based on the query.

Parameters:

Name          | Type           | Required
------------- | -------------  | -------------
dateStart     | date           | true
dateEnd       | date           |
limit         | number         |
skip          | number         |
itemCode      | string         |
orderId       | string         |

Example Params:

```javascript

{
  "dateStart": "2014-8-1",
  "dateEnd": "2014-8-31"
}


```

Example Response:

```javascript
[
  {
    url: 'https://sample.com/invoice?id=54cbXB29FoNY48hbctAvKy',
    status: 'new',
    btcPrice: '0.2272',
    btcDue: '0.2272',
    price: 100,
    currency: 'USD',
    exRates: { USD: 440.12257441601656 },
    invoiceTime: 1411511759973,
    expirationTime: 1411512659973,
    currentTime: 1411511766879,
    id: '54cbXB29FoNY48hbctAvKy',
    btcPaid: '0.0000',
    rate: 440.12,
    exceptionStatus: false,
    transactions: [],
    paymentUrls: {
      BIP21: 'bitcoin:mndc6N5QPCMUgW378D9Nj5wZEwVQ82HGcd?amount=0.2272',
      BIP72: 'bitcoin:mndc6N5QPCMUgW378D9Nj5wZEwVQ82HGcd?amount=0.2272&r=https://sample.com/i/54cbXB29FoNY48hbctAvKy',
      BIP72b: 'bitcoin:?r=https://sample.com/i/54cbXB29FoNY48hbctAvKy',
      BIP73: 'https://sample.com/i/54cbXB29FoNY48hbctAvKy'
    },
    token: '8Esi7g1utRuS8USZqrATQedEA3nhPhuniWJXBUwMbq546fWrUAkCtTEah2pPB6jeW2'
  }
]
```

**Note**: The `paymentUrls` are *temporary* and will change and not be available after 15 minutes when the invoice expires, or the invoice has been paid.

When there is activity on the invoice the status will change, here is the meaning of each status.

Status          | Description
-------------   | -------------
new             | The invoice has not yet been fully paid
paid            | Received payment however has not yet been fully confirmed
complete        | Payment confirmed based on the `transaction` speed settings
expired         | Can no longer receive payments
invalid         | The invoice has received payment, however was invalid

When a payment is received, transactions will become available. An invoice can receive a partial payment and an over payment, and in these situations an `exceptionStatus` will be available with `paidPartial` and `paidOver`. It's also possible to accept an over or under payment via the API, and we will go into that below.

```javascript

    exceptionStatus: 'paidPartial',
    transactions:
     [
       {
         amount: 110000,
         confirmations: 0,
         time: '1970-01-01T00:00:00.000Z',
         receivedTime: '2014-09-23T22:51:20.106Z'
       }
     ],

```

### getInvoice

Retrieves a single invoice with additional transaction information.

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
invoiceId     | BitPay Invoice ID (string) | true


Example Params:

```javascript

{
  "invoiceId": "MKBena5VPtX1SVwtirJYRa"
}

```

Example Response:

```javascript
{
  url: 'https://sample.com/invoice?id=YEh2jnoZUAbYMW2XtE44VD',
  status: 'confirmed',
  btcPrice: '0.0023',
  btcDue: '-0.0007',
  price: 1,
  currency: 'USD',
  exRates: { USD: 435.72 },
  invoiceTime: 1411513113810,
  expirationTime: 1411514013810,
  currentTime: 1411515011556,
  id: 'YEh2jnoZUAbYMW2XtE44VD',
  btcPaid: '0.0030',
  rate: 435.72,
  exceptionStatus: 'paidOver',
  transactions: [
    {
      amount: 300000,
      confirmations: 1,
      time: '1970-01-01T00:00:00.000Z',
      receivedTime: '2014-09-23T22:59:22.443Z'
    }
  ],
  token: '8Esi7g1utRuS8USZqrATQeWGCLPqyLpcexbehA7DwDckiiZjj36LdJtSdmXvPZJTUW'
}
```

**Note**: The token returned from this response includes `merchant` capabilities on the `invoice` resource, including being able to make a refund, this is described in further detail below.

### getLedgers

Will return the current balance for each ledger by currency.

Example Response:

```javascript

[
  {
    currency: 'BTC',
    balance: 0.0076
  },
  {
    currency: 'USD',
    balance: 0
  }
]

```

### getLedgerEntries

Will return entries for a given ledger.

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
currency      | currency code (string)     | true
startDate     | date                       | true
endDate       | date                       | true

Example Params:

```javascript
{
  "currency": "BTC",
  "startDate": "2014-9-1",
  "endDate": "2014-9-30"
}
```

Example Response:

```javascript
[
  {
    code: 1000,
    amount: 230000,
    timestamp: '2014-09-24T00:30:18.737Z',
    scale: 100000000,
    txType: 'sale',
    exRates: { USD: 440.52000000000004 },
    buyerFields: {},
    invoiceId: 'Xdmsdo67EUT9M9XVtEL2A3',
    sourceType: 'invoice',
    customerData: { customData: [] },
    invoiceAmount: 1,
    invoiceCurrency: 'USD'
  },
  {
    description: 'overpayment credit',
    code: 1003,
    timestamp: '2014-09-24T00:53:25.594Z',
    amount: 70000,
    notes: 'BTC credit for overpayment of invoice YEh2jnoZUAbYMW2XtE44VD',
    scale: 100000000,
    txType: 'ACH/other',
    exRates: { USD: 435.72 },
    buyerFields: {},
    invoiceId: 'YEh2jnoZUAbYMW2XtE44VD',
    sourceType: 'invoice',
    customerData: { customData: [] },
    invoiceAmount: 1,
    invoiceCurrency: 'USD'
  }
]

```

### createToken

Will approve a token for the callers merchant resource. A `pairingCode` can be retrieved using `public` capabilities, as described above, and can then be added and approved to the callers merchant resource with this call. A `facade` and `pairingCode` are both required, the `facade` method should match the requesters facade as verification of capabilities.

Parameters:

Name          | Type                       | Required
------------- | -------------------------- | -------------
pairingCode   | string                     | true
facade        | facade *(pos, merchant)*   | true


```javascript
{
  "pairingCode": "X8tj3c2",
  "facade": "merchant"
}
```

Example Response:

```javascript
[
  {
    resource: '51rjLjdhZotGsH76hZpdg5j8KAvsWLWXEaww8ynUa7zh',
    token: 'U4Zeh3tKhUwLguxK8r6TAg',
    facade: 'merchant',
    dateCreated: 1411520974852,
    policies: [{
         policy: 'id',
         method: 'require',
         params: ['TfDzkkFs7vTe8yHyb28NDZ56Jt2VBAF3ysd']
    ]}
   }
]
```

### createPublicPOSToken

This will create a token with point-of-sale capabilities *(the ability to create invoices)*. It will *not* be restricted to a specific Client ID, and can be distributed with mobile applications. The token will appear at *My Account -> API Tokens* and can be further managed.

Example Response:

```javascript
[
  {
    policies: [],
    resource: '51rjLjdhZotGsH76hZpdg5j8KAvsWLWXEaww8ynUa7zh',
    token: '5VZPyPQRczUc8HP3EF9Q5jSwNJtVu4nDvEg5u8iqj8eN',
    facade: 'pos',
    dateCreated: 1411519737345
  }
]
```

### Undocumented
- createBill
- getBills
- getBillById
- getInvoiceBusToken *(see public)*
- getInvoiceSettings *(see point-of-sale)*
- createInvoice *(see point-of-sale)*

## Merchant/Invoice

To use the `merchant` capabilities on an `invoice` resource, you'll need to get a token for the `getInvoice` call, as documented above in the Merchant section. When using this token, the following capabilities will be available for acting upon an invoice.

### refund

Will refund the invoice to any bitcoin address. The invoice will need to have *six confirmations* in the blockchain before a refund can be requested. It's not possible to do a refund while an invoice is partially or overpaid state, see below for more information.

Parameters:

Name               | Type                       | Required
------------------ | -------------------------- | -------------
bitcoinAddress     | bitcoin address (string)   | true
amount             | number                     | true
currency           | currency code (string)     | true

Example Params:

```javascript

{
  "bitcoinAddress": "mtX8nPZZdJ8d3QNLRJ1oJTiEi26Sj6LQXS",
  "amount": 100.00,
  "currency": "USD"
}

```

Example Response:

```javascript

{
  id: 'H9EE8zkSTL5XRCY8pFSf76',
  requestDate: '2014-09-24T00:55:08.347Z',
  status: 'pending',
  token: '6akAeXT66eLfJpCgmAsaT3QLqcXZd1cRseBUSych5KZs7iXAxjLyRvCU8TjMB3DBfw'
}

```

### creditPartialPayment

Will accept a partial payment to complete the payment. Must be done after *six confirmations* and the invoice has been written to the ledger.

Parameters:

Name               | Type                       | Required
------------------ | -------------------------- | -------------
commit             | boolean                    | true

If commit is `true` it will apply the changes, if `false` it will only show the effects of the action.

```javascript
{
  "commit": true
}
```

Example Response:

```javascript

{
  url: 'https://sample.com/invoice?id=8qKF5wGktvwnkGVLmxPgbs',
  status: 'confirmed',
  btcPrice: '0.0010',
  btcDue: '0.0000',
  price: 0.44,
  currency: 'USD',
  exRates: { USD: 426.09000000000003 },
  invoiceTime: 1411586102477,
  expirationTime: 1411587002477,
  currentTime: 1411587051574,
  id: '8qKF5wGktvwnkGVLmxPgbs',
  btcPaid: '0.0010',
  rate: 426.09,
  exceptionStatus: false,
  transactions: [
    {
      amount: 100000,
      confirmations: 2,
      time: '1970-01-01T00:00:00.000Z',
      receivedTime: '2014-09-24T19:17:33.644Z'
    }
  ]
}

```

**Note**: The status has changed to `confirmed`, the `btcPrice` has been adjusted and the `exceptionStatus` has been cleared.

### creditOverpayment

Will accept an over payment to complete the payment. Must be done after *six confirmations*, and the invoice has been written to the ledger.

Parameters:

Name               | Type                       | Required
------------------ | -------------------------- | -------------
commit             | boolean                    | true

If commit is `true` it will apply the changes, if `false` it will only show the effects of the action.

Example Params:

```javascript
{
  "commit": true
}
```

Example Response:

```javascript

{
  url: 'https://sample.com/invoice?id=8KNSxj3m1rcbAWnghSpMKD',
  status: 'complete',
  btcPrice: '0.0023',
  btcDue: '0.0000',
  price: 1,
  currency: 'USD',
  exRates: { USD: 428.33 },
  invoiceTime: 1411588107332,
  expirationTime: 1411589007332,
  currentTime: 1411591834055,
  id: '8KNSxj3m1rcbAWnghSpMKD',
  btcPaid: '0.0023',
  rate: 428.33,
  exceptionStatus: false,
  transactions: [
    {
      amount: 500000,
      confirmations: 6,
      time: '1970-01-01T00:00:00.000Z',
      receivedTime: '2014-09-24T19:49:18.662Z' },
    {
      amount: -270000,
      confirmations: 6,
      time: '2014-09-24T20:50:34.046Z',
      receivedTime: '2014-09-24T20:50:34.046Z'
    }
  ]
}

```

**Note**: The status has changed to `confirmed`, the `btcPaid` has been adjusted, and the additional payment is credited to your ledger.

### getRefunds

Will list the refunds on the invoice.

Example Response:

```javascript
[
  {
    id: '3Ls5s9RfzkN9VtPiTkx8mQ',
    requestDate: '2014-09-24T21:05:17.251Z',
    status: 'pending',
    token: '6akAeXT66eLfJpCgmAsaT3AQ3TpX72kJrMPvHqSDG8BCaGNGGiRuechfsAdtGhgMnk'
  }
]
```

### sendNotification

Will send an IPN notification for the invoice.

Example Response:

```javascript
"Success"
```

### Undocumented

- getOrphans
- applyTx
- getTxRequest
- getTxRequests
- get
