
> [Matrix Specification](https://spec.matrix.org/v1.11/) | [Changelog](https://spec.matrix.org/v1.11/changelog/)

# Changelog

This is version **v1.11** of the Matrix specification.

## v1.11[](https://spec.matrix.org/v1.11/changelog/#v111)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.11](https://github.com/matrix-org/matrix-spec/tree/v1.11)|
|Release date|June 20, 2024|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api)

**Deprecations**

- Authentication using a query string is now deprecated, as per [MSC4126](https://github.com/matrix-org/matrix-spec-proposals/issues/4126). The `Authorization` header should be used instead. ([#1808](https://github.com/matrix-org/matrix-spec/issues/1808))
- Use of the `/_matrix/media/*` endpoints is now deprecated. New, authenticated, endpoints are available instead. ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))

**New Endpoints**

- [`GET /_matrix/client/v1/media/config`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1mediaconfig) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- [`GET /_matrix/client/v1/media/download/{serverName}/{mediaId}`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1mediadownloadservernamemediaid) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- [`GET /_matrix/client/v1/media/download/{serverName}/{mediaId}/{fileName}`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1mediadownloadservernamemediaidfilename) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- [`GET /_matrix/client/v1/media/preview_url`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1mediapreview_url) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- [`GET /_matrix/client/v1/media/thumbnail/{serverName}/{mediaId}`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1mediathumbnailservernamemediaid) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))

**Backwards Compatible Changes**

- Add support for muting in VoIP calls, as per [MSC3291](https://github.com/matrix-org/matrix-spec-proposals/pull/3291). ([#1755](https://github.com/matrix-org/matrix-spec/issues/1755))
- Add optional `animated` query string option to `GET /thumbnail`, as per [MSC2705](https://github.com/matrix-org/matrix-spec-proposals/pull/2705). ([#1757](https://github.com/matrix-org/matrix-spec/issues/1757))
- Specify terms of services at registration, as per [MSC1692](https://github.com/matrix-org/matrix-spec-proposals/pull/1692). ([#1812](https://github.com/matrix-org/matrix-spec/issues/1812))
- Add support for mathematical messages, as per [MSC2191](https://github.com/matrix-org/matrix-spec-proposals/pull/2191). ([#1816](https://github.com/matrix-org/matrix-spec/issues/1816))
- Do not require UIA when first uploading cross-signing keys, as per [MSC3967](https://github.com/matrix-org/matrix-spec-proposals/pull/3967). ([#1828](https://github.com/matrix-org/matrix-spec/issues/1828))
- Add the new `unsigned.membership` property to events, as per [MSC4115](https://github.com/matrix-org/matrix-spec-proposals/pull/4115). ([#1847](https://github.com/matrix-org/matrix-spec/issues/1847))
- Media downloads and thumbnails are now authenticated, as per [MSC3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916). ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- Some media endpoints are now consistently under `/_matrix/client/{version}/media/*` instead of `/_matrix/media/*`, as per [MSC3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916). ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))

**Spec Clarifications**

- Add `/logout` and clarify the endpoints which do not take a JSON request body. ([#1644](https://github.com/matrix-org/matrix-spec/issues/1644))
- Clarify that the `type` of the `POST /login` request must be one of the types returned by the `GET /login` response. ([#1776](https://github.com/matrix-org/matrix-spec/issues/1776))
- Link to existing grammar where possible in types. ([#1813](https://github.com/matrix-org/matrix-spec/issues/1813))
- Rename “recovery key” to “backup decryption key”. ([#1819](https://github.com/matrix-org/matrix-spec/issues/1819))
- Clarify that the device’s Ed25519 signing key should be used in QR code verification (as opposed to the device’s Curve25519 identity key). ([#1829](https://github.com/matrix-org/matrix-spec/issues/1829))
- Fix various typos throughout the specification. ([#1832](https://github.com/matrix-org/matrix-spec/issues/1832), [#1841](https://github.com/matrix-org/matrix-spec/issues/1841), [#1852](https://github.com/matrix-org/matrix-spec/issues/1852), [#1853](https://github.com/matrix-org/matrix-spec/issues/1853))
- Specify the encoding to be used when generating QR codes for device verification. ([#1839](https://github.com/matrix-org/matrix-spec/issues/1839))
- Clarify that an access token is optional on the `POST /account/password` and `POST /account/deactivate` endpoints. ([#1843](https://github.com/matrix-org/matrix-spec/issues/1843))
- Use RFC 2119 keywords more consistently. ([#1846](https://github.com/matrix-org/matrix-spec/issues/1846), [#1861](https://github.com/matrix-org/matrix-spec/issues/1861))
- Move size limits for user, room and event IDs into the appendix and clarify that the length is to be measured in bytes. ([#1850](https://github.com/matrix-org/matrix-spec/issues/1850))
- Clarify that relations recursion should be capped at a certain depth. ([#1854](https://github.com/matrix-org/matrix-spec/issues/1854))
- Add missing secrets, third-party invites and room tagging modules to feature profiles table. ([#1860](https://github.com/matrix-org/matrix-spec/issues/1860))
- Clarify when server name is used and link to the definition. ([#1862](https://github.com/matrix-org/matrix-spec/issues/1862))
- Clarify where keys reside when checking an `m.room.encrypted` event. ([#1863](https://github.com/matrix-org/matrix-spec/issues/1863))
- Clarify that `/media/v3/upload/{serverName}/{mediaId}` requires authentication. ([#1872](https://github.com/matrix-org/matrix-spec/issues/1872))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api)

**Deprecations**

- Use of the Client-Server API `/_matrix/media/*` endpoints is now deprecated. New, authenticated, endpoints are available instead. ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))

**New Endpoints**

- [`GET /_matrix/federation/v1/media/download/{mediaId}`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediadownloadmediaid) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))
- [`GET /_matrix/federation/v1/media/thumbnail/{mediaId}`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediathumbnailmediaid) ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858))

**Backwards Compatible Changes**

- Media downloads and thumbnails are now authenticated, as per [MSC3916](https://github.com/matrix-org/matrix-spec-proposals/pull/3916). ([#1858](https://github.com/matrix-org/matrix-spec/issues/1858), [#1869](https://github.com/matrix-org/matrix-spec/issues/1869))

**Spec Clarifications**

- Link to existing grammar where possible in types. ([#1813](https://github.com/matrix-org/matrix-spec/issues/1813))
- Clarify that whitespace around commas is allowed in the `X-Matrix` `Authorization` header value params list. ([#1818](https://github.com/matrix-org/matrix-spec/issues/1818))
- Clarify that the `event` field of the `/v2/send_join` response is only required when the event is signed by the resident server. ([#1834](https://github.com/matrix-org/matrix-spec/issues/1834), [#1840](https://github.com/matrix-org/matrix-spec/issues/1840))
- Replace references to RFC 7235 and RFC 7230 that are obsoleted by RFC 9110. ([#1844](https://github.com/matrix-org/matrix-spec/issues/1844))
- Fix various typos throughout the specification. ([#1877](https://github.com/matrix-org/matrix-spec/issues/1877))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api)

**Spec Clarifications**

- Clarify that appservices should be notified of events relating to the `sender_localpart` user. ([#1810](https://github.com/matrix-org/matrix-spec/issues/1810))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api)

**Deprecations**

- Authentication using a query string is now deprecated, as per [MSC4126](https://github.com/matrix-org/matrix-spec-proposals/issues/4126). The `Authorization` header should be used instead. ([#1808](https://github.com/matrix-org/matrix-spec/issues/1808))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions)

**Spec Clarifications**

- Clarify that redaction events are still subject to all applicable auth rules. ([#1824](https://github.com/matrix-org/matrix-spec/issues/1824))
- Fix various typos throughout the specification. ([#1827](https://github.com/matrix-org/matrix-spec/issues/1827), [#1848](https://github.com/matrix-org/matrix-spec/issues/1848))
- Fix the rendering of the event format for room versions 1 and 2. ([#1883](https://github.com/matrix-org/matrix-spec/issues/1883))
- Generate the Table of Contents with Hugo rather than JavaScript. ([#1884](https://github.com/matrix-org/matrix-spec/issues/1884))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices)

**Deprecations**

- Deprecate linking to events in rooms identified by alias, as per [MSC4132](https://github.com/matrix-org/matrix-spec-proposals/pull/4132). ([#1823](https://github.com/matrix-org/matrix-spec/issues/1823))

**Spec Clarifications**

- Define ‘Opaque Identifier Grammar’. ([#1791](https://github.com/matrix-org/matrix-spec/issues/1791))
- Define common cryptographic key representation. ([#1819](https://github.com/matrix-org/matrix-spec/issues/1819))
- Move size limits for user, room and event IDs into the appendix and clarify that the length is to be measured in bytes. ([#1850](https://github.com/matrix-org/matrix-spec/issues/1850))

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling)

**Spec Clarifications**

- Update the spec release process and related documentation. ([#1759](https://github.com/matrix-org/matrix-spec/issues/1759))
- Fix npm release script for `@matrix-org/spec`. ([#1765](https://github.com/matrix-org/matrix-spec/issues/1765))
- Formatting fixes in `CONTRIBUTING.rst`. ([#1769](https://github.com/matrix-org/matrix-spec/issues/1769))
- Improve rendering on mobile devices. ([#1770](https://github.com/matrix-org/matrix-spec/issues/1770), [#1771](https://github.com/matrix-org/matrix-spec/issues/1771))
- Fix the OpenAPI definition of the security schemes. ([#1772](https://github.com/matrix-org/matrix-spec/issues/1772))
- Simplify uses of `resolve-refs` partial. ([#1773](https://github.com/matrix-org/matrix-spec/issues/1773))
- Fix Hugo warnings. ([#1775](https://github.com/matrix-org/matrix-spec/issues/1775), [#1788](https://github.com/matrix-org/matrix-spec/issues/1788))
- Fix `github-labels.rst`. ([#1781](https://github.com/matrix-org/matrix-spec/issues/1781))
- Update dependencies. ([#1786](https://github.com/matrix-org/matrix-spec/issues/1786), [#1803](https://github.com/matrix-org/matrix-spec/issues/1803), [#1804](https://github.com/matrix-org/matrix-spec/issues/1804))
- Solve `allOf` recursively in OpenAPI and JSON Schemas. ([#1787](https://github.com/matrix-org/matrix-spec/issues/1787))
- Fix property type resolution in `render-object-table` partial. ([#1789](https://github.com/matrix-org/matrix-spec/issues/1789))
- Factor out common definition of `Tag` type. ([#1793](https://github.com/matrix-org/matrix-spec/issues/1793))
- Update the version of Hugo used to render the spec to v0.124.1. ([#1794](https://github.com/matrix-org/matrix-spec/issues/1794))
- Add support for pattern formats for `patternProperties`. ([#1796](https://github.com/matrix-org/matrix-spec/issues/1796))
- Clean up unnecessary `allOf`s in OpenAPI definitions. ([#1797](https://github.com/matrix-org/matrix-spec/issues/1797))
- Show information about “Additional Properties” in object tables. ([#1798](https://github.com/matrix-org/matrix-spec/issues/1798))
- Fix anchors for schemas under `oneOf`. ([#1799](https://github.com/matrix-org/matrix-spec/issues/1799))
- Use reference to `OneTimeKeys` schema in OpenAPI definitions. ([#1800](https://github.com/matrix-org/matrix-spec/issues/1800))
- Do not use the `title` of objects containing only `additionalProperties` or `patternProperties`. ([#1801](https://github.com/matrix-org/matrix-spec/issues/1801))
- Add anchors in `definition` shortcode. ([#1802](https://github.com/matrix-org/matrix-spec/issues/1802))
- Set python version for the Towncrier CI job. ([#1805](https://github.com/matrix-org/matrix-spec/issues/1805))
- Replace `set-output` with environment files in CI. ([#1806](https://github.com/matrix-org/matrix-spec/issues/1806))
- Render response headers. ([#1809](https://github.com/matrix-org/matrix-spec/issues/1809))
- Use `patternProperties` in more places with supported formats. ([#1813](https://github.com/matrix-org/matrix-spec/issues/1813))
- Add support for rendering string formats. ([#1814](https://github.com/matrix-org/matrix-spec/issues/1814))
- Refactor the OpenAPI definitions of the content repository endpoints. ([#1822](https://github.com/matrix-org/matrix-spec/issues/1822))
- Clean up pull request template. ([#1831](https://github.com/matrix-org/matrix-spec/issues/1831))
- Do not add empty arrays to examples. ([#1849](https://github.com/matrix-org/matrix-spec/issues/1849))
- Generate the Table of Contents with Hugo rather than JavaScript. ([#1851](https://github.com/matrix-org/matrix-spec/issues/1851), [#1885](https://github.com/matrix-org/matrix-spec/issues/1885))
- Fix syntax errors in the spec release issue template. ([#1856](https://github.com/matrix-org/matrix-spec/issues/1856))
- Use environment variables for Netlify build job. ([#1865](https://github.com/matrix-org/matrix-spec/issues/1865))
- Render added/changed in info on request and response content types. ([#1876](https://github.com/matrix-org/matrix-spec/issues/1876))
- Fix validation errors in generated HTML. ([#1880](https://github.com/matrix-org/matrix-spec/issues/1880))
- Ensure most generated HTML IDs are unique. ([#1881](https://github.com/matrix-org/matrix-spec/issues/1881))
- Allow to specify a prefix for generated HTML IDs of API endpoints. ([#1882](https://github.com/matrix-org/matrix-spec/issues/1882))

## v1.10[](https://spec.matrix.org/v1.11/changelog/#v110)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.10](https://github.com/matrix-org/matrix-spec/tree/v1.10)|
|Release date|March 22, 2024|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-1)

**Backwards Compatible Changes**

- Allow `/versions` to optionally accept authentication, as per [MSC4026](https://github.com/matrix-org/matrix-spec-proposals/pull/4026). ([#1728](https://github.com/matrix-org/matrix-spec/issues/1728))
- Add local erasure requests, as per [MSC4025](https://github.com/matrix-org/matrix-spec-proposals/pull/4025). ([#1730](https://github.com/matrix-org/matrix-spec/issues/1730))
- Use the `body` field as optional media caption, as per [MSC2530](https://github.com/matrix-org/matrix-spec-proposals/pull/2530). ([#1731](https://github.com/matrix-org/matrix-spec/issues/1731))
- Add server support discovery endpoint, as per [MSC1929](https://github.com/matrix-org/matrix-spec-proposals/pull/1929). ([#1733](https://github.com/matrix-org/matrix-spec/issues/1733))
- Add support for multi-stream VoIP, as per [MSC3077](https://github.com/matrix-org/matrix-spec-proposals/pull/3077). ([#1735](https://github.com/matrix-org/matrix-spec/issues/1735))
- Specify that the `Retry-After` header may be used to rate-limit a client, as per [MSC4041](https://github.com/matrix-org/matrix-spec-proposals/pull/4041). ([#1737](https://github.com/matrix-org/matrix-spec/issues/1737))
- Add support for recursion on the `GET /relations` endpoints, as per [MSC3981](https://github.com/matrix-org/matrix-spec-proposals/pull/3981). ([#1746](https://github.com/matrix-org/matrix-spec/issues/1746))

**Spec Clarifications**

- The [strike](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/strike) element is deprecated in the HTML spec. Clients should prefer [s](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/s) instead. ([#1629](https://github.com/matrix-org/matrix-spec/issues/1629))
- Clarify that read receipts should be batched by thread as well as by room. ([#1685](https://github.com/matrix-org/matrix-spec/issues/1685))
- Clarify that threads can be created based on replies. ([#1687](https://github.com/matrix-org/matrix-spec/issues/1687))
- Clarify in the reply fallbacks example that the prefix sequence should be repeated for each line. ([#1690](https://github.com/matrix-org/matrix-spec/issues/1690))
- Clarify the format of account data objects for secret storage. ([#1695](https://github.com/matrix-org/matrix-spec/issues/1695), [#1734](https://github.com/matrix-org/matrix-spec/issues/1734))
- Clarify that the key backup MAC is implemented incorrectly and does not pass the ciphertext through HMAC-SHA-256. ([#1712](https://github.com/matrix-org/matrix-spec/issues/1712))
- Clarify one-time key and fallback key types in examples. ([#1715](https://github.com/matrix-org/matrix-spec/issues/1715))
- Clarify that the HKDF calculation for SAS uses base64-encoded keys rather than the raw key bytes. ([#1719](https://github.com/matrix-org/matrix-spec/issues/1719))
- Clarify how to perform the ECDH exchange in step 12 of the SAS process. ([#1720](https://github.com/matrix-org/matrix-spec/issues/1720))
- Document the deprecation policy of HTML tags, as per [MSC4077](https://github.com/matrix-org/matrix-spec-proposals/pull/4077). ([#1732](https://github.com/matrix-org/matrix-spec/issues/1732))
- The [font](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/font) element is deprecated in the HTML spec. Clients should prefer [span](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span) with the `data-mx-bg-color` and `data-mx-color` attributes instead. ([#1739](https://github.com/matrix-org/matrix-spec/issues/1739))
- Disambiguate uses of `PublicRoomsChunk` in the `GET /hierarchy` endpoint. ([#1740](https://github.com/matrix-org/matrix-spec/issues/1740))
- Clarify that `sdpMid` and `sdpMLineIndex` are not required in `m.call.candidates`. ([#1742](https://github.com/matrix-org/matrix-spec/issues/1742))
- Fix various typos throughout the specification. ([#1748](https://github.com/matrix-org/matrix-spec/issues/1748))
- Clearly indicate that each `Content-Type` may have distinct behaviour on non-JSON requests/responses. ([#1756](https://github.com/matrix-org/matrix-spec/issues/1756))
- Clarify that the `m.push_rules` account data type cannot be set using the `/account_data` API, as per [MSC4010](https://github.com/matrix-org/matrix-spec-proposals/pull/4010). ([#1763](https://github.com/matrix-org/matrix-spec/issues/1763))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-1)

**Spec Clarifications**

- Clarify Server-Server API request signing example by using the `POST` HTTP method, as `GET` requests don’t have request bodies. ([#1721](https://github.com/matrix-org/matrix-spec/issues/1721))
- Disambiguate uses of `PublicRoomsChunk` in the `GET /hierarchy` endpoint. ([#1740](https://github.com/matrix-org/matrix-spec/issues/1740))
- Clarify that the `children_state`, `room_type` and `allowed_room_ids` properties in the items of the `children` array of the response of the `GET /hierarchy` endpoint are not required. ([#1741](https://github.com/matrix-org/matrix-spec/issues/1741))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-1)

**Spec Clarifications**

- Clarify that the `/login` and `/register` endpoints should fail when using the `m.login.application_service` login type without a valid `as_token`. ([#1744](https://github.com/matrix-org/matrix-spec/issues/1744))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-1)

No significant changes.

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-1)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-1)

**Spec Clarifications**

- For room versions 7 through 11: Clarify that `invite->knock` is not a legal transition. ([#1717](https://github.com/matrix-org/matrix-spec/issues/1717))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-1)

No significant changes.

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-1)

**Spec Clarifications**

- Update the spec release process. ([#1680](https://github.com/matrix-org/matrix-spec/issues/1680))
- Minor clarifications to the contributing guide. ([#1697](https://github.com/matrix-org/matrix-spec/issues/1697))
- Update Docsy to v0.8.0. ([#1699](https://github.com/matrix-org/matrix-spec/issues/1699), [#1762](https://github.com/matrix-org/matrix-spec/issues/1762))
- Fix npm release script for `@matrix-org/spec`. ([#1713](https://github.com/matrix-org/matrix-spec/issues/1713))
- Add some clarifications around implementation requirements for MSCs. ([#1718](https://github.com/matrix-org/matrix-spec/issues/1718))
- Update HTML templates to include links to object schema definitions. ([#1724](https://github.com/matrix-org/matrix-spec/issues/1724))
- Factor out all the common parameters of the various `/relations` apis. ([#1745](https://github.com/matrix-org/matrix-spec/issues/1745))
- Add support for `$ref` URIs containing fragments in OpenAPI definitions and JSON schemas. ([#1751](https://github.com/matrix-org/matrix-spec/issues/1751), [#1754](https://github.com/matrix-org/matrix-spec/issues/1754))

## v1.9[](https://spec.matrix.org/v1.11/changelog/#v19)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.9](https://github.com/matrix-org/matrix-spec/tree/v1.9)|
|Release date|November 29, 2023|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-2)

**Backwards Compatible Changes**

- Add the `m.rule.suppress_edits` default push rule, as per [MSC3958](https://github.com/matrix-org/matrix-spec-proposals/pull/3958). ([#1617](https://github.com/matrix-org/matrix-spec/issues/1617))

**Spec Clarifications**

- Fix `m.call.negotiate` schema and example. ([#1546](https://github.com/matrix-org/matrix-spec/issues/1546))
- Clarify that the `via` property is required for `m.space.parent` and `m.space.child` as per [MSC1772](https://github.com/matrix-org/matrix-spec-proposals/pull/1772). Contributed by @PaarthShah. ([#1618](https://github.com/matrix-org/matrix-spec/issues/1618))
- Add a note to the `/publicRooms` API that the server name is case sensitive. ([#1638](https://github.com/matrix-org/matrix-spec/issues/1638))
- Clarify that an `m.room.name` event with an absent `name` field is not expected behavior. ([#1639](https://github.com/matrix-org/matrix-spec/issues/1639))
- Fix schemas used for account data and presence events in `GET /initialSync`. ([#1647](https://github.com/matrix-org/matrix-spec/issues/1647))
- Fix various typos throughout the specification. ([#1658](https://github.com/matrix-org/matrix-spec/issues/1658), [#1661](https://github.com/matrix-org/matrix-spec/issues/1661), [#1665](https://github.com/matrix-org/matrix-spec/issues/1665))
- Fix `.m.rule.suppress_notices` push rule not being valid JSON. ([#1671](https://github.com/matrix-org/matrix-spec/issues/1671))
- Add missing properties for `event_property_is` and `event_property_contains` push conditions to `PushConditions` object. ([#1673](https://github.com/matrix-org/matrix-spec/issues/1673))
- Indicate that fallback keys should have a `fallback` property set to `true`. ([#1676](https://github.com/matrix-org/matrix-spec/issues/1676))
- Clarify that thread roots are not considered within the thread. ([#1677](https://github.com/matrix-org/matrix-spec/issues/1677))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-2)

**Spec Clarifications**

- Fix schema of `m.receipt` EDU. ([#1636](https://github.com/matrix-org/matrix-spec/issues/1636))
- Fix various typos throughout the specification. ([#1661](https://github.com/matrix-org/matrix-spec/issues/1661))
- Clarify that federation requests for non-local users are invalid. ([#1672](https://github.com/matrix-org/matrix-spec/issues/1672))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-2)

No significant changes.

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-2)

No significant changes.

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-2)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-2)

No significant changes.

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-2)

**Spec Clarifications**

- Clarify timestamp specification with respect to leap seconds. ([#1627](https://github.com/matrix-org/matrix-spec/issues/1627))
- Fix various typos throughout the specification. ([#1652](https://github.com/matrix-org/matrix-spec/issues/1652))

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-2)

**Backwards Compatible Changes**

- Add more CI checks for OpenAPI definitions and JSON Schemas. ([#1656](https://github.com/matrix-org/matrix-spec/issues/1656))
- Generate server-server OpenAPI definition. ([#1657](https://github.com/matrix-org/matrix-spec/issues/1657))

**Spec Clarifications**

- Replace all mentions of Swagger by OpenAPI. ([#1633](https://github.com/matrix-org/matrix-spec/issues/1633))
- Fix enum types in JSON schemas. ([#1634](https://github.com/matrix-org/matrix-spec/issues/1634))
- Fix schema of `m.mentions` object. ([#1635](https://github.com/matrix-org/matrix-spec/issues/1635))
- Fix rendering of `m.receipt` event in Client-Server API. ([#1637](https://github.com/matrix-org/matrix-spec/issues/1637))
- Remove required `fieldname` in appservice Protocol definition. ([#1646](https://github.com/matrix-org/matrix-spec/issues/1646))
- Fix github action workflow responsible for releasing of @matrix-org/spec package. ([#1648](https://github.com/matrix-org/matrix-spec/issues/1648))
- Upgrade GitHub actions. ([#1660](https://github.com/matrix-org/matrix-spec/issues/1660))

## v1.8[](https://spec.matrix.org/v1.11/changelog/#v18)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.8](https://github.com/matrix-org/matrix-spec/tree/v1.8)|
|Release date|August 23, 2023|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-3)

**Backwards Compatible Changes**

- Require callers to be joined to the room to report its events, as per [MSC2249](https://github.com/matrix-org/matrix-spec-proposals/pull/2249). ([#1517](https://github.com/matrix-org/matrix-spec/issues/1517))

**Spec Clarifications**

- Fix missing `type` property in the JSON schema definition of the `m.reaction` event. Contributed by @chebureki. ([#1552](https://github.com/matrix-org/matrix-spec/issues/1552))
- Make sure examples types match schema in definitions. ([#1563](https://github.com/matrix-org/matrix-spec/issues/1563))
- Allow `null` in `room_types` in `POST /publicRooms` endpoints schemas. ([#1564](https://github.com/matrix-org/matrix-spec/issues/1564))
- Fix broken header formatting. Contributed by @midnightveil. ([#1578](https://github.com/matrix-org/matrix-spec/issues/1578))
- Render binary request and response bodies. ([#1579](https://github.com/matrix-org/matrix-spec/issues/1579))
- Fix description of MAC calculation in SAS verification. ([#1590](https://github.com/matrix-org/matrix-spec/issues/1590))
- Update link to SAS emoji definition data. ([#1593](https://github.com/matrix-org/matrix-spec/issues/1593))
- Fix various typos throughout the specification. ([#1597](https://github.com/matrix-org/matrix-spec/issues/1597))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-3)

**Deprecations**

- Deprecate `matrix` SRV lookup steps during server discovery, as per [MSC4040](https://github.com/matrix-org/matrix-spec-proposals/pull/4040). ([#1624](https://github.com/matrix-org/matrix-spec/issues/1624))

**Backwards Compatible Changes**

- Add `matrix-fed` SRV lookup steps to server discovery, as per [MSC4040](https://github.com/matrix-org/matrix-spec-proposals/pull/4040). ([#1624](https://github.com/matrix-org/matrix-spec/issues/1624))

**Spec Clarifications**

- Document why `/state_ids` can respond with a 404. ([#1521](https://github.com/matrix-org/matrix-spec/issues/1521))
- Fix response definition for `POST /_matrix/federation/v1/user/keys/claim`. ([#1559](https://github.com/matrix-org/matrix-spec/issues/1559))
- Fix examples in server keys definition. ([#1560](https://github.com/matrix-org/matrix-spec/issues/1560))
- Make sure examples types match schema in definitions. ([#1563](https://github.com/matrix-org/matrix-spec/issues/1563))
- Allow `null` in `room_types` in `POST /publicRooms` endpoints schemas. ([#1564](https://github.com/matrix-org/matrix-spec/issues/1564))
- Fix broken header formatting. Contributed by @midnightveil. ([#1578](https://github.com/matrix-org/matrix-spec/issues/1578))
- Remove spurious mention of a “default port” with respect to SRV record lookup. ([#1615](https://github.com/matrix-org/matrix-spec/issues/1615))
- Switch to ordered list for server name resolution steps. ([#1623](https://github.com/matrix-org/matrix-spec/issues/1623))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-3)

**Spec Clarifications**

- Fix type of custom `fields` in thirdparty lookup queries. ([#1584](https://github.com/matrix-org/matrix-spec/issues/1584))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-3)

**Spec Clarifications**

- Make sure examples types match schema in definitions. ([#1563](https://github.com/matrix-org/matrix-spec/issues/1563))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-3)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-3)

**Backwards Compatible Changes**

- Add room version 11 as per [MSC3820](https://github.com/matrix-org/matrix-spec-proposals/pull/3820). ([#1604](https://github.com/matrix-org/matrix-spec/issues/1604))
- Move `redacts` from top level to `content` on `m.room.redaction` events in room version 11, as per [MSC2174](https://github.com/matrix-org/matrix-spec-proposals/pull/2174). ([#1604](https://github.com/matrix-org/matrix-spec/issues/1604))
- Remove `creator` from `m.room.creator` events in room version 11, as per [MSC2175](https://github.com/matrix-org/matrix-spec-proposals/pull/2175). ([#1604](https://github.com/matrix-org/matrix-spec/issues/1604))
- Remove remaining usage of `origin` from events in room version 11, as per [MSC3989](https://github.com/matrix-org/matrix-spec-proposals/pull/3989). ([#1604](https://github.com/matrix-org/matrix-spec/issues/1604))
- Update the redaction rules in room version 11, as per [MSC2176](https://github.com/matrix-org/matrix-spec-proposals/pull/2176) and [MSC3821](https://github.com/matrix-org/matrix-spec-proposals/pull/3821). ([#1604](https://github.com/matrix-org/matrix-spec/issues/1604))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-3)

**Backwards Compatible Changes**

- Allow `+` in Matrix IDs, as per [MSC4009](https://github.com/matrix-org/matrix-spec-proposals/pull/4009). ([#1583](https://github.com/matrix-org/matrix-spec/issues/1583))

**Spec Clarifications**

- Clarify spec re canonical JSON to handle negative-zero; also, give an example of negative-zero and a large power of ten. ([#1573](https://github.com/matrix-org/matrix-spec/issues/1573))

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-3)

**Backwards Compatible Changes**

- Upgrade Swagger data to OpenAPI 3.1. ([#1310](https://github.com/matrix-org/matrix-spec/issues/1310))
- Create `@matrix-org/spec` npm package to ship the SAS Emoji data definitions & translations. ([#1620](https://github.com/matrix-org/matrix-spec/issues/1620))

**Spec Clarifications**

- Update the CI to validate the file extension of changelog entries. ([#1542](https://github.com/matrix-org/matrix-spec/issues/1542))
- Disclosure sections now only display their title when collapsed. ([#1549](https://github.com/matrix-org/matrix-spec/issues/1549))
- Fix the sidebar in recent versions of Hugo. ([#1551](https://github.com/matrix-org/matrix-spec/issues/1551))
- Bump jsonschema to validate JSON Schemas against Draft 2020-12. ([#1556](https://github.com/matrix-org/matrix-spec/issues/1556))
- Use Redocly CLI to validate OpenAPI definitions. ([#1558](https://github.com/matrix-org/matrix-spec/issues/1558))
- Use tag name as the OpenAPI definition version. ([#1561](https://github.com/matrix-org/matrix-spec/issues/1561))
- Make sure version in `x-changedInMatrixVersion` is a string. ([#1562](https://github.com/matrix-org/matrix-spec/issues/1562))
- Clarify usage of ABNF for grammar in the documentation style guide. ([#1582](https://github.com/matrix-org/matrix-spec/issues/1582))
- Remove unnecessary `oneOf`s in JSON schemas. ([#1585](https://github.com/matrix-org/matrix-spec/issues/1585))
- Update the version of Hugo used to render the spec to v0.113.0. ([#1591](https://github.com/matrix-org/matrix-spec/issues/1591))
- Fix rendered changelog with new version of towncrier. ([#1598](https://github.com/matrix-org/matrix-spec/issues/1598))
- Improve the layout of tables on desktop displays. Contributed by Martin Fischer. ([#1601](https://github.com/matrix-org/matrix-spec/issues/1601))

## v1.7[](https://spec.matrix.org/v1.11/changelog/#v17)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.7](https://github.com/matrix-org/matrix-spec/tree/v1.7)|
|Release date|May 25, 2023|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-4)

**New Endpoints**

- [`POST /_matrix/media/v1/create`](https://spec.matrix.org/v1.11/client-server-api/#post_matrixmediav1create) ([#1499](https://github.com/matrix-org/matrix-spec/issues/1499))
- [`PUT /_matrix/media/v3/upload/{serverName}/{mediaId}`](https://spec.matrix.org/v1.11/client-server-api/#put_matrixmediav3uploadservernamemediaid) ([#1499](https://github.com/matrix-org/matrix-spec/issues/1499))
- [`POST /_matrix/client/v1/login/get_token`](https://spec.matrix.org/v1.11/client-server-api/#post_matrixclientv1loginget_token) ([#1530](https://github.com/matrix-org/matrix-spec/issues/1530))

**Backwards Compatible Changes**

- Changes to the server-side aggregation of `m.replace` (edit) events, as per [MSC3925](https://github.com/matrix-org/matrix-spec-proposals/pull/3925). ([#1440](https://github.com/matrix-org/matrix-spec/issues/1440), [#1525](https://github.com/matrix-org/matrix-spec/issues/1525))
- Add new push rule conditions `event_property_is` and `event_property_contains`, as per [MSC3758](https://github.com/matrix-org/matrix-spec-proposals/pull/3758) and [MSC3966](https://github.com/matrix-org/matrix-spec-proposals/pull/3966). ([#1464](https://github.com/matrix-org/matrix-spec/issues/1464))
- Add `m.annotation` relations (reactions), as per [MSC2677](https://github.com/matrix-org/matrix-spec-proposals/pull/2677). ([#1475](https://github.com/matrix-org/matrix-spec/issues/1475), [#1531](https://github.com/matrix-org/matrix-spec/issues/1531))
- Support asynchronous media uploads, as per [MSC2246](https://github.com/matrix-org/matrix-spec-proposals/pull/2246). ([#1499](https://github.com/matrix-org/matrix-spec/issues/1499), [#1510](https://github.com/matrix-org/matrix-spec/issues/1510))
- Document the `m.mentions` property; the `.m.rule.is_user_mention` and `.m.rule.is_room_mention` push rules; and other notification behaviour, as per [MSC3952](https://github.com/matrix-org/matrix-spec-proposals/pull/3952). ([#1508](https://github.com/matrix-org/matrix-spec/issues/1508))
- Improve VoIP signaling, as per [MSC2746](https://github.com/matrix-org/matrix-spec-proposals/pull/2746). ([#1511](https://github.com/matrix-org/matrix-spec/issues/1511), [#1540](https://github.com/matrix-org/matrix-spec/issues/1540))
- Update the scope of transaction IDs, as per [MSC3970](https://github.com/matrix-org/matrix-spec-proposals/pull/3970). ([#1526](https://github.com/matrix-org/matrix-spec/issues/1526))
- Add an ability to redirect media downloads, as per [MSC3860](https://github.com/matrix-org/matrix-spec-proposals/pull/3860). ([#1529](https://github.com/matrix-org/matrix-spec/issues/1529))
- Add an ability to use an existing session to log in another, as per [MSC3882](https://github.com/matrix-org/matrix-spec-proposals/pull/3882). ([#1530](https://github.com/matrix-org/matrix-spec/issues/1530))

**Spec Clarifications**

- Clarify the sections of the specification concerning aggregation of child events. ([#1424](https://github.com/matrix-org/matrix-spec/issues/1424))
- Fix various typos throughout the specification. ([#1432](https://github.com/matrix-org/matrix-spec/issues/1432), [#1442](https://github.com/matrix-org/matrix-spec/issues/1442), [#1447](https://github.com/matrix-org/matrix-spec/issues/1447), [#1455](https://github.com/matrix-org/matrix-spec/issues/1455), [#1465](https://github.com/matrix-org/matrix-spec/issues/1465), [#1500](https://github.com/matrix-org/matrix-spec/issues/1500), [#1509](https://github.com/matrix-org/matrix-spec/issues/1509))
- Clarify that reply chain fallback for threads might not be present. ([#1439](https://github.com/matrix-org/matrix-spec/issues/1439))
- Clarify what event property the content-specific push rules match against. ([#1441](https://github.com/matrix-org/matrix-spec/issues/1441))
- Clarify the semantics that make requests idempotent. ([#1449](https://github.com/matrix-org/matrix-spec/issues/1449))
- Improve documentation of how clients use push rules. ([#1461](https://github.com/matrix-org/matrix-spec/issues/1461))
- Clarify that servers should enforce a default `limit` on a filter if one is not specified. ([#1463](https://github.com/matrix-org/matrix-spec/issues/1463))
- Disambiguate using property names with dots in them during push rule processing, as per [MSC3873](https://github.com/matrix-org/matrix-spec-proposals/pull/3873) and [MSC3980](https://github.com/matrix-org/matrix-spec-proposals/pull/3980). ([#1464](https://github.com/matrix-org/matrix-spec/issues/1464))
- Fix phrasing & typography in the registration endpoint description. Contributed by @HarHarLinks. ([#1474](https://github.com/matrix-org/matrix-spec/issues/1474))
- Remove outdated text saying that `state_default` is 0 if there is no `m.room.power_levels` event in a room. ([#1479](https://github.com/matrix-org/matrix-spec/issues/1479))
- Remove fictitious `token` parameter on `/keys/query` endpoint. ([#1485](https://github.com/matrix-org/matrix-spec/issues/1485))
- Fix rendering of properties with a list of types. ([#1487](https://github.com/matrix-org/matrix-spec/issues/1487))
- Clarify parts of the cross-signing signature upload request. ([#1495](https://github.com/matrix-org/matrix-spec/issues/1495))
- Remove the `dont_notify` and `coalesce` push rule actions, as per [MSC3987](https://github.com/matrix-org/matrix-spec-proposals/pull/3987). ([#1501](https://github.com/matrix-org/matrix-spec/issues/1501))
- Clarify `m.location` scheme by partially reverting [f1f32d3](https://github.com/matrix-org/matrix-spec/commit/f1f32d3a15c325ee8aa9d2c6bafd96c38069bb53). Contributed by @HarHarLinks. ([#1507](https://github.com/matrix-org/matrix-spec/issues/1507))
- Add missing `knock_restricted` join rule to the `m.room.join_rules` schema. ([#1535](https://github.com/matrix-org/matrix-spec/issues/1535))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-4)

**Spec Clarifications**

- Fix various typos throughout the specification. ([#1431](https://github.com/matrix-org/matrix-spec/issues/1431), [#1447](https://github.com/matrix-org/matrix-spec/issues/1447), [#1466](https://github.com/matrix-org/matrix-spec/issues/1466), [#1518](https://github.com/matrix-org/matrix-spec/issues/1518))
- Fix PDU examples by removing invalid OpenAPI reference to `examples/minimal_pdu.json`. ([#1454](https://github.com/matrix-org/matrix-spec/issues/1454))
- Remove leftover `{key_id}` from `/_matrix/key/v2/server/`. ([#1473](https://github.com/matrix-org/matrix-spec/issues/1473))
- Remove extraneous `age_ts` field from the reference hash calculation section. ([#1536](https://github.com/matrix-org/matrix-spec/issues/1536))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-4)

**New Endpoints**

- [`POST /_matrix/app/v1/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixappv1ping) ([#1516](https://github.com/matrix-org/matrix-spec/issues/1516))
- [`POST /_matrix/client/v1/appservice/{appserviceId}/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping) ([#1516](https://github.com/matrix-org/matrix-spec/issues/1516))

**Backwards Compatible Changes**

- Add homeserver->appservice ping mechanism, as per [MSC2659](https://github.com/matrix-org/matrix-spec-proposals/pull/2659). Contributed by @tulir at @beeper. ([#1516](https://github.com/matrix-org/matrix-spec/issues/1516), [#1541](https://github.com/matrix-org/matrix-spec/issues/1541))

**Spec Clarifications**

- Fix various typos throughout the specification. ([#1447](https://github.com/matrix-org/matrix-spec/issues/1447))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-4)

**Spec Clarifications**

- Corrections to the response format of `/_matrix/identity/v2/store-invite`. ([#1486](https://github.com/matrix-org/matrix-spec/issues/1486))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-4)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-4)

**Spec Clarifications**

- Clarifications of event ID formats in early room versions ([#1484](https://github.com/matrix-org/matrix-spec/issues/1484))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-4)

**Spec Clarifications**

- Clarify that the term “Canonical JSON” is a specific thing within the Matrix specification. ([#1468](https://github.com/matrix-org/matrix-spec/issues/1468))
- Remove references to groups. ([#1483](https://github.com/matrix-org/matrix-spec/issues/1483))
- Clarifications of event ID formats in early room versions. ([#1484](https://github.com/matrix-org/matrix-spec/issues/1484))

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-4)

**Spec Clarifications**

- Update references to Inter font. ([#1444](https://github.com/matrix-org/matrix-spec/issues/1444))
- Endpoint disclosures now hide everything but the URL. ([#1446](https://github.com/matrix-org/matrix-spec/issues/1446))
- Wrap $ref in allOf where other attributes are present, to improve OpenAPI compliance. ([#1457](https://github.com/matrix-org/matrix-spec/issues/1457))
- Minor cleanups to the GitHub Actions workflows ([#1476](https://github.com/matrix-org/matrix-spec/issues/1476))
- Fix generation of anchors for additional properties. ([#1488](https://github.com/matrix-org/matrix-spec/issues/1488))
- Fix various typos throughout the specification. ([#1534](https://github.com/matrix-org/matrix-spec/issues/1534))
- Document more of the spec release timeline/process. ([#1538](https://github.com/matrix-org/matrix-spec/issues/1538))

## v1.6[](https://spec.matrix.org/v1.11/changelog/#v16)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.6](https://github.com/matrix-org/matrix-spec/tree/v1.6)|
|Release date|February 14, 2023|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-5)

**Backwards Compatible Changes**

- Add information on standard error responses for unknown endpoints/methods, as per [MSC3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743). ([#1347](https://github.com/matrix-org/matrix-spec/issues/1347))
- Add `/rooms/<roomID>/timestamp_to_event` endpoint, as per [MSC3030](https://github.com/matrix-org/matrix-spec-proposals/pull/3030). ([#1366](https://github.com/matrix-org/matrix-spec/issues/1366))
- Define `hkdf-hmac-sha256.v2` MAC method for SAS verification, as per [MSC3783](https://github.com/matrix-org/matrix-spec-proposals/pull/3783). ([#1412](https://github.com/matrix-org/matrix-spec/issues/1412))

**Spec Clarifications**

- Clarify the power levels integer range. ([#1169](https://github.com/matrix-org/matrix-spec/issues/1169), [#1355](https://github.com/matrix-org/matrix-spec/issues/1355))
- Clarify that `/context` always returns `event` even if `limit` is zero. Contributed by @sumnerevans at @beeper. ([#1239](https://github.com/matrix-org/matrix-spec/issues/1239))
- Clarify what fields are required when deleting a pusher ([#1321](https://github.com/matrix-org/matrix-spec/issues/1321))
- Improve the presentation of push rule kinds and actions. ([#1348](https://github.com/matrix-org/matrix-spec/issues/1348))
- Add missing description to m.call.answer schema. ([#1353](https://github.com/matrix-org/matrix-spec/issues/1353))
- Fix various typos throughout the specification. ([#1363](https://github.com/matrix-org/matrix-spec/issues/1363))
- Clarify parts of the end-to-end encryption sections. ([#1381](https://github.com/matrix-org/matrix-spec/issues/1381))
- Move login API definitions to the right heading. Contributed by @HarHarLinks. ([#1382](https://github.com/matrix-org/matrix-spec/issues/1382))
- Clarify which events will be included in Stripped State. Contributed by @andybalaam. ([#1409](https://github.com/matrix-org/matrix-spec/issues/1409))
- Add links to the spec for the definition of 3PID `medium`. ([#1417](https://github.com/matrix-org/matrix-spec/issues/1417))
- Correct the order of the default override pushrules in the spec. ([#1421](https://github.com/matrix-org/matrix-spec/issues/1421))
- Improve distinction between tags and their attributes in the rich text section. Contributed by Nico. ([#1433](https://github.com/matrix-org/matrix-spec/issues/1433))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-5)

**Breaking Changes**

- Remove `keyId` from the server `/keys` endpoints, as per [MSC3938](https://github.com/matrix-org/matrix-spec-proposals/pull/3938). ([#1350](https://github.com/matrix-org/matrix-spec/issues/1350))

**Backwards Compatible Changes**

- Add information on standard error responses for unknown endpoints/methods, as per [MSC3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743). ([#1347](https://github.com/matrix-org/matrix-spec/issues/1347))
- Add `/timestamp_to_event/<roomID>` endpoint, as per [MSC3030](https://github.com/matrix-org/matrix-spec-proposals/pull/3030). ([#1366](https://github.com/matrix-org/matrix-spec/issues/1366))
- Extend `/_matrix/federation/v2/send_join` to allow omitting membership events, per [MSC3706](https://github.com/matrix-org/matrix-doc/pull/3706). ([#1393](https://github.com/matrix-org/matrix-spec/issues/1393), [#1398](https://github.com/matrix-org/matrix-spec/issues/1398))
- Note that `/_matrix/federation/v2/send_join` should include heroes for nameless rooms, even when allowed to omit membership events, per [MSC3943](https://github.com/matrix-org/matrix-doc/pull/3943). ([#1425](https://github.com/matrix-org/matrix-spec/issues/1425))

**Spec Clarifications**

- Include examples inline instead of using a reference for invite endpoint definitions. ([#1349](https://github.com/matrix-org/matrix-spec/issues/1349))
- Fix `POST _matrix/federation/v1/user/keys/claim` response schema. ([#1351](https://github.com/matrix-org/matrix-spec/issues/1351))
- Correct the default invite level definition in the “Checks performed on receipt of a PDU” section. ([#1371](https://github.com/matrix-org/matrix-spec/issues/1371))
- Clarify that CNAMEs are permissible for server names. ([#1376](https://github.com/matrix-org/matrix-spec/issues/1376))
- Fix `edu_type` in EDU examples. ([#1383](https://github.com/matrix-org/matrix-spec/issues/1383))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-5)

**Backwards Compatible Changes**

- Add information on standard error responses for unknown endpoints/methods, as per [MSC3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743). ([#1347](https://github.com/matrix-org/matrix-spec/issues/1347))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-5)

**Backwards Compatible Changes**

- Add information on standard error responses for unknown endpoints/methods, as per [MSC3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743). ([#1347](https://github.com/matrix-org/matrix-spec/issues/1347))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-5)

**Backwards Compatible Changes**

- Add information on standard error responses for unknown endpoints/methods, as per [MSC3743](https://github.com/matrix-org/matrix-spec-proposals/pull/3743). ([#1347](https://github.com/matrix-org/matrix-spec/issues/1347))

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-5)

**Backwards Compatible Changes**

- Update the default room version to 10, as per [MSC3904](https://github.com/matrix-org/matrix-spec-proposals/pull/3904). ([#1397](https://github.com/matrix-org/matrix-spec/issues/1397))

**Spec Clarifications**

- Clarify the grammar for room versions. ([#1422](https://github.com/matrix-org/matrix-spec/issues/1422))
- Fix various typos throughout the specification. ([#1423](https://github.com/matrix-org/matrix-spec/issues/1423))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-5)

No significant changes.

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-5)

**Spec Clarifications**

- Add link to the unstable spec to the README. ([#1357](https://github.com/matrix-org/matrix-spec/issues/1357))
- Improve safety of the proposals retrieval script in the event of failure. ([#1368](https://github.com/matrix-org/matrix-spec/issues/1368))
- Rename `<content>` to `content` in the OpenAPI files for content uploads. ([#1370](https://github.com/matrix-org/matrix-spec/issues/1370))
- Stop autogenerating examples where we already have an example. ([#1384](https://github.com/matrix-org/matrix-spec/issues/1384))
- Improve formatting of definitions in the Push Notifications section. ([#1415](https://github.com/matrix-org/matrix-spec/issues/1415))

## v1.5[](https://spec.matrix.org/v1.11/changelog/#v15)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.5](https://github.com/matrix-org/matrix-spec/tree/v1.5)|
|Release date|November 17, 2022|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-6)

**Backwards Compatible Changes**

- Add `m.reference` relations, as per [MSC3267](https://github.com/matrix-org/matrix-spec-proposals/pull/3267). ([#1206](https://github.com/matrix-org/matrix-spec/issues/1206))
- Add missing documentation for `m.key.verification.request` msgtype for in-room verification. ([#1271](https://github.com/matrix-org/matrix-spec/issues/1271))

**Spec Clarifications**

- Fix various typos throughout the specification. ([#1260](https://github.com/matrix-org/matrix-spec/issues/1260), [#1265](https://github.com/matrix-org/matrix-spec/issues/1265), [#1276](https://github.com/matrix-org/matrix-spec/issues/1276))
- Fix naming of `device_one_time_keys_count` in `/sync`. ([#1266](https://github.com/matrix-org/matrix-spec/issues/1266))
- Improve display of event subtypes. ([#1283](https://github.com/matrix-org/matrix-spec/issues/1283))
- Improve documentation about ephemeral events. ([#1284](https://github.com/matrix-org/matrix-spec/issues/1284))
- Define a 400 response from `/_matrix/client/v3/directory/rooms/{roomAlias}`. ([#1286](https://github.com/matrix-org/matrix-spec/issues/1286))
- Clarify parts of the end-to-end encryption sections. ([#1294](https://github.com/matrix-org/matrix-spec/issues/1294), [#1345](https://github.com/matrix-org/matrix-spec/issues/1345))
- Various clarifications throughout the specification. ([#1306](https://github.com/matrix-org/matrix-spec/issues/1306))
- Replace `set_sound` push rule action by `set_tweak`. ([#1318](https://github.com/matrix-org/matrix-spec/issues/1318))
- Clarify the behavior of `PUT /_matrix/client/v3/pushrules/{scope}/{kind}/{ruleId}`. ([#1319](https://github.com/matrix-org/matrix-spec/issues/1319))
- Clarify that `.m.rule.master` has a higher priority than any push rule. ([#1320](https://github.com/matrix-org/matrix-spec/issues/1320))
- Require request field `refresh_token` at endpoint `POST /_matrix/client/v3/refresh`. ([#1323](https://github.com/matrix-org/matrix-spec/issues/1323))
- Fix a number of broken links in the specification. ([#1330](https://github.com/matrix-org/matrix-spec/issues/1330))
- Add example read receipt to `GET /_matrix/client/v3/sync` response example. ([#1341](https://github.com/matrix-org/matrix-spec/issues/1341))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-6)

**Spec Clarifications**

- Fix a number of broken links in the specification. ([#1330](https://github.com/matrix-org/matrix-spec/issues/1330))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-6)

**Spec Clarifications**

- Clarify that application services can only register an interest in local users, as per [MSC3905](https://github.com/matrix-org/matrix-spec-proposals/issues/3905). ([#1305](https://github.com/matrix-org/matrix-spec/issues/1305))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-6)

**Spec Clarifications**

- Fix a number of broken links in the specification. ([#1330](https://github.com/matrix-org/matrix-spec/issues/1330))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-6)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-6)

**Spec Clarifications**

- Reword the event auth rules to clarify that users cannot demote other users with the same power level. ([#1269](https://github.com/matrix-org/matrix-spec/issues/1269))
- Various clarifications to the text on event authorisation rules. ([#1270](https://github.com/matrix-org/matrix-spec/issues/1270))
- Fix a number of broken links in the specification. ([#1330](https://github.com/matrix-org/matrix-spec/issues/1330))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-6)

No significant changes.

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-6)

**Backwards Compatible Changes**

- Update docsy theme to v0.5.0 + matrix.org modifications ([https://github.com/matrix-org/docsy/commit/a0032f8db919a6c67ba6cdef2c455f105b6272a2)](https://github.com/matrix-org/docsy/commit/a0032f8db919a6c67ba6cdef2c455f105b6272a2)). ([#1295](https://github.com/matrix-org/matrix-spec/issues/1295))

**Spec Clarifications**

- Improve error messages emitted by `resolve-additional-types` template. ([#1303](https://github.com/matrix-org/matrix-spec/issues/1303))
- Fix link to API viewer. ([#1308](https://github.com/matrix-org/matrix-spec/issues/1308))
- Stop rendering the subsections of the Client-Server API and Room Versions specs as their own separate pages. ([#1317](https://github.com/matrix-org/matrix-spec/issues/1317))
- Use a link checker to ensure that we do not have broken links. ([#1329](https://github.com/matrix-org/matrix-spec/issues/1329), [#1338](https://github.com/matrix-org/matrix-spec/issues/1338))
- Update instructions to preview Swagger definitions. ([#1331](https://github.com/matrix-org/matrix-spec/issues/1331))
- Make definition anchors more unique. ([#1339](https://github.com/matrix-org/matrix-spec/issues/1339))
- Generate the unstable changelogs with towncrier, for consistency. ([#1340](https://github.com/matrix-org/matrix-spec/issues/1340))
- Update CONTRIBUTING.md to mention that non-content changes to this repo should have an “internal” changelog entry. ([#1342](https://github.com/matrix-org/matrix-spec/issues/1342))
- Update module summary table with new modules: Event Replacements, Threading and Reference Relations. ([#1344](https://github.com/matrix-org/matrix-spec/issues/1344))
- Disable RSS generation for the spec. ([#1346](https://github.com/matrix-org/matrix-spec/issues/1346))

## v1.4[](https://spec.matrix.org/v1.11/changelog/#v14)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.4](https://github.com/matrix-org/matrix-spec/tree/v1.4)|
|Release date|September 29, 2022|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-7)

**Removed Endpoints**

- Remove unused policy room sharing mechanism, as per [MSC3844](https://github.com/matrix-org/matrix-spec-proposals/pull/3844). ([#1196](https://github.com/matrix-org/matrix-spec/issues/1196))

**Backwards Compatible Changes**

- Add a `.m.rule.room.server_acl` push rule to match `m.room.server_acl` events, as per [MSC3786](https://github.com/matrix-org/matrix-spec-proposals/pull/3786). ([#1190](https://github.com/matrix-org/matrix-spec/issues/1190), [#1201](https://github.com/matrix-org/matrix-spec/issues/1201))
- Add `Cross-Origin-Resource-Policy` (CORP) headers to media repository, as per [MSC3828](https://github.com/matrix-org/matrix-spec-proposals/pull/3828). ([#1197](https://github.com/matrix-org/matrix-spec/issues/1197))
- Copy a room’s `type` when upgrading it, as per [MSC3818](https://github.com/matrix-org/matrix-spec-proposals/pull/3818). ([#1198](https://github.com/matrix-org/matrix-spec/issues/1198))
- Add `room_types` filter and `room_type` response to `/publicRooms`, as per [MSC3827](https://github.com/matrix-org/matrix-spec-proposals/pull/3827). ([#1199](https://github.com/matrix-org/matrix-spec/issues/1199))
- Add `m.replace` relations (event edits), as per [MSC2676](https://github.com/matrix-org/matrix-spec-proposals/pull/2676). ([#1211](https://github.com/matrix-org/matrix-spec/issues/1211))
- Add `m.read.private` receipts, as per [MSC2285](https://github.com/matrix-org/matrix-spec-proposals/pull/2285). ([#1216](https://github.com/matrix-org/matrix-spec/issues/1216))
- Make `m.fully_read` optional on `/read_markers`, as per [MSC2285](https://github.com/matrix-org/matrix-spec-proposals/pull/2285). ([#1216](https://github.com/matrix-org/matrix-spec/issues/1216))
- Allow `m.fully_read` markers to be set from `/receipts`, as per [MSC2285](https://github.com/matrix-org/matrix-spec-proposals/pull/2285). ([#1216](https://github.com/matrix-org/matrix-spec/issues/1216))
- Add threading via `m.thread` relations, as per [MSC3440](https://github.com/matrix-org/matrix-spec-proposals/pull/3440), [MSC3816](https://github.com/matrix-org/matrix-spec-proposals/pull/3816), [MSC3856](https://github.com/matrix-org/matrix-spec-proposals/pull/3856), and [MSC3715](https://github.com/matrix-org/matrix-spec-proposals/pull/3715). ([#1254](https://github.com/matrix-org/matrix-spec/issues/1254))
- Add per-thread notifications and read receipts, as per [MSC3771](https://github.com/matrix-org/matrix-spec-proposals/pull/3771) and [MSC3773](https://github.com/matrix-org/matrix-spec-proposals/pull/3773). ([#1255](https://github.com/matrix-org/matrix-spec/issues/1255))
- Add `thread_id` to the `/receipt` endpoint, as per [MSC3771](https://github.com/matrix-org/matrix-spec-proposals/pull/3771). ([#1261](https://github.com/matrix-org/matrix-spec/issues/1261))

**Spec Clarifications**

- Mention that the `/rooms/{roomId}/invite` endpoint will return a 200 response if the user is already invited to the room. ([#1084](https://github.com/matrix-org/matrix-spec/issues/1084))
- Fix various typos throughout the specification. ([#1135](https://github.com/matrix-org/matrix-spec/issues/1135), [#1161](https://github.com/matrix-org/matrix-spec/issues/1161), [#1164](https://github.com/matrix-org/matrix-spec/issues/1164), [#1170](https://github.com/matrix-org/matrix-spec/issues/1170), [#1180](https://github.com/matrix-org/matrix-spec/issues/1180), [#1215](https://github.com/matrix-org/matrix-spec/issues/1215), [#1238](https://github.com/matrix-org/matrix-spec/issues/1238), [#1243](https://github.com/matrix-org/matrix-spec/issues/1243), [#1263](https://github.com/matrix-org/matrix-spec/issues/1263))
- Describe return codes for account data endpoints, and clarify that per-room data does not inherit from the global data. ([#1155](https://github.com/matrix-org/matrix-spec/issues/1155))
- Clarify that policy rule globs work like ACL globs. Contributed by Nico. ([#1165](https://github.com/matrix-org/matrix-spec/issues/1165))
- Clarify the format of some structures in the End-to-end encryption module. ([#1166](https://github.com/matrix-org/matrix-spec/issues/1166))
- Add HTML anchors for object definitions in the formatted specification. ([#1174](https://github.com/matrix-org/matrix-spec/issues/1174))
- Tweak the styling of `<code>` snippets in tables rendered from OpenAPI definitions. ([#1179](https://github.com/matrix-org/matrix-spec/issues/1179))
- Update “API Standards” section to clarify how JSON is used. ([#1185](https://github.com/matrix-org/matrix-spec/issues/1185))
- Clarify that the “device_id”, “user_id” and “access_token” fields are required in the response body of `POST /_matrix/client/v3/login`. ([#1210](https://github.com/matrix-org/matrix-spec/issues/1210))
- Reinforce the relationship of refreshed access tokens to transaction IDs. ([#1236](https://github.com/matrix-org/matrix-spec/issues/1236))
- Clarify enum values by separating possible values with commas. ([#1240](https://github.com/matrix-org/matrix-spec/issues/1240))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-7)

**Backwards Compatible Changes**

- Add per-thread notifications and read receipts, as per [MSC3771](https://github.com/matrix-org/matrix-spec-proposals/pull/3771) and [MSC3773](https://github.com/matrix-org/matrix-spec-proposals/pull/3773). ([#1255](https://github.com/matrix-org/matrix-spec/issues/1255))

**Spec Clarifications**

- Add HTML anchors for object definitions in the formatted specification. ([#1174](https://github.com/matrix-org/matrix-spec/issues/1174))
- Tweak the styling of `<code>` snippets in tables rendered from OpenAPI definitions. ([#1179](https://github.com/matrix-org/matrix-spec/issues/1179))
- Update “API Standards” section to clarify how JSON is used. ([#1185](https://github.com/matrix-org/matrix-spec/issues/1185))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-7)

**Breaking Changes**

- Replace homeserver authorization approach with an `Authorization` header instead of `access_token` when talking to the application service, as per [MSC2832](https://github.com/matrix-org/matrix-spec-proposals/pull/2832). ([#1200](https://github.com/matrix-org/matrix-spec/issues/1200))

**Spec Clarifications**

- Add HTML anchors for object definitions in the formatted specification. ([#1174](https://github.com/matrix-org/matrix-spec/issues/1174))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-7)

**Spec Clarifications**

- Add HTML anchors for object definitions in the formatted specification. ([#1174](https://github.com/matrix-org/matrix-spec/issues/1174))
- Update “API Standards” section to clarify how JSON is used. ([#1185](https://github.com/matrix-org/matrix-spec/issues/1185))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-7)

**Spec Clarifications**

- Add HTML anchors for object definitions in the formatted specification. ([#1174](https://github.com/matrix-org/matrix-spec/issues/1174))

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-7)

**Spec Clarifications**

- For room versions 1 through 10, clarify that events with rejected `auth_events` must be rejected. ([#1137](https://github.com/matrix-org/matrix-spec/issues/1137))
- For room versions 2–10: correct a mistaken clarification to the state resolution algorithm. ([#1158](https://github.com/matrix-org/matrix-spec/issues/1158))
- For room versions 7 through 10: Clarify that `invite->knock` is actually a legal transition. ([#1175](https://github.com/matrix-org/matrix-spec/issues/1175))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-7)

No significant changes.

### Internal Changes/Tooling[](https://spec.matrix.org/v1.11/changelog/#internal-changestooling-7)

**Backwards Compatible Changes**

- Add internal changes changelog section. ([#1194](https://github.com/matrix-org/matrix-spec/issues/1194))

**Spec Clarifications**

- Render HTML anchors for object definition tables. ([#1191](https://github.com/matrix-org/matrix-spec/issues/1191))
- Give rendered-data sections a background and some padding. ([#1195](https://github.com/matrix-org/matrix-spec/issues/1195))
- Fix rendering of shortcodes within the client-server API. ([#1205](https://github.com/matrix-org/matrix-spec/issues/1205))
- Fix the spacing of mapping types generated from the OpenAPI spec. ([#1230](https://github.com/matrix-org/matrix-spec/issues/1230))

## v1.3[](https://spec.matrix.org/v1.11/changelog/#v13)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-spec/tree/v1.3](https://github.com/matrix-org/matrix-spec/tree/v1.3)|
|Release date|June 15, 2022|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-8)

**Deprecations**

- Deprecate the `sender_key` and `device_id` on `m.megolm.v1.aes-sha2` events, and the `sender_key` on `m.room_key_request` to-device messages, as per [MSC3700](https://github.com/matrix-org/matrix-spec-proposals/pull/3700). ([#1101](https://github.com/matrix-org/matrix-spec/issues/1101))

**Backwards Compatible Changes**

- Make `from` optional on `GET /_matrix/client/v3/messages` to allow requesting events from the start or end of the room history, as per [MSC3567](https://github.com/matrix-org/matrix-spec-proposals/pull/3567). ([#1002](https://github.com/matrix-org/matrix-spec/issues/1002))
- Add refresh tokens, per [MSC2918](https://github.com/matrix-org/matrix-spec-proposals/pull/2918). ([#1056](https://github.com/matrix-org/matrix-spec/issues/1056), [#1113](https://github.com/matrix-org/matrix-spec/issues/1113))
- Relax the restrictions on Rich Replies, as per [MSC3676](https://github.com/matrix-org/matrix-spec-proposals/pull/3676). ([#1062](https://github.com/matrix-org/matrix-spec/issues/1062))
- Describe a structured system for event relationships, as per [MSC2674](https://github.com/matrix-org/matrix-spec-proposals/pull/2674). ([#1062](https://github.com/matrix-org/matrix-spec/issues/1062))
- Describe how relationships between events can be “aggregated”, as per [MSC2675](https://github.com/matrix-org/matrix-spec-proposals/pull/2675) and [MSC3666](https://github.com/matrix-org/matrix-spec-proposals/pull/3666). ([#1062](https://github.com/matrix-org/matrix-spec/issues/1062))
- Add support for a new `knock_restricted` join rule in supported room versions, as per [MSC3787](https://github.com/matrix-org/matrix-spec-proposals/pull/3787). ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))

**Spec Clarifications**

- Clarify that the url field in `m.room.avatar` events is not required. ([#987](https://github.com/matrix-org/matrix-spec/issues/987))
- Clarify that the `type` in user-interactive authentication can be omitted. ([#989](https://github.com/matrix-org/matrix-spec/issues/989))
- Adjust the OpenAPI specification so that the type `Flow information` is explicitly defined when the client-server API is rendered. ([#1003](https://github.com/matrix-org/matrix-spec/issues/1003))
- Correct the default value for `invite` where it is not specified in an `m.room.power_levels` event. ([#1021](https://github.com/matrix-org/matrix-spec/issues/1021))
- Update various links which pointed to the old `matrix-doc` github repository. ([#1032](https://github.com/matrix-org/matrix-spec/issues/1032))
- Removed `m.room.message.feedback` per [MSC3582](https://github.com/matrix-org/matrix-spec-proposals/pull/3582). ([#1035](https://github.com/matrix-org/matrix-spec/issues/1035))
- Fix various typos throughout the specification. ([#1051](https://github.com/matrix-org/matrix-spec/issues/1051), [#1054](https://github.com/matrix-org/matrix-spec/issues/1054), [#1059](https://github.com/matrix-org/matrix-spec/issues/1059), [#1081](https://github.com/matrix-org/matrix-spec/issues/1081), [#1097](https://github.com/matrix-org/matrix-spec/issues/1097), [#1110](https://github.com/matrix-org/matrix-spec/issues/1110), [#1115](https://github.com/matrix-org/matrix-spec/issues/1115), [#1126](https://github.com/matrix-org/matrix-spec/issues/1126), [#1127](https://github.com/matrix-org/matrix-spec/issues/1127), [#1128](https://github.com/matrix-org/matrix-spec/issues/1128), [#1129](https://github.com/matrix-org/matrix-spec/issues/1129), [#3681](https://github.com/matrix-org/matrix-spec-proposals/issues/3681), [#3708](https://github.com/matrix-org/matrix-spec-proposals/issues/3708))
- Clarify that state keys starting with `@` are in fact reserved. Regressed from [#3658](https://github.com/matrix-org/matrix-spec-proposals/pull/3658). ([#1100](https://github.com/matrix-org/matrix-spec/issues/1100))
- Remove unenforced size limit on the `name` field of `m.room.name` events. ([#3669](https://github.com/matrix-org/matrix-spec-proposals/issues/3669))
- Remove erroneous `room_id` field from examples of `m.read`, `m.typing` in `/sync` and `m.fully_read` in room account data. ([#3679](https://github.com/matrix-org/matrix-spec-proposals/issues/3679))
- Clarify the behaviour of `event_match` in push rule conditions. ([#3690](https://github.com/matrix-org/matrix-spec-proposals/issues/3690))
- Fix incorrectly referenced `m.login.appservice` login identifier, instead using `m.login.application_service`. ([#3711](https://github.com/matrix-org/matrix-spec-proposals/issues/3711))
- Fix membership state transitions to denote that `invite->knock` and `external->leave` are valid transitions. ([#3730](https://github.com/matrix-org/matrix-spec-proposals/issues/3730))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-8)

**Backwards Compatible Changes**

- Add a `destination` property to the Authorization header, as per [MSC3383](https://github.com/matrix-org/matrix-spec-proposals/pull/3383). ([#1067](https://github.com/matrix-org/matrix-spec/issues/1067))

**Spec Clarifications**

- Remove largely unused `origin` field from PDUs. ([#998](https://github.com/matrix-org/matrix-spec/issues/998))
- Update various links which pointed to the old `matrix-doc` github repository. ([#1032](https://github.com/matrix-org/matrix-spec/issues/1032))
- Clarify the format for the Authorization header. ([#1038](https://github.com/matrix-org/matrix-spec/issues/1038), [#1067](https://github.com/matrix-org/matrix-spec/issues/1067))
- Clarify what a “valid event” means when performing checks on a received PDU. ([#1045](https://github.com/matrix-org/matrix-spec/issues/1045))
- Clarify that `valid_until_ts` is in milliseconds, like other timestamps used in Matrix. ([#1055](https://github.com/matrix-org/matrix-spec/issues/1055))
- Clarify that checks on PDUs should refer to the state _before_ an event. ([#1070](https://github.com/matrix-org/matrix-spec/issues/1070))
- Clarify the historical handling of non-integer power levels. ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))
- Fix various typos throughout the specification. ([#1110](https://github.com/matrix-org/matrix-spec/issues/1110))
- Correct misleading text for `/send_join` response. ([#3703](https://github.com/matrix-org/matrix-spec-proposals/issues/3703))
- Clarify that the `content` for `X-Matrix` signature validation is the parsed JSON body. ([#3727](https://github.com/matrix-org/matrix-spec-proposals/issues/3727))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-8)

**Backwards Compatible Changes**

- Add timestamp massaging as per [MSC3316](https://github.com/matrix-org/matrix-spec-proposals/pull/3316). ([#1094](https://github.com/matrix-org/matrix-spec/issues/1094))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-8)

No significant changes.

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-8)

No significant changes.

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-8)

**Backwards Compatible Changes**

- Add room version 10 as per [MSC3604](https://github.com/matrix-org/matrix-spec-proposals/pull/3604). ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))
- Enforce integer power levels in room version 10 as per [MSC3667](https://github.com/matrix-org/matrix-spec-proposals/pull/3667). ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))
- Add a `knock_restricted` join rule supported by room version 10 as per [MSC3787](https://github.com/matrix-org/matrix-spec-proposals/pull/3787). ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))
- Update the default room version to 9 as per [MSC3589](https://github.com/matrix-org/matrix-spec-proposals/pull/3589). ([#3739](https://github.com/matrix-org/matrix-spec-proposals/issues/3739))

**Spec Clarifications**

- Improve readability and understanding of the state resolution algorithms. ([#1037](https://github.com/matrix-org/matrix-spec/issues/1037), [#1042](https://github.com/matrix-org/matrix-spec/issues/1042), [#1043](https://github.com/matrix-org/matrix-spec/issues/1043), [#1120](https://github.com/matrix-org/matrix-spec/issues/1120))
- Improve readability of the authorization rules. ([#1050](https://github.com/matrix-org/matrix-spec/issues/1050))
- For room versions 8, 9, and 10: clarify which homeserver is required to sign the join event. ([#1093](https://github.com/matrix-org/matrix-spec/issues/1093))
- Clarify that room versions 1 through 9 accept stringy power levels, as noted by [MSC3667](https://github.com/matrix-org/matrix-spec-proposals/pull/3667). ([#1099](https://github.com/matrix-org/matrix-spec/issues/1099))
- For all room versions: Add `m.federate` to the authorization rules, as originally intended. ([#1103](https://github.com/matrix-org/matrix-spec/issues/1103))
- For room versions 2 through 10: More explicitly define the mainline of a power event and the mainline ordering of other events. ([#1107](https://github.com/matrix-org/matrix-spec/issues/1107))
- For room versions 7, 8, 9, and 10: fix join membership authorization rules when `join_rule` is `knock`. ([#3737](https://github.com/matrix-org/matrix-spec-proposals/issues/3737))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-8)

No significant changes.

## v1.2[](https://spec.matrix.org/v1.11/changelog/#v12)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-doc/tree/v1.2](https://github.com/matrix-org/matrix-doc/tree/v1.2)|
|Release date|February 02, 2022|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-9)

**Breaking Changes**

- The `prev_content` field is now returned inside the `unsigned` property of events, rather than at the top level, as per [MSC3442](https://github.com/matrix-org/matrix-doc/pull/3442). ([#3524](https://github.com/matrix-org/matrix-doc/issues/3524))
- The `aliases` property from the chunks returned by `/publicRooms`, as per [MSC2432](https://github.com/matrix-org/matrix-doc/pull/2432). ([#3624](https://github.com/matrix-org/matrix-doc/issues/3624))

**New Endpoints**

- Add the Space Hierarchy API (`GET /_matrix/client/v1/rooms/{roomId}/hierarchy`) as per [MSC2946](https://github.com/matrix-org/matrix-doc/pull/2946). ([#3610](https://github.com/matrix-org/matrix-doc/issues/3610))
- Add `/_matrix/client/v1/register/m.login.registration_token/validity` as per [MSC3231](https://github.com/matrix-org/matrix-doc/pull/3231). ([#3616](https://github.com/matrix-org/matrix-doc/issues/3616))

**Backwards Compatible Changes**

- Extend `/_matrix/client/r0/login` to accept a `m.login.appservice`, as per [MSC2778](https://github.com/matrix-org/matrix-doc/pull/2778). ([#3324](https://github.com/matrix-org/matrix-doc/issues/3324))
- Add support for `restricted` rooms as per [MSC3083](https://github.com/matrix-org/matrix-doc/pull/3083), [MSC3289](https://github.com/matrix-org/matrix-doc/pull/3289), and [MSC3375](https://github.com/matrix-org/matrix-doc/pull/3375). ([#3387](https://github.com/matrix-org/matrix-doc/issues/3387))
- Add `is_guest` to `/account/whoami` as per [MSC3069](https://github.com/matrix-org/matrix-doc/pull/3069). ([#3605](https://github.com/matrix-org/matrix-doc/issues/3605))
- Expand guest access to sending any room event and state event as per [MSC3419](https://github.com/matrix-org/matrix-doc/pull/3419). ([#3605](https://github.com/matrix-org/matrix-doc/issues/3605))
- Add Spaces and room types as per [MSC1772](https://github.com/matrix-org/matrix-doc/pull/1772) and [MSC2946](https://github.com/matrix-org/matrix-doc/pull/2946). ([#3610](https://github.com/matrix-org/matrix-doc/issues/3610))
- Add new `m.set_displayname`, `m.set_avatar_url`, and `m.3pid_changes` capabilities as per [MSC3283](https://github.com/matrix-org/matrix-doc/pull/3283). ([#3614](https://github.com/matrix-org/matrix-doc/issues/3614))
- Add support for fallback keys (optional keys used once one-time keys run out), as per [MSC2732](https://github.com/matrix-org/matrix-doc/pull/2732). ([#3615](https://github.com/matrix-org/matrix-doc/issues/3615))
- Add token-authenticated registration support as per [MSC3231](https://github.com/matrix-org/matrix-doc/pull/3231). ([#3616](https://github.com/matrix-org/matrix-doc/issues/3616))

**Spec Clarifications**

- Make `AesHmacSha2KeyDescription` consistent with `KeyDescription` in marking `name` as optional. ([#3481](https://github.com/matrix-org/matrix-doc/issues/3481))
- Fix various typos throughout the specification. ([#3482](https://github.com/matrix-org/matrix-doc/issues/3482), [#3495](https://github.com/matrix-org/matrix-doc/issues/3495), [#3509](https://github.com/matrix-org/matrix-doc/issues/3509), [#3535](https://github.com/matrix-org/matrix-doc/issues/3535), [#3591](https://github.com/matrix-org/matrix-doc/issues/3591), [#3601](https://github.com/matrix-org/matrix-doc/issues/3601), [#3611](https://github.com/matrix-org/matrix-doc/issues/3611), [#3671](https://github.com/matrix-org/matrix-doc/issues/3671), [#3680](https://github.com/matrix-org/matrix-doc/issues/3680))
- Explicitly mention RFC5870 in the definition of `m.location` events ([#3492](https://github.com/matrix-org/matrix-doc/issues/3492))
- Add `403 M_FORBIDDEN` error code to `/profile/{userId}` as per [MSC3550](https://github.com/matrix-org/matrix-doc/pull/3550). ([#3530](https://github.com/matrix-org/matrix-doc/issues/3530))
- Clarify the description of the `/sync` API, including a fix to an ASCII art diagram. ([#3543](https://github.com/matrix-org/matrix-doc/issues/3543))
- Clarify that `base_url` in client `well_known` may or may not include trailing slash. ([#3562](https://github.com/matrix-org/matrix-doc/issues/3562))
- Clarify which signature to check when decrypting `m.olm.v1.curve25519-aes-sha2` messages. ([#3573](https://github.com/matrix-org/matrix-doc/issues/3573))
- Clarify what “Stripped State” is and what purpose it serves, as per [MSC3173](https://github.com/matrix-org/matrix-doc/pull/3173). ([#3606](https://github.com/matrix-org/matrix-doc/issues/3606))
- Clarify how to interpret missing one-time key counts. ([#3636](https://github.com/matrix-org/matrix-doc/issues/3636))
- Correct the schema for the responses for various API endpoints. ([#3650](https://github.com/matrix-org/matrix-doc/issues/3650))
- Clarify that group mentions are no longer in the specification. ([#3652](https://github.com/matrix-org/matrix-doc/issues/3652))
- Distinguish between “federation” event format as exchanged by the Federation API, and the “client” event formats as used in the client-server and AS APIs. ([#3658](https://github.com/matrix-org/matrix-doc/issues/3658))
- Fix the rendering of the responses for various API endpoints. ([#3674](https://github.com/matrix-org/matrix-doc/issues/3674))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-9)

**New Endpoints**

- Add the Space Hierarchy API (`GET /_matrix/federation/v1/hierarchy/{roomId}`) as per [MSC2946](https://github.com/matrix-org/matrix-doc/pull/2946). ([#3610](https://github.com/matrix-org/matrix-doc/issues/3610), [#3660](https://github.com/matrix-org/matrix-doc/issues/3660))

**Backwards Compatible Changes**

- Add support for `restricted` rooms as per [MSC3083](https://github.com/matrix-org/matrix-doc/pull/3083), [MSC3289](https://github.com/matrix-org/matrix-doc/pull/3289), and [MSC3375](https://github.com/matrix-org/matrix-doc/pull/3375). ([#3387](https://github.com/matrix-org/matrix-doc/issues/3387))

**Spec Clarifications**

- Fix various typos throughout the specification. ([#3527](https://github.com/matrix-org/matrix-doc/issues/3527), [#3528](https://github.com/matrix-org/matrix-doc/issues/3528))
- Clarify that `GET /_matrix/federation/v1/event_auth/{roomId}/{eventId}` does _not_ return the auth chain for the full state of the room. ([#3583](https://github.com/matrix-org/matrix-doc/issues/3583))
- Fix the rendering of the responses for various API endpoints. ([#3674](https://github.com/matrix-org/matrix-doc/issues/3674))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-9)

**Spec Clarifications**

- Distinguish between “federation” event format as exchanged by the Federation API, and the “client” event formats as used in the client-server and AS APIs. ([#3658](https://github.com/matrix-org/matrix-doc/issues/3658))
- Fix the rendering of the responses for various API endpoints. ([#3674](https://github.com/matrix-org/matrix-doc/issues/3674))
- Correct the documentation for the response value for `GET /_matrix/app/v1/thirdparty/protocol/{protocol}`. ([#3675](https://github.com/matrix-org/matrix-doc/issues/3675))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-9)

**Backwards Compatible Changes**

- Add the `room_type` to stored invites as per [MSC3288](https://github.com/matrix-org/matrix-doc/pull/3288). ([#3610](https://github.com/matrix-org/matrix-doc/issues/3610))

**Spec Clarifications**

- Fix the rendering of the responses for various API endpoints. ([#3674](https://github.com/matrix-org/matrix-doc/issues/3674))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-9)

**Spec Clarifications**

- Fix the rendering of the responses for various API endpoints. ([#3674](https://github.com/matrix-org/matrix-doc/issues/3674))

### Room Versions[](https://spec.matrix.org/v1.11/changelog/#room-versions-9)

**Backwards Compatible Changes**

- Add Room Version 8 as per [MSC3289](https://github.com/matrix-org/matrix-doc/pull/3289). ([#3387](https://github.com/matrix-org/matrix-doc/issues/3387))
- Add Room Version 9 as per [MSC3375](https://github.com/matrix-org/matrix-doc/pull/3375). ([#3387](https://github.com/matrix-org/matrix-doc/issues/3387))

**Spec Clarifications**

- Fully specify room versions to indicate what exactly is carried over from parent versions. ([#3432](https://github.com/matrix-org/matrix-doc/issues/3432), [#3661](https://github.com/matrix-org/matrix-doc/issues/3661))
- Clarifications to sections on event IDs and event formats. ([#3501](https://github.com/matrix-org/matrix-doc/issues/3501))
- Remove a number of fields which were incorrectly shown to form part of the `unsigned` data of a Federation PDU. ([#3522](https://github.com/matrix-org/matrix-doc/issues/3522))
- Fix heading order of room version specifications to be consistent. ([#3683](https://github.com/matrix-org/matrix-doc/issues/3683))
- Add missing “Signing key validity period” section to room version 6. ([#3683](https://github.com/matrix-org/matrix-doc/issues/3683))
- Fix auth rules to allow membership of `knock` -> `leave` in v7, v8, and v9. ([#3694](https://github.com/matrix-org/matrix-doc/issues/3694))

### Appendices[](https://spec.matrix.org/v1.11/changelog/#appendices-9)

**Backwards Compatible Changes**

- Describe “Common Namespaced Identifier Grammar” as per [MSC2758](https://github.com/matrix-org/matrix-doc/pull/2758). ([#3171](https://github.com/matrix-org/matrix-doc/issues/3171))
- Describe the `matrix:` URI scheme as per [MSC2312](https://github.com/matrix-org/matrix-doc/pull/2312). ([#3608](https://github.com/matrix-org/matrix-doc/issues/3608))

## v1.1[](https://spec.matrix.org/v1.11/changelog/#v11)

|   |   |
|---|---|
|Git commit|[https://github.com/matrix-org/matrix-doc/tree/v1.1](https://github.com/matrix-org/matrix-doc/tree/v1.1)|
|Release date|November 09, 2021|

### Client-Server API[](https://spec.matrix.org/v1.11/changelog/#client-server-api-10)

**Breaking Changes**

- Document `curve25519-hkdf-sha256` key agreement method for SAS verification, and deprecate old method as per [MSC2630](https://github.com/matrix-org/matrix-doc/pull/2630). ([#2687](https://github.com/matrix-org/matrix-doc/issues/2687))
- Add `m.key.verification.ready` and `m.key.verification.done` to key verification framework as per [MSC2366](https://github.com/matrix-org/matrix-doc/pull/2366). ([#3139](https://github.com/matrix-org/matrix-doc/issues/3139))

**Deprecations**

- Deprecate starting verifications that don’t start with `m.key.verification.request` as per [MSC3122](https://github.com/matrix-org/matrix-doc/pull/3122). ([#3199](https://github.com/matrix-org/matrix-doc/issues/3199))

**New Endpoints**

- Add key backup (`/room_keys/*`) endpoints as per [MSC1219](https://github.com/matrix-org/matrix-doc/pull/1219). ([#2387](https://github.com/matrix-org/matrix-doc/issues/2387), [#2639](https://github.com/matrix-org/matrix-doc/issues/2639))
- Add `POST /keys/device_signing/upload` and `POST /keys/signatures/upload` as per [MSC1756](https://github.com/matrix-org/matrix-doc/pull/1756). ([#2536](https://github.com/matrix-org/matrix-doc/issues/2536))
- Add `/knock` endpoint as per [MSC2403](https://github.com/matrix-org/matrix-doc/pull/2403). ([#3154](https://github.com/matrix-org/matrix-doc/issues/3154))
- Add `/login/sso/redirect/{idpId}` as per [MSC2858](https://github.com/matrix-org/matrix-doc/pull/2858). ([#3163](https://github.com/matrix-org/matrix-doc/issues/3163))

**Removed Endpoints**

- Remove unimplemented `m.login.oauth2` and `m.login.token` user-interactive authentication mechanisms as per [MSC2610](https://github.com/matrix-org/matrix-doc/pull/2610) and [MSC2611](https://github.com/matrix-org/matrix-doc/pull/2611). ([#2609](https://github.com/matrix-org/matrix-doc/issues/2609))

**Backwards Compatible Changes**

- Document how clients can advise recipients that it is withholding decryption keys as per [MSC2399](https://github.com/matrix-org/matrix-doc/pull/2399). ([#2399](https://github.com/matrix-org/matrix-doc/issues/2399))
- Add cross-signing properties to the response of `POST /keys/query` as per [MSC1756](https://github.com/matrix-org/matrix-doc/pull/1756). ([#2536](https://github.com/matrix-org/matrix-doc/issues/2536))
- Document Secure Secret Storage and Sharing as per [MSC1946](https://github.com/matrix-org/matrix-doc/pull/1946) and [MSC2472](https://github.com/matrix-org/matrix-doc/pull/2472). ([#2597](https://github.com/matrix-org/matrix-doc/issues/2597))
- Add a `device_id` parameter to login fallback as per [MSC2604](https://github.com/matrix-org/matrix-doc/pull/2604). ([#2709](https://github.com/matrix-org/matrix-doc/issues/2709))
- Added a common set of translations for SAS Emoji. ([#2728](https://github.com/matrix-org/matrix-doc/issues/2728))
- Added support for `reason` on all membership events and related endpoints as per [MSC2367](https://github.com/matrix-org/matrix-doc/pull/2367). ([#2795](https://github.com/matrix-org/matrix-doc/issues/2795))
- Add a 404 `M_NOT_FOUND` error to push rule endpoints as per [MSC2663](https://github.com/matrix-org/matrix-doc/pull/2663). ([#2796](https://github.com/matrix-org/matrix-doc/issues/2796))
- Make `reason` and `score` parameters optional in the content reporting API as per [MSC2414](https://github.com/matrix-org/matrix-doc/pull/2414). ([#2807](https://github.com/matrix-org/matrix-doc/issues/2807))
- Allow guests to get the list of members for a room as per [MSC2689](https://github.com/matrix-org/matrix-doc/pull/2689). ([#2808](https://github.com/matrix-org/matrix-doc/issues/2808))
- Add support for spoilers as per [MSC2010](https://github.com/matrix-org/matrix-doc/pull/2010) and [MSC2557](https://github.com/matrix-org/matrix-doc/pull/2557), and `color` attribute as per [MSC2422](https://github.com/matrix-org/matrix-doc/pull/2422). ([#3098](https://github.com/matrix-org/matrix-doc/issues/3098))
- Add `<details>` and `<summary>` to the suggested HTML subset as per [MSC2184](https://github.com/matrix-org/matrix-doc/pull/2184). ([#3100](https://github.com/matrix-org/matrix-doc/issues/3100))
- Add key verification using in-room messages as per [MSC2241](https://github.com/matrix-org/matrix-doc/pull/2241). ([#3139](https://github.com/matrix-org/matrix-doc/issues/3139), [#3150](https://github.com/matrix-org/matrix-doc/issues/3150))
- Add information about using SSSS for cross-signing and key backup. ([#3147](https://github.com/matrix-org/matrix-doc/issues/3147))
- Add key verification method using QR codes as per [MSC1544](https://github.com/matrix-org/matrix-doc/pull/1544). ([#3149](https://github.com/matrix-org/matrix-doc/issues/3149))
- Document how clients can simplify usage of Secure Secret Storage as per [MSC2874](https://github.com/matrix-org/matrix-doc/pull/2874). ([#3151](https://github.com/matrix-org/matrix-doc/issues/3151))
- Add support for knocking, as per [MSC2403](https://github.com/matrix-org/matrix-doc/pull/2403). ([#3154](https://github.com/matrix-org/matrix-doc/issues/3154), [#3254](https://github.com/matrix-org/matrix-doc/issues/3254))
- Multiple SSO providers are possible through `m.login.sso` as per [MSC2858](https://github.com/matrix-org/matrix-doc/pull/2858). ([#3163](https://github.com/matrix-org/matrix-doc/issues/3163))
- Add `device_id` to `/account/whoami` response as per [MSC2033](https://github.com/matrix-org/matrix-doc/pull/2033). ([#3166](https://github.com/matrix-org/matrix-doc/issues/3166))
- Downgrade identity server discovery failures to `FAIL_PROMPT` as per [MSC2284](https://github.com/matrix-org/matrix-doc/pull/2284). ([#3169](https://github.com/matrix-org/matrix-doc/issues/3169))
- Re-version all endpoints to be `v3` as a starting point instead of `r0` as per [MSC2844](https://github.com/matrix-org/matrix-doc/pull/2844). ([#3421](https://github.com/matrix-org/matrix-doc/issues/3421))

**Spec Clarifications**

- Fix issues with `age` and `unsigned` being shown in the wrong places. ([#2591](https://github.com/matrix-org/matrix-doc/issues/2591))
- Fix definitions for room version capabilities. ([#2592](https://github.com/matrix-org/matrix-doc/issues/2592))
- Fix various typos throughout the specification. ([#2594](https://github.com/matrix-org/matrix-doc/issues/2594), [#2599](https://github.com/matrix-org/matrix-doc/issues/2599), [#2809](https://github.com/matrix-org/matrix-doc/issues/2809), [#2878](https://github.com/matrix-org/matrix-doc/issues/2878), [#2885](https://github.com/matrix-org/matrix-doc/issues/2885), [#2888](https://github.com/matrix-org/matrix-doc/issues/2888), [#3116](https://github.com/matrix-org/matrix-doc/issues/3116), [#3339](https://github.com/matrix-org/matrix-doc/issues/3339))
- Clarify link to OpenID Connect specification. ([#2605](https://github.com/matrix-org/matrix-doc/issues/2605))
- Clarify the behaviour of SSO login and UI-Auth. ([#2608](https://github.com/matrix-org/matrix-doc/issues/2608))
- Remove spurious `room_id` from `/sync` examples. ([#2629](https://github.com/matrix-org/matrix-doc/issues/2629))
- Reorganize information in Push Notifications module for clarity. ([#2634](https://github.com/matrix-org/matrix-doc/issues/2634))
- Improve consistency and clarity of event schema `title`s. ([#2647](https://github.com/matrix-org/matrix-doc/issues/2647))
- Fix schema issues in `m.key.verification.accept` and secret storage. ([#2653](https://github.com/matrix-org/matrix-doc/issues/2653))
- Reword “UI Authorization” to “User-Interactive Authentication” to be more clear. ([#2667](https://github.com/matrix-org/matrix-doc/issues/2667))
- Fix schemas for push rule actions to represent their alternative object form. ([#2669](https://github.com/matrix-org/matrix-doc/issues/2669))
- Fix usage of `highlight` tweak for consistency. ([#2670](https://github.com/matrix-org/matrix-doc/issues/2670))
- Clarify the behaviour of `state` for `/sync` with lazy-loading. ([#2754](https://github.com/matrix-org/matrix-doc/issues/2754))
- Clarify description of `m.room.redaction` event. ([#2814](https://github.com/matrix-org/matrix-doc/issues/2814))
- Mark `messages` as a required JSON body field in `PUT /_matrix/client/r0/sendToDevice/{eventType}/{txnId}` calls. ([#2928](https://github.com/matrix-org/matrix-doc/issues/2928))
- Correct examples of `client_secret` request body parameters so that they do not include invalid characters. ([#2985](https://github.com/matrix-org/matrix-doc/issues/2985))
- Fix example MXC URI for `m.presence`. ([#3091](https://github.com/matrix-org/matrix-doc/issues/3091))
- Clarify that event bodies are untrusted, as per [MSC2801](https://github.com/matrix-org/matrix-doc/pull/2801). ([#3099](https://github.com/matrix-org/matrix-doc/issues/3099))
- Fix the maximum event size restriction (65535 bytes -> 65536). ([#3127](https://github.com/matrix-org/matrix-doc/issues/3127))
- Update `Access-Control-Allow-Headers` recommendation to fit CORS specification. ([#3225](https://github.com/matrix-org/matrix-doc/issues/3225))
- Explicitly state that `replacement_room` is a room ID in `m.room.tombstone` events. ([#3233](https://github.com/matrix-org/matrix-doc/issues/3233))
- Clarify that all request bodies are required. ([#3238](https://github.com/matrix-org/matrix-doc/issues/3238), [#3332](https://github.com/matrix-org/matrix-doc/issues/3332))
- Add missing titles to some scheams. ([#3330](https://github.com/matrix-org/matrix-doc/issues/3330))
- Add User-Interactive Authentication fields to cross-signing APIs as per [MSC1756](https://github.com/matrix-org/matrix-doc/pull/1756). ([#3331](https://github.com/matrix-org/matrix-doc/issues/3331))
- Mention that a canonical alias event should be added when a room is created with an alias. ([#3337](https://github.com/matrix-org/matrix-doc/issues/3337))
- Add an ‘API conventions’ section to the Appendices. ([#3350](https://github.com/matrix-org/matrix-doc/issues/3350))
- Clarify the documentation around the pagination tokens used by `/sync`, `/rooms/{room_id}/messages`, `/initialSync`, `/rooms/{room_id}/initialSync`, and `/notifications`. ([#3353](https://github.com/matrix-org/matrix-doc/issues/3353))
- Remove the inaccurate ‘Pagination’ section. ([#3366](https://github.com/matrix-org/matrix-doc/issues/3366))
- Clarify how `redacted_because` is meant to work. ([#3411](https://github.com/matrix-org/matrix-doc/issues/3411))
- Remove extraneous `mimetype` from `EncryptedFile` examples, as per [MSC2582](https://github.com/matrix-org/matrix-doc/pull/2582). ([#3412](https://github.com/matrix-org/matrix-doc/issues/3412))
- Describe how [MSC2844](https://github.com/matrix-org/matrix-doc/pull/2844) affects the `/versions` endpoint. ([#3420](https://github.com/matrix-org/matrix-doc/issues/3420))
- Fix documentation errors around `threepid_creds`. ([#3471](https://github.com/matrix-org/matrix-doc/issues/3471))

### Server-Server API[](https://spec.matrix.org/v1.11/changelog/#server-server-api-10)

**New Endpoints**

- Add `/make_knock` and `/send_knock` endpoints as per [MSC2403](https://github.com/matrix-org/matrix-doc/pull/2403). ([#3154](https://github.com/matrix-org/matrix-doc/issues/3154))

**Backwards Compatible Changes**

- Add cross-signing information to `GET /user/keys` and `GET /user/devices/{userId}`, `m.device_list_update` EDU, and a new `m.signing_key_update` EDU as per [MSC1756](https://github.com/matrix-org/matrix-doc/pull/1756). ([#2536](https://github.com/matrix-org/matrix-doc/issues/2536))
- Add support for knocking, as per [MSC2403](https://github.com/matrix-org/matrix-doc/pull/2403). ([#3154](https://github.com/matrix-org/matrix-doc/issues/3154))

**Spec Clarifications**

- Specify that `GET /_matrix/federation/v1/make_join/{roomId}/{userId}` can return a 404 if the room is unknown. ([#2688](https://github.com/matrix-org/matrix-doc/issues/2688))
- Fix various typos throughout the specification. ([#2888](https://github.com/matrix-org/matrix-doc/issues/2888), [#3116](https://github.com/matrix-org/matrix-doc/issues/3116), [#3128](https://github.com/matrix-org/matrix-doc/issues/3128), [#3207](https://github.com/matrix-org/matrix-doc/issues/3207))
- Correct the `/_matrix/federation/v1/user/devices/{userId}` response which actually returns `"self_signing_key"` instead of `"self_signing_keys"`. ([#3312](https://github.com/matrix-org/matrix-doc/issues/3312))
- Explain the reasons why `<hostname>` TLS certificate is needed rather than `<delegated_hostname>` for SRV delegation. ([#3322](https://github.com/matrix-org/matrix-doc/issues/3322))
- Tweak the example PDU diagram to better demonstrate situations with multiple `prev_events`. ([#3340](https://github.com/matrix-org/matrix-doc/issues/3340))

### Application Service API[](https://spec.matrix.org/v1.11/changelog/#application-service-api-10)

**Spec Clarifications**

- Fix various typos throughout the specification. ([#2888](https://github.com/matrix-org/matrix-doc/issues/2888))

### Identity Service API[](https://spec.matrix.org/v1.11/changelog/#identity-service-api-10)

**New Endpoints**

- Add `GET /_matrix/identity/versions` API as per [MSC2320](https://github.com/matrix-org/matrix-doc/pull/2320). ([#3101](https://github.com/matrix-org/matrix-doc/issues/3101))

**Removed Endpoints**

- The v1 identity service API has been removed in favour of the v2 API, as per [MSC2713](https://github.com/matrix-org/matrix-doc/pull/2713). ([#3170](https://github.com/matrix-org/matrix-doc/issues/3170))

**Spec Clarifications**

- Fix various typos throughout the specification. ([#2888](https://github.com/matrix-org/matrix-doc/issues/2888))
- Clarify that some identifiers must be case folded prior to processing, as per [MSC2265](https://github.com/matrix-org/matrix-doc/pull/2265). ([#3167](https://github.com/matrix-org/matrix-doc/issues/3167), [#3176](https://github.com/matrix-org/matrix-doc/issues/3176))
- Describe how [MSC2844](https://github.com/matrix-org/matrix-doc/pull/2844) affects the `/versions` endpoint. ([#3459](https://github.com/matrix-org/matrix-doc/issues/3459))

### Push Gateway API[](https://spec.matrix.org/v1.11/changelog/#push-gateway-api-10)

**Spec Clarifications**

- Clarify where to get information about the various parameter values for the notify endpoint. ([#2763](https://github.com/matrix-org/matrix-doc/issues/2763))

## Historical versions[](https://spec.matrix.org/v1.11/changelog/#historical-versions)

Before version 1.1, versioning was applied at the level of individual API specifications. This section includes links to these versions of the APIs.

- **Client-Server API**
    
    - [r0.6.1](https://matrix.org/docs/spec/client_server/r0.6.1.html)
    - [r0.6.0](https://matrix.org/docs/spec/client_server/r0.6.0.html)
    - [r0.5.0](https://matrix.org/docs/spec/client_server/r0.5.0.html)
    - [r0.4.0](https://matrix.org/docs/spec/client_server/r0.4.0.html)
    - [r0.3.0](https://matrix.org/docs/spec/client_server/r0.3.0.html)
    - [r0.2.0](https://matrix.org/docs/spec/client_server/r0.2.0.html)
    - [r0.1.0](https://matrix.org/docs/spec/client_server/r0.1.0.html)
    - [r0.0.1](https://matrix.org/docs/spec/r0.0.1/client_server.html)
    - [r0.0.0](https://matrix.org/docs/spec/r0.0.0/client_server.html)
    - [Legacy](https://matrix.org/docs/spec/legacy/#client-server-api): The last draft before the spec was formally released in version r0.0.0.
- **Server-Server API**
    
    - [r0.1.4](https://matrix.org/docs/spec/server_server/r0.1.4.html)
    - [r0.1.3](https://matrix.org/docs/spec/server_server/r0.1.3.html)
    - [r0.1.2](https://matrix.org/docs/spec/server_server/r0.1.2.html)
    - [r0.1.1](https://matrix.org/docs/spec/server_server/r0.1.1.html)
    - [r0.1.0](https://matrix.org/docs/spec/server_server/r0.1.0.html)
- **Application Service API**
    
    - [r0.1.1](https://matrix.org/docs/spec/application_service/r0.1.1.html)
    - [r0.1.0](https://matrix.org/docs/spec/application_service/r0.1.0.html)
- **Identity Service API**
    
    - [r0.3.0](https://matrix.org/docs/spec/identity_service/r0.3.0.html)
    - [r0.2.1](https://matrix.org/docs/spec/identity_service/r0.2.1.html)
    - [r0.2.0](https://matrix.org/docs/spec/identity_service/r0.2.0.html)
    - [r0.1.0](https://matrix.org/docs/spec/identity_service/r0.1.0.html)
- **Push Gateway API**
    
    - [r0.1.0](https://matrix.org/docs/spec/push_gateway/r0.1.0.html)