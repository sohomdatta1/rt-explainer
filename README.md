# Add HTTP Method to Resource-timing entry

Proposal to add a new field `httpMethod` to PerformanceResourceTiming which is an [ByteString](https://webidl.spec.whatwg.org/#idl-ByteString) highlighting the method used to fetch a resource.

## Usecases

The addition of this field will allow for the following scenarios (per [w3c/resource-timing#373](https://github.com/w3c/resource-timing/issues/373)):

- Being able to debug and tease out specific cases where a POST request causes more overhead than a GET request and vice-versa.
- Being able to segregate the performance of URLs based of the HTTP methods used to access them (for example when monitoring a REST API)

## API Changes

The PerformanceResourceTiming interface in the [w3c/resource-timing](https://github.com/w3c/resource-timing) specification would need to be updated:

```diff
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    readonly attribute DOMString initiatorType;
    readonly attribute DOMString deliveryType;
    readonly attribute ByteString nextHopProtocol;
+   readonly attribute ByteString httpMethod
    readonly attribute DOMHighResTimeStamp workerStart;
    ...
    readonly attribute unsigned short responseStatus;
    readonly attribute RenderBlockingStatusType renderBlockingStatus;
    [Default] object toJSON();
};
```

## Example code

An example of how the `httpMethod` field can be used is shown below, in this case the code is logging all resources that were fetched using the `POST` method.

```js
const entries = performance.getEntries();
for( entry in entries ) {
    if ( entry.httpMethod === 'POST' ) {
        console.log( entry );
    }
}
```

## Specification changes

The following changes to specifications will be required to implement this feature:

- Fetch
  - New `httpMethod` field added to [fetch_timing_info](https://fetch.spec.whatwg.org/#fetch-timing-info)
  - The [Fetching algorithm](https://fetch.spec.whatwg.org/#fetching) is modified to initialize [fetch_timing_info](https://fetch.spec.whatwg.org/#fetch-timing-info)'s `httpMethod` field with the [request's method](https://fetch.spec.whatwg.org/#concept-request-method)

- ResourceTiming
  - Add `httpMethod` field to [PerformanceResourceTiming interface](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming)
  - A PerformanceResourceTiming has an associated ByteString http method.
  - The httpMethod getter steps are to return this's [normalized](https://fetch.spec.whatwg.org/#concept-method-normalize) http method.

## Security/Privacy Considerations

The field should not reveal any new cross-origin information. The ResourceTiming API only reports data about entries in the current frame, thus no cross-origin information is revealed.

Additionally, the data made available via this field can already be obtained by a page by proxying/wrapping the fetch API globally.

### [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01. What information might this feature expose to Web sites or other parties,
>      and for what purposes is that exposure necessary?

It exposes the method that is used to fetch a specific resource.

> 02. Do features in your specification expose the minimum amount of information
>      necessary to enable their intended uses?

Yes

> 03. How do the features in your specification deal with personal information,
>      personally-identifiable information (PII), or information derived from
>      them?

It does not deal with such information.

> 04. How do the features in your specification deal with sensitive information?

It does not deal with sensitive information.

> 05. Do the features in your specification introduce new state for an origin
>      that persists across browsing sessions?

No.

> 06. Do the features in your specification expose information about the
>      underlying platform to origins?

No.

> 07. Does this specification allow an origin to send data to the underlying
>      platform?

No.

> 08. Do features in this specification enable access to device sensors?

No.

> 09. Do features in this specification enable new script execution/loading
>      mechanisms?

No.

> 10. Do features in this specification allow an origin to access other devices?

No.

> 11. Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

No.

> 12. What temporary identifiers do the features in this specification create or
>      expose to the web?

None.

> 13. How does this specification distinguish between behavior in first-party and
>      third-party contexts?

No distinction.

> 14. How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

No difference.

> 15. Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

No.

> 16. Do features in your specification enable origins to downgrade default
>      security protections?

No.

> 17. How does your feature handle non-"fully active" documents?

No difference.

## Changelog
