---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - ruby
  - shell
  - javascript

toc_footers:
  - <a href='https://roadster.com/contact'>Contact us for your developer key</a>
  - <a href='https://roadster.com'>Become a Roadster Dealer</a>

includes:
  - errors

search: true
---

# Introduction

Access to Roadster APIs is limited to current dealer, manufacturer or data partners. We support a limited set of endpoints that allow our partners to access certain features of the Roadster platform for use externally or to pass us data required to better integrate our systems into the dealer's software ecosystem.

This initial API documentation has been produced to help introduce our partners to our data and experience endpoints. For additional quesitons, please contact our support teams at [dealersupport@roadster.com](mailto:dealersupport@roadster.com)

<aside class="notice">
All requests should be made using HTTPS.
</aside>

# Authentication

## Token-based API endpoints

> Examples of using the key-based API authorization.

```ruby
require 'net/http'

req = Net::HTTP::Post.new('https://express-dealer-site/api')
req['Authorization'] = 'your_api_key'
```

```shell
# With shell, you can just pass the correct header with each request
curl "https://express-dealer-site/api"
  -H "Authorization: your_api_key"
```

> Make sure to replace `your_api_key` with your API key and `express-dealer-site` with the hostname of the Express Store for the dealership you are attempting to access.

Authentication method depends on the endpoint being used. In general, you should have been provided an API key that you will use in the header of each of your calls. For applications that will be passing us customer records as part of the user interface, you will instead encrypt the data to AES-256-CBC standards using a cipher key provided by Roadster.

`Authorization: your_api_key`

<aside class="notice">
You must replace <code>your_api_key</code> with your personal API key.
</aside>

## Browser-based data passing

> Examples of using a public encryption key

```ruby
require 'openssl'
require 'base64'

key = Base64.decode64("your_encryption_key")
key = Digest::SHA256.digest(key) if(key.kind_of?(String) && 32 != key.bytesize)
aes = OpenSSL::Cipher.new('AES-256-CBC')
aes.encrypt
aes.key = key
iv = aes.random_iv
```

```shell
# With shell you can use OpenSSL to encrypt and decrypt data

echo data | openssl aes-256-cbc -a -salt
enter aes-256-cbc encryption password: your_encryption_key
Verifying - enter aes-256-cbc encryption password: your_encryption_key
```

> Make sure to replace `your_encryption_key` with your API key and the base URL with the URL of the dealership you are attempting to access.

Passing data to Roadster via the browser is possible using encrypted query parameters. Encryption to AES-256-CBC standards is required using an encryption key provided to you by Roadster. All parameters should be encrypted before passing them via the URL query string.

<aside class="notice">
You must replace <code>your_encryption_key</code> with your personal encryption key.
</aside>


# Create Customer

## POST /v1/customers

> Example request for creating a customer with a dealer and VIN

```ruby
req = Net::HTTP::Post.new('https://api.roadster.com/v1/customers')
req['Authorization'] = 'your_api_key'
req.body = {
  customer: {
    first_name: "Steve",
    last_name: "McQueen",
    email: "steve.mcqueen@roadster.com",
    phone: "+14155556666",
    partner_id: "238612817"
  },
  vehicle: {
    vin: "3TMKU72N16M008970",
    make: "Ford",
    model: "Mustang",
    year: "1968"
  },
  dealer: {
    partner_id: "12345"
  }
}
```

```shell
curl 'https://api.roadster.com/v1/customers' -XPOST -d @api-post.json -H 'Authorization: Token your_api_key' -H 'Content-type: application/json' -i

--@api-post.json
{
  "customer": {
    "first_name": "Ralph",
    "last_name": "Dorfin",
    "email": "ralf4@testing.zz",
    "phone": "800-555-1234",
    "partner_id": "1359"
  },
  "vehicle": {
    "vin": "JM1NDAM77K0306400",
    "make": "Mazda",
    "model": "MX-5 Miata",
    "year": "2019"
  },
  "dealer": {
    "partner_id": "51101"
  }
}
```
> Make sure to replace `your_api_key` with your API key.

You can create a new customer in the Roadster Express Strorefront for a specified dealer using a POST to the `/customers` endpoint. The response will include the unique identifier of the user as well as a unique URL to the specified piece of inventory on the dealer's site which will automatically store the provided user as active in the session.

Parameter | Type | Required | Description
----------|------|----------|------------
**customer** | **object** | **yes** | **Object including the user data**
first_name | string | yes | The user's first name
last_name | string | yes | The user's last name
email | string | yes | The user's email address
phone | string | no | The user's phone number (E.164 formatting is preferred)
partner_id | string | no | A unique identifier provided by the partner to be appended to the customer record for reporting purposes.
**dealer** | **object** | **no** | **Data to force user creation against a specific dealer. If included, one of the identifiers below must be used.**
partner_id | string | no | Your unique identifier for the dealership (must be registered with Roadster to impact dealer selection otherwise stored as data only)
dpid | string | no | Roadster's unique identifier for a dealership
**vehicle** | **object** | **no** | **Object including the user's vehicle preference which will also be the resulting landing page of the returned URL.
vin | string | no | The preferred VIN. Without this we will return search level results at a make or model level.
make | string | no | The preferred make, this is required if there is no VIN but model or submodel info is provided.
model | string | no | The preferred model, this is not required if a VIN or submodel is provided or make-level results are sufficient.
year | string | no | The preferred model year, not required if a VIN is provided or the preferred search set should be limited to new.


## Expected response

> Example success response

```ruby
HTTP/1.1 200 OK
{
  "status": "ok",
  "customer": {
    "id": 1650260,
    "redirect_url": "http://brickellmazda.roadsterlocal.com:3000/express/JM1NDAM77K0306400?ucid=1650260"
  }
}
```

```shell
HTTP/1.1 200 OK
{
  "status": "ok",
  "customer": {
    "id": 1650260,
    "redirect_url": "http://brickellmazda.roadster.com/express/JM1NDAM77K0306400?ucid=1650260"
  }
}
```


> Example error response

```ruby
HTTP/1.1 400 Bad Request
{
  "status": "error",
  "error": "Missing email"
}
```

```shell
HTTP/1.1 400 Bad Request
{
  "status": "error",
  "error": "Missing email"
}
```


A request to the create user endpoint will create a customer and respond with a `redirect_url` that can be used to redirect the user to a landing page on the corresponding dealer site with account active in the session and pricing unlocked. In the case that the user exists, the response will still be `success` but the `duplicate` parameter will also be true.

Response field | Type | Description
---------------|------|------------
status | string | The HTTP response status
error | string | Error message (if status of error returned)
**customer** | **object** | **Object containing the customer data that was just created**
id | string | The unque identifier of that customer on the Roadster platform
redirect_url | string | The URL of the landing page that corresponds to the grainularity of data provided in the request. When redirecting to this page will automatically log the user as the active account in the session and unlock any pricing (if applicable).





## Browser-based customer submission

```ruby
require 'openssl'
require 'base64'

def aes256_cbc_encrypt(key, data)
  key = Digest::SHA256.digest(key) if(key.kind_of?(String) && 32 != key.bytesize)
  aes = OpenSSL::Cipher.new('AES-256-CBC')
  aes.encrypt
  aes.key = key
  iv = aes.random_iv
  return aes.update(data) + aes.final, iv
end

def generate_link(vin, key = "your_encryption_key")
  data = {fname: 'Testing', lname: 'Bot', email: 'testing@roadster.com', phone: '800-505-1000'}

  block, iv = aes256_cbc_encrypt key, data.to_query
  mem_token = Base64.encode64(iv + block)
  escaped_token = CGI.escape mem_token
  url = "/express/" + vin + "?mem=" + escaped_token
end
```

```shell
# The query string should be URL encoded BEFORE encryption

echo "fname%3DTesting%26lname%3DBot%26email%3Dtesting%40roadster.com%26phone%3D800-505-1000" | openssl aes-256-cbc -a -salt
enter aes-256-cbc encryption password: your_encryption_key
Verifying - enter aes-256-cbc encryption password: your_encryption_key

curl "https://express-dealer-site/express/[vin]?mem=" + encryption_result
```

> The customer data should be URL encoded _before_ encryption


This browser-based URL request format allows partners to securely pass customer data into the Roadster platform using URL query parameters. The customer data is then used to create an account on the dealer's Express Storefront and bypass any price locks that may be enabled for that dealership. The request also results in lead submission consistent with the lead routing rules the dealership has setup for "Unlock Inquiry" lead types.

### HTTP Request

`GET https://express-dealer-site/express/[vin]?mem=[encrypted_parameters]`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
vin | yes | The vehicle identification number of the requested details landing page.
encrypted_parameters | yes | The customer's encrypted contact information (see detail below).

<aside class="notice">
Replace the <code>express-dealer-site</code> with the hostname for the dealership you are referring to.
</aside>

The `mem` query parameter should contain the encrypted URL query string of the customer's contact information that you are referring to the dealership. This contact information can include any of the following key / value pairs:

Parameter | Required | Description
--------- | ------- | -----------
fname | no | The first name of the customer.
lname | no | The last name of the customer.
email | yes | The email address of the customer (used as the unique identifier for each dealership).
phone | no | The phone number of the customer
custid | no | A unique identifier from the attribution source (only used for reporting purposes).


<aside class="notice">
These parameters should be placed in a standard URL query string and URL encoded _before_ encryption.
E.g. <code>fname%3DTesting%26lname%3DBot%26email%3Dtesting%40roadster.com%26phone%3D800-505-1000</code>
</aside>



