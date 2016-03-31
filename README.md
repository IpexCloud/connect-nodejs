# Anvil Connect client for Nodejs
[![Build Status](https://travis-ci.org/anvilresearch/connect-nodejs.svg?branch=master)](https://travis-ci.org/anvilresearch/connect-nodejs)

[Anvil Connect][connect] is a modern authorization server built to authenticate 
your users and protect your APIs. It's based on [OAuth 2.0][oauth2] and 
[OpenID Connect][oidc]. 

[This library][connect-nodejs] is a low level OpenID Connect and Anvil Connect 
API client. Previous versions included Express-specific functions and 
middleware. These higher-level functions are being split out into a 
[separate library][connect-express].

[oauth2]: http://tools.ietf.org/html/rfc6749
[oidc]: http://openid.net/connect/
[connect]: https://github.com/anvilresearch/connect
[connect-nodejs]: https://github.com/anvilresearch/connect-nodejs
[connect-express]: https://github.com/anvilresearch/connect-express



### Install

```bash
$ npm install anvil-connect-nodejs --save
```

### Configure

#### new AnvilConnect(config)

```javascript
var AnvilConnect = require('anvil-connect-nodejs');

var anvil = new AnvilConnect({
  issuer: 'https://connect.example.com',
  client_id: 'CLIENT_ID',
  client_secret: 'CLIENT_SECRET',
  redirect_uri: 'REDIRECT_URI',
  scope: 'realm'
})
```

**options**

* `issuer` – REQUIRED uri of your OpenID Connect provider
* `client_id` – OPTIONAL client identifier issued by OIDC provider during registration
* `client_secret` – OPTIONAL confidential value issued by OIDC provider during registration
* `redirect_uri` – OPTIONAL uri users will be redirected back to after authenticating with the issuer
* `scope` – OPTIONAL array of strings, or space delimited string value containing scopes to be included in authorization requests. Defaults to `openid profile`


### OpenID Connect

#### anvil.discover()

Returns a promise providing [OpenID Metadata][oidc-meta] retrieved from the `.well-known/openid-configuration` endpoint for the configured issuer. Sets the response data as `anvil.configuration`.

[oidc-meta]: http://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata

**example**

```javascript
anvil.discover()
  .then(function (openidMetadata) {
    // anvil.configuration === openidMetadata
  })
  .catch(function (error) {
    // ...
  })
```

#### anvil.getJWKs()

Returns a promise providing the JWK set published by the configured issuer. Depends on a prior call to `anvil.discover()`.

**example**

```javascript
anvil.getJWKs()
  .then(function (jwks) {
    // anvil.jwks === jwks
  })
  .catch(function (error) {
    // ...
  })
```

#### anvil.register(registration)

Dynamically registers a new client with the configured issuer and returns a promise for the new client registration. You can learn more about [dynamic registration for Anvil Connect][dynamic-registration] in the docs. Depends on a prior call to `anvil.discover()`.

[dynamic-registration]: https://github.com/anvilresearch/connect-docs/blob/master/clients.md#dynamic-registration

**example**

```javascript
anvil.register({
  client_name: 'Antisocial Network',
  client_uri: 'https://app.example.com',
  logo_uri: 'https://app.example.com/assets/logo.png',
  response_types: ['code'],
  grant_types: ['authorization_code', 'refresh_token'],
  default_max_age: 86400, // one day in seconds
  redirect_uris: ['https://app.example.com/callback.html', 'https://app.example.com/other.html'],
  post_logout_redirect_uris: ['https://app.example.com']
})
```

#### anvil.authorizationUri([endpoint|options])

Accepts a string specifying a non-default endpoint or an options object and returns an authorization URI. Depends on a prior call to `anvil.discover()` and `client_id` being configured.

**options**

* All options accepted by `anvil.authorizationParams()`.
* `endpoint` – This value is used for the path in the returned URI. Defaults to `authorize`. 

**example**

```javascript
anvil.authorizationUri()
// 'https://connect.example.com/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=openid%20profile%20more'

anvil.authorizationUri('signin')
// 'https://connect.example.com/signin?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=openid%20profile%20more'

anvil.authorizationUri({
  endpoint: 'connect/google',
  response_type: 'code id_token token',
  redirect_uri: 'OTHER_REDIRECT_URI',
  scope: 'openid profile extra'
})
// 'https://connect.example.com/connect/google?response_type=code%20id_token%20token&client_id=CLIENT_ID&redirect_uri=OTHER_REDIRECT_URI&scope=openid%20profile%20extra'
```


#### anvil.authorizationParams(options)

Accepts an options object and returns an object containing authorization params including default values. Depends on `client_id` being configured.

**options**

* `response_type` – defaults to `code`
* `redirect_uri` – defaults to the `redirect_uri` configured for this client
* `scope` – defaults to the scope configured for this client
* `state`
* `response_mode`
* `nonce`
* `display`
* `prompt`
* `max_age`
* `ui_locales`
* `id_token_hint`
* `login_hint`
* `acr_values`
* `email`
* `password`
* `provider`

#### anvil.token(options)

Given an authorization code is provided as the `code` option, this method will exchange the auth code for a set of token credentials, then verify the signatures and decode the payloads. Depends on `client_id` and `client_secret` being configured, and prior calls to `anvil.discover()` and `anvil.getJWKs()`.

**options**

 * `code` – value obtained from a successful authorization request with `code` in the `response_types` request param

**example**

```javascript
anvil.token({ code: 'AUTHORIZATION_CODE' })
```

#### anvil.refresh(options)

Given an refresh_token is provided as the `refresh_token` option, this method will exchange the refresh_token for a set of token credentials, then verify the signatures. Depends on `client_id` and `client_secret` being configured, and prior calls to `anvil.discover()` and `anvil.getJWKs()`.

**options**

* `refresh_token` – value obtained from a successful authorization request with `token` in the `response_types` request param

**example**

```javascript
anvil.refresh({ refresh_token: 'REFRESH_TOKEN' })
```

#### anvil.userInfo(options)

Get user info from the issuer.

**options**

* `token` – access token

**example**

```javascript
anvil.userInfo({ token: 'ACCESS_TOKEN' })
```

#### anvil.verify(token, options)

### Anvil Connect API

#### Clients

#### anvil.clients.list()
#### anvil.clients.get(id)
#### anvil.clients.create(data)
#### anvil.clients.update(id, data)
#### anvil.clients.delete(id)

#### Roles

#### anvil.roles.list()
#### anvil.roles.get(id)
#### anvil.roles.create(data)
#### anvil.roles.update(id, data)
#### anvil.roles.delete(id)

#### Scopes

#### anvil.scopes.list()
#### anvil.scopes.get(id)
#### anvil.scopes.create(data)
#### anvil.scopes.update(id, data)
#### anvil.scopes.delete(id)

#### Users

#### anvil.users.list()
#### anvil.users.get(id)
#### anvil.users.create(data)
#### anvil.users.update(id, data)
#### anvil.users.delete(id)

### Example

```javascript
var AnvilConnect = require('anvil-connect-nodejs');

var anvil = new AnvilConnect({
  issuer: 'https://connect.example.com',
  client_id: 'CLIENT_ID',
  client_secret: 'CLIENT_SECRET',
  redirect_uri: 'REDIRECT_URI'
}) 

// get the discovery document for the OpenID Connect provider
anvil.discover()
  .then(function (configuration) {
    // the call to discover() cached the configuration on the client instance
    console.log(anvil.configuration)

    // get the public keys for verifying tokens
    return anvil.getJWKs()  
  })
  .then(function (jwks) {
    // the call to getJwks() cached the JWK set on the client instance too
    console.log(jwks)

    // get an authorization uri
    return anvil.authorizationUri()
  })
  .then(function (uri) {
    console.log(uri)

    // handle an authorization response
    // this verifies the signatures on tokens received from the authorization server
    return anvil.token({ code: 'AUTHORIZATION_CODE' })
  })
  .then(function (tokens) {
    // a successful call to tokens() gives us id_token, access_token, 
    // refresh_token, expiration, and the decoded payloads of the JWTs
    console.log(tokens)

    // get userinfo
    return anvil.userInfo({ token: tokens.access_token })
  })
  .then(function (userInfo) {
    console.log(userInfo)

    // verify an access token received by an API service
    return anvil.verify(JWT, { scope: 'research' })
  })
```
