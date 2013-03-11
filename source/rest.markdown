REST API
--------

This guide covers the inner working of Spree’s RESTful API. It assumes a
basic understanding of the principles of
[REST](http://en.wikipedia.org/wiki/Representational_State_Transfer).
More specifically, the guide will provide information on the following:

-   The various resources available through the API
-   How to make a basic API call
-   Specific examples on how to access certain resources
-   Tools and other suggestions for working with the API

endprologue.

WARNING: We now have a dedicated online resource with very detailed
information on the REST API. Be sure to check out the [Spree API
Website](http://api.spreecommerce.com) for more details.

### Overview

The REST API is designed to give developers a convenient way to access
data contained within Spree. With a standard read/write interface to
store data, it is now very simple to write third party applications (ex.
iPhone) that can talk to Spree. It is also possible to build
sophisticated [middleware](http://en.wikipedia.org/wiki/Middleware)
applications that can serve as a bridge between Spree and a warehouse or
inventory system.

### Available Resources

INFO: The API currently only supports a limited number of resources. The
current list reflects the needs of paying clients who funded the
development of the API. The list will be expanded soon to cover
additional resources. Adding more resources is simply a matter of making
the time for testing and investigating possible security implications.

Spree currently supports RESTful access to the following resources:

-   Products
-   Variants
-   Orders
-   Line Items

### JSON Data

Developers communicate with the Spree API using the
[JSON](http://www.json.org/) data format. Requests for data are
communicated in the standard manner using the HTTP protocol.

For example, the request *GET /api/products/a-product.json* will produce
output similar to the following:

<plain>\
{\
 [product]() {\
 [id]() 1,\
 [name]() “A product”,\
 [price]() “19.99”,\
 [available\_on]() “2011-03-23T21:11:50Z”,\
 “permalink”, “a-product”\
 }\
}\
</plain>

### Making an API Call

#### Authentication

You will need an authentication token to access the API. These keys can
be generated on the user edit screen within the admin interface. To make
a request to the API, pass a *X-Spree-Token* header along with the
request:

<shell>\
curl —header “X-Spree-Token: YOUR\_KEY\_HERE”
http://example.com/api/products.json\
</shell>

Alternatively, you may also pass through the token as a parameter in the
request if a header just won’t suit your purposes (i.e. JavaScript
console debugging).

<shell>\
curl http://example.com/api/products.json?token=YOUR\_KEY\_HERE\
</shell>

The token allows the request to assume the same level of permissions as
the actual user to whom the token belongs.

#### Other Supported Formats

Currently the API supports only the [JSON](http://www.json.org/) format.

INFO: Supporting the XML format should not be difficult since Rails
supports it out of the box. Volunteers who are willing to provide a
patch (and tests) for XML support are encouraged to do so.

### API Reference

WARNING: We now have a dedicated online resource with very detailed
information on the REST API. Be sure to check out the [Spree API
Website](http://api.spreecommerce.com) for more details.

#### API Rules

-   All successful requests for the API will return a status of 200.
-   Successful create and update requests will result in a status of 201
    and 200 respectively.
-   Both create and update requests will return Spree’s representation
    of the data upon success.
-   If a create or update request fails, a status code of 422 will be
    returned, with a hash containing an “error” key, and an “errors”
    key. The errors value will contain all ActiveRecord validation
    errors encounterd when saving this record.
-   Delete requests will return status of 200, and no content.
-   Requests that list collections, such as */api/products* will return
    a limited result set back.
-   Requests that list collections can be paginated through by passing a
    *page* parameter that is a number greater than 0.
-   If a resource can not be found, the API will return a status of 404.
-   Unauthorized requests will be met with a 401 response.

Additional details and documentation for the Spree API can be found on
our [Spree API Website](http://api.spreecommerce.com).
