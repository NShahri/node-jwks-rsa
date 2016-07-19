# jwks-rsa - Express Example

The `jwks-rsa` library provides a small helper that makes it easy to configure `express-jwt` with the `RS256` algorithm. Using `expressJwtSecret` you can generate a secret provider that will provide the right signing certificate to `express-jwt` based on the `kid` in the JWT header.

```js
const Express = require('express');
const jwt = require('express-jwt');
const jwksRsa = require('jwks-rsa');

...

// Initialize the app.
const app = new Express();
app.use(jwt({
  secret: jwksRsa.expressJwtSecret({
    cache: true,
    rateLimit: true,
    jwksRequestsPerMinute: 5,
    jwksUri: `https://my-authz-server/.well-known/jwks.json`
  }),
  audience: 'urn:my-resource-server',
  issuer: 'https://my-authz-server/',
  algorithms: [ 'RS256' ]
}));
```

## Running the sample

```bash
DEBUG=express,jwks JWKS_HOST=https://my-authz-server AUDIENCE=urn:my-resource-server ISSUER=https://my-authz-server/ node server.js
```

> Tip: You can use Auth0 to test this.

## How this work?

When you have the sample running you'll need to get a token from your Authorization Server, which will look like this:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlJrSTVNakk1T1VZNU9EYzFOMFE0UXpNME9VWXpOa1ZHTVRKRE9VRXpRa0ZDT1RVM05qRTJSZyJ9.eyJpc3MiOiJodHRwczovL3NhbmRyaW5vLmF1dGgwLmNvbS8iLCJzdWIiOiJhdXRoMHw1NjMyNTAxZjQ2OGYwZjE3NTZmNGNhYjAiLCJhdWQiOiJQN2JhQnRTc3JmQlhPY3A5bHlsMUZEZVh0ZmFKUzRyViIsImV4cCI6MTQ2ODk2NDkyNiwiaWF0IjoxNDY4OTI4OTI2fQ.NaNeRSDCNu522u4hcVhV65plQOiGPStgSzVW4vR0liZYQBlZ_3OKqCmHXsu28NwVHW7_KfVgOz4m3BK6eMDZk50dAKf9LQzHhiG8acZLzm5bNMU3iobSAJdRhweRht544ZJkzJ-scS1fyI4gaPS5aD3SaLRYWR0Xsb6N1HU86trnbn-XSYSspNqzIUeJjduEpPwC53V8E2r1WZXbqEHwM9_BGEeNTQ8X9NqCUvbQtnylgYR3mfJRL14JsCWNFmmamgNNHAI0uAJo84mu_03I25eVuCK0VYStLPd0XFEyMVFpk48Bg9KNWLMZ7OUGTB_uv_1u19wKYtqeTbt9m1YcPMQ
```

You can then call the sample application like this:

```js
var request = require("request");

var options = {
  method: 'GET',
  url: 'http://localhost:4001/me',
  headers: { authorization: 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI...' }
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

A few things will happen now:

 1. `express-jwt` will decode the token and pass the request, the header and the payload to `jwksRsa.expressJwtSecret`
 2. `jwks-rsa` will then download all signing keys from the JWKS endpoint and see if a one of the signing keys matches the `kid` in the header of the JWT.
   a. If none of the signing keys match the incoming `kid`, an error will be thrown
   b. If we have a match, we will pass the right signing key to `express-jwt`
 3. `express-jwt` will the continue its own logic to validate the signature of the token, the expiration, audience, issuer, ...

If you repeat this call a few times you'll see in the console output that we're not calling the JWKS endpoint anymore, because caching has been enabled.

If you then make multiple calls with a `kid` that is not defined in the JWKS endpoint, you'll see that rate limiting will kick in.