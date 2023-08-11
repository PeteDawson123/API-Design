# Collections and Pagination

---

This guidance is mostly derived from [Microsofts API guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#9-collections).

## Serving collections over HTTP

Much like

    /users/1

should return the User entity with Id 1, the collection of Users should be returned as follows:

    /users

Collections can choose to either return a list of complete entities OR a list of summary objects, which act as an index for subsequent queries. In either case, if the number of items that could occur in such a list is unknown or can grow arbitrarily, **the collection should return a paginated list**.

## Empty Collections

Empty collections should always return a HTTP 200 and an empty array. Any other status code could indicate to the client that there was an error.

## Summary Objects

When nested collections refer to other root entities, use summary objects to reference these resources.

    /customers/1/orders -> returns summary of orders
    /orders/1 -> returns full order objects

Summary objects **MUST** contain at least the URI of the full resource, and it's ID.
Summary objects **MAY** contain other properties, but they MUST be consistent with the full resource.

Summary objects are most comment when nesting one aggregate root inside another - such as a list of references to `orders` inside a `user` entity. Situationally, summary objects may contain additional data to prevent the client `enumerating lists` for predictable values (e.g. return the `OrderReference` on an `OrderSummary` rather than having the client issue multiple requests where possible).

## Pagination

RESTful APIs that return collections SHOULD return partial sets, to allow for a better experience to clients and to avoid performance issues. Consumers of these services MUST expect partial result sets and correctly page through to retrieve an entire set.

There are two forms of pagination that MAY be supported by RESTful APIs. Server-driven paging mitigates against denial-of-service attacks by forcibly paginating a request over multiple response payloads. Client-driven paging enables clients to request only the number of resources that it can use at a given time.

Sorting and filtering parameters MUST be preserved across pages.

A sample minimal response will look like:
```json
    {
    "items": [/*...*/],
    "next": "{opaqueUrl}",
    }
```

Paginated collections properties:

- MUST have a `next` property in the response that indicates the next page of results. If the property is missing, this is the last page.
- SHOULD have a `prev` property with a link to the previous page. If a API response define such property, when missing it indicates it is the first page.
- COULD have the `first`, `last` and `self` properties pointing to the first, last and current page of the resultset.
- COULD have a `query` property containing an object defining the query and sorting used.

### Supporting Client-Driven paging

Clients MAY use `$top` and `$skip` query parameters to specify a number of results to return and an offset into the collection.

The server SHOULD honor the values specified by the client; however, clients MUST be prepared to handle responses that contain a different page size or contain a continuation token.

When both `$top` and `$skip` are given by a client, the server SHOULD first apply `$skip` and then `$top` on the collection.

Clients MAY request server-driven paging with a specific page size by specifying a `$maxpagesize` preference. The server **SHOULD** honor this preference if the specified page size is smaller than the server's default page size.

Note: If the server can't honor `$top` and/or `$skip`, the server MUST return an error to the client informing about it instead of just ignoring the query options. This will avoid the risk of the client making assumptions about the data returned.
