
> [Matrix Specification](https://spec.matrix.org/v1.11/) | [Server-Server API](https://spec.matrix.org/v1.11/server-server-api/)

# Server-Server API

Matrix homeservers use the Federation APIs (also known as server-server APIs) to communicate with each other. Homeservers use these APIs to push messages to each other in real-time, to retrieve historic messages from each other, and to query profile and presence information about users on each other’s servers.

The APIs are implemented using HTTPS requests between each of the servers. These HTTPS requests are strongly authenticated using public key signatures at the TLS transport layer and using public key signatures in HTTP Authorization headers at the HTTP layer.

There are three main kinds of communication that occur between homeservers:

Persistent Data Units (PDUs): These events are broadcast from one homeserver to any others that have joined the same room (identified by Room ID). They are persisted in long-term storage and record the history of messages and state for a room.

Like email, it is the responsibility of the originating server of a PDU to deliver that event to its recipient servers. However PDUs are signed using the originating server’s private key so that it is possible to deliver them through third-party servers.

Ephemeral Data Units (EDUs): These events are pushed between pairs of homeservers. They are not persisted and are not part of the history of a room, nor does the receiving homeserver have to reply to them.

Queries: These are single request/response interactions between a given pair of servers, initiated by one side sending an HTTPS GET request to obtain some information, and responded by the other. They are not persisted and contain no long-term significant history. They simply request a snapshot state at the instant the query is made.

EDUs and PDUs are further wrapped in an envelope called a Transaction, which is transferred from the origin to the destination homeserver using an HTTPS PUT request.

# API standards[](https://spec.matrix.org/v1.11/server-server-api/#api-standards)

The mandatory baseline for server-server communication in Matrix is exchanging JSON objects over HTTPS APIs. More efficient transports may be specified in future as optional extensions.

All `POST` and `PUT` endpoints require the requesting server to supply a request body containing a (potentially empty) JSON object. Requesting servers should supply a `Content-Type` header of `application/json` for all requests with JSON bodies, but this is not required.

Similarly, all endpoints in this specification require the destination server to return a JSON object. Servers must include a `Content-Type` header of `application/json` for all JSON responses.

All JSON data, in requests or responses, must be encoded using UTF-8.

## TLS[](https://spec.matrix.org/v1.11/server-server-api/#tls)

Server-server communication must take place over HTTPS.

The destination server must provide a TLS certificate signed by a known Certificate Authority.

Requesting servers are ultimately responsible for determining the trusted Certificate Authorities, however are strongly encouraged to rely on the operating system’s judgement. Servers can offer administrators a means to override the trusted authorities list. Servers can additionally skip the certificate validation for a given whitelist of domains or netmasks for the purposes of testing or in networks where verification is done elsewhere, such as with `.onion` addresses.

Servers should respect SNI when making requests where possible: a SNI should be sent for the certificate which is expected, unless that certificate is expected to be an IP address in which case SNI is not supported and should not be sent.

Servers are encouraged to make use of the [Certificate Transparency](https://www.certificate-transparency.org/) project.

## Unsupported endpoints[](https://spec.matrix.org/v1.11/server-server-api/#unsupported-endpoints)

If a request for an unsupported (or unknown) endpoint is received then the server must respond with a 404 `M_UNRECOGNIZED` error.

Similarly, a 405 `M_UNRECOGNIZED` error is used to denote an unsupported method to a known endpoint.

# Server discovery[](https://spec.matrix.org/v1.11/server-server-api/#server-discovery)

## Resolving server names[](https://spec.matrix.org/v1.11/server-server-api/#resolving-server-names)

Each Matrix homeserver is identified by a server name consisting of a hostname and an optional port, as described by the [grammar](https://spec.matrix.org/v1.11/appendices#server-name). Where applicable, a delegated server name uses the same grammar.

Server names are resolved to an IP address and port to connect to, and have various conditions affecting which certificates and `Host` headers to send. The process overall is as follows:

1. If the hostname is an IP literal, then that IP address should be used, together with the given port number, or 8448 if no port is given. The target server must present a valid certificate for the IP address. The `Host` header in the request should be set to the server name, including the port if the server name included one.
    
2. If the hostname is not an IP literal, and the server name includes an explicit port, resolve the hostname to an IP address using CNAME, AAAA or A records. Requests are made to the resolved IP address and given port with a `Host` header of the original server name (with port). The target server must present a valid certificate for the hostname.
    
3. If the hostname is not an IP literal, a regular HTTPS request is made to `https://<hostname>/.well-known/matrix/server`, expecting the schema defined later in this section. 30x redirects should be followed, however redirection loops should be avoided. Responses (successful or otherwise) to the `/.well-known` endpoint should be cached by the requesting server. Servers should respect the cache control headers present on the response, or use a sensible default when headers are not present. The recommended sensible default is 24 hours. Servers should additionally impose a maximum cache time for responses: 48 hours is recommended. Errors are recommended to be cached for up to an hour, and servers are encouraged to exponentially back off for repeated failures. The schema of the `/.well-known` request is later in this section. If the response is invalid (bad JSON, missing properties, non-200 response, etc), skip to step 4. If the response is valid, the `m.server` property is parsed as `<delegated_hostname>[:<delegated_port>]` and processed as follows:
    
    1. If `<delegated_hostname>` is an IP literal, then that IP address should be used together with the `<delegated_port>` or 8448 if no port is provided. The target server must present a valid TLS certificate for the IP address. Requests must be made with a `Host` header containing the IP address, including the port if one was provided.
    2. If `<delegated_hostname>` is not an IP literal, and `<delegated_port>` is present, an IP address is discovered by looking up CNAME, AAAA or A records for `<delegated_hostname>`. The resulting IP address is used, alongside the `<delegated_port>`. Requests must be made with a `Host` header of `<delegated_hostname>:<delegated_port>`. The target server must present a valid certificate for `<delegated_hostname>`.
    3. **[Added in `v1.8`]** If `<delegated_hostname>` is not an IP literal and no `<delegated_port>` is present, an SRV record is looked up for `_matrix-fed._tcp.<delegated_hostname>`. This may result in another hostname (to be resolved using AAAA or A records) and port. Requests should be made to the resolved IP address and port with a `Host` header containing the `<delegated_hostname>`. The target server must present a valid certificate for `<delegated_hostname>`.
    4. **[Deprecated]** If `<delegated_hostname>` is not an IP literal, no `<delegated_port>` is present, and a `_matrix-fed._tcp.<delegated_hostname>` SRV record was not found, an SRV record is looked up for `_matrix._tcp.<delegated_hostname>`. This may result in another hostname (to be resolved using AAAA or A records) and port. Requests should be made to the resolved IP address and port with a `Host` header containing the `<delegated_hostname>`. The target server must present a valid certificate for `<delegated_hostname>`.
    5. If no SRV record is found, an IP address is resolved using CNAME, AAAA or A records. Requests are then made to the resolved IP address and a port of 8448, using a `Host` header of `<delegated_hostname>`. The target server must present a valid certificate for `<delegated_hostname>`.
4. **[Added in `v1.8`]** If the `/.well-known` request resulted in an error response, a server is found by resolving an SRV record for `_matrix-fed._tcp.<hostname>`. This may result in a hostname (to be resolved using AAAA or A records) and port. Requests are made to the resolved IP address and port, with a `Host` header of `<hostname>`. The target server must present a valid certificate for `<hostname>`.
    
5. **[Deprecated]** If the `/.well-known` request resulted in an error response, and a `_matrix-fed._tcp.<hostname>` SRV record was not found, a server is found by resolving an SRV record for `_matrix._tcp.<hostname>`. This may result in a hostname (to be resolved using AAAA or A records) and port. Requests are made to the resolved IP address and port, with a `Host` header of `<hostname>`. The target server must present a valid certificate for `<hostname>`.
    
6. If the `/.well-known` request returned an error response, and no SRV records were found, an IP address is resolved using CNAME, AAAA and A records. Requests are made to the resolved IP address using port 8448 and a `Host` header containing the `<hostname>`. The target server must present a valid certificate for `<hostname>`.
    
> [!info] info:
> The reasons we require `<hostname>` rather than `<delegated_hostname>` for SRV delegation are:
> 
> 1. DNS is insecure (not all domains have DNSSEC), so the target of the delegation must prove that it is a valid delegate for `<hostname>` via TLS.
> 2. Consistency with the recommendations in [RFC6125](https://datatracker.ietf.org/doc/html/rfc6125#section-6.2.1) and other applications using SRV records such [XMPP](https://datatracker.ietf.org/doc/html/rfc6120#section-13.7.2.1).

> [!info] info:
> Note that the target of a SRV record may _not_ be a CNAME, as mandated by [RFC2782](https://www.rfc-editor.org/rfc/rfc2782.html):
> 
> > the name MUST NOT be an alias (in the sense of RFC 1034 or RFC 2181)

> [!info] info:
> Steps 3.4 and 5 are deprecated because they use a service name not registered by IANA. They may be removed in a future version of the specification. Server admins are encouraged to use `.well-known` over any form of SRV records.
> 
> The IANA registration for port 8448 and `matrix-fed` can be found [here](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=matrix-fed).

### GET /.well-known/matrix/server[](https://spec.matrix.org/v1.11/server-server-api/#getwell-knownmatrixserver)

---

Gets information about the delegated server for server-server communication between Matrix homeservers. Servers should follow 30x redirects, carefully avoiding redirect loops, and use normal X.509 certificate validation.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

#### Request

No request parameters or request body.

---

#### Responses

|Status|Description|
|---|---|
|`200`|The delegated server information. The `Content-Type` for this response SHOULD be `application/json`, however servers parsing the response should assume that the body is JSON regardless of type. Failures parsing the JSON or invalid data provided in the resulting parsed JSON should not result in discovery failure - consult the server discovery process for information on how to continue.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`m.server`|`string`|The server name to delegate server-server communications to, with optional port. The delegated server name uses the same grammar as [server names in the appendices](https://spec.matrix.org/v1.11/appendices/#server-name).|

```
{
  "m.server": "delegated.example.com:1234"
}
```

## Server implementation[](https://spec.matrix.org/v1.11/server-server-api/#server-implementation)

### GET /_matrix/federation/v1/version[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1version)

---

Get the implementation name and version of this homeserver.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

#### Request

No request parameters or request body.

---

#### Responses

|Status|Description|
|---|---|
|`200`|The implementation name and version of this homeserver.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`server`|[Server](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1version_response-200_server)||

|Server|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`name`|`string`|Arbitrary name that identify this implementation.|
|`version`|`string`|Version of this implementation. The version format depends on the implementation.|

```
{
  "server": {
    "name": "My_Homeserver_Implementation",
    "version": "ArbitraryVersionNumber"
  }
}
```

## Retrieving server keys[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-server-keys)

> [!info] info:
> There was once a “version 1” of the key exchange. It has been removed from the specification due to lack of significance. It may be reviewed [from the historical draft](https://github.com/matrix-org/matrix-doc/blob/51faf8ed2e4a63d4cfd6d23183698ed169956cc0/specification/server_server_api.rst#232version-1).

Each homeserver publishes its public keys under `/_matrix/key/v2/server`. Homeservers query for keys by either getting `/_matrix/key/v2/server` directly or by querying an intermediate notary server using a `/_matrix/key/v2/query/{serverName}` API. Intermediate notary servers query the `/_matrix/key/v2/server` API on behalf of another server and sign the response with their own key. A server may query multiple notary servers to ensure that they all report the same public keys.

This approach is borrowed from the [Perspectives Project](https://web.archive.org/web/20170702024706/https://perspectives-project.org/), but modified to include the NACL keys and to use JSON instead of XML. It has the advantage of avoiding a single trust-root since each server is free to pick which notary servers they trust and can corroborate the keys returned by a given notary server by querying other servers.

### Publishing Keys[](https://spec.matrix.org/v1.11/server-server-api/#publishing-keys)

Homeservers publish their signing keys in a JSON object at `/_matrix/key/v2/server`. The response contains a list of `verify_keys` that are valid for signing federation requests made by the homeserver and for signing events. It contains a list of `old_verify_keys` which are only valid for signing events.

#### GET /_matrix/key/v2/server[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server)

---

Gets the homeserver’s published signing keys. The homeserver may have any number of active keys and may have a number of old keys.

Intermediate notary servers should cache a response for half of its lifetime to avoid serving a stale response. Originating servers should avoid returning responses that expire in less than an hour to avoid repeated requests for a certificate that is about to expire. Requesting servers should limit how frequently they query for certificates to avoid flooding a server with requests.

If the server fails to respond to this request, intermediate notary servers should continue to return the last response they received from the server so that the signatures of old events can still be checked.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

##### Request

No request parameters or request body.

---

##### Responses

|Status|Description|
|---|---|
|`200`|The homeserver’s keys|

###### 200 response

|Server Keys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server_response-200_old-verify-key)}|The public keys that the server used to use and when it stopped using them.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `0ldK3y` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|
|`server_name`|`string`|**Required:** DNS name of the homeserver.|
|`signatures`|`{string: {string: string}}`|Digital signatures for this object signed using the `verify_keys`.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`valid_until_ts`|`integer`|POSIX timestamp in milliseconds when the list of valid keys should be refreshed. This field MUST be ignored in room versions 1, 2, 3, and 4. Keys used beyond this timestamp MUST be considered invalid, depending on the [room version specification](https://spec.matrix.org/v1.11/rooms).<br><br>Servers MUST use the lesser of this field and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server_response-200_verify-key)}|**Required:**<br><br>Public keys of the homeserver for verifying digital signatures.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `abc123` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|

|Old Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`expired_ts`|`integer`|**Required:** POSIX timestamp in milliseconds for when this key expired.|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

|Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

```
{
  "old_verify_keys": {
    "ed25519:0ldk3y": {
      "expired_ts": 1532645052628,
      "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
    }
  },
  "server_name": "example.org",
  "signatures": {
    "example.org": {
      "ed25519:auto2": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
    }
  },
  "valid_until_ts": 1652262000000,
  "verify_keys": {
    "ed25519:abc123": {
      "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
    }
  }
}
```

### Querying Keys Through Another Server[](https://spec.matrix.org/v1.11/server-server-api/#querying-keys-through-another-server)

Servers may query another server’s keys through a notary server. The notary server may be another homeserver. The notary server will retrieve keys from the queried servers through use of the `/_matrix/key/v2/server` API. The notary server will additionally sign the response from the queried server before returning the results.

Notary servers can return keys for servers that are offline or having issues serving their own keys by using cached responses. Keys can be queried from multiple servers to mitigate against DNS spoofing.

#### POST /_matrix/key/v2/query[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query)

---

Query for keys from multiple servers in a batch format. The receiving (notary) server must sign the keys returned by the queried servers.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

##### Request

###### Request body

|Name|Type|Description|
|---|---|---|
|`server_keys`|{string: {string: [Query Criteria](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_request_query-criteria)}}|**Required:**<br><br>The query criteria. The outer `string` key on the object is the server name (eg: `matrix.org`). The inner `string` key is the Key ID to query for the particular server. If no key IDs are given to be queried, the notary server should query for all keys. If no servers are given, the notary server must return an empty `server_keys` array in the response.<br><br>The notary server may return multiple keys regardless of the Key IDs given.|

|Query Criteria|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`minimum_valid_until_ts`|`integer`|A millisecond POSIX timestamp in milliseconds indicating when the returned certificates will need to be valid until to be useful to the requesting server.<br><br>If not supplied, the current time as determined by the notary server is used.|

###### Request body example

```
{
  "server_keys": {
    "example.org": {
      "ed25519:abc123": {
        "minimum_valid_until_ts": 1234567890
      }
    }
  }
}
```

---

##### Responses

|Status|Description|
|---|---|
|`200`|The keys for the queried servers, signed by the notary server. Servers which are offline and have no cached keys will not be included in the result. This may result in an empty array.|

###### 200 response

|Name|Type|Description|
|---|---|---|
|`server_keys`|[[Server Keys](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_server-keys)]|The queried server’s keys, signed by the notary server.|

|Server Keys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_old-verify-key)}|The public keys that the server used to use and when it stopped using them.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `0ldK3y` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|
|`server_name`|`string`|**Required:** DNS name of the homeserver.|
|`signatures`|`{string: {string: string}}`|Digital signatures for this object signed using the `verify_keys`.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`valid_until_ts`|`integer`|POSIX timestamp in milliseconds when the list of valid keys should be refreshed. This field MUST be ignored in room versions 1, 2, 3, and 4. Keys used beyond this timestamp MUST be considered invalid, depending on the [room version specification](https://spec.matrix.org/v1.11/rooms).<br><br>Servers MUST use the lesser of this field and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_verify-key)}|**Required:**<br><br>Public keys of the homeserver for verifying digital signatures.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `abc123` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|

|Old Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`expired_ts`|`integer`|**Required:** POSIX timestamp in milliseconds for when this key expired.|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

|Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

```
{
  "server_keys": [
    {
      "old_verify_keys": {
        "ed25519:0ldk3y": {
          "expired_ts": 1532645052628,
          "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
        }
      },
      "server_name": "example.org",
      "signatures": {
        "example.org": {
          "ed25519:abc123": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
        },
        "notary.server.com": {
          "ed25519:010203": "VGhpcyBpcyBhbm90aGVyIHNpZ25hdHVyZQ"
        }
      },
      "valid_until_ts": 1652262000000,
      "verify_keys": {
        "ed25519:abc123": {
          "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
        }
      }
    }
  ]
}
```

#### GET /_matrix/key/v2/query/{serverName}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername)

---

Query for another server’s keys. The receiving (notary) server must sign the keys returned by the queried server.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

##### Request

###### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`serverName`|`string`|**Required:** The server’s DNS name to query|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`minimum_valid_until_ts`|`integer`|A millisecond POSIX timestamp in milliseconds indicating when the returned certificates will need to be valid until to be useful to the requesting server.<br><br>If not supplied, the current time as determined by the notary server is used.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The keys for the server, or an empty array if the server could not be reached and no cached keys were available.|

###### 200 response

|Name|Type|Description|
|---|---|---|
|`server_keys`|[[Server Keys](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_server-keys)]|The queried server’s keys, signed by the notary server.|

|Server Keys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_old-verify-key)}|The public keys that the server used to use and when it stopped using them.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `0ldK3y` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|
|`server_name`|`string`|**Required:** DNS name of the homeserver.|
|`signatures`|`{string: {string: string}}`|Digital signatures for this object signed using the `verify_keys`.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`valid_until_ts`|`integer`|POSIX timestamp in milliseconds when the list of valid keys should be refreshed. This field MUST be ignored in room versions 1, 2, 3, and 4. Keys used beyond this timestamp MUST be considered invalid, depending on the [room version specification](https://spec.matrix.org/v1.11/rooms).<br><br>Servers MUST use the lesser of this field and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_verify-key)}|**Required:**<br><br>Public keys of the homeserver for verifying digital signatures.<br><br>The object’s key is the algorithm and version combined (`ed25519` being the algorithm and `abc123` being the version in the example below). Together, this forms the Key ID. The version must have characters matching the regular expression `[a-zA-Z0-9_]`.|

|Old Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`expired_ts`|`integer`|**Required:** POSIX timestamp in milliseconds for when this key expired.|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

|Verify Key|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`key`|`string`|**Required:** The [Unpadded base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) encoded key.|

```
{
  "server_keys": [
    {
      "old_verify_keys": {
        "ed25519:0ldk3y": {
          "expired_ts": 1532645052628,
          "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
        }
      },
      "server_name": "example.org",
      "signatures": {
        "example.org": {
          "ed25519:abc123": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
        },
        "notary.server.com": {
          "ed25519:010203": "VGhpcyBpcyBhbm90aGVyIHNpZ25hdHVyZQ"
        }
      },
      "valid_until_ts": 1652262000000,
      "verify_keys": {
        "ed25519:abc123": {
          "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
        }
      }
    }
  ]
}
```

# Authentication[](https://spec.matrix.org/v1.11/server-server-api/#authentication)

## Request Authentication[](https://spec.matrix.org/v1.11/server-server-api/#request-authentication)

Every HTTP request made by a homeserver is authenticated using public key digital signatures. The request method, target and body are signed by wrapping them in a JSON object and signing it using the JSON signing algorithm. The resulting signatures are added as an Authorization header with an auth scheme of `X-Matrix`. Note that the target field should include the full path starting with `/_matrix/...`, including the `?` and any query parameters if present, but should not include the leading `https:`, nor the destination server’s hostname.

Step 1 sign JSON:

```
{
    "method": "POST",
    "uri": "/target",
    "origin": "origin.hs.example.com",
    "destination": "destination.hs.example.com",
    "content": <JSON-parsed request body>,
    "signatures": {
        "origin.hs.example.com": {
            "ed25519:key1": "ABCDEF..."
        }
    }
}
```

The server names in the JSON above are the server names for each homeserver involved. Delegation from the [server name resolution section](https://spec.matrix.org/v1.11/server-server-api/#resolving-server-names) above do not affect these - the server names from before delegation would take place are used. This same condition applies throughout the request signing process.

Step 2 add Authorization header:

```
POST /target HTTP/1.1
Authorization: X-Matrix origin="origin.hs.example.com",destination="destination.hs.example.com",key="ed25519:key1",sig="ABCDEF..."
Content-Type: application/json

<JSON-encoded request body>
```

Example python code:

```python
def authorization_headers(origin_name, origin_signing_key,
                          destination_name, request_method, request_target,
                          content=None):
    request_json = {
         "method": request_method,
         "uri": request_target,
         "origin": origin_name,
         "destination": destination_name,
    }

    if content is not None:
        # Assuming content is already parsed as JSON
        request_json["content"] = content

    signed_json = sign_json(request_json, origin_name, origin_signing_key)

    authorization_headers = []

    for key, sig in signed_json["signatures"][origin_name].items():
        authorization_headers.append(bytes(
            "X-Matrix origin=\"%s\",destination=\"%s\",key=\"%s\",sig=\"%s\"" % (
                origin_name, destination_name, key, sig,
            )
        ))

    return ("Authorization", authorization_headers[0])

```

The format of the Authorization header is given in [Section 11.4 of RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110#section-11.4). In summary, the header begins with authorization scheme `X-Matrix`, followed by one or more spaces, followed by a comma-separated list of parameters written as name=value pairs. Zero or more spaces and tabs around each comma are allowed. The names are case insensitive and order does not matter. The values must be enclosed in quotes if they contain characters that are not allowed in `token`s, as defined in [Section 5.6.2 of RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110#section-5.6.2); if a value is a valid `token`, it may or may not be enclosed in quotes. Quoted values may include backslash-escaped characters. When parsing the header, the recipient must unescape the characters. That is, a backslash-character pair is replaced by the character that follows the backslash.

For compatibility with older servers, the sender should

- only include one space after `X-Matrix`,
- only use lower-case names,
- avoid using backslashes in parameter values, and
- avoid including whitespace around the commas between name=value pairs.

For compatibility with older servers, the recipient should allow colons to be included in values without requiring the value to be enclosed in quotes.

The authorization parameters to include are:

- `origin`: the server name of the sending server. This is the same as the `origin` field from JSON described in step 1.
- `destination`: **[Added in `v1.3`]** the server name of the receiving server. This is the same as the `destination` field from the JSON described in step 1. For compatibility with older servers, recipients should accept requests without this parameter, but MUST always send it. If this property is included, but the value does not match the receiving server’s name, the receiving server must deny the request with an HTTP status code 401 Unauthorized.
- `key`: the ID, including the algorithm name, of the sending server’s key used to sign the request.
- `signature`: the signature of the JSON as calculated in step 1.

Unknown parameters are ignored.


> [!info] info:
> **[Changed in `v1.11`]** This section used to reference [RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235#section-2.1) and [RFC 7230](https://datatracker.ietf.org/doc/html/rfc9110#section-5.6.2), that were obsoleted by RFC 9110 without changes to the sections of interest here.

## Response Authentication[](https://spec.matrix.org/v1.11/server-server-api/#response-authentication)

Responses are authenticated by the TLS server certificate. A homeserver should not send a request until it has authenticated the connected server to avoid leaking messages to eavesdroppers.

## Client TLS Certificates[](https://spec.matrix.org/v1.11/server-server-api/#client-tls-certificates)

Requests are authenticated at the HTTP layer rather than at the TLS layer because HTTP services like Matrix are often deployed behind load balancers that handle the TLS and these load balancers make it difficult to check TLS client certificates.

A homeserver may provide a TLS client certificate and the receiving homeserver may check that the client certificate matches the certificate of the origin homeserver.

# Transactions[](https://spec.matrix.org/v1.11/server-server-api/#transactions)

The transfer of EDUs and PDUs between homeservers is performed by an exchange of Transaction messages, which are encoded as JSON objects, passed over an HTTP PUT request. A Transaction is meaningful only to the pair of homeservers that exchanged it; they are not globally-meaningful.

Transactions are limited in size; they can have at most 50 PDUs and 100 EDUs.

### PUT /_matrix/federation/v1/send/{txnId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid)

---

Push messages representing live activity to another server. The destination name will be set to that of the receiving server itself. Each embedded PDU in the transaction body will be processed.

The sending server must wait and retry for a 200 OK response before sending a transaction with a different `txnId` to the receiving server.

Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`txnId`|`string`|**Required:** The transaction ID.|

##### Request body

|Transaction|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`edus`|[[Ephemeral Data Unit](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid_request_ephemeral-data-unit)]|List of ephemeral messages. May be omitted if there are no ephemeral messages to be sent. Must not include more than 100 EDUs.|
|`origin`|`string`|**Required:** The `server_name` of the homeserver sending this transaction.|
|`origin_server_ts`|`integer`|**Required:** POSIX timestamp in milliseconds on originating homeserver when this transaction started.|
|`pdus`|`[PDU]`|**Required:** List of persistent updates to rooms. Must not include more than 50 PDUs. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

|Ephemeral Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`object`|**Required:** The content of the ephemeral message.|
|`edu_type`|`string`|**Required:** The type of ephemeral message.|

##### Request body example

```
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The result of processing the transaction. The server is to use this response even in the event of one or more PDUs failing to be processed.|

##### 200 response

|PDU Processing Results|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`pdus`|{[Event ID](https://spec.matrix.org/v1.11/appendices#event-ids): [PDU Processing Result](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid_response-200_pdu-processing-result)}|**Required:** The PDUs from the original transaction. The string key represents the ID of the PDU (event) that was processed.|

|PDU Processing Result|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`error`|`string`|A human readable description about what went wrong in processing this PDU. If no error is present, the PDU can be considered successfully handled.|

```
{
  "pdus": {
    "$failed_event:example.org": {
      "error": "You are not allowed to send a message to this room."
    },
    "$successful_event:example.org": {}
  }
}
```

# PDUs[](https://spec.matrix.org/v1.11/server-server-api/#pdus)

Each PDU contains a single Room Event which the origin server wants to send to the destination.

The `prev_events` field of a PDU identifies the “parents” of the event, and thus establishes a partial ordering on events within the room by linking them into a Directed Acyclic Graph (DAG). The sending server should populate this field with all of the events in the room for which it has not yet seen a child - thus demonstrating that the event comes after all other known events.

For example, consider a room whose events form the DAG shown below. A server creating a new event in this room should populate the new event’s `prev_events` field with both `E4` and `E6`, since neither event yet has a child:

```
E1
^
|
E2 <--- E5
^       ^

|       |
E3      E6
^

|
E4
```

For a full schema of what a PDU looks like, see the [room version specification](https://spec.matrix.org/v1.11/rooms).

## Checks performed on receipt of a PDU[](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu)

Whenever a server receives an event from a remote server, the receiving server must ensure that the event:

1. Is a valid event, otherwise it is dropped. For an event to be valid, it must contain a `room_id`, and it must comply with the event format of that [room version](https://spec.matrix.org/v1.11/rooms).
2. Passes signature checks, otherwise it is dropped.
3. Passes hash checks, otherwise it is redacted before being processed further.
4. Passes authorization rules based on the event’s auth events, otherwise it is rejected.
5. Passes authorization rules based on the state before the event, otherwise it is rejected.
6. Passes authorization rules based on the current state of the room, otherwise it is “soft failed”.

Further details of these checks, and how to handle failures, are described below.

The [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events) section has more information on which hashes and signatures are expected on events, and how to calculate them.

### Definitions[](https://spec.matrix.org/v1.11/server-server-api/#definitions)

Required Power Level

A given event type has an associated _required power level_. This is given by the current [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api/#mroompower_levels) event. The event type is either listed explicitly in the `events` section or given by either `state_default` or `events_default` depending on if the event is a state event or not.

Invite Level, Kick Level, Ban Level, Redact Level

The levels given by the `invite`, `kick`, `ban`, and `redact` properties in the current [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api/#mroompower_levels) state. The invite level defaults to 0 if unspecified. The kick level, ban level and redact level each default to 50 if unspecified.

Target User

For an [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api/#mroommember) state event, the user given by the `state_key` of the event.

> [!warning] warning:
> 
> Some [room versions](https://spec.matrix.org/v1.11/rooms) accept power level values to be represented as strings rather than integers. This is strictly for backwards compatibility. A homeserver should take reasonable precautions to prevent users from sending new power level events with string values (eg: by rejecting the API request), and must never populate the default power levels in a room as string values.
> 
> See the [room version specification](https://spec.matrix.org/v1.11/rooms) for more information.

### Authorization rules[](https://spec.matrix.org/v1.11/server-server-api/#authorization-rules)

The rules governing whether an event is authorized depends on a set of state. A given event is checked multiple times against different sets of state, as specified above. Each room version can have a different algorithm for how the rules work, and which rules are applied. For more detailed information, please see the [room version specification](https://spec.matrix.org/v1.11/rooms).

#### Auth events selection[](https://spec.matrix.org/v1.11/server-server-api/#auth-events-selection)

The `auth_events` field of a PDU identifies the set of events which give the sender permission to send the event. The `auth_events` for the `m.room.create` event in a room is empty; for other events, it should be the following subset of the room state:

- The `m.room.create` event.
    
- The current `m.room.power_levels` event, if any.
    
- The sender’s current `m.room.member` event, if any.
    
- If type is `m.room.member`:
    
    - The target’s current `m.room.member` event, if any.
    - If `membership` is `join` or `invite`, the current `m.room.join_rules` event, if any.
    - If membership is `invite` and `content` contains a `third_party_invite` property, the current `m.room.third_party_invite` event with `state_key` matching `content.third_party_invite.signed.token`, if any.
    - If `content.join_authorised_via_users_server` is present, and the [room version supports restricted rooms](https://spec.matrix.org/v1.11/rooms/#feature-matrix), then the `m.room.member` event with `state_key` matching `content.join_authorised_via_users_server`.

### Rejection[](https://spec.matrix.org/v1.11/server-server-api/#rejection)

If an event is rejected it should neither be relayed to clients nor be included as a prev event in any new events generated by the server. Subsequent events from other servers that reference rejected events should be allowed if they still pass the auth rules. The state used in the checks should be calculated as normal, except not updating with the rejected event where it is a state event.

If an event in an incoming transaction is rejected, this should not cause the transaction request to be responded to with an error response.

> [!info] info:
> This means that events may be included in the room DAG even though they should be rejected.

> [!info] info:
> This is in contrast to redacted events which can still affect the state of the room. For example, a redacted `join` event will still result in the user being considered joined.

### Soft failure[](https://spec.matrix.org/v1.11/server-server-api/#soft-failure)

> [!info] RATIONALE:
> 
> It is important that we prevent users from evading bans (or other power restrictions) by creating events which reference old parts of the DAG. For example, a banned user could continue to send messages to a room by having their server send events which reference the event before they were banned. Note that such events are entirely valid, and we cannot simply reject them, as it is impossible to distinguish such an event from a legitimate one which has been delayed. We must therefore accept such events and let them participate in state resolution and the federation protocol as normal. However, servers may choose not to send such events on to their clients, so that end users won’t actually see the events.
> 
> When this happens it is often fairly obvious to servers, as they can see that the new event doesn’t actually pass auth based on the “current state” (i.e. the resolved state across all forward extremities). While the event is technically valid, the server can choose to not notify clients about the new event.
> 
> This discourages servers from sending events that evade bans etc. in this way, as end users won’t actually see the events.
> 
When the homeserver receives a new event over federation it should also check whether the event passes auth checks based on the current state of the room (as well as based on the state at the event). If the event does not pass the auth checks based on the _current state_ of the room (but does pass the auth checks based on the state at that event) it should be “soft failed”.

When an event is “soft failed” it should not be relayed to the client nor be referenced by new events created by the homeserver (i.e. they should not be added to the server’s list of forward extremities of the room). Soft failed events are otherwise handled as usual.

> [!info] info:
> Soft failed events participate in state resolution as normal if further events are received which reference it. It is the job of the state resolution algorithm to ensure that malicious events cannot be injected into the room state via this mechanism.

> [!info] info:
> Because soft failed state events participate in state resolution as normal, it is possible for such events to appear in the current state of the room. In that case the client should be told about the soft failed event in the usual way (e.g. by sending it down in the `state` section of a sync response).

> [!info] info:
> A soft failed event should be returned in response to federation requests where appropriate (e.g. in `/event/<event_id>`). Note that soft failed events are returned in `/backfill` and `/get_missing_events` responses only if the requests include events referencing the soft failed events.

Example

As an example consider the event graph:

```
  A
 /
B
```

where `B` is a ban of a user `X`. If the user `X` tries to set the topic by sending an event `C` while evading the ban:

```
  A
 / \
B   C
```

servers that receive `C` after `B` should soft fail event `C`, and so will neither relay `C` to its clients nor send any events referencing `C`.

If later another server sends an event `D` that references both `B` and `C` (this can happen if it received `C` before `B`):

```
  A
 / \
B   C
 \ /
  D
```

then servers will handle `D` as normal. `D` is sent to the servers' clients (assuming `D` passes auth checks). The state at `D` may resolve to a state that includes `C`, in which case clients should also to be told that the state has changed to include `C`. (_Note_: This depends on the exact state resolution algorithm used. In the original version of the algorithm `C` would be in the resolved state, whereas in latter versions the algorithm tries to prioritise the ban over the topic change.)

Note that this is essentially equivalent to the situation where one server doesn’t receive `C` at all, and so asks another server for the state of the `C` branch.

Let’s go back to the graph before `D` was sent:

```
  A
 / \
B   C
```

If all the servers in the room saw `B` before `C` and so soft fail `C`, then any new event `D'` will not reference `C`:

```
  A
 / \
B   C
|
D'
```

### Retrieving event authorization information[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-event-authorization-information)

The homeserver may be missing event authorization information, or wish to check with other servers to ensure it is receiving the correct auth chain. These APIs give the homeserver an avenue for getting the information it needs.

#### GET /_matrix/federation/v1/event_auth/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1event_authroomideventid)

---

Retrieves the complete auth chain for a given event.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID to get the auth chain of.|
|`roomId`|`string`|**Required:** The room ID to get the auth chain for.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The auth chain for the event.|

###### 200 response

|Name|Type|Description|
|---|---|---|
|`auth_chain`|`[PDU]`|**Required:** The [PDUs](https://spec.matrix.org/v1.11/server-server-api/#pdus) forming the auth chain of the given event. The event format varies depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

# EDUs[](https://spec.matrix.org/v1.11/server-server-api/#edus)

EDUs, by comparison to PDUs, do not have an ID, a room ID, or a list of “previous” IDs. They are intended to be non-persistent data such as user presence, typing notifications, etc.

## `Ephemeral Data Unit`

---

An ephemeral data unit.

|Ephemeral Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`object`|**Required:** The content of the ephemeral message.|
|`edu_type`|`string`|**Required:** The type of ephemeral message.|

### Examples

```
{
  "content": {
    "key": "value"
  },
  "edu_type": "m.presence"
}
```

# Room State Resolution[](https://spec.matrix.org/v1.11/server-server-api/#room-state-resolution)

The _state_ of a room is a map of `(event_type, state_key)` to `event_id`. Each room starts with an empty state, and each state event which is accepted into the room updates the state of that room.

Where each event has a single `prev_event`, it is clear what the state of the room after each event should be. However, when two branches in the event graph merge, the state of those branches might differ, so a _state resolution_ algorithm must be used to determine the resultant state.

For example, consider the following event graph (where the oldest event, E0, is at the top):

```
  E0
  |
  E1
 /  \
E2  E4

|    |
E3   |
 \  /
  E5
```

Suppose E3 and E4 are both `m.room.name` events which set the name of the room. What should the name of the room be at E5?

The algorithm to be used for state resolution depends on the room version. For a description of each room version’s algorithm, please see the [room version specification](https://spec.matrix.org/v1.11/rooms).

# Backfilling and retrieving missing events[](https://spec.matrix.org/v1.11/server-server-api/#backfilling-and-retrieving-missing-events)

Once a homeserver has joined a room, it receives all the events emitted by other homeservers in that room, and is thus aware of the entire history of the room from that moment onwards. Since users in that room are able to request the history by the `/messages` client API endpoint, it’s possible that they might step backwards far enough into history before the homeserver itself was a member of that room.

To cover this case, the federation API provides a server-to-server analog of the `/messages` client API, allowing one homeserver to fetch history from another. This is the `/backfill` API.

To request more history, the requesting homeserver picks another homeserver that it thinks may have more (most likely this should be a homeserver for some of the existing users in the room at the earliest point in history it has currently), and makes a `/backfill` request.

Similar to backfilling a room’s history, a server may not have all the events in the graph. That server may use the `/get_missing_events` API to acquire the events it is missing.

### GET /_matrix/federation/v1/backfill/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1backfillroomid)

---

Retrieves a sliding-window history of previous PDUs that occurred in the given room. Starting from the PDU ID(s) given in the `v` argument, the PDUs given in `v` and the PDUs that preceded them are retrieved, up to the total number given by the `limit`.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID to backfill.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`limit`|`integer`|**Required:** The maximum number of PDUs to retrieve, including the given events.|
|`v`|`[string]`|**Required:** The event IDs to backfill from.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A transaction containing the PDUs that preceded the given event(s), including the given event(s), up to the given limit.<br><br>**Note:** Though the PDU definitions require that `prev_events` and `auth_events` be limited in number, the response of backfill MUST NOT be validated on these specific restrictions.<br><br>Due to historical reasons, it is possible that events which were previously accepted would now be rejected by these limitations. The events should be rejected per usual by the `/send`, `/get_missing_events`, and remaining endpoints.|

##### 200 response

|Transaction|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`origin`|`string`|**Required:** The `server_name` of the homeserver sending this transaction.|
|`origin_server_ts`|`integer`|**Required:** POSIX timestamp in milliseconds on originating homeserver when this transaction started.|
|`pdus`|`[PDU]`|**Required:** List of persistent updates to rooms. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### POST /_matrix/federation/v1/get_missing_events/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1get_missing_eventsroomid)

---

Retrieves previous events that the sender is missing. This is done by doing a breadth-first walk of the `prev_events` for the `latest_events`, ignoring any events in `earliest_events` and stopping at the `limit`.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID to search in.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`earliest_events`|`[string]`|**Required:** The latest event IDs that the sender already has. These are skipped when retrieving the previous events of `latest_events`.|
|`latest_events`|`[string]`|**Required:** The event IDs to retrieve the previous events for.|
|`limit`|`integer`|The maximum number of events to retrieve. Defaults to 10.|
|`min_depth`|`integer`|The minimum depth of events to retrieve. Defaults to 0.|

##### Request body example

```
{
  "earliest_events": [
    "$missing_event:example.org"
  ],
  "latest_events": [
    "$event_that_has_the_missing_event_as_a_previous_event:example.org"
  ],
  "limit": 10
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The previous events for `latest_events`, excluding any `earliest_events`, up to the provided `limit`.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`events`|`[PDU]`|**Required:** The missing events. The event format varies depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
{
  "events": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

# Retrieving events[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-events)

In some circumstances, a homeserver may be missing a particular event or information about the room which cannot be easily determined from backfilling. These APIs provide homeservers with the option of getting events and the state of the room at a given point in the timeline.

### GET /_matrix/federation/v1/event/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1eventeventid)

---

Retrieves a single event.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID to get.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A transaction containing a single PDU which is the event requested.|

##### 200 response

|Transaction|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`origin`|`string`|**Required:** The `server_name` of the homeserver sending this transaction.|
|`origin_server_ts`|`integer`|**Required:** POSIX timestamp in milliseconds on originating homeserver when this transaction started.|
|`pdus`|`[PDU]`|**Required:** A single PDU. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### GET /_matrix/federation/v1/state/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1stateroomid)

---

Retrieves a snapshot of a room’s state at a given event.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID to get state for.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`event_id`|`string`|**Required:** An event ID in the room to retrieve the state at.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The fully resolved state for the room, prior to considering any state changes induced by the requested event. Includes the authorization chain for the events.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`auth_chain`|`[PDU]`|**Required:** The full set of authorization events that make up the state of the room, and their authorization events, recursively. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`pdus`|`[PDU]`|**Required:** The fully resolved state of the room at the given event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ],
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### GET /_matrix/federation/v1/state_ids/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1state_idsroomid)

---

Retrieves a snapshot of a room’s state at a given event, in the form of event IDs. This performs the same function as calling `/state/{roomId}`, however this returns just the event IDs rather than the full events.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID to get state for.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`event_id`|`string`|**Required:** An event ID in the room to retrieve the state at.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The fully resolved state for the room, prior to considering any state changes induced by the requested event. Includes the authorization chain for the events.|
|`404`|The given `event_id` was not found or the server doesn’t know about the state at that event to return anything useful.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`auth_chain_ids`|`[string]`|**Required:** The full set of authorization events that make up the state of the room, and their authorization events, recursively.|
|`pdu_ids`|`[string]`|**Required:** The fully resolved state of the room at the given event.|

```
{
  "auth_chain_ids": [
    "$an_event:example.org"
  ],
  "pdu_ids": [
    "$an_event:example.org"
  ]
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
  "errcode": "M_NOT_FOUND",
  "error": "Could not find event $Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg"
}
```

### GET /_matrix/federation/v1/timestamp_to_event/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1timestamp_to_eventroomid)

---

**Added in `v1.6`**

Get the ID of the event closest to the given timestamp, in the direction specified by the `dir` parameter.

This is primarily used when handling the corresponding [client-server endpoint](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1roomsroomidtimestamp_to_event) when the server does not have all of the room history, and does not have an event suitably close to the requested timestamp.

The heuristics for deciding when to ask another homeserver for a closer event if your homeserver doesn’t have something close, are left up to the homeserver implementation, although the heuristics will probably be based on whether the closest event is a forward/backward extremity indicating it’s next to a gap of events which are potentially closer.

A good heuristic for which servers to try first is to sort by servers that have been in the room the longest because they’re most likely to have anything we ask about.

After the local homeserver receives the response, it should determine, using the `origin_server_ts` property, whether the returned event is closer to the requested timestamp than the closest event that it could find locally. If so, it should try to backfill this event via the [`/event/{event_id}`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1eventeventid) endpoint so that it is available to for a client to query.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The ID of the room to search|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`dir`|`string`|**Required:** The direction in which to search. `f` for forwards, `b` for backwards.<br><br>One of: `[f, b]`.|
|`ts`|`integer`|**Required:** The timestamp to search from, as given in milliseconds since the Unix epoch.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|An event was found matching the search parameters.|
|`404`|No event was found.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`event_id`|`string`|**Required:** The ID of the event found|
|`origin_server_ts`|`integer`|**Required:** The event’s timestamp, in milliseconds since the Unix epoch.|

```
{
  "event_id": "$143273582443PhrSn:example.org",
  "origin_server_ts": 1432735824653
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
  "errcode": "M_NOT_FOUND",
  "error": "Unable to find event from 1432684800000 in forward direction"
}
```

# Joining Rooms[](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms)

When a new user wishes to join a room that the user’s homeserver already knows about, the homeserver can immediately determine if this is allowable by inspecting the state of the room. If it is acceptable, it can generate, sign, and emit a new `m.room.member` state event adding the user into that room. When the homeserver does not yet know about the room it cannot do this directly. Instead, it must take a longer multi-stage handshaking process by which it first selects a remote homeserver which is already participating in that room, and use it to assist in the joining process. This is the remote join handshake.

This handshake involves the homeserver of the new member wishing to join (referred to here as the “joining” server), the directory server hosting the room alias the user is requesting to join with, and a homeserver where existing room members are already present (referred to as the “resident” server).

In summary, the remote join handshake consists of the joining server querying the directory server for information about the room alias; receiving a room ID and a list of join candidates. The joining server then requests information about the room from one of the residents. It uses this information to construct an `m.room.member` event which it finally sends to a resident server.

Conceptually these are three different roles of homeserver. In practice the directory server is likely to be resident in the room, and so may be selected by the joining server to be the assisting resident. Likewise, it is likely that the joining server picks the same candidate resident for both phases of event construction, though in principle any valid candidate may be used at each time. Thus, any join handshake can potentially involve anywhere from two to four homeservers, though most in practice will use just two.

```
+---------+          +---------------+            +-----------------+ +-----------------+
| Client  |          | JoiningServer |            | DirectoryServer | | ResidentServer  |
+---------+          +---------------+            +-----------------+ +-----------------+
     |                       |                             |                   |
     | join request          |                             |                   |
     |---------------------->|                             |                   |
     |                       |                             |                   |
     |                       | directory request           |                   |
     |                       |---------------------------->|                   |
     |                       |                             |                   |
     |                       |          directory response |                   |
     |                       |<----------------------------|                   |
     |                       |                             |                   |
     |                       | make_join request           |                   |
     |                       |------------------------------------------------>|
     |                       |                             |                   |
     |                       |                             |make_join response |
     |                       |<------------------------------------------------|
     |                       |                             |                   |
     |                       | send_join request           |                   |
     |                       |------------------------------------------------>|
     |                       |                             |                   |
     |                       |                             |send_join response |
     |                       |<------------------------------------------------|
     |                       |                             |                   |
     |         join response |                             |                   |
     |<----------------------|                             |                   |
     |                       |                             |                   |
```

The first part of the handshake usually involves using the directory server to request the room ID and join candidates through the [`/query/directory`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1querydirectory) API endpoint. In the case of a new user joining a room as a result of a received invite, the joining user’s homeserver could optimise this step away by picking the origin server of that invite message as the join candidate. However, the joining server should be aware that the origin server of the invite might since have left the room, so should be prepared to fall back on the regular join flow if this optimisation fails.

Once the joining server has the room ID and the join candidates, it then needs to obtain enough information about the room to fill in the required fields of the `m.room.member` event. It obtains this by selecting a resident from the candidate list, and using the `GET /make_join` endpoint. The resident server will then reply with enough information for the joining server to fill in the event.

The joining server is expected to add or replace the `origin`, `origin_server_ts`, and `event_id` on the templated event received by the resident server. This event is then signed by the joining server.

To complete the join handshake, the joining server submits this new event to the resident server it used for `GET /make_join`, using the `PUT /send_join` endpoint.

The resident homeserver then adds its signature to this event and accepts it into the room’s event graph. The joining server receives the full set of state for the newly-joined room as well as the freshly signed membership event. The resident server must also send the event to other servers participating in the room.

### GET /_matrix/federation/v1/make_join/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid)

---

Asks the receiving server to return information that the sending server will need to prepare a join event to get into the room.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID that is about to be joined.|
|`userId`|`string`|**Required:** The user ID the join event will be for.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`ver`|`[string]`|The room versions the sending server has support for. Defaults to `[1]`.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A template to be used for the rest of the [Joining Rooms](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms) handshake. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The response body here describes the common event fields in more detail and may be missing other required fields for a PDU.**|
|`400`|The request is invalid, the room the server is attempting to join has a version that is not listed in the `ver` parameters, or the server was unable to validate [restricted room conditions](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms).<br><br>The error should be passed through to clients so that they may give better feedback to users.<br><br>New in `v1.2`, the following error conditions might happen:<br><br>If the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and none of the conditions can be validated by the server then the `errcode` `M_UNABLE_TO_AUTHORISE_JOIN` must be used. This can happen if the server does not know about any of the rooms listed as conditions, for example.<br><br>`M_UNABLE_TO_GRANT_JOIN` is returned to denote that a different server should be attempted for the join. This is typically because the resident server can see that the joining user satisfies one or more conditions, such as in the case of [restricted rooms](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms), but the resident server would be unable to meet the auth rules governing `join_authorised_via_users_server` on the resulting `m.room.member` event.|
|`403`|The room that the joining server is attempting to join does not permit the user to join.|
|`404`|The room that the joining server is attempting to join is unknown to the receiving server.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`event`|[Event Template](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid_response-200_event-template)|An unsigned template event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`room_version`|`string`|The version of the room where the server is trying to join. If not provided, the room version is assumed to be either “1” or “2”.|

|Event Template|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid_response-200_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the resident homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the resident homeserver.|
|`sender`|`string`|**Required:** The user ID of the joining member.|
|`state_key`|`string`|**Required:** The user ID of the joining member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`join_authorised_via_users_server`|`string`|Required if the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and is joining through one of the conditions available. If the user is responding to an invite, this is not required.<br><br>An arbitrary user ID belonging to the resident server in the room being joined that is able to issue invites to other users. This is used in later validation of the auth rules for the `m.room.member` event.<br><br>**Added in `v1.2`**|
|`membership`|`string`|**Required:** The value `join`.|

```
{
  "event": {
    "content": {
      "join_authorised_via_users_server": "@anyone:resident.example.org",
      "membership": "join"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "2"
}
```

##### 400 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|
|`room_version`|`string`|The version of the room. Required if the `errcode` is `M_INCOMPATIBLE_ROOM_VERSION`.|

```
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to join this room",
  "room_version": "3"
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not invited to this room"
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
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```

### PUT /_matrix/federation/v1/send_join/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_joinroomideventid)

---

> [!warning] warning:
> This API is deprecated and will be removed from a future release.

**Note:** Servers should instead prefer to use the v2 `/send_join` endpoint.

Submits a signed join event to the resident server for it to accept it into the room’s graph. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response body here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the join event.|
|`roomId`|`string`|**Required:** The room ID that is about to be joined.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_joinroomideventid_request_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the joining homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the joining homeserver.|
|`sender`|`string`|**Required:** The user ID of the joining member.|
|`state_key`|`string`|**Required:** The user ID of the joining member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`join_authorised_via_users_server`|`string`|Required if the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and is joining through one of the conditions available. If the user is responding to an invite, this is not required.<br><br>An arbitrary user ID belonging to the resident server in the room being joined that is able to issue invites to other users. This is used in later validation of the auth rules for the `m.room.member` event.<br><br>The resident server which owns the provided user ID must have a valid signature on the event. If the resident server is receiving the `/send_join` request, the signature must be added before sending or persisting the event to other servers.<br><br>**Added in `v1.2`**|
|`membership`|`string`|**Required:** The value `join`.|

##### Request body example

```
{
  "content": {
    "membership": "join"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The join event has been accepted into the room.|

##### 200 response

Array of `integer, Room State`.

|Room State|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_chain`|`[PDU]`|**Required:**<br><br>The auth chain for the entire current room state prior to the join event.<br><br>Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`origin`|`string`|**Required:** The resident server’s DNS name.|
|`state`|`[PDU]`|**Required:**<br><br>The resolved current room state prior to the join event.<br><br>The event format varies depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

```
[
  200,
  {
    "auth_chain": [
      {
        "content": {
          "see_room_version_spec": "The event format changes depending on the room version."
        },
        "room_id": "!somewhere:example.org",
        "type": "m.room.minimal_pdu"
      }
    ],
    "event": {
      "auth_events": [
        "$urlsafe_base64_encoded_eventid",
        "$a-different-event-id"
      ],
      "content": {
        "join_authorised_via_users_server": "@arbitrary:resident.example.com",
        "membership": "join"
      },
      "depth": 12,
      "hashes": {
        "sha256": "thishashcoversallfieldsincasethisisredacted"
      },
      "origin": "example.com",
      "origin_server_ts": 1404838188000,
      "prev_events": [
        "$urlsafe_base64_encoded_eventid",
        "$a-different-event-id"
      ],
      "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
      "sender": "@alice:example.com",
      "signatures": {
        "example.com": {
          "ed25519:key_version": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
        },
        "resident.example.com": {
          "ed25519:other_key_version": "a different signature"
        }
      },
      "state_key": "@alice:example.com",
      "type": "m.room.member",
      "unsigned": {
        "age": 4612
      }
    },
    "origin": "matrix.org",
    "state": [
      {
        "content": {
          "see_room_version_spec": "The event format changes depending on the room version."
        },
        "room_id": "!somewhere:example.org",
        "type": "m.room.minimal_pdu"
      }
    ]
  }
]
```

### PUT /_matrix/federation/v2/send_join/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_joinroomideventid)

---

**Note:** This API is nearly identical to the v1 API with the exception of the response format being fixed.

This endpoint is preferred over the v1 API as it provides a more standardised response format. Senders which receive a 400, 404, or other status code which indicates this endpoint is not available should retry using the v1 API instead.

Submits a signed join event to the resident server for it to accept it into the room’s graph. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response body here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the join event.|
|`roomId`|`string`|**Required:** The room ID that is about to be joined.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`omit_members`|`boolean`|If `true`, indicates that that calling server can accept a reduced response, in which membership events are omitted from `state` and redundant events are omitted from `auth_chain`.<br><br>If the room to be joined has no `m.room.name` nor `m.room.canonical_alias` events in its current state, the resident server should determine the room members who would be included in the `m.heroes` property of the room summary as defined in the [Client-Server /sync response](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv3sync). The resident server should include these members’ membership events in the response `state` field, and include the auth chains for these membership events in the response `auth_chain` field.<br><br>**Added in `v1.6`**|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_joinroomideventid_request_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the joining homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the joining homeserver.|
|`sender`|`string`|**Required:** The user ID of the joining member.|
|`state_key`|`string`|**Required:** The user ID of the joining member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`join_authorised_via_users_server`|`string`|Required if the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and is joining through one of the conditions available. If the user is responding to an invite, this is not required.<br><br>An arbitrary user ID belonging to the resident server in the room being joined that is able to issue invites to other users. This is used in later validation of the auth rules for the `m.room.member` event.<br><br>The resident server which owns the provided user ID must have a valid signature on the event. If the resident server is receiving the `/send_join` request, the signature must be added before sending or persisting the event to other servers.<br><br>**Added in `v1.2`**|
|`membership`|`string`|**Required:** The value `join`.|

##### Request body example

```
{
  "content": {
    "membership": "join"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The join event has been accepted into the room.|
|`400`|The request is invalid in some way.<br><br>The error should be passed through to clients so that they may give better feedback to users.<br><br>New in `v1.2`, the following error conditions might happen:<br><br>If the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and none of the conditions can be validated by the server then the `errcode` `M_UNABLE_TO_AUTHORISE_JOIN` must be used. This can happen if the server does not know about any of the rooms listed as conditions, for example.<br><br>`M_UNABLE_TO_GRANT_JOIN` is returned to denote that a different server should be attempted for the join. This is typically because the resident server can see that the joining user satisfies one or more conditions, such as in the case of [restricted rooms](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms), but the resident server would be unable to meet the auth rules governing `join_authorised_via_users_server` on the resulting `m.room.member` event.|
|`403`|The room that the joining server is attempting to join does not permit the user to join.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`auth_chain`|`[PDU]`|**Required:**<br><br>All events in the auth chain for the new join event, as well as those in the auth chains for any events returned in `state`.<br><br>If the `omit_members` query parameter was set to `true`, then any events that are returned in `state` may be omitted from `auth_chain`, whether or not membership events are omitted from `state`.<br><br>Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.<br><br>  <br>  <br>**Changed in `v1.6`:** reworded to include only consider state events returned in `state`, and to allow elision of redundant events.|
|`event`|`SignedMembershipEvent`|The membership event sent to other servers by the resident server including a signature from the resident server. Required if the room is [restricted](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and the joining user is authorised by one of the conditions.<br><br>**Added in `v1.2`**|
|`members_omitted`|`boolean`|`true` if `m.room.member` events have been omitted from `state`.<br><br>**Added in `v1.6`**|
|`origin`|`string`|**Required:** The resident server’s DNS name.|
|`servers_in_room`|`[string]`|**Required** if `members_omitted` is true.<br><br>A list of the servers active in the room (ie, those with joined members) before the join.<br><br>**Added in `v1.6`**|
|`state`|`[PDU]`|**Required:**<br><br>The resolved current room state prior to the join event.<br><br>If the request had `omit_members` set to `true`, events of type `m.room.member` may be omitted from the response to reduce the size of the response. If this is done, `members_omitted` must be set to `true`.<br><br>  <br>  <br>**Changed in `v1.6`:** permit omission of membership events.|

```
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ],
  "event": {
    "auth_events": [
      "$urlsafe_base64_encoded_eventid",
      "$a-different-event-id"
    ],
    "content": {
      "join_authorised_via_users_server": "@arbitrary:resident.example.com",
      "membership": "join"
    },
    "depth": 12,
    "hashes": {
      "sha256": "thishashcoversallfieldsincasethisisredacted"
    },
    "origin": "example.com",
    "origin_server_ts": 1404838188000,
    "prev_events": [
      "$urlsafe_base64_encoded_eventid",
      "$a-different-event-id"
    ],
    "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
    "sender": "@alice:example.com",
    "signatures": {
      "example.com": {
        "ed25519:key_version": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
      },
      "resident.example.com": {
        "ed25519:other_key_version": "a different signature"
      }
    },
    "state_key": "@alice:example.com",
    "type": "m.room.member",
    "unsigned": {
      "age": 4612
    }
  },
  "members_omitted": true,
  "origin": "matrix.org",
  "servers_in_room": [
    "matrix.org",
    "example.com"
  ],
  "state": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
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
  "errcode": "M_UNABLE_TO_GRANT_JOIN",
  "error": "This server cannot send invites to you."
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not invited to this room"
}
```

## Restricted rooms[](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms)

Restricted rooms are described in detail in the [client-server API](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms) and are available in room versions [which support restricted join rules](https://spec.matrix.org/v1.11/rooms/#feature-matrix).

A resident server processing a request to join a restricted room must ensure that the joining server satisfies at least one of the conditions specified by `m.room.join_rules`. If no conditions are available, or none match the required schema, then the joining server is considered to have failed all conditions.

The resident server uses a 400 `M_UNABLE_TO_AUTHORISE_JOIN` error on `/make_join` and `/send_join` to denote that the resident server is unable to validate any of the conditions, usually because the resident server does not have state information about rooms required by the conditions.

The resident server uses a 400 `M_UNABLE_TO_GRANT_JOIN` error on `/make_join` and `/send_join` to denote that the joining server should try a different server. This is typically because the resident server can see that the joining user satisfies one of the conditions, though the resident server would be unable to meet the auth rules governing `join_authorised_via_users_server` on the resulting `m.room.member` event.

If the joining server fails all conditions then a 403 `M_FORBIDDEN` error is used by the resident server.

# Knocking upon a room[](https://spec.matrix.org/v1.11/server-server-api/#knocking-rooms)

Rooms can permit knocking through the join rules, and if permitted this gives users a way to request to join (be invited) to the room. Users who knock on a room where the server is already a resident of the room can just send the knock event directly without using this process, however much like [joining rooms](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms) the server must handshake their way into having the knock sent on its behalf.

The handshake is largely the same as the joining rooms handshake, where instead of a “joining server” there is a “knocking server”, and the APIs to be called are different (`/make_knock` and `/send_knock`).

Servers can retract knocks over federation by leaving the room, as described below for rejecting invites.

### GET /_matrix/federation/v1/make_knock/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid)

---

**Added in `v1.1`**

Asks the receiving server to return information that the sending server will need to prepare a knock event for the room.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID that is about to be knocked.|
|`userId`|`string`|**Required:** The user ID the knock event will be for.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`ver`|`[string]`|**Required:** The room versions the sending server has support for.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A template to be used for the rest of the [Knocking Rooms](https://spec.matrix.org/v1.11/server-server-api/#knocking-rooms) handshake. Note that events have a different format depending on room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The response body here describes the common event fields in more detail and may be missing other required fields for a PDU.**|
|`400`|The request is invalid or the room the server is attempting to knock upon has a version that is not listed in the `ver` parameters.<br><br>The error should be passed through to clients so that they may give better feedback to users.|
|`403`|The knocking server or user is not permitted to knock on the room, such as when the server/user is banned or the room is not set up for receiving knocks.|
|`404`|The room that the knocking server is attempting to knock upon is unknown to the receiving server.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`event`|[Event Template](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid_response-200_event-template)|**Required:** An unsigned template event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`room_version`|`string`|**Required:** The version of the room where the server is trying to knock.|

|Event Template|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid_response-200_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the resident homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the resident homeserver.|
|`sender`|`string`|**Required:** The user ID of the knocking member.|
|`state_key`|`string`|**Required:** The user ID of the knocking member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `knock`.|

```
{
  "event": {
    "content": {
      "membership": "knock"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "7"
}
```

##### 400 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|
|`room_version`|`string`|The version of the room. Required if the `errcode` is `M_INCOMPATIBLE_ROOM_VERSION`.|

```
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to knock on this room",
  "room_version": "7"
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not permitted to knock on this room"
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
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```

### PUT /_matrix/federation/v1/send_knock/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid)

---

**Added in `v1.1`**

Submits a signed knock event to the resident server for it to accept into the room’s graph. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response body here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the knock event.|
|`roomId`|`string`|**Required:** The room ID that is about to be knocked upon.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid_request_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the knocking homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the knocking homeserver.|
|`sender`|`string`|**Required:** The user ID of the knocking member.|
|`state_key`|`string`|**Required:** The user ID of the knocking member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `knock`.|

##### Request body example

```
{
  "content": {
    "membership": "knock"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|Information about the room to pass along to clients regarding the knock.|
|`403`|The knocking server or user is not permitted to knock on the room, such as when the server/user is banned or the room is not set up for receiving knocks.|
|`404`|The room that the knocking server is attempting to knock upon is unknown to the receiving server.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`knock_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid_response-200_strippedstateevent)]|**Required:** A list of [stripped state events](https://spec.matrix.org/v1.11/client-server-api/#stripped-state) to help the initiator of the knock identify the room.|

|StrippedStateEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`EventContent`|**Required:** The `content` for the event.|
|`sender`|`string`|**Required:** The `sender` for the event.|
|`state_key`|`string`|**Required:** The `state_key` for the event.|
|`type`|`string`|**Required:** The `type` for the event.|

```
{
  "knock_room_state": [
    {
      "content": {
        "name": "Example Room"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.name"
    },
    {
      "content": {
        "join_rule": "knock"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.join_rules"
    }
  ]
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not permitted to knock on this room"
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
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```

# Inviting to a room[](https://spec.matrix.org/v1.11/server-server-api/#inviting-to-a-room)

When a user on a given homeserver invites another user on the same homeserver, the homeserver may sign the membership event itself and skip the process defined here. However, when a user invites another user on a different homeserver, a request to that homeserver to have the event signed and verified must be made.

Note that invites are used to indicate that knocks were accepted. As such, receiving servers should be prepared to manually link up a previous knock to an invite if the invite event does not directly reference the knock.

### PUT /_matrix/federation/v1/invite/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid)

---

Invites a remote user to a room. Once the event has been signed by both the inviting homeserver and the invited homeserver, it can be sent to all of the servers in the room by the inviting homeserver.

Servers should prefer to use the v2 API for invites instead of the v1 API. Servers which receive a v1 invite request must assume that the room version is either `"1"` or `"2"`.

Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response bodies here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the invite event, generated by the inviting server.|
|`roomId`|`string`|**Required:** The room ID that the user is being invited to.|

##### Request body

|InviteEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_membership-event-content)|**Required:** The content of the event, matching what is available in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/). Must include a `membership` of `invite`.|
|`origin`|`string`|**Required:** The name of the inviting homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the inviting homeserver.|
|`sender`|`string`|**Required:** The matrix ID of the user who sent the original `m.room.third_party_invite`.|
|`state_key`|`string`|**Required:** The user ID of the invited member.|
|`type`|`string`|**Required:** The value `m.room.member`.|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_unsigneddata)|Information included alongside the event that is not signed. May include more than what is listed here.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `invite`.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`invite_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_strippedstateevent)]|An optional list of [stripped state events](https://spec.matrix.org/v1.11/client-server-api/#stripped-state) to help the receiver of the invite identify the room.|

|StrippedStateEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`EventContent`|**Required:** The `content` for the event.|
|`sender`|`string`|**Required:** The `sender` for the event.|
|`state_key`|`string`|**Required:** The `state_key` for the event.|
|`type`|`string`|**Required:** The `type` for the event.|

##### Request body example

```
{
  "content": {
    "membership": "invite"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@joe:elsewhere.com",
  "type": "m.room.member",
  "unsigned": {
    "invite_room_state": [
      {
        "content": {
          "name": "Example Room"
        },
        "sender": "@bob:example.org",
        "state_key": "",
        "type": "m.room.name"
      },
      {
        "content": {
          "join_rule": "invite"
        },
        "sender": "@bob:example.org",
        "state_key": "",
        "type": "m.room.join_rules"
      }
    ]
  }
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The event with the invited server’s signature added. All other fields of the events should remain untouched. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`403`|The invite is not allowed. This could be for a number of reasons, including:<br><br>- The sender is not allowed to send invites to the target user/homeserver.<br>- The homeserver does not permit anyone to invite its users.<br>- The homeserver refuses to participate in the room.|

##### 200 response

Array of `integer, Event Container`.

|Event Container|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_response-200_inviteevent)|**Required:** An invite event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

|InviteEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_response-200_membership-event-content)|**Required:** The content of the event, matching what is available in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/). Must include a `membership` of `invite`.|
|`origin`|`string`|**Required:** The name of the inviting homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the inviting homeserver.|
|`sender`|`string`|**Required:** The matrix ID of the user who sent the original `m.room.third_party_invite`.|
|`state_key`|`string`|**Required:** The user ID of the invited member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `invite`.|

```
[
  200,
  {
    "event": {
      "content": {
        "membership": "invite"
      },
      "origin": "example.org",
      "origin_server_ts": 1549041175876,
      "room_id": "!somewhere:example.org",
      "sender": "@someone:example.org",
      "signatures": {
        "elsewhere.com": {
          "ed25519:k3y_versi0n": "SomeOtherSignatureHere"
        },
        "example.com": {
          "ed25519:key_version": "SomeSignatureHere"
        }
      },
      "state_key": "@someone:example.org",
      "type": "m.room.member",
      "unsigned": {
        "invite_room_state": [
          {
            "content": {
              "name": "Example Room"
            },
            "sender": "@bob:example.org",
            "state_key": "",
            "type": "m.room.name"
          },
          {
            "content": {
              "join_rule": "invite"
            },
            "sender": "@bob:example.org",
            "state_key": "",
            "type": "m.room.join_rules"
          }
        ]
      }
    }
  }
]
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "User cannot invite the target user."
}
```

### PUT /_matrix/federation/v2/invite/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid)

---

**Note:** This API is nearly identical to the v1 API with the exception of the request body being different, and the response format fixed.

Invites a remote user to a room. Once the event has been signed by both the inviting homeserver and the invited homeserver, it can be sent to all of the servers in the room by the inviting homeserver.

This endpoint is preferred over the v1 API as it is more useful for servers. Senders which receive a 400 or 404 response to this endpoint should retry using the v1 API as the server may be older, if the room version is “1” or “2”.

Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response bodies here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the invite event, generated by the inviting server.|
|`roomId`|`string`|**Required:** The room ID that the user is being invited to.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_inviteevent)|**Required:** An invite event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`invite_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_strippedstateevent)]|An optional list of [stripped state events](https://spec.matrix.org/v1.11/client-server-api/#stripped-state) to help the receiver of the invite identify the room.|
|`room_version`|`string`|**Required:** The version of the room where the user is being invited to.|

|InviteEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_membership-event-content)|**Required:** The content of the event, matching what is available in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/). Must include a `membership` of `invite`.|
|`origin`|`string`|**Required:** The name of the inviting homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the inviting homeserver.|
|`sender`|`string`|**Required:** The matrix ID of the user who sent the original `m.room.third_party_invite`.|
|`state_key`|`string`|**Required:** The user ID of the invited member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `invite`.|

|StrippedStateEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`EventContent`|**Required:** The `content` for the event.|
|`sender`|`string`|**Required:** The `sender` for the event.|
|`state_key`|`string`|**Required:** The `state_key` for the event.|
|`type`|`string`|**Required:** The `type` for the event.|

##### Request body example

```
{
  "event": {
    "content": {
      "membership": "invite"
    },
    "origin": "matrix.org",
    "origin_server_ts": 1234567890,
    "sender": "@someone:example.org",
    "state_key": "@joe:elsewhere.com",
    "type": "m.room.member"
  },
  "invite_room_state": [
    {
      "content": {
        "name": "Example Room"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.name"
    },
    {
      "content": {
        "join_rule": "invite"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.join_rules"
    }
  ],
  "room_version": "2"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The event with the invited server’s signature added. All other fields of the events should remain untouched. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`400`|The request is invalid or the room the server is attempting to join has a version that is not listed in the `ver` parameters.<br><br>The error should be passed through to clients so that they may give better feedback to users.|
|`403`|The invite is not allowed. This could be for a number of reasons, including:<br><br>- The sender is not allowed to send invites to the target user/homeserver.<br>- The homeserver does not permit anyone to invite its users.<br>- The homeserver refuses to participate in the room.|

##### 200 response

|Event Container|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_response-200_inviteevent)|**Required:** An invite event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|

|InviteEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_response-200_membership-event-content)|**Required:** The content of the event, matching what is available in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/). Must include a `membership` of `invite`.|
|`origin`|`string`|**Required:** The name of the inviting homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the inviting homeserver.|
|`sender`|`string`|**Required:** The matrix ID of the user who sent the original `m.room.third_party_invite`.|
|`state_key`|`string`|**Required:** The user ID of the invited member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `invite`.|

```
{
  "event": {
    "content": {
      "membership": "invite"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "signatures": {
      "elsewhere.com": {
        "ed25519:k3y_versi0n": "SomeOtherSignatureHere"
      },
      "example.com": {
        "ed25519:key_version": "SomeSignatureHere"
      }
    },
    "state_key": "@someone:example.org",
    "type": "m.room.member",
    "unsigned": {
      "invite_room_state": [
        {
          "content": {
            "name": "Example Room"
          },
          "sender": "@bob:example.org",
          "state_key": "",
          "type": "m.room.name"
        },
        {
          "content": {
            "join_rule": "invite"
          },
          "sender": "@bob:example.org",
          "state_key": "",
          "type": "m.room.join_rules"
        }
      ]
    }
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
|`room_version`|`string`|The version of the room. Required if the `errcode` is `M_INCOMPATIBLE_ROOM_VERSION`.|

```
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to join this room",
  "room_version": "3"
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "User cannot invite the target user."
}
```

# Leaving Rooms (Rejecting Invites)[](https://spec.matrix.org/v1.11/server-server-api/#leaving-rooms-rejecting-invites)

Normally homeservers can send appropriate `m.room.member` events to have users leave the room, to reject local invites, or to retract a knock. Remote invites/knocks from other homeservers do not involve the server in the graph and therefore need another approach to reject the invite. Joining the room and promptly leaving is not recommended as clients and servers will interpret that as accepting the invite, then leaving the room rather than rejecting the invite.

Similar to the [Joining Rooms](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms) handshake, the server which wishes to leave the room starts with sending a `/make_leave` request to a resident server. In the case of rejecting invites, the resident server may be the server which sent the invite. After receiving a template event from `/make_leave`, the leaving server signs the event and replaces the `event_id` with its own. This is then sent to the resident server via `/send_leave`. The resident server will then send the event to other servers in the room.

### GET /_matrix/federation/v1/make_leave/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid)

---

Asks the receiving server to return information that the sending server will need to prepare a leave event to get out of the room.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID that is about to be left.|
|`userId`|`string`|**Required:** The user ID the leave event will be for.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A template to be used to call `/send_leave`. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The response body here describes the common event fields in more detail and may be missing other required fields for a PDU.**|
|`403`|The request is not authorized. This could mean that the user is not in the room.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`event`|[Event Template](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid_response-200_event-template)|An unsigned template event. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats.|
|`room_version`|`string`|The version of the room where the server is trying to leave. If not provided, the room version is assumed to be either “1” or “2”.|

|Event Template|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid_response-200_membership-event-content)|**Required:** The content of the event.|
|`origin`|`string`|**Required:** The name of the resident homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the resident homeserver.|
|`sender`|`string`|**Required:** The user ID of the leaving member.|
|`state_key`|`string`|**Required:** The user ID of the leaving member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `leave`.|

```
{
  "event": {
    "content": {
      "membership": "leave"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "2"
}
```

##### 403  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "User is not in the room."
}
```

### PUT /_matrix/federation/v1/send_leave/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_leaveroomideventid)

---

> [!warning] warning:
> This API is deprecated and will be removed from a future release.

**Note:** Servers should instead prefer to use the v2 `/send_leave` endpoint.

Submits a signed leave event to the resident server for it to accept it into the room’s graph. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response body here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the leave event.|
|`roomId`|`string`|**Required:** The room ID that is about to be left.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_leaveroomideventid_request_membership-event-content)|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** This field must be present but is ignored; it may be 0.|
|`origin`|`string`|**Required:** The name of the leaving homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the leaving homeserver.|
|`sender`|`string`|**Required:** The user ID of the leaving member.|
|`state_key`|`string`|**Required:** The user ID of the leaving member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `leave`.|

##### Request body example

```
{
  "content": {
    "membership": "leave"
  },
  "depth": 12,
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|An empty response to indicate the event was accepted into the graph by the receiving homeserver.|

##### 200 response

Array of `integer, Empty Object`.

```
[
  200,
  {}
]
```

### PUT /_matrix/federation/v2/send_leave/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_leaveroomideventid)

---

**Note:** This API is nearly identical to the v1 API with the exception of the response format being fixed.

This endpoint is preferred over the v1 API as it provides a more standardised response format. Senders which receive a 400, 404, or other status code which indicates this endpoint is not available should retry using the v1 API instead.

Submits a signed leave event to the resident server for it to accept it into the room’s graph. Note that events have a different format depending on the room version - check the [room version specification](https://spec.matrix.org/v1.11/rooms) for precise event formats. **The request and response body here describe the common event fields in more detail and may be missing other required fields for a PDU.**

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`eventId`|`string`|**Required:** The event ID for the leave event.|
|`roomId`|`string`|**Required:** The room ID that is about to be left.|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_leaveroomideventid_request_membership-event-content)|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** This field must be present but is ignored; it may be 0.|
|`origin`|`string`|**Required:** The name of the leaving homeserver.|
|`origin_server_ts`|`integer`|**Required:** A timestamp added by the leaving homeserver.|
|`sender`|`string`|**Required:** The user ID of the leaving member.|
|`state_key`|`string`|**Required:** The user ID of the leaving member.|
|`type`|`string`|**Required:** The value `m.room.member`.|

|Membership Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The value `leave`.|

##### Request body example

```
{
  "content": {
    "membership": "leave"
  },
  "depth": 12,
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|An empty response to indicate the event was accepted into the graph by the receiving homeserver.|

##### 200 response

```
{}
```

# Third-party invites[](https://spec.matrix.org/v1.11/server-server-api/#third-party-invites)

> [!info] info:
> More information about third-party invites is available in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api) under the Third-party Invites module.

When a user wants to invite another user in a room but doesn’t know the Matrix ID to invite, they can do so using a third-party identifier (e.g. an e-mail or a phone number).

This identifier and its bindings to Matrix IDs are verified by an identity server implementing the [Identity Service API](https://spec.matrix.org/v1.11/identity-service-api).

## Cases where an association exists for a third-party identifier[](https://spec.matrix.org/v1.11/server-server-api/#cases-where-an-association-exists-for-a-third-party-identifier)

If the third-party identifier is already bound to a Matrix ID, a lookup request on the identity server will return it. The invite is then processed by the inviting homeserver as a standard `m.room.member` invite event. This is the simplest case.

## Cases where an association doesn’t exist for a third-party identifier[](https://spec.matrix.org/v1.11/server-server-api/#cases-where-an-association-doesnt-exist-for-a-third-party-identifier)

If the third-party identifier isn’t bound to any Matrix ID, the inviting homeserver will request the identity server to store an invite for this identifier and to deliver it to whoever binds it to its Matrix ID. It will also send an `m.room.third_party_invite` event in the room to specify a display name, a token and public keys the identity server provided as a response to the invite storage request.

When a third-party identifier with pending invites gets bound to a Matrix ID, the identity server will send a POST request to the ID’s homeserver as described in the [Invitation Storage](https://spec.matrix.org/v1.11/identity-service-api#invitation-storage) section of the Identity Service API.

The following process applies for each invite sent by the identity server:

The invited homeserver will create an `m.room.member` invite event containing a special `third_party_invite` section containing the token and a signed object, both provided by the identity server.

If the invited homeserver is in the room the invite came from, it can auth the event and send it.

However, if the invited homeserver isn’t in the room the invite came from, it will need to request the room’s homeserver to auth the event.

### PUT /_matrix/federation/v1/3pid/onbind[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind)

---

Used by identity servers to notify the homeserver that one of its users has bound a third-party identifier successfully, including any pending room invites the identity server has been made aware of.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The third-party identifier itself. For example, an email address.|
|`invites`|[[Third-party Invite](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_third-party-invite)]|**Required:** A list of pending invites that the third-party identifier has received.|
|`medium`|`string`|**Required:** The type of third-party identifier. Currently only “email” is a possible value.|
|`mxid`|`string`|**Required:** The user that is now bound to the third-party identifier.|

|Third-party Invite|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`address`|`string`|**Required:** The third-party identifier that received the invite.|
|`medium`|`string`|**Required:** The type of third-party invite issues. Currently only “email” is used.|
|`mxid`|`string`|**Required:** The now-bound user ID that received the invite.|
|`room_id`|`string`|**Required:** The room ID the invite is valid for.|
|`sender`|`string`|**Required:** The user ID that sent the invite.|
|`signed`|[Identity Server Signatures](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_identity-server-signatures)|**Required:** Signature from the identity server using a long-term private key.|

|Identity Server Signatures|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`mxid`|`string`|**Required:** The user ID that has been bound to the third-party identifier.|
|`signatures`|{string: [Identity Server Domain Signature](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_identity-server-domain-signature)}|**Required:** The signature from the identity server. The `string` key is the identity server’s domain name, such as vector.im|
|`token`|`string`|**Required:** A token.|

|Identity Server Domain Signature|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`ed25519:0`|`string`|**Required:** The signature.|

##### Request body example

```
{
  "address": "alice@example.com",
  "invites": [
    {
      "address": "alice@example.com",
      "medium": "email",
      "mxid": "@alice:matrix.org",
      "room_id": "!somewhere:example.org",
      "sender": "@bob:matrix.org",
      "signed": {
        "mxid": "@alice:matrix.org",
        "signatures": {
          "vector.im": {
            "ed25519:0": "SomeSignatureGoesHere"
          }
        },
        "token": "Hello World"
      }
    }
  ],
  "medium": "email",
  "mxid": "@alice:matrix.org"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The homeserver has processed the notification.|

##### 200 response

```
{}
```

### PUT /_matrix/federation/v1/exchange_third_party_invite/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid)

---

The receiving server will verify the partial `m.room.member` event given in the request body. If valid, the receiving server will issue an invite as per the [Inviting to a room](https://spec.matrix.org/v1.11/server-server-api/#inviting-to-a-room) section before returning a response to this request.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID to exchange a third-party invite in|

##### Request body

|Name|Type|Description|
|---|---|---|
|`content`|[Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_event-content)|**Required:** The event content|
|`room_id`|`string`|**Required:** The room ID the event is for. Must match the ID given in the path.|
|`sender`|`string`|**Required:** The user ID of the user who sent the original `m.room.third_party_invite` event.|
|`state_key`|`string`|**Required:** The user ID of the invited user|
|`type`|`string`|**Required:** The event type. Must be `m.room.member`|

|Event Content|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`membership`|`string`|**Required:** The membership state. Must be `invite`|
|`third_party_invite`|[Third-party Invite](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_third-party-invite)|**Required:** The third-party invite|

|Third-party Invite|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`display_name`|`string`|**Required:** A name which can be displayed to represent the user instead of their third-party identifier.|
|`signed`|[Invite Signatures](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_invite-signatures)|**Required:** A block of content which has been signed, which servers can use to verify the event.|

|Invite Signatures|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`mxid`|`string`|**Required:** The invited matrix user ID|
|`signatures`|`{string: {string: string}}`|**Required:**<br><br>The server signatures for this event.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`token`|`string`|**Required:** The token used to verify the event|

##### Request body example

```
{
  "content": {
    "membership": "invite",
    "third_party_invite": {
      "display_name": "alice",
      "signed": {
        "mxid": "@alice:localhost",
        "signatures": {
          "magic.forest": {
            "ed25519:3": "fQpGIW1Snz+pwLZu6sTy2aHy/DYWWTspTJRPyNp0PKkymfIsNffysMl6ObMMFdIJhk6g6pwlIqZ54rxo8SLmAg"
          }
        },
        "token": "abc123"
      }
    }
  },
  "room_id": "!abc123:matrix.org",
  "sender": "@joe:matrix.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The invite has been issued successfully.|

##### 200 response

```
{}
```

### Verifying the invite[](https://spec.matrix.org/v1.11/server-server-api/#verifying-the-invite)

When a homeserver receives an `m.room.member` invite event for a room it’s in with a `third_party_invite` object, it must verify that the association between the third-party identifier initially invited to the room and the Matrix ID that claims to be bound to it has been verified without having to rely on a third-party server.

To do so, it will fetch from the room’s state events the `m.room.third_party_invite` event for which the state key matches with the value for the `token` key in the `third_party_invite` object from the `m.room.member` event’s content to fetch the public keys initially delivered by the identity server that stored the invite.

It will then use these keys to verify that the `signed` object (in the `third_party_invite` object from the `m.room.member` event’s content) was signed by the same identity server.

Since this `signed` object can only be delivered once in the POST request emitted by the identity server upon binding between the third-party identifier and the Matrix ID, and contains the invited user’s Matrix ID and the token delivered when the invite was stored, this verification will prove that the `m.room.member` invite event comes from the user owning the invited third-party identifier.

# Public Room Directory[](https://spec.matrix.org/v1.11/server-server-api/#public-room-directory)

To complement the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api)’s room directory, homeservers need a way to query the public rooms for another server. This can be done by making a request to the `/publicRooms` endpoint for the server the room directory should be retrieved for.

### GET /_matrix/federation/v1/publicRooms[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1publicrooms)

---

Gets all the public rooms for the homeserver. This should not return rooms that are listed on another homeserver’s directory, just those listed on the receiving homeserver’s directory.

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
|`include_all_networks`|`boolean`|Whether or not to include all networks/protocols defined by application services on the homeserver. Defaults to false.|
|`limit`|`integer`|The maximum number of rooms to return. Defaults to 0 (no limit).|
|`since`|`string`|A pagination token from a previous call to this endpoint to fetch more rooms.|
|`third_party_instance_id`|`string`|The specific third-party network/protocol to request from the homeserver. Can only be used if `include_all_networks` is false.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The public room list for the homeserver.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`chunk`|[[PublicRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1publicrooms_response-200_publicroomschunk)]|**Required:** A paginated chunk of public rooms.|
|`next_batch`|`string`|A pagination token for the response. The absence of this token means there are no more results to fetch and the client should stop paginating.|
|`prev_batch`|`string`|A pagination token that allows fetching previous results. The absence of this token means there are no results before this batch, i.e. this is the first batch.|
|`total_room_count_estimate`|`integer`|An estimate on the total number of public rooms, if the server has an estimate.|

|PublicRoomsChunk|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|The URL for the room’s avatar, if one is set.|
|`canonical_alias`|`string`|The canonical alias of the room, if any.|
|`guest_can_join`|`boolean`|**Required:** Whether guest users may join the room and participate in it. If they can, they will be subject to ordinary power level rules like any other user.|
|`join_rule`|`string`|The room’s join rule. When not present, the room is assumed to be `public`. Note that rooms with `invite` join rules are not expected here, but rooms with `knock` rules are given their near-public nature.|
|`name`|`string`|The name of the room, if any.|
|`num_joined_members`|`integer`|**Required:** The number of members joined to the room.|
|`room_id`|`string`|**Required:** The ID of the room.|
|`room_type`|`string`|The `type` of room (from [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)), if any.<br><br>**Added in `v1.4`**|
|`topic`|`string`|The topic of the room, if any.|
|`world_readable`|`boolean`|**Required:** Whether the room may be viewed by guest users without joining.|

```
{
  "chunk": [
    {
      "avatar_url": "mxc://bleecker.street/CHEDDARandBRIE",
      "guest_can_join": false,
      "join_rule": "public",
      "name": "CHEESE",
      "num_joined_members": 37,
      "room_id": "!ol19s:bleecker.street",
      "room_type": "m.space",
      "topic": "Tasty tasty cheese",
      "world_readable": true
    }
  ],
  "next_batch": "p190q",
  "prev_batch": "p1902",
  "total_room_count_estimate": 115
}
```

### POST /_matrix/federation/v1/publicRooms[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms)

---

Lists the public rooms on the server, with optional filter.

This API returns paginated responses. The rooms are ordered by the number of joined members, with the largest rooms first.

Note that this endpoint receives and returns the same format that is seen in the Client-Server API’s `POST /publicRooms` endpoint.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`filter`|[Filter](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms_request_filter)|Filter to apply to the results.|
|`include_all_networks`|`boolean`|Whether or not to include all known networks/protocols from application services on the homeserver. Defaults to false.|
|`limit`|`integer`|Limit the number of results returned.|
|`since`|`string`|A pagination token from a previous request, allowing servers to get the next (or previous) batch of rooms. The direction of pagination is specified solely by which token is supplied, rather than via an explicit flag.|
|`third_party_instance_id`|`string`|The specific third-party network/protocol to request from the homeserver. Can only be used if `include_all_networks` is false.|

|Filter|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`generic_search_term`|`string`|An optional string to search for in the room metadata, e.g. name, topic, canonical alias, etc.|
|`room_types`|`[string\|null]`|An optional list of [room types](https://spec.matrix.org/v1.11/client-server-api/#types) to search for. To include rooms without a room type, specify `null` within this list. When not specified, all applicable rooms (regardless of type) are returned.<br><br>**Added in `v1.4`**|

##### Request body example

```
{
  "filter": {
    "generic_search_term": "foo",
    "room_types": [
      null,
      "m.space"
    ]
  },
  "include_all_networks": false,
  "limit": 10,
  "third_party_instance_id": "irc"
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|A list of the rooms on the server.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`chunk`|[[PublicRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms_response-200_publicroomschunk)]|**Required:** A paginated chunk of public rooms.|
|`next_batch`|`string`|A pagination token for the response. The absence of this token means there are no more results to fetch and the client should stop paginating.|
|`prev_batch`|`string`|A pagination token that allows fetching previous results. The absence of this token means there are no results before this batch, i.e. this is the first batch.|
|`total_room_count_estimate`|`integer`|An estimate on the total number of public rooms, if the server has an estimate.|

|PublicRoomsChunk|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|The URL for the room’s avatar, if one is set.|
|`canonical_alias`|`string`|The canonical alias of the room, if any.|
|`guest_can_join`|`boolean`|**Required:** Whether guest users may join the room and participate in it. If they can, they will be subject to ordinary power level rules like any other user.|
|`join_rule`|`string`|The room’s join rule. When not present, the room is assumed to be `public`. Note that rooms with `invite` join rules are not expected here, but rooms with `knock` rules are given their near-public nature.|
|`name`|`string`|The name of the room, if any.|
|`num_joined_members`|`integer`|**Required:** The number of members joined to the room.|
|`room_id`|`string`|**Required:** The ID of the room.|
|`room_type`|`string`|The `type` of room (from [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)), if any.<br><br>**Added in `v1.4`**|
|`topic`|`string`|The topic of the room, if any.|
|`world_readable`|`boolean`|**Required:** Whether the room may be viewed by guest users without joining.|

```
{
  "chunk": [
    {
      "avatar_url": "mxc://bleecker.street/CHEDDARandBRIE",
      "guest_can_join": false,
      "join_rule": "public",
      "name": "CHEESE",
      "num_joined_members": 37,
      "room_id": "!ol19s:bleecker.street",
      "room_type": "m.space",
      "topic": "Tasty tasty cheese",
      "world_readable": true
    }
  ],
  "next_batch": "p190q",
  "prev_batch": "p1902",
  "total_room_count_estimate": 115
}
```

# Spaces[](https://spec.matrix.org/v1.11/server-server-api/#spaces)

To complement the [Client-Server API’s Spaces module](https://spec.matrix.org/v1.11/client-server-api/#spaces), homeservers need a way to query information about spaces from other servers.

### GET /_matrix/federation/v1/hierarchy/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid)

---

**Added in `v1.2`**

Federation version of the Client-Server [`GET /hierarchy`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1roomsroomidhierarchy) endpoint. Unlike the Client-Server API version, this endpoint does not paginate. Instead, all the space-room’s children the requesting server could feasibly peek/join are returned. The requesting server is responsible for filtering the results further down for the user’s request.

Only [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) state events of the room are considered. Invalid child rooms and parent events are not covered by this endpoint.

Responses to this endpoint should be cached for a period of time.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomId`|`string`|**Required:** The room ID of the space to get a hierarchy for.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`suggested_only`|`boolean`|Optional (default `false`) flag to indicate whether or not the server should only consider suggested rooms. Suggested rooms are annotated in their [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) event contents.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The space room and its children.|
|`404`|The room is not known to the server or the requesting server is unable to peek/join it (if it were to attempt this).|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`children`|[[SpaceHierarchyChildRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_spacehierarchychildroomschunk)]|**Required:** A summary of the space’s children. Rooms which the requesting server cannot peek/join will be excluded.|
|`inaccessible_children`|`[string]`|**Required:**<br><br>The list of room IDs the requesting server doesn’t have a viable way to peek/join. Rooms which the responding server cannot provide details on will be outright excluded from the response instead.<br><br>Assuming both the requesting and responding server are well behaved, the requesting server should consider these room IDs as not accessible from anywhere. They should not be re-requested.|
|`room`|[SpaceHierarchyParentRoom](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_spacehierarchyparentroom)|**Required:** A summary of the room requested.|

|SpaceHierarchyChildRoomsChunk|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`allowed_room_ids`|`[string]`|If the room is a [restricted room](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms), these are the room IDs which are specified by the join rules. Empty or omitted otherwise.|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|The URL for the room’s avatar, if one is set.|
|`canonical_alias`|`string`|The canonical alias of the room, if any.|
|`guest_can_join`|`boolean`|**Required:** Whether guest users may join the room and participate in it. If they can, they will be subject to ordinary power level rules like any other user.|
|`join_rule`|`string`|The room’s join rule. When not present, the room is assumed to be `public`.|
|`name`|`string`|The name of the room, if any.|
|`num_joined_members`|`integer`|**Required:** The number of members joined to the room.|
|`room_id`|`string`|**Required:** The ID of the room.|
|`room_type`|`string`|The `type` of room (from [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)), if any.<br><br>**Added in `v1.4`**|
|`topic`|`string`|The topic of the room, if any.|
|`world_readable`|`boolean`|**Required:** Whether the room may be viewed by guest users without joining.|

|SpaceHierarchyParentRoom|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`allowed_room_ids`|`[string]`|If the room is a [restricted room](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms), these are the room IDs which are specified by the join rules. Empty or omitted otherwise.|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|The URL for the room’s avatar, if one is set.|
|`canonical_alias`|`string`|The canonical alias of the room, if any.|
|`children_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_strippedstateevent)]|**Required:**<br><br>The [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) events of the space-room, represented as [Stripped State Events](https://spec.matrix.org/v1.11/client-server-api/#stripped-state) with an added `origin_server_ts` key.<br><br>If the room is not a space-room, this should be empty.|
|`guest_can_join`|`boolean`|**Required:** Whether guest users may join the room and participate in it. If they can, they will be subject to ordinary power level rules like any other user.|
|`join_rule`|`string`|The room’s join rule. When not present, the room is assumed to be `public`.|
|`name`|`string`|The name of the room, if any.|
|`num_joined_members`|`integer`|**Required:** The number of members joined to the room.|
|`room_id`|`string`|**Required:** The ID of the room.|
|`room_type`|`string`|The `type` of room (from [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)), if any.<br><br>**Added in `v1.4`**|
|`topic`|`string`|The topic of the room, if any.|
|`world_readable`|`boolean`|**Required:** Whether the room may be viewed by guest users without joining.|

|StrippedStateEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`EventContent`|**Required:** The `content` for the event.|
|`origin_server_ts`|`integer`|**Required:** The `origin_server_ts` for the event.|
|`sender`|`string`|**Required:** The `sender` for the event.|
|`state_key`|`string`|**Required:** The `state_key` for the event.|
|`type`|`string`|**Required:** The `type` for the event.|

```
{
  "children": [
    {
      "allowed_room_ids": [
        "!upstream:example.org"
      ],
      "avatar_url": "mxc://example.org/abcdef2",
      "canonical_alias": "#general:example.org",
      "children_state": [
        {
          "content": {
            "via": [
              "remote.example.org"
            ]
          },
          "origin_server_ts": 1629422222222,
          "sender": "@alice:example.org",
          "state_key": "!b:example.org",
          "type": "m.space.child"
        }
      ],
      "guest_can_join": false,
      "join_rule": "restricted",
      "name": "The ~~First~~ Second Space",
      "num_joined_members": 42,
      "room_id": "!second_room:example.org",
      "room_type": "m.space",
      "topic": "Hello world",
      "world_readable": true
    }
  ],
  "inaccessible_children": [
    "!secret:example.org"
  ],
  "room": {
    "allowed_room_ids": [],
    "avatar_url": "mxc://example.org/abcdef",
    "canonical_alias": "#general:example.org",
    "children_state": [
      {
        "content": {
          "via": [
            "remote.example.org"
          ]
        },
        "origin_server_ts": 1629413349153,
        "sender": "@alice:example.org",
        "state_key": "!a:example.org",
        "type": "m.space.child"
      }
    ],
    "guest_can_join": false,
    "join_rule": "public",
    "name": "The First Space",
    "num_joined_members": 42,
    "room_id": "!space:example.org",
    "room_type": "m.space",
    "topic": "No other spaces were created first, ever",
    "world_readable": true
  }
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
  "errcode": "M_NOT_FOUND",
  "error": "Room does not exist."
}
```

# Typing Notifications[](https://spec.matrix.org/v1.11/server-server-api/#typing-notifications)

When a server’s users send typing notifications, those notifications need to be sent to other servers in the room so their users are aware of the same state. Receiving servers should verify that the user is in the room, and is a user belonging to the sending server.

## `m.typing`

---

A typing notification EDU for a user in a room.

|m.typing|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Typing Notification](https://spec.matrix.org/v1.11/server-server-api/#definition-mtyping_typing-notification)|**Required:** The typing notification.|
|`edu_type`|`string`|**Required:** The string `m.typing`<br><br>One of: `[m.typing]`.|

|Typing Notification|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`room_id`|`string`|**Required:** The room where the user’s typing status has been updated.|
|`typing`|`boolean`|**Required:** Whether the user is typing in the room or not.|
|`user_id`|`string`|**Required:** The user ID that has had their typing status changed.|

### Examples

```
{
  "content": {
    "room_id": "!somewhere:matrix.org",
    "typing": true,
    "user_id": "@john:matrix.org"
  },
  "edu_type": "m.typing"
}
```

# Presence[](https://spec.matrix.org/v1.11/server-server-api/#presence)

The server API for presence is based entirely on exchange of the following EDUs. There are no PDUs or Federation Queries involved.

Servers should only send presence updates for users that the receiving server would be interested in. Such as the receiving server sharing a room with a given user.

## `m.presence`

---

An EDU representing presence updates for users of the sending homeserver.

|m.presence|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Presence Update](https://spec.matrix.org/v1.11/server-server-api/#definition-mpresence_presence-update)|**Required:** The presence updates and requests.|
|`edu_type`|`string`|**Required:** The string `m.presence`<br><br>One of: `[m.presence]`.|

|Presence Update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`push`|[[User Presence Update](https://spec.matrix.org/v1.11/server-server-api/#definition-mpresence_user-presence-update)]|**Required:** A list of presence updates that the receiving server is likely to be interested in.|

|User Presence Update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`currently_active`|`boolean`|True if the user is likely to be interacting with their client. This may be indicated by the user having a `last_active_ago` within the last few minutes. Defaults to false.|
|`last_active_ago`|`integer`|**Required:** The number of milliseconds that have elapsed since the user last did something.|
|`presence`|`string`|**Required:** The presence of the user.<br><br>One of: `[offline, unavailable, online]`.|
|`status_msg`|`string`|An optional description to accompany the presence.|
|`user_id`|`string`|**Required:** The user ID this presence EDU is for.|

### Examples

```
{
  "content": {
    "push": [
      {
        "currently_active": true,
        "last_active_ago": 5000,
        "presence": "online",
        "status_msg": "Making cupcakes",
        "user_id": "@john:matrix.org"
      }
    ]
  },
  "edu_type": "m.presence"
}
```

# Receipts[](https://spec.matrix.org/v1.11/server-server-api/#receipts)

Receipts are EDUs used to communicate a marker for a given event. Currently the only kind of receipt supported is a “read receipt”, or where in the event graph the user has read up to.

Read receipts for events that a user sent do not need to be sent. It is implied that by sending the event the user has read up to the event.

## `m.receipt`

---

An EDU representing receipt updates for users of the sending homeserver. When receiving receipts, the server should only update entries that are listed in the EDU. Receipts previously received that do not appear in the EDU should not be removed or otherwise manipulated.

|m.receipt|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|{[Room ID](https://spec.matrix.org/v1.11/appendices#room-ids): [Room Receipts](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_room-receipts)}|**Required:** Receipts for a particular room. The string key is the room ID for which the receipts under it belong.|
|`edu_type`|`string`|**Required:** The string `m.receipt`<br><br>One of: `[m.receipt]`.|

|Room Receipts|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`m.read`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [User Read Receipt](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_user-read-receipt)}|**Required:** Read receipts for users in the room. The string key is the user ID the receipt belongs to.|

|User Read Receipt|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`data`|[Read Receipt Metadata](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_read-receipt-metadata)|**Required:** Metadata for the read receipt.|
|`event_ids`|`[string]`|**Required:** The extremity event IDs that the user has read up to.|

|Read Receipt Metadata|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`thread_id`|`string`|The root thread event’s ID (or `main`) for which thread this receipt is intended to be under. If not specified, the read receipt is _unthreaded_ (default).<br><br>**Added in `v1.4`**|
|`ts`|`integer`|**Required:** A POSIX timestamp in milliseconds for when the user read the event specified in the read receipt.|

### Examples

```
{
  "content": {
    "!some_room:example.org": {
      "m.read": {
        "@john:matrix.org": {
          "data": {
            "ts": 1533358089009
          },
          "event_ids": [
            "$read_this_event:matrix.org"
          ]
        }
      }
    }
  },
  "edu_type": "m.receipt"
}
```

# Querying for information[](https://spec.matrix.org/v1.11/server-server-api/#querying-for-information)

Queries are a way to retrieve information from a homeserver about a resource, such as a user or room. The endpoints here are often called in conjunction with a request from a client on the client-server API in order to complete the call.

There are several types of queries that can be made. The generic endpoint to represent all queries is described first, followed by the more specific queries that can be made.

### GET /_matrix/federation/v1/query/directory[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1querydirectory)

---

Performs a query to get the mapped room ID and list of resident homeservers in the room for a given room alias. Homeservers should only query room aliases that belong to the target server (identified by the DNS Name in the alias).

Servers may wish to cache the response to this query to avoid requesting the information too often.

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
|`room_alias`|`string`|**Required:** The room alias to query.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The corresponding room ID and list of known resident homeservers for the room.|
|`404`|The room alias was not found.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`room_id`|`string`|**Required:** The room ID mapped to the queried room alias.|
|`servers`|`[string]`|**Required:** An array of server names that are likely to hold the given room. This list may or may not include the server answering the query.|

```
{
  "room_id": "!roomid1234:example.org",
  "servers": [
    "example.org",
    "example.com",
    "another.example.com:8449"
  ]
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
  "errcode": "M_NOT_FOUND",
  "error": "Room alias not found."
}
```

### GET /_matrix/federation/v1/query/profile[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1queryprofile)

---

Performs a query to get profile information, such as a display name or avatar, for a given user. Homeservers should only query profiles for users that belong to the target server (identified by the DNS Name in the user ID).

Servers may wish to cache the response to this query to avoid requesting the information too often.

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
|`field`|`string`|The field to query. If specified, the server will only return the given field in the response. If not specified, the server will return the full profile for the user.<br><br>One of: `[displayname, avatar_url]`.|
|`user_id`|`string`|**Required:** The user ID to query. Must be a user local to the receiving homeserver.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The profile for the user. If a `field` is specified in the request, only the matching field should be included in the response. If no `field` was specified, the response should include the fields of the user’s profile that can be made public, such as the display name and avatar.<br><br>If the user does not have a particular field set on their profile, the server should exclude it from the response body or give it the value `null`.|
|`404`|The user does not exist or does not have a profile.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`avatar_url`|`string`|The avatar URL for the user’s avatar. May be omitted if the user does not have an avatar set.|
|`displayname`|`string`|The display name of the user. May be omitted if the user does not have a display name set.|

```
{
  "avatar_url": "mxc://matrix.org/MyC00lAvatar",
  "displayname": "John Doe"
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
  "errcode": "M_NOT_FOUND",
  "error": "User does not exist."
}
```

### GET /_matrix/federation/v1/query/{queryType}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1queryquerytype)

---

Performs a single query request on the receiving homeserver. The query string arguments are dependent on which type of query is being made. Known query types are specified as their own endpoints as an extension to this definition.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`queryType`|`string`|**Required:** The type of query to make|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The query response. The schema varies depending on the query being made.|

# OpenID[](https://spec.matrix.org/v1.11/server-server-api/#openid)

Third-party services can exchange an access token previously generated by the Client-Server API for information about a user. This can help verify that a user is who they say they are without granting full access to the user’s account.

Access tokens generated by the OpenID API are only good for the OpenID API and nothing else.

### GET /_matrix/federation/v1/openid/userinfo[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1openiduserinfo)

---

Exchanges an OpenID access token for information about the user who generated the token. Currently this only exposes the Matrix User ID of the owner.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

#### Request

##### Request parameters

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`access_token`|`string`|**Required:** The OpenID access token to get information about the owner for.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|Information about the user who generated the OpenID access token.|
|`401`|The token was not recognized or has expired.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`sub`|`string`|**Required:** The Matrix User ID who generated the token.|

```
{
  "sub": "@alice:example.com"
}
```

##### 401 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_UNKNOWN_TOKEN",
  "error": "Access token unknown or expired"
}
```

# Device Management[](https://spec.matrix.org/v1.11/server-server-api/#device-management)

Details of a user’s devices must be efficiently published to other users and kept up-to-date. This is critical for reliable end-to-end encryption, in order for users to know which devices are participating in a room. It’s also required for to-device messaging to work. This section is intended to complement the [Device Management module](https://spec.matrix.org/v1.11/client-server-api#device-management) of the Client-Server API.

Matrix currently uses a custom pubsub system for synchronising information about the list of devices for a given user over federation. When a server wishes to determine a remote user’s device list for the first time, it should populate a local cache from the result of a `/user/keys/query` API on the remote server. However, subsequent updates to the cache should be applied by consuming `m.device_list_update` EDUs. Each new `m.device_list_update` EDU describes an incremental change to one device for a given user which should replace any existing entry in the local server’s cache of that device list. Servers must send `m.device_list_update` EDUs to all the servers who share a room with a given local user, and must be sent whenever that user’s device list changes (i.e. for new or deleted devices, when that user joins a room which contains servers which are not already receiving updates for that user’s device list, or changes in device information such as the device’s human-readable name).

Servers send `m.device_list_update` EDUs in a sequence per origin user, each with a unique `stream_id`. They also include a pointer to the most recent previous EDU(s) that this update is relative to in the `prev_id` field. To simplify implementation for clustered servers which could send multiple EDUs at the same time, the `prev_id` field should include all `m.device_list_update` EDUs which have not been yet been referenced in a EDU. If EDUs are emitted in series by a server, there should only ever be one `prev_id` in the EDU.

This forms a simple directed acyclic graph of `m.device_list_update` EDUs, showing which EDUs a server needs to have received in order to apply an update to its local copy of the remote user’s device list. If a server receives an EDU which refers to a `prev_id` it does not recognise, it must resynchronise its list by calling the `/user/keys/query API` and resume the process. The response contains a `stream_id` which should be used to correlate with subsequent `m.device_list_update` EDUs.

### GET /_matrix/federation/v1/user/devices/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid)

---

Gets information on all of the user’s devices

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`userId`|`string`|**Required:** The user ID to retrieve devices for. Must be a user local to the receiving homeserver.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The user’s devices.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`devices`|[[User Device](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_user-device)]|**Required:** The user’s devices. May be empty.|
|`master_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_crosssigningkey)|The user's master cross-signing key.|
|`self_signing_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_crosssigningkey)|The user's self-signing key.|
|`stream_id`|`integer`|**Required:** A unique ID for a given user_id which describes the version of the returned device list. This is matched with the `stream_id` field in `m.device_list_update` EDUs in order to incrementally update the returned device_list.|
|`user_id`|`string`|**Required:** The user ID devices were requested for.|

|User Device|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`device_display_name`|`string`|Optional display name for the device.|
|`device_id`|`string`|**Required:** The device ID.|
|`keys`|[DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_devicekeys)|**Required:** Identity keys for the device.|

|DeviceKeys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`algorithms`|`[string]`|**Required:** The encryption algorithms supported by this device.|
|`device_id`|`string`|**Required:** The ID of the device these keys belong to. Must match the device ID used when logging in.|
|`keys`|`{string: string}`|**Required:** Public identity keys. The names of the properties should be in the format `<algorithm>:<device_id>`. The keys themselves should be encoded as specified by the key algorithm.|
|`signatures`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**Required:**<br><br>Signatures for the device key object. A map from user ID, to a map from `<algorithm>:<device_id>` to the signature.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`user_id`|`string`|**Required:** The ID of the user the device belongs to. Must match the user ID used when logging in.|

|CrossSigningKey|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`keys`|`{string: string}`|**Required:** The public key. The object must have exactly one property, whose name is in the form `<algorithm>:<unpadded_base64_public_key>`, and whose value is the unpadded base64 public key.|
|`signatures`|`Signatures`|Signatures of the key, calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json). Optional for the master key. Other keys must be signed by the user's master key.|
|`usage`|`[string]`|**Required:** What the key is used for.|
|`user_id`|`string`|**Required:** The ID of the user the key belongs to.|

```
{
  "devices": [
    {
      "device_display_name": "Alice's Mobile Phone",
      "device_id": "JLAFKJWSCS",
      "keys": {
        "algorithms": [
          "m.olm.v1.curve25519-aes-sha2",
          "m.megolm.v1.aes-sha2"
        ],
        "device_id": "JLAFKJWSCS",
        "keys": {
          "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
          "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
        },
        "signatures": {
          "@alice:example.com": {
            "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
          }
        },
        "user_id": "@alice:example.com"
      }
    }
  ],
  "master_key": {
    "keys": {
      "ed25519:base64+master+public+key": "base64+master+public+key"
    },
    "usage": [
      "master"
    ],
    "user_id": "@alice:example.com"
  },
  "self_signing_key": {
    "keys": {
      "ed25519:base64+self+signing+public+key": "base64+self+signing+master+public+key"
    },
    "signatures": {
      "@alice:example.com": {
        "ed25519:base64+master+public+key": "signature+of+self+signing+key"
      }
    },
    "usage": [
      "self_signing"
    ],
    "user_id": "@alice:example.com"
  },
  "stream_id": 5,
  "user_id": "@alice:example.org"
}
```

## `m.device_list_update`

---

**Added in `v1.1`**

An EDU that lets servers push details to each other when one of their users adds a new device to their account, required for E2E encryption to correctly target the current set of devices for a given user. This event will also be sent when an existing device gets a new cross-signing signature.

|m.device_list_update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Device List Update](https://spec.matrix.org/v1.11/server-server-api/#definition-mdevice_list_update_device-list-update)|**Required:** The description of the device whose details has changed.|
|`edu_type`|`string`|**Required:** The string `m.device_list_update`.<br><br>One of: `[m.device_list_update]`.|

|Device List Update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`deleted`|`boolean`|True if the server is announcing that this device has been deleted.|
|`device_display_name`|`string`|The public human-readable name of this device. Will be absent if the device has no name.|
|`device_id`|`string`|**Required:** The ID of the device whose details are changing.|
|`keys`|[DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#definition-mdevice_list_update_devicekeys)|The updated identity keys (if any) for this device. May be absent if the device has no E2E keys defined.|
|`prev_id`|`[integer]`|The stream_ids of any prior m.device_list_update EDUs sent for this user which have not been referred to already in an EDU’s prev_id field. If the receiving server does not recognise any of the prev_ids, it means an EDU has been lost and the server should query a snapshot of the device list via `/user/keys/query` in order to correctly interpret future `m.device_list_update` EDUs. May be missing or empty for the first EDU in a sequence.|
|`stream_id`|`integer`|**Required:** An ID sent by the server for this update, unique for a given user_id. Used to identify any gaps in the sequence of `m.device_list_update` EDUs broadcast by a server.|
|`user_id`|`string`|**Required:** The user ID who owns this device.|

|DeviceKeys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`algorithms`|`[string]`|**Required:** The encryption algorithms supported by this device.|
|`device_id`|`string`|**Required:** The ID of the device these keys belong to. Must match the device ID used when logging in.|
|`keys`|`{string: string}`|**Required:** Public identity keys. The names of the properties should be in the format `<algorithm>:<device_id>`. The keys themselves should be encoded as specified by the key algorithm.|
|`signatures`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**Required:**<br><br>Signatures for the device key object. A map from user ID, to a map from `<algorithm>:<device_id>` to the signature.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`user_id`|`string`|**Required:** The ID of the user the device belongs to. Must match the user ID used when logging in.|

### Examples

```
{
  "content": {
    "device_display_name": "Mobile",
    "device_id": "QBUAZIFURK",
    "keys": {
      "algorithms": [
        "m.olm.v1.curve25519-aes-sha2",
        "m.megolm.v1.aes-sha2"
      ],
      "device_id": "JLAFKJWSCS",
      "keys": {
        "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
        "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
      },
      "signatures": {
        "@alice:example.com": {
          "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
        }
      },
      "user_id": "@alice:example.com"
    },
    "prev_id": [
      5
    ],
    "stream_id": 6,
    "user_id": "@john:example.com"
  },
  "edu_type": "m.device_list_update"
}
```

# End-to-End Encryption[](https://spec.matrix.org/v1.11/server-server-api/#end-to-end-encryption)

This section complements the [End-to-End Encryption module](https://spec.matrix.org/v1.11/client-server-api#end-to-end-encryption) of the Client-Server API. For detailed information about end-to-end encryption, please see that module.

The APIs defined here are designed to be able to proxy much of the client’s request through to federation, and have the response also be proxied through to the client.

### POST /_matrix/federation/v1/user/keys/claim[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysclaim)

---

Claims one-time keys for use in pre-key messages.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`one_time_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**Required:** The keys to be claimed. A map from user ID, to a map from device ID to algorithm name. Requested users must be local to the receiving homeserver.|

##### Request body example

```
{
  "one_time_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": "signed_curve25519"
    }
  }
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The claimed keys.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`one_time_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: {string: string\|[KeyObject](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysclaim_response-200_keyobject)}}}|**Required:**<br><br>One-time keys for the queried devices. A map from user ID, to a map from devices to a map from `<algorithm>:<key_id>` to the key object.<br><br>See the [Client-Server Key Algorithms](https://spec.matrix.org/v1.11/client-server-api/#key-algorithms) section for more information on the Key Object format.|

|KeyObject|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`key`|`string`|**Required:** The key, encoded using unpadded base64.|
|`signatures`|`{string: {string: string}}`|**Required:**<br><br>Signature of the key object.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|

```
{
  "one_time_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": {
        "signed_curve25519:AAAAHg": {
          "key": "zKbLg+NrIjpnagy+pIY6uPL4ZwEG2v+8F9lmgsnlZzs",
          "signatures": {
            "@alice:example.com": {
              "ed25519:JLAFKJWSCS": "FLWxXqGbwrb8SM3Y795eB6OA8bwBcoMZFXBqnTn58AYWZSqiD45tlBVcDa2L7RwdKXebW/VzDlnfVJ+9jok1Bw"
            }
          }
        }
      }
    }
  }
}
```

### POST /_matrix/federation/v1/user/keys/query[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery)

---

Returns the current devices and identity keys for the given users.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

#### Request

##### Request body

|Name|Type|Description|
|---|---|---|
|`device_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [string]}|**Required:** The keys to be downloaded. A map from user ID, to a list of device IDs, or to an empty list to indicate all devices for the corresponding user. Requested users must be local to the receiving homeserver.|

##### Request body example

```
{
  "device_keys": {
    "@alice:example.com": []
  }
}
```

---

#### Responses

|Status|Description|
|---|---|
|`200`|The device information.|

##### 200 response

|Name|Type|Description|
|---|---|---|
|`device_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: [DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_devicekeys)}}|**Required:** Information on the queried devices. A map from user ID, to a map from device ID to device information. For each device, the information returned will be the same as uploaded via `/keys/upload`, with the addition of an `unsigned` property.|
|`master_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_crosssigningkey)}|Information on the master cross-signing keys of the queried users. A map from user ID, to master key information. For each key, the information returned will be the same as uploaded via `/keys/device_signing/upload`, along with the signatures uploaded via `/keys/signatures/upload` that the user is allowed to see.<br><br>**Added in `v1.1`**|
|`self_signing_keys`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_crosssigningkey)}|Information on the self-signing keys of the queried users. A map from user ID, to self-signing key information. For each key, the information returned will be the same as uploaded via `/keys/device_signing/upload`.<br><br>**Added in `v1.1`**|

|DeviceKeys|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`algorithms`|`[string]`|**Required:** The encryption algorithms supported by this device.|
|`device_id`|`string`|**Required:** The ID of the device these keys belong to. Must match the device ID used when logging in.|
|`keys`|`{string: string}`|**Required:** Public identity keys. The names of the properties should be in the format `<algorithm>:<device_id>`. The keys themselves should be encoded as specified by the key algorithm.|
|`signatures`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**Required:**<br><br>Signatures for the device key object. A map from user ID, to a map from `<algorithm>:<device_id>` to the signature.<br><br>The signature is calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json).|
|`unsigned`|[UnsignedDeviceInfo](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_unsigneddeviceinfo)|Additional data added to the device key information by intermediate servers, and not covered by the signatures.|
|`user_id`|`string`|**Required:** The ID of the user the device belongs to. Must match the user ID used when logging in.|

|UnsignedDeviceInfo|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`device_display_name`|`string`|The display name which the user set on the device.|

|CrossSigningKey|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`keys`|`{string: string}`|**Required:** The public key. The object must have exactly one property, whose name is in the form `<algorithm>:<unpadded_base64_public_key>`, and whose value is the unpadded base64 public key.|
|`signatures`|`Signatures`|Signatures of the key, calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json). Optional for the master key. Other keys must be signed by the user's master key.|
|`usage`|`[string]`|**Required:** What the key is used for.|
|`user_id`|`string`|**Required:** The ID of the user the key belongs to.|

```
{
  "device_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": {
        "algorithms": [
          "m.olm.v1.curve25519-aes-sha2",
          "m.megolm.v1.aes-sha2"
        ],
        "device_id": "JLAFKJWSCS",
        "keys": {
          "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
          "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
        },
        "signatures": {
          "@alice:example.com": {
            "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
          }
        },
        "unsigned": {
          "device_display_name": "Alice's mobile phone"
        },
        "user_id": "@alice:example.com"
      }
    }
  }
}
```

## `m.signing_key_update`

---

**Added in `v1.1`**

An EDU that lets servers push details to each other when one of their users updates their cross-signing keys.

|m.signing_key_update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[Signing Key Update](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_signing-key-update)|**Required:** The updated signing keys.|
|`edu_type`|`string`|**Required:** The string `m.signing_update`.<br><br>One of: `[m.signing_key_update]`.|

|Signing Key Update|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`master_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_crosssigningkey)|Cross signing key|
|`self_signing_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_crosssigningkey)|Cross signing key|
|`user_id`|`string`|**Required:** The user ID whose cross-signing keys have changed.|

|CrossSigningKey|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`keys`|`{string: string}`|**Required:** The public key. The object must have exactly one property, whose name is in the form `<algorithm>:<unpadded_base64_public_key>`, and whose value is the unpadded base64 public key.|
|`signatures`|`Signatures`|Signatures of the key, calculated using the process described at [Signing JSON](https://spec.matrix.org/v1.11/appendices/#signing-json). Optional for the master key. Other keys must be signed by the user's master key.|
|`usage`|`[string]`|**Required:** What the key is used for.|
|`user_id`|`string`|**Required:** The ID of the user the key belongs to.|

### Examples

```
{
  "content": {
    "master_key": {
      "keys": {
        "ed25519:base64+master+public+key": "base64+master+public+key"
      },
      "usage": [
        "master"
      ],
      "user_id": "@alice:example.com"
    },
    "self_signing_key": {
      "keys": {
        "ed25519:base64+self+signing+public+key": "base64+self+signing+master+public+key"
      },
      "signatures": {
        "@alice:example.com": {
          "ed25519:base64+master+public+key": "signature+of+self+signing+key"
        }
      },
      "usage": [
        "self_signing"
      ],
      "user_id": "@alice:example.com"
    },
    "user_id": "@alice:example.com"
  },
  "edu_type": "m.signing_key_update"
}
```

# Send-to-device messaging[](https://spec.matrix.org/v1.11/server-server-api/#send-to-device-messaging)

The server API for send-to-device messaging is based on the `m.direct_to_device` EDU. There are no PDUs or Federation Queries involved.

Each send-to-device message should be sent to the destination server using the following EDU:

## `m.direct_to_device`

---

An EDU that lets servers push send events directly to a specific device on a remote server - for instance, to maintain an Olm E2E encrypted message channel between a local and remote device.

|m.direct_to_device|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|[To Device Message](https://spec.matrix.org/v1.11/server-server-api/#definition-mdirect_to_device_to-device-message)|**Required:** The description of the direct-to-device message.|
|`edu_type`|`string`|**Required:** The string `m.direct_to_device`.<br><br>One of: `[m.direct_to_device]`.|

|To Device Message|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`message_id`|`string`|**Required:** Unique ID for the message, used for idempotence. Arbitrary utf8 string, of maximum length 32 codepoints.|
|`messages`|{[User ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: Device Message Contents}}|**Required:** The contents of the messages to be sent. These are arranged in a map of user IDs to a map of device IDs to message bodies. The device ID may also be `*`, meaning all known devices for the user.|
|`sender`|`string`|**Required:** User ID of the sender.|
|`type`|`string`|**Required:** Event type for the message.|

### Examples

```
{
  "content": {
    "message_id": "hiezohf6Hoo7kaev",
    "messages": {
      "@alice:example.org": {
        "IWHQUZUIAH": {
          "algorithm": "m.megolm.v1.aes-sha2",
          "room_id": "!Cuyf34gef24t:localhost",
          "session_id": "X3lUlvLELLYxeTx4yOVu6UDpasGEVO0Jbu+QFnm0cKQ",
          "session_key": "AgAAAADxKHa9uFxcXzwYoNueL5Xqi69IkD4sni8LlfJL7qNBEY..."
        }
      }
    },
    "sender": "@john:example.com",
    "type": "m.room_key_request"
  },
  "edu_type": "m.direct_to_device"
}
```

# Content Repository[](https://spec.matrix.org/v1.11/server-server-api/#content-repository)

Attachments to events (images, files, etc) are uploaded to a homeserver via the Content Repository described in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/#content-repository). When a server wishes to serve content originating from a remote server, it needs to ask the remote server for the media.

Servers MUST use the server described in the [Matrix Content URI](https://spec.matrix.org/v1.11/client-server-api/#matrix-content-mxc-uris). Formatted as `mxc://{ServerName}/{MediaID}`, servers MUST download the media from `ServerName` using the below endpoints.

> [!info] RATIONALE:
> **[Changed in `v1.11`]** Servers were previously advised to use the `/_matrix/media/*` endpoints described by the [Content Repository module in the Client-Server API](https://spec.matrix.org/v1.11/client-server-api/#content-repository), however, those endpoints have been deprecated. New endpoints are introduced which require authentication. Naturally, as a server is not a user, they cannot provide the required access token to those endpoints. Instead, servers MUST try the endpoints described below before falling back to the deprecated `/_matrix/media/*` endpoints when they receive a `404 M_UNRECOGNIZED` error. When falling back, servers MUST be sure to set `allow_remote` to `false`.

### GET /_matrix/federation/v1/media/download/{mediaId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediadownloadmediaid)

---

**Added in `v1.11`**

|   |   |
|---|---|
|Rate-limited:|Yes|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`mediaId`|`string`|**Required:** The media ID from the `mxc://` URI (the path component).|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`timeout_ms`|`integer`|The maximum number of milliseconds that the client is willing to wait to start receiving data, in the case that the content has not yet been uploaded. The default value is 20000 (20 seconds). The content repository SHOULD impose a maximum value for this parameter. The content repository MAY respond before the timeout.<br><br>**Added in `v1.7`**|

---

#### Responses

|Status|Description|
|---|---|
|`200`|The content that was previously uploaded.|
|`429`|This request was rate-limited.|
|`502`|The content is too large for the server to serve.|
|`504`|The content is not yet available. A [standard error response](https://spec.matrix.org/v1.11/client-server-api/#standard-error-response) will be returned with the `errcode` `M_NOT_YET_UPLOADED`.|

##### 200 response

|Headers|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`Content-Type`|`string`|Must be `multipart/mixed`.|

|Content-Type|Description|
|---|---|
|`multipart/mixed`|**Required.** MUST contain a `boundary` (per [RFC 2046](https://datatracker.ietf.org/doc/html/rfc2046#section-5.1)) delineating exactly two parts:<br><br>The first part has a `Content-Type` header of `application/json` and describes the media’s metadata, if any. Currently, this will always be an empty object.<br><br>The second part is either:<br><br>1. the bytes of the media itself, using `Content-Type` and `Content-Disposition` headers as appropriate;<br>    <br>2. or a `Location` header to redirect the caller to where the media can be retrieved. The URL at `Location` SHOULD have appropriate `Content-Type` and `Content-Disposition` headers which describe the media.<br>    <br>    When `Location` is present, servers SHOULD NOT cache the URL. The remote server may have applied time limits on its validity. If the caller requires an up-to-date URL, it SHOULD re-request the media download.|

##### 429 response

|RateLimitError|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** The M_LIMIT_EXCEEDED error code|
|`error`|`string`|A human-readable error message.|
|`retry_after_ms`|`integer`|The amount of time in milliseconds the client should wait before trying the request again.|

```
{
  "errcode": "M_LIMIT_EXCEEDED",
  "error": "Too many requests",
  "retry_after_ms": 2000
}
```

##### 502  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to serve"
}
```

##### 504 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_NOT_YET_UPLOADED",
  "error": "Content has not yet been uploaded"
}
```

### GET /_matrix/federation/v1/media/thumbnail/{mediaId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediathumbnailmediaid)

---

**Added in `v1.11`**

Download a thumbnail of content from the content repository. See the [Client-Server API Thumbnails](https://spec.matrix.org/v1.11/client-server-api/#thumbnails) section for more information.

|   |   |
|---|---|
|Rate-limited:|Yes|
|Requires authentication:|Yes|

---

#### Request

##### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`mediaId`|`string`|**Required:** The media ID from the `mxc://` URI (the path component).|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`animated`|`boolean`|Indicates preference for an animated thumbnail from the server, if possible. Animated thumbnails typically use the content types `image/gif`, `image/png` (with APNG format), `image/apng`, and `image/webp` instead of the common static `image/png` or `image/jpeg` content types.<br><br>When `true`, the server SHOULD return an animated thumbnail if possible and supported. When `false`, the server MUST NOT return an animated thumbnail. For example, returning a static `image/png` or `image/jpeg` thumbnail. When not provided, the server SHOULD NOT return an animated thumbnail.<br><br>Servers SHOULD prefer to return `image/webp` thumbnails when supporting animation.<br><br>When `true` and the media cannot be animated, such as in the case of a JPEG or PDF, the server SHOULD behave as though `animated` is `false`.<br><br>**Added in `v1.11`**|
|`height`|`integer`|**Required:** The _desired_ height of the thumbnail. The actual thumbnail may be larger than the size specified.|
|`method`|`string`|The desired resizing method. See the [Client-Server API Thumbnails](https://spec.matrix.org/v1.11/client-server-api/#thumbnails) section for more information.<br><br>One of: `[crop, scale]`.|
|`timeout_ms`|`integer`|The maximum number of milliseconds that the client is willing to wait to start receiving data, in the case that the content has not yet been uploaded. The default value is 20000 (20 seconds). The content repository SHOULD impose a maximum value for this parameter. The content repository MAY respond before the timeout.<br><br>**Added in `v1.7`**|
|`width`|`integer`|**Required:** The _desired_ width of the thumbnail. The actual thumbnail may be larger than the size specified.|

---

#### Responses

|Status|Description|
|---|---|
|`200`|A thumbnail of the requested content.|
|`400`|The request does not make sense to the server, or the server cannot thumbnail the content. For example, the caller requested non-integer dimensions or asked for negatively-sized images.|
|`413`|The local content is too large for the server to thumbnail.|
|`429`|This request was rate-limited.|
|`502`|The remote content is too large for the server to thumbnail.|
|`504`|The content is not yet available. A [standard error response](https://spec.matrix.org/v1.11/client-server-api/#standard-error-response) will be returned with the `errcode` `M_NOT_YET_UPLOADED`.|

##### 200 response

|Headers|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`Content-Type`|`string`|Must be `multipart/mixed`.|

|Content-Type|Description|
|---|---|
|`multipart/mixed`|**Required.** MUST contain a `boundary` (per [RFC 2046](https://datatracker.ietf.org/doc/html/rfc2046#section-5.1)) delineating exactly two parts:<br><br>The first part has a `Content-Type` header of `application/json` and describes the media’s metadata, if any. Currently, this will always be an empty object.<br><br>The second part is either:<br><br>1. the bytes of the media itself, using `Content-Type` and `Content-Disposition` headers as appropriate;<br>    <br>2. or a `Location` header to redirect the caller to where the media can be retrieved. The URL at `Location` SHOULD have appropriate `Content-Type` and `Content-Disposition` headers which describe the media.<br>    <br>    When `Location` is present, servers SHOULD NOT cache the URL. The remote server may have applied time limits on its validity. If the caller requires an up-to-date URL, it SHOULD re-request the media download.<br>    <br><br>The `Content-Type` for the second part SHOULD be one of:<br><br>- `image/png` (possibly of the APNG variety)<br>- `image/apng`<br>- `image/jpeg`<br>- `image/gif`<br>- `image/webp`|

##### 400 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_UNKNOWN",
  "error": "Cannot generate thumbnails for the requested content"
}
```

##### 413 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to thumbnail"
}
```

##### 429 response

|RateLimitError|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** The M_LIMIT_EXCEEDED error code|
|`error`|`string`|A human-readable error message.|
|`retry_after_ms`|`integer`|The amount of time in milliseconds the client should wait before trying the request again.|

```
{
  "errcode": "M_LIMIT_EXCEEDED",
  "error": "Too many requests",
  "retry_after_ms": 2000
}
```

##### 502  response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to thumbnail"
}
```

##### 504 response

|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_NOT_YET_UPLOADED",
  "error": "Content has not yet been uploaded"
}
```

# Server Access Control Lists (ACLs)[](https://spec.matrix.org/v1.11/server-server-api/#server-access-control-lists-acls)

Server ACLs and their purpose are described in the [Server ACLs](https://spec.matrix.org/v1.11/client-server-api#server-access-control-lists-acls-for-rooms) section of the Client-Server API.

When a remote server makes a request, it MUST be verified to be allowed by the server ACLs. If the server is denied access to a room, the receiving server MUST reply with a 403 HTTP status code and an `errcode` of `M_FORBIDDEN`.

The following endpoint prefixes MUST be protected:

- `/_matrix/federation/v1/send` (on a per-PDU basis)
- `/_matrix/federation/v1/make_join`
- `/_matrix/federation/v1/make_leave`
- `/_matrix/federation/v1/send_join`
- `/_matrix/federation/v2/send_join`
- `/_matrix/federation/v1/send_leave`
- `/_matrix/federation/v2/send_leave`
- `/_matrix/federation/v1/invite`
- `/_matrix/federation/v2/invite`
- `/_matrix/federation/v1/make_knock`
- `/_matrix/federation/v1/send_knock`
- `/_matrix/federation/v1/state`
- `/_matrix/federation/v1/state_ids`
- `/_matrix/federation/v1/backfill`
- `/_matrix/federation/v1/event_auth`
- `/_matrix/federation/v1/get_missing_events`

# Signing Events[](https://spec.matrix.org/v1.11/server-server-api/#signing-events)

Signing events is complicated by the fact that servers can choose to redact non-essential parts of an event.

## Adding hashes and signatures to outgoing events[](https://spec.matrix.org/v1.11/server-server-api/#adding-hashes-and-signatures-to-outgoing-events)

Before signing the event, the _content hash_ of the event is calculated as described below. The hash is encoded using [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) and stored in the event object, in a `hashes` object, under a `sha256` key.

The event object is then _redacted_, following the [redaction algorithm](https://spec.matrix.org/v1.11/client-server-api#redactions). Finally it is signed as described in [Signing JSON](https://spec.matrix.org/v1.11/appendices#signing-json), using the server’s signing key (see also [Retrieving server keys](https://spec.matrix.org/v1.11/server-server-api/#retrieving-server-keys)).

The signature is then copied back to the original event object.

For an example of a signed event, see the [room version specification](https://spec.matrix.org/v1.11/rooms).

## Validating hashes and signatures on received events[](https://spec.matrix.org/v1.11/server-server-api/#validating-hashes-and-signatures-on-received-events)

When a server receives an event over federation from another server, the receiving server should check the hashes and signatures on that event.

First the signature is checked. The event is redacted following the [redaction algorithm](https://spec.matrix.org/v1.11/client-server-api#redactions), and the resultant object is checked for a signature from the originating server, following the algorithm described in [Checking for a signature](https://spec.matrix.org/v1.11/appendices#checking-for-a-signature). Note that this step should succeed whether we have been sent the full event or a redacted copy.

The signatures expected on an event are:

- The `sender`’s server, unless the invite was created as a result of 3rd party invite. The sender must already match the 3rd party invite, and the server which actually sends the event may be a different server.
- For room versions 1 and 2, the server which created the `event_id`. Other room versions do not track the `event_id` over federation and therefore do not need a signature from those servers.

If the signature is found to be valid, the expected content hash is calculated as described below. The content hash in the `hashes` property of the received event is base64-decoded, and the two are compared for equality.

If the hash check fails, then it is assumed that this is because we have only been given a redacted version of the event. To enforce this, the receiving server should use the redacted copy it calculated rather than the full copy it received.

## Calculating the reference hash for an event[](https://spec.matrix.org/v1.11/server-server-api/#calculating-the-reference-hash-for-an-event)

The _reference hash_ of an event covers the essential fields of an event, including content hashes. It is used for event identifiers in some room versions. See the [room version specification](https://spec.matrix.org/v1.11/rooms) for more information. It is calculated as follows.

1. The event is put through the redaction algorithm.
2. The `signatures` and `unsigned` properties are removed from the event, if present.
3. The event is converted into [Canonical JSON](https://spec.matrix.org/v1.11/appendices#canonical-json).
4. A sha256 hash is calculated on the resulting JSON object.

## Calculating the content hash for an event[](https://spec.matrix.org/v1.11/server-server-api/#calculating-the-content-hash-for-an-event)

The _content hash_ of an event covers the complete event including the _unredacted_ contents. It is calculated as follows.

First, any existing `unsigned`, `signature`, and `hashes` members are removed. The resulting object is then encoded as [Canonical JSON](https://spec.matrix.org/v1.11/appendices#canonical-json), and the JSON is hashed using SHA-256.

## Example code[](https://spec.matrix.org/v1.11/server-server-api/#example-code)

```python
def hash_and_sign_event(event_object, signing_key, signing_name):
    # First we need to hash the event object.
    content_hash = compute_content_hash(event_object)
    event_object["hashes"] = {"sha256": encode_unpadded_base64(content_hash)}

    # Strip all the keys that would be removed if the event was redacted.
    # The hashes are not stripped and cover all the keys in the event.
    # This means that we can tell if any of the non-essential keys are
    # modified or removed.
    stripped_object = strip_non_essential_keys(event_object)

    # Sign the stripped JSON object. The signature only covers the
    # essential keys and the hashes. This means that we can check the
    # signature even if the event is redacted.
    signed_object = sign_json(stripped_object, signing_key, signing_name)

    # Copy the signatures from the stripped event to the original event.
    event_object["signatures"] = signed_object["signatures"]

def compute_content_hash(event_object):
    # take a copy of the event before we remove any keys.
    event_object = dict(event_object)

    # Keys under "unsigned" can be modified by other servers.
    # They are useful for conveying information like the age of an
    # event that will change in transit.
    # Since they can be modified we need to exclude them from the hash.
    event_object.pop("unsigned", None)

    # Signatures will depend on the current value of the "hashes" key.
    # We cannot add new hashes without invalidating existing signatures.
    event_object.pop("signatures", None)

    # The "hashes" key might contain multiple algorithms if we decide to
    # migrate away from SHA-2. We don't want to include an existing hash
    # output in our hash so we exclude the "hashes" dict from the hash.
    event_object.pop("hashes", None)

    # Encode the JSON using a canonical encoding so that we get the same
    # bytes on every server for the same JSON object.
    event_json_bytes = encode_canonical_json(event_object)

    return hashlib.sha256(event_json_bytes)

```

# Security considerations[](https://spec.matrix.org/v1.11/server-server-api/#security-considerations)

When a domain’s ownership changes, the new controller of the domain can masquerade as the previous owner, receiving messages (similarly to email) and request past messages from other servers. In the future, proposals like [MSC1228](https://github.com/matrix-org/matrix-spec-proposals/issues/1228) will address this issue.
