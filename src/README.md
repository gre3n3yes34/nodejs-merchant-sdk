# COINQVEST Merchant SDK (NodeJS)

Official COINQVEST Merchant API SDK for NodeJS by www.coinqvest.com

This SDK implements the REST API documented at https://www.coinqvest.com/en/api-docs

For SDKs in different programming languages, see https://www.coinqvest.com/en/api-docs#sdks

Read our Merchant API [development guide](https://www.coinqvest.com/en/blog/guide-mastering-cryptocurrency-checkouts-with-coinqvest-merchant-apis-321ac139ce15) and the examples below to help you get started.

Requirements
------------
* NodeJS >= 10.14.0
* axios >= 0.21.1

Installation with npm
---------------------
`npm install coinqvest-merchant-sdk`

**Usage Client**
```javascript
require('coinqvest-merchant-sdk');

const client = new CoinqvestClient(
    'YOUR-API-KEY',
    'YOUR-API-SECRET'
);
```
Get your API key and secret here: https://www.coinqvest.com/en/api-settings

## Examples

**Create a Customer** (https://www.coinqvest.com/en/api-docs#post-customer)

Creates a customer object, which can be associated with checkouts, payments, and invoices. Checkouts associated with a customer generate more transaction details, help with your accounting, and can automatically create invoices for your customer and yourself.

```javascript
let response = await client.post('/customer', {
    customer:{
        email: 'john@doe.com',
        firstname: 'John',
        lastname: 'Doe',
        company: 'ACME Inc.',
        adr1: '810 Beach St',
        adr2: 'Finance Department',
        zip: 'CA 94133',
        city: 'San Francisco',
        countrycode: 'US'
    }
});

console.log(response.status);
console.log(response.data);

if (response.status !== 200) {
    // something went wrong, let's abort and debug by looking at our log file
    console.log('Could not create customer. Inspect above log entry.');
    return;
}

let customerId = response.data['customerId']; // store this persistently in your database
```

**Create a Hosted Checkout** (https://www.coinqvest.com/en/api-docs#post-checkout-hosted)

Hosted checkouts are the simplest form of getting paid using the COINQVEST platform.

Using this endpoint, your server submits a set of parameters, such as the payment details including optional tax items, customer information, and settlement currency. Your server then receives a checkout URL in return, which is displayed back to your customer.

Upon visiting the URL, your customer is presented with a checkout page hosted on COINQVEST servers. This page displays all the information the customer needs to complete payment.

```javascript
let response = await client.post('/checkout/hosted', {
    charge:{
        customerId: customerId, // associates this charge with a customer
        billingCurrency: 'USD', // specifies the billing currency
        lineItems: [{ // a list of line items included in this charge
            description: 'T-Shirt',
            netAmount: 10,
            quantity: 1
        }],
        discountItems: [{ // an optional list of discounts
            description: 'Loyalty Discount',
            netAmount: 0.5
        }],
        shippingCostItems: [{ // an optional list of shipping and handling costs
            description: 'Shipping and Handling',
            netAmount: 3.99,
            taxable: false // sometimes shipping costs are taxable
        }],
        taxItems: [{
            name: 'CA Sales Tax',
            percent: 0.0825 // 8.25% CA sales tax
        }]
    },
    settlementAsset: 'USDC:GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN' // your settlement asset as given by GET /assets (or ORIGIN to omit conversion)
});

console.log(response.status);
console.log(response.data);

if (response.status !== 200) {
    // something went wrong, let's abort and debug by looking at our log file
    console.log('Could not create checkout.');
    return;
}

// the checkout was created
// response.data now contains an object as specified in the success response here: https://www.coinqvest.com/en/api-docs#post-checkout
let checkoutId = response.data['checkoutId']; // store this persistently in your database
let url = response.data['url']; // redirect your customer to this URL to complete the payment
```

**Monitor Payment State** (https://www.coinqvest.com/en/api-docs#get-checkout)

Once the payment is captured we notify you via email, [webhook](https://www.coinqvest.com/en/api-docs#webhook-concepts). You can also poll [GET /checkout](https://www.coinqvest.com/en/api-docs#get-checkout) for payment status updates:

```javascript
let response = await client.get('/checkout',  {id: checkoutId});

console.log(response.status);
console.log(response.data);

if (response.status === 200) {
    let state = response.data['checkout']['state'];
    if (state === 'CHECKOUT_COMPLETED') {
        console.log("The payment has completed and your account was credited. You can now ship the goods.");
    } else {
        // try again in 30 seconds or so...
    }
}
```

**Query your USD Wallet** (https://www.coinqvest.com/en/api-docs#get-wallet)
```javascript
let response = await client.get('/wallet', {assetCode: 'USD'});

console.log(response.status);
console.log(response.data);
```

**Query all Wallets** (https://www.coinqvest.com/en/api-docs#get-wallets)
```javascript
let response = await client.get('/wallets');

console.log(response.status);
console.log(response.data);
```

**Withdraw to your Bitcoin Account** (https://www.coinqvest.com/en/api-docs#post-withdrawal)
```javascript
let response = await client.post('/withdrawal', {
    sourceAsset: 'USDC:GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN', // withdraw from your USD wallet
    sourceAmount: 100,
    targetNetwork: 'BITCOIN', // a target network as given by GET /networks
    targetAccount: {
        address: 'bc1qj633nx575jm28smgcp3mx6n3gh0zg6ndr0ew23'
    }
});

console.log(response.status);
console.log(response.data);
```

**Withdraw USDC to your Stellar Account** (https://www.coinqvest.com/en/api-docs#post-withdrawal)
```javascript
let response = await client.post('/withdrawal', {
    sourceAsset: 'USDC:GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN', // withdraw from your USD wallet
    sourceAmount: 100,
    targetNetwork: 'STELLAR', // a target network as given by GET /networks
    targetAccount: {
        account: 'bc1qj633nx575jm28smgcp3mx6n3gh0zg6ndr0ew23',
        memo: 'Transfer Note',
        memoType: 'text'
    }
});

console.log(response.status);
console.log(response.data);
```

**Update a Customer** (https://www.coinqvest.com/en/api-docs#put-customer)
```javascript
let response = await client.put('/customer', {customer:{id: 'CUSTOMER-ID', email: 'john@doe-2.com'}});

console.log(response.status);
console.log(response.data);
```

**Delete a Customer** (https://www.coinqvest.com/en/api-docs#delete-customer)
```javascript
let response = await client.delete('/customer', {id: 'CUSTOMER-ID'});

console.log(response.status);
console.log(response.data);
```

**List your 250 newest customers** (https://www.coinqvest.com/en/api-docs#get-customers)
```javascript
let response = await client.get('/wallet', {limit: 250});

console.log(response.status);
console.log(response.data);
```

**List all available assets** (https://www.coinqvest.com/en/api-docs#get-assets)
```javascript
let response = await client.get('/assets');

console.log(response.status);
console.log(response.data);

**List all available networks** (https://www.coinqvest.com/en/api-docs#get-networks)
```javascript
let response = await client.get('/networks');

console.log(response.status);
console.log(response.data);
```

**The response object** ([axios](https://github.com/axios/axios) HTTP response as given to your callback function)
```javascript
{
    // `data` is the response that was provided by the server
    data: {},

    // `status` is the HTTP status code from the server response
    status: 200,

        // `statusText` is the HTTP status message from the server response
        statusText: 'OK',

        // `headers` the HTTP headers that the server responded with
        // All header names are lower cased and can be accessed using the bracket notation.
        // Example: `response.headers['content-type']`
        headers: {},

    // `config` is the config that was provided to `axios` for the request
    config: {},

    // `request` is the request that generated this response
    // The last ClientRequest instance in node.js (in redirects)
    request: {}
}
```

Please inspect https://www.coinqvest.com/en/api-docs for detailed API documentation or send us an email to service@coinqvest.com.

Support and Feedback
--------------------
Your feedback is appreciated! If you have specific problems or bugs with this SDK, please file an issue on Github. For general feedback and support requests, send an email to service@coinqvest.com.

Contributing
------------

1. Fork it ( https://github.com/COINQVEST/nodejs-merchant-sdk/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

