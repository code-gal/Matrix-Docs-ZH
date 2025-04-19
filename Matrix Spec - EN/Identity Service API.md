> [Matrix Specification](https://spec.matrix.org/unstable/) | [Identity Service API](https://spec.matrix.org/unstable/identity-service-api/)

# Identity Service API

The Matrix client-server and server-server APIs are largely expressed in Matrix user identifiers. From time to time, it is useful to refer to users by other (“third-party”) identifiers, or “3PID"s, e.g. their email address or phone number. This Identity Service Specification describes how mappings between third-party identifiers and Matrix user identifiers can be established, validated, and used. This description technically may apply to any 3PID, but in practice has only been applied specifically to email addresses and phone numbers.

# General principles[](https://spec.matrix.org/unstable/identity-service-api/#general-principles)

The purpose of an identity server is to validate, store, and answer questions about the identities of users. In particular, it stores associations of the form “identifier X represents the same user as identifier Y”, where identities may exist on different systems (such as email addresses, phone numbers, Matrix user IDs, etc).

The identity server has some private-public keypairs. When asked about an association, it will sign details of the association with its private key. Clients may validate the assertions about associations by verifying the signature with the public key of the identity server.

In general, identity servers are treated as reliable oracles. They do not necessarily provide evidence that they have validated associations, but claim to have done so. Establishing the trustworthiness of an individual identity server is left as an exercise for the client.

3PID types are described in [3PID Types](https://spec.matrix.org/unstable/appendices#3pid-types) Appendix.

# API standards[](https://spec.matrix.org/unstable/identity-service-api/#api-standards)

The mandatory baseline for identity server communication in Matrix is exchanging JSON objects over HTTP APIs. HTTPS is required for communication.

All `POST` and `PUT` endpoints, with the exception (for historical reasons) of [`POST /_matrix/identity/v2/account/logout`](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountlogout), require the client to supply a request body containing a (potentially empty) JSON object. Clients should supply a `Content-Type` header of `application/json` for all requests with JSON bodies, but this is not required.

Similarly, all endpoints require the server to return a JSON object. Servers must include a `Content-Type` header of `application/json` for all JSON responses.

All JSON data, in requests or responses, must be encoded using UTF-8.

## Standard error response[](https://spec.matrix.org/unstable/identity-service-api/#standard-error-response)

Any errors which occur at the Matrix API level MUST return a “standard error response”. This is a JSON object which looks like:

```
{
  "errcode": "<error code>",
  "error": "<error message>"
}
```

The `error` string will be a human-readable error message, usually a sentence explaining what went wrong. The `errcode` string will be a unique string which can be used to handle an error message e.g. `M_FORBIDDEN`. There may be additional keys depending on the error, but the keys `error` and `errcode` MUST always be present.

Some standard error codes are below:

`M_NOT_FOUND` The resource requested could not be located.

`M_MISSING_PARAMS` The request was missing one or more parameters.

`M_INVALID_PARAM` The request contained one or more invalid parameters.

`M_SESSION_NOT_VALIDATED` The session has not been validated.

`M_NO_VALID_SESSION` A session could not be located for the given parameters.

`M_SESSION_EXPIRED` The session has expired and must be renewed.

`M_INVALID_EMAIL` The email address provided was not valid.

`M_EMAIL_SEND_ERROR` There was an error sending an email. Typically seen when attempting to verify ownership of a given email address.

`M_INVALID_ADDRESS` The provided third-party address was not valid.

`M_SEND_ERROR` There was an error sending a notification. Typically seen when attempting to verify ownership of a given third-party address.

`M_UNRECOGNIZED` The request contained an unrecognised value, such as an unknown token or medium.

This is also used as the response if a server did not understand the request. This is expected to be returned with a 404 HTTP status code if the endpoint is not implemented or a 405 HTTP status code if the endpoint is implemented, but the incorrect HTTP method is used.

`M_THREEPID_IN_USE` The third-party identifier is already in use by another user. Typically this error will have an additional `mxid` property to indicate who owns the third-party identifier.

`M_UNKNOWN` An unknown error has occurred.

# Privacy[](https://spec.matrix.org/unstable/identity-service-api/#privacy)

Identity is a privacy-sensitive issue. While the identity server exists to provide identity information, access should be restricted to avoid leaking potentially sensitive data. In particular, being able to construct large-scale connections between identities should be avoided. To this end, in general APIs should allow a 3PID to be mapped to a Matrix user identity, but not in the other direction (i.e. one should not be able to get all 3PIDs associated with a Matrix user ID, or get all 3PIDs associated with a 3PID).

# Web browser clients[](https://spec.matrix.org/unstable/identity-service-api/#web-browser-clients)

It is realistic to expect that some clients will be written to be run within a web browser or similar environment. In these cases, the identity server should respond to pre-flight requests and supply Cross-Origin Resource Sharing (CORS) headers on all requests.

When a client approaches the server with a pre-flight (OPTIONS) request, the server should respond with the CORS headers for that route. The recommended CORS headers to be returned by servers on all requests are:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization
```

# API Version check[](https://spec.matrix.org/unstable/identity-service-api/#api-version-check)

## GET /_matrix/identity/versions[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityversions)

---

**Added in `v1.1`**

Gets the versions of the specification supported by the server.

Values will take the form `vX.Y` or `rX.Y.Z` in historical cases. See [the Specification Versioning](https://spec.matrix.org/unstable/#specification-versions) for more information.

All supported versions, including patch versions, are reported by the server.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

No request parameters or request body.

---

### Responses

|Status|Description|
|---|---|
|`200`|The versions supported by the server.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`versions`|`[string]`|**Required:** The supported versions.|

```
{
  "versions": [
    "r0.2.0",
    "r0.2.1",
    "v1.1"
  ]
}
```

# Authentication[](https://spec.matrix.org/unstable/identity-service-api/#authentication)

Most endpoints in the Identity Service API require authentication in order to ensure that the requesting user has accepted all relevant policies and is otherwise permitted to make the request.

Identity Servers use a scheme similar to the Client-Server API’s concept of access tokens to authenticate users. The access tokens provided by an Identity Server cannot be used to authenticate Client-Server API requests.

Access tokens may be provided via a request header, using the Authentication Bearer scheme: `Authorization: Bearer TheTokenHere`.

Clients may alternatively provide the access token via a query string parameter: `access_token=TheTokenHere`. This method is deprecated to prevent the access token being leaked in access/HTTP logs and SHOULD NOT be used by clients.

Identity Servers MUST support both methods.

> [!info] info
> **[Changed in `v1.11`]** Sending the access token as a query string parameter is now deprecated.

When credentials are required but missing or invalid, the HTTP call will return with a status of 401 and the error code `M_UNAUTHORIZED`.

## GET /_matrix/identity/v2/account[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2account)

---

Gets information about what user owns the access token used in the request.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

### Request

No request parameters or request body.

---

### Responses

|Status|Description|
|---|---|
|`200`|The token holder’s information.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`user_id`|`string`|**Required:** The user ID which registered the token.|

```
{
  "user_id": "@alice:example.org"
}
```

#### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

## POST /_matrix/identity/v2/account/logout[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountlogout)

---

Logs out the access token, preventing it from being used to authenticate future requests to the server.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

### Request

No request parameters or request body.

---

### Responses

|Status|Description|
|---|---|
|`200`|The token was successfully logged out.|
|`401`|The token is not registered or is otherwise unknown to the server.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

#### 200 response

```
{}
```

#### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_UNKNOWN_TOKEN",
  "error": "Unrecognised access token"
}
```

#### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

## POST /_matrix/identity/v2/account/register[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountregister)

---

Exchanges an OpenID token from the homeserver for an access token to access the identity server. The request body is the same as the values returned by `/openid/request_token` in the Client-Server API.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

#### Request body
|OpenIdCredentials|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`access_token`|`string`|**Required:** An access token the consumer may use to verify the identity of the person who generated the token. This is given to the federation API `GET /openid/userinfo` to verify the user’s identity.|
|`expires_in`|`integer`|**Required:** The number of seconds before this token expires and a new one must be generated.|
|`matrix_server_name`|`string`|**Required:** The homeserver domain the consumer should use when attempting to verify the user’s identity.|
|`token_type`|`string`|**Required:** The string `Bearer`.|

#### Request body example

```
{}
```

---

### Responses

|Status|Description|
|---|---|
|`200`|A token which can be used to authenticate future requests to the identity server.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`token`|`string`|**Required:** An opaque string representing the token to authenticate future requests to the identity server with.|

```
{
  "token": "abc123_OpaqueString"
}
```

# Terms of service[](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)

Identity Servers are encouraged to have terms of service (or similar policies) to ensure that users have agreed to their data being processed by the server. To facilitate this, an identity server can respond to almost any authenticated API endpoint with an HTTP 403 and the error code `M_TERMS_NOT_SIGNED`. The error code is used to indicate that the user must accept new terms of service before being able to continue.

All endpoints which support authentication can return the `M_TERMS_NOT_SIGNED` error. When clients receive the error, they are expected to make a call to `GET /terms` to find out what terms the server offers. The client compares this to the `m.accepted_terms` account data for the user (described later) and presents the user with option to accept the still-missing terms of service. After the user has made their selection, if applicable, the client sends a request to `POST /terms` to indicate the user’s acceptance. The server cannot expect that the client will send acceptance for all pending terms, and the client should not expect that the server will not respond with another `M_TERMS_NOT_SIGNED` on their next request. The terms the user has just accepted are appended to `m.accepted_terms`.

## `m.accepted_terms`[](https://spec.matrix.org/unstable/identity-service-api/#maccepted_terms)

---

A list of terms URLs the user has previously accepted. Clients SHOULD use this to avoid presenting the user with terms they have already agreed to.

|   |   |
|---|---|
|Event type:|Message event|

### Content

|Name|Type|Description|
|---|---|---|
|`accepted`|`[string]`|The list of URLs the user has previously accepted. Should be appended to when the user agrees to new terms.|

### Examples

```
{
  "content": {
    "accepted": [
      "https://example.org/somewhere/terms-1.2-en.html",
      "https://example.org/somewhere/privacy-1.2-en.html"
    ]
  },
  "type": "m.accepted_terms"
}
```

## GET /_matrix/identity/v2/terms[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms)

---

Gets all the terms of service offered by the server. The client is expected to filter through the terms to determine which terms need acceptance from the user. Note that this endpoint does not require authentication.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

No request parameters or request body.

---

### Responses

|Status|Description|
|---|---|
|`200`|The terms of service offered by the server.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`policies`|{string: [Policy Object](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms_response-200_policy-object)}|**Required:** The policies the server offers. Mapped from arbitrary ID (unused in this version of the specification) to a Policy Object.|

|Policy Object|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`version`|`string`|**Required:** The version for the policy. There are no requirements on what this might be and could be “alpha”, semantically versioned, or arbitrary.|
|<Other properties>|[Internationalised Policy](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms_response-200_internationalised-policy)|The policy information for the specified language.|

|Internationalised Policy|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`name`|`string`|**Required:** The translated name of the policy.|
|`url`|`string`|**Required:** The URL, which should include the policy ID, version, and language in it, to be presented to the user as the policy. URLs should have all three criteria to avoid conflicts when the policy is updated in the future: for example, if this was “[https://example.org/terms.html"](https://example.org/terms.html%22) then the server would be unable to update it because the client would have already added that URL to the `m.accepted_terms` collection.|

```
{
  "policies": {
    "privacy_policy": {
      "en": {
        "name": "Privacy Policy",
        "url": "https://example.org/somewhere/privacy-1.2-en.html"
      },
      "fr": {
        "name": "Politique de confidentialité",
        "url": "https://example.org/somewhere/privacy-1.2-fr.html"
      },
      "version": "1.2"
    },
    "terms_of_service": {
      "en": {
        "name": "Terms of Service",
        "url": "https://example.org/somewhere/terms-2.0-en.html"
      },
      "fr": {
        "name": "Conditions d'utilisation",
        "url": "https://example.org/somewhere/terms-2.0-fr.html"
      },
      "version": "2.0"
    }
  }
}
```

## POST /_matrix/identity/v2/terms[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2terms)

---

Called by a client to indicate that the user has accepted/agreed to the included set of URLs. Servers MUST NOT assume that the client will be sending all previously accepted URLs and should therefore append the provided URLs to what the server already knows has been accepted.

Clients MUST provide the URL of the policy in the language that was presented to the user. Servers SHOULD consider acceptance of any one language’s URL as acceptance for all other languages of that policy.

The server should avoid returning `M_TERMS_NOT_SIGNED` because the client may not be accepting all terms at once.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

### Request

#### Request body

|Name|Type|Description|
|---|---|---|
|`user_accepts`|`[string]`|**Required:** The URLs the user is accepting in this request.|

#### Request body example

```
{
  "user_accepts": [
    "https://example.org/somewhere/terms-2.0-en.html"
  ]
}
```

---

### Responses

|Status|Description|
|---|---|
|`200`|The server has considered the user as having accepted the provided URLs.|

#### 200 response

```
{}
```

# Status check[](https://spec.matrix.org/unstable/identity-service-api/#status-check)

## GET /_matrix/identity/v2[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2)

---

Checks that an identity server is available at this API endpoint.

To discover that an identity server is available at a specific URL, this endpoint can be queried and will return an empty object.

This is primarily used for auto-discovery and health check purposes by entities acting as a client for the identity server.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

No request parameters or request body.

---

### Responses

|Status|Description|
|---|---|
|`200`|An identity server is ready to serve requests.|

#### 200 response

```
{}
```

# Key management[](https://spec.matrix.org/unstable/identity-service-api/#key-management)

An identity server has some long-term public-private keypairs. These are named in a scheme `algorithm:identifier`, e.g. `ed25519:0`. When signing an association, the standard [Signing JSON](https://spec.matrix.org/unstable/appendices#signing-json) algorithm applies.

The identity server may also keep track of some short-term public-private keypairs, which may have different usage and lifetime characteristics than the service’s long-term keys.

## GET /_matrix/identity/v2/pubkey/ephemeral/isvalid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeyephemeralisvalid)

---

Check whether a short-term public key is valid.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

#### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`public_key`|`string`|**Required:** The unpadded base64-encoded public key to check.|

---

### Responses

|Status|Description|
|---|---|
|`200`|The validity of the public key.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`valid`|`boolean`|**Required:** Whether the public key is recognised and is currently valid.|

```
{
  "valid": true
}
```

## GET /_matrix/identity/v2/pubkey/isvalid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeyisvalid)

---

Check whether a long-term public key is valid. The response should always be the same, provided the key exists.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

#### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`public_key`|`string`|**Required:** The unpadded base64-encoded public key to check.|

---

### Responses

|Status|Description|
|---|---|
|`200`|The validity of the public key.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`valid`|`boolean`|**Required:** Whether the public key is recognised and is currently valid.|

```
{
  "valid": true
}
```

## GET /_matrix/identity/v2/pubkey/{keyId}[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeykeyid)

---

Get the public key for the passed key ID.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

### Request

#### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`keyId`|`string`|**Required:** The ID of the key. This should take the form algorithm:identifier where algorithm identifies the signing algorithm, and the identifier is an opaque string.|

---

### Responses

|Status|Description|
|---|---|
|`200`|The public key exists.|
|`404`|The public key was not found.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`public_key`|`string`|**Required:** Unpadded Base64 encoded public key.|

```
{
  "public_key": "VXuGitF39UH5iRfvbIknlvlAVKgD1BsLDMvBf0pmp7c"
}
```

#### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_NOT_FOUND",
  "error": "The public key was not found"
}
```

# Association lookup[](https://spec.matrix.org/unstable/identity-service-api/#association-lookup)

### GET /_matrix/identity/v2/hash_details[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2hash_details)

---

Gets parameters for hashing identifiers from the server. This can include any of the algorithms defined in this specification.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

No request parameters or request body.

---

#### Responses

|Status|Description|
|---|---|
|`200`|The hash function information.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`algorithms`|`[string]`|**Required:** The algorithms the server supports. Must contain at least `sha256`.|
|`lookup_pepper`|`string`|**Required:**<br><br>The pepper the client MUST use in hashing identifiers, and MUST supply to the `/lookup` endpoint when performing lookups.<br><br>Servers SHOULD rotate this string often.|

```
{
  "algorithms": [
    "none",
    "sha256"
  ],
  "lookup_pepper": "matrixrocks"
}
```

### POST /_matrix/identity/v2/lookup[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2lookup)

---

Looks up the set of Matrix User IDs which have bound the 3PIDs given, if bindings are available. Note that the format of the addresses is defined later in this specification.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`addresses`|`[string]`|**Required:**<br><br>The addresses to look up. The format of the entries here depend on the `algorithm` used. Note that queries which have been incorrectly hashed or formatted will lead to no matches.<br><br>Note that addresses are case sensitive: review the [3PID Types](https://spec.matrix.org/unstable/appendices#3pid-types) to verify the intended case an identifier should be prior to submission/hashing.|
|`algorithm`|`string`|**Required:** The algorithm the client is using to encode the `addresses`. This should be one of the available options from `/hash_details`.|
|`pepper`|`string`|**Required:** The pepper from `/hash_details`. This is required even when the `algorithm` does not make use of it.|

##### Request body example

```
{
  "addresses": [
    "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc",
    "nlo35_T5fzSGZzJApqu8lgIudJvmOQtDaHtr-I4rU7I"
  ],
  "algorithm": "sha256",
  "pepper": "matrixrocks"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The associations for any matched `addresses`.|
|`400`|The client’s request was invalid in some way. One possible problem could be the `pepper` being invalid after the server has rotated it - this is presented with the `M_INVALID_PEPPER` error code. Clients SHOULD make a call to `/hash_details` to get a new pepper in this scenario, being careful to avoid retry loops. `M_INVALID_PARAM` can also be returned to indicate the client supplied an `algorithm` that is unknown to the server.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`mappings`|`{string: string}`|**Required:** Any applicable mappings of `addresses` to Matrix User IDs. Addresses which do not have associations will not be included, which can make this property be an empty object.|

```
{
  "mappings": {
    "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc": "@alice:example.org"
  }
}
```

##### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_INVALID_PEPPER",
  "error": "Unknown or invalid pepper - has it been rotated?"
}
```

## Client behaviour[](https://spec.matrix.org/unstable/identity-service-api/#client-behaviour)

Prior to performing a lookup clients SHOULD make a request to the `/hash_details` endpoint to determine what algorithms the server supports (described in more detail below). The client then uses this information to form a `/lookup` request and receive known bindings from the server.

Clients MUST support at least the `sha256` algorithm.

## Server behaviour[](https://spec.matrix.org/unstable/identity-service-api/#server-behaviour)

Servers, upon receipt of a `/lookup` request, will compare the query against known bindings it has, hashing the identifiers it knows about as needed to verify exact matches to the request.

Servers MUST support at least the `sha256` algorithm.

## Algorithms[](https://spec.matrix.org/unstable/identity-service-api/#algorithms)

Some algorithms are defined as part of the specification, however other formats can be negotiated between the client and server using `/hash_details`.

### `sha256`[](https://spec.matrix.org/unstable/identity-service-api/#sha256)

This algorithm MUST be supported by clients and servers at a minimum. It is additionally the preferred algorithm for lookups.

When using this algorithm, the client converts the query first into strings separated by spaces in the format `<address> <medium> <pepper>`. The `<pepper>` is retrieved from `/hash_details`, the `<medium>` is typically `email` or `msisdn` (both lowercase), and the `<address>` is the 3PID to search for. For example, if the client wanted to know about `alice@example.org`’s bindings, it would first format the query as `alice@example.org email ThePepperGoesHere`.

> [!info] RATIONALE:
> Mediums and peppers are appended to the address to prevent a common prefix for each 3PID, helping prevent attackers from pre-computing the internal state of the hash function.

After formatting each query, the string is run through SHA-256 as defined by [RFC 4634](https://tools.ietf.org/html/rfc4634). The resulting bytes are then encoded using URL-Safe [Unpadded Base64](https://spec.matrix.org/unstable/appendices#unpadded-base64) (similar to [room version 4’s event ID format](https://spec.matrix.org/unstable/rooms/v4#event-ids)).

An example set of queries when using the pepper `matrixrocks` would be:


```
"alice@example.com email matrixrocks" -> "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc"
"bob@example.com email matrixrocks"   -> "LJwSazmv46n0hlMlsb_iYxI0_HXEqy_yj6Jm636cdT8"
"18005552067 msisdn matrixrocks"      -> "nlo35_T5fzSGZzJApqu8lgIudJvmOQtDaHtr-I4rU7I"
```


The set of hashes is then given as the `addresses` array in `/lookup`. Note that the pepper used MUST be supplied as `pepper` in the `/lookup` request.

### `none`[](https://spec.matrix.org/unstable/identity-service-api/#none)

This algorithm performs cleartext lookups on the identity server. Typically this algorithm should not be used due to the security concerns of unhashed identifiers, however some scenarios (such as LDAP-backed identity servers) prevent the use of hashed identifiers. Identity servers (and optionally clients) can use this algorithm to perform those kinds of lookups.

Similar to the `sha256` algorithm, the client converts the queries into strings separated by spaces in the format `<address> <medium>` - note the lack of `<pepper>`. For example, if the client wanted to know about `alice@example.org`’s bindings, it would format the query as `alice@example.org email`.

The formatted strings are then given as the `addresses` in `/lookup`. Note that the `pepper` is still required, and must be provided to ensure the client has made an appropriate request to `/hash_details` first.

## Security considerations[](https://spec.matrix.org/unstable/identity-service-api/#security-considerations)

> [!info] info:
> [MSC2134](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/2134-identity-hash-lookup.md) has much more information about the security considerations made for this section of the specification. This section covers the high-level details for why the specification is the way it is.

Typically the lookup endpoint is used when a client has an unknown 3PID it wants to find a Matrix User ID for. Clients normally do this kind of lookup when inviting new users to a room or searching a user’s address book to find any Matrix users they may not have discovered yet. Rogue or malicious identity servers could harvest this unknown information and do nefarious things with it if it were sent in plain text. In order to protect the privacy of users who might not have a Matrix identifier bound to their 3PID addresses, the specification attempts to make it difficult to harvest 3PIDs.

> [!info] RATIONALE:
> Hashing identifiers, while not perfect, helps make the effort required to harvest identifiers significantly higher. Phone numbers in particular are still difficult to protect with hashing, however hashing is objectively better than not.
> 
> An alternative to hashing would be using bcrypt or similar with many rounds, however by nature of needing to serve mobile clients and clients on limited hardware the solution needs be kept relatively lightweight.

Clients should be cautious of servers not rotating their pepper very often, and potentially of servers which use a weak pepper - these servers may be attempting to brute force the identifiers or use rainbow tables to mine the addresses. Similarly, clients which support the `none` algorithm should consider at least warning the user of the risks in sending identifiers in plain text to the identity server.

Addresses are still potentially reversible using a calculated rainbow table given some identifiers, such as phone numbers, common email address domains, and leaked addresses are easily calculated. For example, phone numbers can have roughly 12 digits to them, making them an easier target for attack than email addresses.

# Establishing associations[](https://spec.matrix.org/unstable/identity-service-api/#establishing-associations)

The flow for creating an association is session-based.

Within a session, one may prove that one has ownership of a 3PID. Once this has been established, the user can form an association between that 3PID and a Matrix user ID. Note that this association is only proved one way; a user can associate _any_ Matrix user ID with a validated 3PID, i.e. I can claim that any email address I own is associated with @billg:microsoft.com.

Sessions are time-limited; a session is considered to have been modified when it was created, and then when a validation is performed within it. A session can only be checked for validation, and validation can only be performed within a session, within a 24-hour period since its most recent modification. Any attempts to perform these actions after the expiry will be rejected, and a new session should be created and used instead.

To start a session, the client makes a request to the appropriate `/requestToken` endpoint. The identity server then sends a validation token to the user, and the user provides the token to the client. The client then provides the token to the appropriate `/submitToken` endpoint, completing the session. At this point, the client should `/bind` the third-party identifier or leave it for another entity to bind.

## Format of a validation token[](https://spec.matrix.org/unstable/identity-service-api/#format-of-a-validation-token)

The format of the validation token is left up to the identity server: it should choose one appropriate to the 3PID type. (For example, it would be inappropriate to expect a user to copy a long passphrase including punctuation from an SMS message into a client.)

Whatever format the identity server uses, the validation token must consist of at most 255 Unicode codepoints. Clients must pass the token through without modification.

## Email associations[](https://spec.matrix.org/unstable/identity-service-api/#email-associations)

### POST /_matrix/identity/v2/validate/email/requestToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validateemailrequesttoken)

---

Create a session for validating an email address.

The identity server will send an email containing a token. If that token is presented to the identity server in the future, it indicates that that user was able to read the email for that email address, and so we validate ownership of the email address.

Note that homeservers offer APIs that proxy this API, adding additional behaviour on top, for example, `/register/email/requestToken` is designed specifically for use when registering an account and therefore will inform the user if the email address given is already registered on the server.

Note: for backwards compatibility with previous drafts of this specification, the parameters may also be specified as `application/x-form-www-urlencoded` data. However, this usage is deprecated.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** A unique string generated by the client, and used to identify the validation attempt. It must be a string consisting of the characters `[0-9a-zA-Z.=_-]`. Its length must not exceed 255 characters and it must not be empty.|
|`email`|`string`|**Required:** The email address to validate.|
|`next_link`|`string`|Optional. When the validation is completed, the identity server will redirect the user to this URL. This option is ignored when submitting 3PID validation information through a POST request.|
|`send_attempt`|`integer`|**Required:** The server will only send an email if the `send_attempt` is a number greater than the most recent one which it has seen, scoped to that `email` + `client_secret` pair. This is to avoid repeatedly sending the same email in the case of request retries between the POSTing user and the identity server. The client should increment this value if they desire a new email (e.g. a reminder) to be sent. If they do not, the server should respond with success but not resend the email.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "email": "alice@example.org",
  "next_link": "https://example.org/congratulations.html",
  "send_attempt": 1
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|Session created.|
|`400`|An error occurred. Some possible errors are:<br><br>- `M_INVALID_EMAIL`: The email address provided was invalid.<br>- `M_EMAIL_SEND_ERROR`: The validation email could not be sent.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`sid`|`string`|**Required:** The session ID. Session IDs are opaque strings generated by the identity server. They must consist entirely of the characters `[0-9a-zA-Z.=_-]`. Their length must not exceed 255 characters and they must not be empty.|

```
{
  "sid": "123abc"
}
```

##### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_INVALID_EMAIL",
  "error": "The email address is not valid"
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### GET /_matrix/identity/v2/validate/email/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2validateemailsubmittoken)

---

Validate ownership of an email address.

If the three parameters are consistent with a set generated by a `requestToken` call, ownership of the email address is considered to have been validated. This does not publish any information publicly, or associate the email address with any Matrix user ID. Specifically, calls to `/lookup` will not show a binding.

Note that, in contrast with the POST version, this endpoint will be used by end-users, and so the response should be human-readable.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret that was supplied to the `requestToken` call.|
|`sid`|`string`|**Required:** The session ID, generated by the `requestToken` call.|
|`token`|`string`|**Required:** The token generated by the `requestToken` call and emailed to the user.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|Email address is validated.|
|`3XX`|Email address is validated, and the `next_link` parameter was provided to the `requestToken` call. The user must be redirected to the URL provided by the `next_link` parameter.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|
|`4XX`|Validation failed.|

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### POST /_matrix/identity/v2/validate/email/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validateemailsubmittoken)

---

Validate ownership of an email address.

If the three parameters are consistent with a set generated by a `requestToken` call, ownership of the email address is considered to have been validated. This does not publish any information publicly, or associate the email address with any Matrix user ID. Specifically, calls to `/lookup` will not show a binding.

The identity server is free to match the token case-insensitively, or carry out other mapping operations such as unicode normalisation. Whether to do so is an implementation detail for the identity server. Clients must always pass on the token without modification.

Note: for backwards compatibility with previous drafts of this specification, the parameters may also be specified as `application/x-form-www-urlencoded` data. However, this usage is deprecated.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret that was supplied to the `requestToken` call.|
|`sid`|`string`|**Required:** The session ID, generated by the `requestToken` call.|
|`token`|`string`|**Required:** The token generated by the `requestToken` call and emailed to the user.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "sid": "1234",
  "token": "atoken"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The success of the validation.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`success`|`boolean`|**Required:** Whether the validation was successful or not.|

```
{
  "success": true
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

## Phone number associations[](https://spec.matrix.org/unstable/identity-service-api/#phone-number-associations)

### POST /_matrix/identity/v2/validate/msisdn/requestToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validatemsisdnrequesttoken)

---

Create a session for validating a phone number.

The identity server will send an SMS message containing a token. If that token is presented to the identity server in the future, it indicates that that user was able to read the SMS for that phone number, and so we validate ownership of the phone number.

Note that homeservers offer APIs that proxy this API, adding additional behaviour on top, for example, `/register/msisdn/requestToken` is designed specifically for use when registering an account and therefore will inform the user if the phone number given is already registered on the server.

Note: for backwards compatibility with previous drafts of this specification, the parameters may also be specified as `application/x-form-www-urlencoded` data. However, this usage is deprecated.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** A unique string generated by the client, and used to identify the validation attempt. It must be a string consisting of the characters `[0-9a-zA-Z.=_-]`. Its length must not exceed 255 characters and it must not be empty.|
|`country`|`string`|**Required:** The two-letter uppercase ISO-3166-1 alpha-2 country code that the number in `phone_number` should be parsed as if it were dialled from.|
|`next_link`|`string`|Optional. When the validation is completed, the identity server will redirect the user to this URL. This option is ignored when submitting 3PID validation information through a POST request.|
|`phone_number`|`string`|**Required:** The phone number to validate.|
|`send_attempt`|`integer`|**Required:** The server will only send an SMS if the `send_attempt` is a number greater than the most recent one which it has seen, scoped to that `country` + `phone_number` + `client_secret` triple. This is to avoid repeatedly sending the same SMS in the case of request retries between the POSTing user and the identity server. The client should increment this value if they desire a new SMS (e.g. a reminder) to be sent.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "country": "GB",
  "next_link": "https://example.org/congratulations.html",
  "phone_number": "07700900001",
  "send_attempt": 1
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|Session created.|
|`400`|An error occurred. Some possible errors are:<br><br>- `M_INVALID_ADDRESS`: The phone number provided was invalid.<br>- `M_SEND_ERROR`: The validation SMS could not be sent.<br>- `M_DESTINATION_REJECTED`: The identity server cannot deliver an SMS to the provided country or region.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`sid`|`string`|**Required:** The session ID. Session IDs are opaque strings generated by the identity server. They must consist entirely of the characters `[0-9a-zA-Z.=_-]`. Their length must not exceed 255 characters and they must not be empty.|

```
{
  "sid": "123abc"
}
```

##### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_INVALID_ADDRESS",
  "error": "The phone number is not valid"
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### GET /_matrix/identity/v2/validate/msisdn/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2validatemsisdnsubmittoken)

---

Validate ownership of a phone number.

If the three parameters are consistent with a set generated by a `requestToken` call, ownership of the phone number address is considered to have been validated. This does not publish any information publicly, or associate the phone number with any Matrix user ID. Specifically, calls to `/lookup` will not show a binding.

Note that, in contrast with the POST version, this endpoint will be used by end-users, and so the response should be human-readable.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret that was supplied to the `requestToken` call.|
|`sid`|`string`|**Required:** The session ID, generated by the `requestToken` call.|
|`token`|`string`|**Required:** The token generated by the `requestToken` call and sent to the user.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|Phone number is validated.|
|`3XX`|Phone number address is validated, and the `next_link` parameter was provided to the `requestToken` call. The user must be redirected to the URL provided by the `next_link` parameter.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|
|`4XX`|Validation failed.|

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### POST /_matrix/identity/v2/validate/msisdn/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validatemsisdnsubmittoken)

---

Validate ownership of a phone number.

If the three parameters are consistent with a set generated by a `requestToken` call, ownership of the phone number is considered to have been validated. This does not publish any information publicly, or associate the phone number address with any Matrix user ID. Specifically, calls to `/lookup` will not show a binding.

The identity server is free to match the token case-insensitively, or carry out other mapping operations such as unicode normalisation. Whether to do so is an implementation detail for the identity server. Clients must always pass on the token without modification.

Note: for backwards compatibility with previous drafts of this specification, the parameters may also be specified as `application/x-form-www-urlencoded` data. However, this usage is deprecated.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret that was supplied to the `requestToken` call.|
|`sid`|`string`|**Required:** The session ID, generated by the `requestToken` call.|
|`token`|`string`|**Required:** The token generated by the `requestToken` call and sent to the user.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "sid": "1234",
  "token": "atoken"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The success of the validation.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`success`|`boolean`|**Required:** Whether the validation was successful or not.|

```
{
  "success": true
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

## General[](https://spec.matrix.org/unstable/identity-service-api/#general)

### POST /_matrix/identity/v2/3pid/bind[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidbind)

---

Publish an association between a session and a Matrix user ID.

Future calls to `/lookup` for any of the session's 3pids will return this association.

Note: for backwards compatibility with previous drafts of this specification, the parameters may also be specified as `application/x-form-www-urlencoded` data. However, this usage is deprecated.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret passed to the `requestToken` call.|
|`mxid`|`string`|**Required:** The Matrix user ID to associate with the 3pids.|
|`sid`|`string`|**Required:** The Session ID generated by the `requestToken` call.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "mxid": "@ears:matrix.org",
  "sid": "1234"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The association was published.|
|`400`|The association was not published.<br><br>If the session has not been validated, then `errcode` will be `M_SESSION_NOT_VALIDATED`. If the session has timed out, then `errcode` will be `M_SESSION_EXPIRED`.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|
|`404`|The Session ID or client secret were not found|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The 3pid address of the user being looked up.|
|`medium`|`string`|**Required:** The medium type of the 3pid.|
|`mxid`|`string`|**Required:** The Matrix user ID associated with the 3pid.|
|`not_after`|`integer`|**Required:** A unix timestamp after which the association is not known to be valid.|
|`not_before`|`integer`|**Required:** A unix timestamp before which the association is not known to be valid.|
|`signatures`|`{string: {string: string}}`|**Required:** The signatures of the verifying identity servers which show that the association should be trusted, if you trust the verifying identity services.|
|`ts`|`integer`|**Required:** The unix timestamp at which the association was verified.|

```
{
  "address": "louise@bobs.burgers",
  "medium": "email",
  "mxid": "@ears:matrix.org",
  "not_after": 4582425849161,
  "not_before": 1428825849161,
  "signatures": {
    "matrix.org": {
      "ed25519:0": "ENiU2YORYUJgE6WBMitU0mppbQjidDLanAusj8XS2nVRHPu+0t42OKA/r6zV6i2MzUbNQ3c3MiLScJuSsOiVDQ"
    }
  },
  "ts": 1428825849161
}
```

##### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_SESSION_NOT_VALIDATED",
  "error": "This validation session has not yet been completed"
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

##### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_NO_VALID_SESSION",
  "error": "No valid session was found matching that sid and client secret"
}
```

### GET /_matrix/identity/v2/3pid/getValidated3pid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv23pidgetvalidated3pid)

---

Determines if a given 3pid has been validated by a user.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|**Required:** The client secret passed to the `requestToken` call.|
|`sid`|`string`|**Required:** The Session ID generated by the `requestToken` call.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|Validation information for the session.|
|`400`|The session has not been validated.<br><br>If the session has not been validated, then `errcode` will be `M_SESSION_NOT_VALIDATED`. If the session has timed out, then `errcode` will be `M_SESSION_EXPIRED`.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|
|`404`|The Session ID or client secret were not found.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The address of the 3pid being looked up.|
|`medium`|`string`|**Required:** The medium type of the 3pid.|
|`validated_at`|`integer`|**Required:** Timestamp, in milliseconds, indicating the time that the 3pid was validated.|

```
{
  "address": "louise@bobs.burgers",
  "medium": "email",
  "validated_at": 1457622739026
}
```

##### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_SESSION_NOT_VALIDATED",
  "error": "This validation session has not yet been completed"
}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

##### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_NO_VALID_SESSION",
  "error": "No valid session was found matching that sid and client secret"
}
```

### POST /_matrix/identity/v2/3pid/unbind[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidunbind)

---

Remove an association between a session and a Matrix user ID.

Future calls to `/lookup` for any of the session’s 3pids will not return the removed association.

The identity server should authenticate the request in one of two ways:

1. The request is signed by the homeserver which controls the `user_id`.
2. The request includes the `sid` and `client_secret` parameters, as per `/3pid/bind`, which proves ownership of the 3PID.

If this endpoint returns a JSON Matrix error, that error should be passed through to the client requesting an unbind through a homeserver, if the homeserver is acting on behalf of a client.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`client_secret`|`string`|The client secret passed to the `requestToken` call.|
|`mxid`|`string`|**Required:** The Matrix user ID to remove from the 3pids.|
|`sid`|`string`|The Session ID generated by the `requestToken` call.|
|`threepid`|[3PID](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidunbind_request_3pid)|**Required:** The 3PID to remove. Must match the 3PID used to generate the session if using `sid` and `client_secret` to authenticate this request.|

|3PID|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The 3PID address to remove.|
|`medium`|`string`|**Required:** A medium from the [3PID Types](https://spec.matrix.org/unstable/appendices/#3pid-types) Appendix, matching the medium of the identifier to unbind.|

##### Request body example

```
{
  "client_secret": "monkeys_are_GREAT",
  "mxid": "@ears:example.org",
  "sid": "1234",
  "threepid": {
    "address": "monkeys_have_ears@example.org",
    "medium": "email"
  }
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The association was successfully removed.|
|`400`|If the response body is not a JSON Matrix error, the identity server does not support unbinds. If a JSON Matrix error is in the response body, the requesting party should respect the error.|
|`403`|The credentials supplied to authenticate the request were invalid. This may also be returned if the identity server does not support the chosen authentication method (such as blocking homeservers from unbinding identifiers).<br><br>Another common error code is `M_TERMS_NOT_SIGNED` where the user needs to [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service) in order to continue.|
|`404`|If the response body is not a JSON Matrix error, the identity server does not support unbinds. If a JSON Matrix error is in the response body, the requesting party should respect the error.|
|`501`|If the response body is not a JSON Matrix error, the identity server does not support unbinds. If a JSON Matrix error is in the response body, the requesting party should respect the error.|

##### 200 response

```
{}
```

##### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "Invalid homeserver signature"
}
```

# Invitation storage[](https://spec.matrix.org/unstable/identity-service-api/#invitation-storage)

An identity server can store pending invitations to a user’s 3PID, which will be retrieved and can be either notified on or look up when the 3PID is associated with a Matrix user ID.

At a later point, if the owner of that particular 3PID binds it with a Matrix user ID, the identity server will attempt to make an HTTP POST to the Matrix user’s homeserver via the [/3pid/onbind](https://spec.matrix.org/unstable/server-server-api#put_matrixfederationv13pidonbind) endpoint. The request MUST be signed with a long-term private key for the identity server.

## POST /_matrix/identity/v2/store-invite[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2store-invite)

---

Store pending invitations to a user’s 3pid.

In addition to the request parameters specified below, an arbitrary number of other parameters may also be specified. These may be used in the invite message generation described below.

The service will generate a random token and an ephemeral key used for accepting the invite.

The service also generates a `display_name` for the inviter, which is a redacted version of `address` which does not leak the full contents of the `address`.

The service records persistently all of the above information.

It also generates an email containing all of this data, sent to the `address` parameter, notifying them of the invitation. The email should reference the `inviter_name`, `room_name`, `room_avatar`, and `room_type` (if present) from the request here.

Also, the generated ephemeral public key will be listed as valid on requests to `/_matrix/identity/v2/pubkey/ephemeral/isvalid`.

Currently, invites may only be issued for 3pids of the `email` medium.

Optional fields in the request should be populated to the best of the server’s ability. Identity servers may use these variables when notifying the `address` of the pending invite for display purposes.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

### Request

#### Request body

|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The email address of the invited user.|
|`medium`|`string`|**Required:** The literal string `email`.|
|`room_alias`|`string`|The Matrix room alias for the room to which the user is invited. This should be retrieved from the `m.room.canonical_alias` state event.|
|`room_avatar_url`|`string`|The Content URI for the room to which the user is invited. This should be retrieved from the `m.room.avatar` state event.|
|`room_id`|`string`|**Required:** The Matrix room ID to which the user is invited|
|`room_join_rules`|`string`|The `join_rule` for the room to which the user is invited. This should be retrieved from the `m.room.join_rules` state event.|
|`room_name`|`string`|The name of the room to which the user is invited. This should be retrieved from the `m.room.name` state event.|
|`room_type`|`string`|The `type` from the `m.room.create` event’s `content`. If the create event doesn’t have a specified `type`, this field is not included.|
|`sender`|`string`|**Required:** The Matrix user ID of the inviting user|
|`sender_avatar_url`|`string`|The Content URI for the avatar of the user ID initiating the invite.|
|`sender_display_name`|`string`|The display name of the user ID initiating the invite.|

#### Request body example

```
{
  "address": "foo@example.com",
  "medium": "email",
  "room_alias": "#somewhere:example.org",
  "room_avatar_url": "mxc://example.org/s0meM3dia",
  "room_id": "!something:example.org",
  "room_join_rules": "public",
  "room_name": "Bob's Emporium of Messages",
  "room_type": "m.space",
  "sender": "@bob:example.com",
  "sender_avatar_url": "mxc://example.org/an0th3rM3dia",
  "sender_display_name": "Bob Smith"
}
```

---

### Responses

|Status|Description|
|---|---|
|`200`|The invitation was stored.|
|`400`|An error has occurred.<br><br>If the 3pid is already bound to a Matrix user ID, the error code will be `M_THREEPID_IN_USE`. If the medium is unsupported, the error code will be `M_UNRECOGNIZED`.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`display_name`|`string`|**Required:** The generated (redacted) display name.|
|`public_keys`|[[PublicKey](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2store-invite_response-200_publickey)]|**Required:** A list of [server’s long-term public key, generated ephemeral public key].|
|`token`|`string`|**Required:** The generated token. Must be a string consisting of the characters `[0-9a-zA-Z.=_-]`. Its length must not exceed 255 characters and it must not be empty.|

|PublicKey|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`key_validity_url`|`string`|**Required:** The URI of an endpoint where the validity of this key can be checked by passing it as a `public_key` query parameter. See [key management](https://spec.matrix.org/unstable/identity-service-api/#key-management).|
|`public_key`|`string`|**Required:** The public key, encoded using [unpadded Base64](https://spec.matrix.org/unstable/appendices/#unpadded-base64).|

```
{
  "display_name": "f...@b...",
  "public_keys": [
    {
      "key_validity_url": "https://example.com/_matrix/identity/v2/pubkey/isvalid",
      "public_key": "serverPublicKeyBase64"
    },
    {
      "key_validity_url": "https://example.com/_matrix/identity/v2/pubkey/ephemeral/isvalid",
      "public_key": "ephemeralPublicKeyBase64"
    }
  ],
  "token": "sometoken"
}
```

#### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_THREEPID_IN_USE",
  "error": "Binding already known",
  "mxid": "@alice:example.com"
}
```

#### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

# Ephemeral invitation signing[](https://spec.matrix.org/unstable/identity-service-api/#ephemeral-invitation-signing)

To aid clients who may not be able to perform crypto themselves, the identity server offers some crypto functionality to help in accepting invitations. This is less secure than the client doing it itself, but may be useful where this isn’t possible.

## POST /_matrix/identity/v2/sign-ed25519[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2sign-ed25519)

---

Sign invitation details.

The identity server will look up `token` which was stored in a call to `store-invite`, and fetch the sender of the invite.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

### Request

#### Request body

|Name|Type|Description|
|---|---|---|
|`mxid`|`string`|**Required:** The Matrix user ID of the user accepting the invitation.|
|`private_key`|`string`|**Required:** The private key, encoded as [Unpadded base64](https://spec.matrix.org/unstable/appendices/#unpadded-base64).|
|`token`|`string`|**Required:** The token from the call to `store-invite`.|

#### Request body example

```
{
  "mxid": "@foo:bar.com",
  "private_key": "base64encodedkey",
  "token": "sometoken"
}
```

---

### Responses

|Status|Description|
|---|---|
|`200`|The signed JSON of the mxid, sender, and token.|
|`403`|The user must do something in order to use this endpoint. One example is an `M_TERMS_NOT_SIGNED` error where the user must [agree to more terms](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service).|
|`404`|The token was not found.|

#### 200 response

|Name|Type|Description|
|---|---|---|
|`mxid`|`string`|**Required:** The Matrix user ID of the user accepting the invitation.|
|`sender`|`string`|**Required:** The Matrix user ID of the user who sent the invitation.|
|`signatures`|`{string: {string: string}}`|**Required:** The signature of the mxid, sender, and token.|
|`token`|`string`|**Required:** The token for the invitation.|

```
{
  "mxid": "@foo:bar.com",
  "sender": "@baz:bar.com",
  "signatures": {
    "my.id.server": {
      "ed25519:0": "def987"
    }
  },
  "token": "abc123"
}
```

#### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

#### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_UNRECOGNIZED",
  "error": "Didn't recognize token"
}
```
