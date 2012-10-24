# General API Information

The Rackspace Service Registry API provides a RESTful web service
interface. All requests to authenticate and operate against the Rackspace
Rackspace Service Registry API are performed using SSL over HTTPS on TCP
port 443.

## Endpoint Access

Rackspace Service Registry runs as a separate instance in multiple
regions. You are advised to pick the region which is closest to your
servers.

During preview, only a single instance in DFW is active. You can reach it at
the address bellow:

https://dfw.registry.api.rackspacecloud.com/v1.0/1234

Replace the sample account ID number, 1234, with your actual account number.
Your account number is returned as part of the authentication service
response, after the final '/' in the X-Server-Management-Url header. See
[Authentication](general-api-information-authentication) bellow for more
information.

## Authentication

### The Authentication Endpoint

You'll send your authentication request to the Rackspace Cloud Authentication
Service using the following URL if you have a US account:

https://identity.api.rackspacecloud.com/v2.0/tokens

or the following URL if you have a UK account:

https://lon.identity.api.rackspacecloud.com/v2.0/tokens

### The Authentication Process

When sending an authentication request, you'll send your Rackspace account
username and API access key to the Rackspace Cloud Authentication Service.
After receiving your request, the Authentication Service validates your
credentials and returns an authorization token and a list of the services
you can access along with your account number. You'll need to include the
authentication token and your account number with each request you make to
the API. We'll show you how in the next section.

### Example Authentication

You'll need your username and API key in order to authenticate. You can find
your API key like so:

* Log into the Control Panel at https://mycloud.rackspacecloud.com.
* When the site loads, click your username in the upper-right corner of the
window. Your username appears as the menu title. Select API Keys from the
drop-down menu. The API Access page appears.
* From the API Access page you can generate a new key or Show/Hide an
existing key.

Once you have the API key, you can make a request to authenticate like so:

### Using cURL to Authenticate

```bash
curl -X POST https://identity.api.rackspacecloud.com/v2.0/tokens -d
'{ "auth":{ "RAX-KSKEY:apiKeyCredentials":{ "username":"theUserName", "apiKey":"00a00000a000a0000000a000a00aaa0a" } } }' -H "Content-type: application/json"

```

Successful authentication returns a token which you can use as evidence that
your identity has already been authenticated. To use the token, pass it to
Service Registry as an X-Auth-Token header.

## Request / Response Types

The Rackspace Service Registry API supports JSON data serialization
format. The request format is specified using the Content-Type header and is
required for operations that have a request body.

### Supported Response Types

Format | Accept
------ | ------
JSON | application/json

## Versions

The Rackspace Service Registry API uses a URI versioning scheme. In the
URI scheme, the first element of the path contains the target version
identifier (e.g. https://dfw.registry.api.rackspacecloud.com/v1.0/…).

```shell
GET /v1.0/1234/sessions HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

New features and functionality that do not break API-compatibility will be
introduced in the current version of the API and the URI will remain
unchanged. Features or functionality changes that would necessitate a break
in API-compatibility will require a new version, which will result in the
URI version being updated accordingly. When new API versions are released,
older versions will be marked as DEPRECATED.

## Paginated Collections

To reduce load on the service, list operations will return a maximum number
of items at a time. To navigate the collection, the parameters "limit" and
"marker" can be set in the URI (for example, ?limit=200&marker=enCCCCCC). The
marker parameter is the ID of the first item in the next page. This item
can be found in the metadata object under the `next_key` tag. Items are
sorted by the ID name in lexicographic order. The limit parameter sets the
page size. It defaults to `100` items per page and the maximum value is
`1000`. Both parameters are optional. If the client requests a limit beyond
that which is supported, an invalidLimit (400) fault may be thrown.

For convenience, collections contain a link to the next page (the
`next_href` attribute in the metadata object). If there are no more
objects, the `next_key` and next_href attributes will be empty. The
following examples illustrate two pages in a collection of services. The
first page was retrieved via a GET to
`https://dfw.api.rackspacecloud.com/v1.0/1234/services?limit=1`. In these
examples, the limit parameter sets the page size to a single item. The
subsequent next_href link will honor the initial page size. Thus, a client
may follow this link to traverse a paginated collection without having to
input the limit parameter.

### List Services, First Page (?limit=1)

```javascript
{
    "values": [
        {
            "id": "dfw1-api",
            "session_id": "seOne",
            "tags": [],
            "metadata": {}
        }
    ],
    "metadata": {
        "count": 1,
        "limit": 1,
        "marker": null,
        "next_marker": "dfw1-db1",
        "next_href": "https://todo.api.rackspacecloud.com/v1.0/7777/services?limit=1&marker=dfw1-db1"
    }
}
```

### List Services, Second Page (?marker=dfw1-messenger)

```javascript
{
    "values": [
        {
            "id": "dfw1-db1",
            "session_id": "seOne",
            "tags": [
                "db",
                "mysql"
            ],
            "metadata": {
                "region": "dfw",
                "port": "3306",
                "ip": "127.0.0.1",
                "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
            }
        }
    ],
    "metadata": {
        "count": 1,
        "limit": 100,
        "marker": "dfw1-db1",
        "next_href": null
    }
}
```

In the JSON representation, paginated collections contain a `values`
property that contains the items in the collection. The link to the next
page is located in the metadata object (`metadata.next_href` attribute).
Clients must follow the next_href link to continue to retrieve additional
entities belonging to an account.

## Rate Limits

## Faults

When an error occurs the system returns an HTTP error response code denoting
the type of error and additional information in the fault response body.

The following examples show a response body for a 404 notFoundError.

### Not Found Error Response

```javascript
{
    "type": "notFoundError",
    "code": 404,
    "message": "object does not exist",
    "details": "Object \"Session\" with key \"seNotFound\" does not exist",
    "txnId": ".rh-qyek.h-farscape.r-q3i5psGp.c-3.ts-1347320188220.v-0.1"
}
```

### Fault and Error Code Descriptions

Fault Element | Associated Error Codes | Description
---------- | ---------------- | ---------
badRequest | 400 | The system received an invalid value in a request
serviceWithThisIdExists | 400 | Service with the specified if already exists
invalidLimit | 400 | Invalid limit has been specified.
badRequestHttpsOnly | 400 | API endpoint is only accessible over a secure (https) connection
limitReached | 400 | Limit has been reached. Please contact sr@rackspace.com to increase your limit.
rateLimitReached | 400 | Rate limit has been reached. Please wait before trying again.
unauthorizedError | 401 | The system received a request from a user that is not authenticated.
forbiddenError | 403 | The system received a request that the user is forbidden to make.
notFoundError | 404 | The URL requested is not found in the system.
requestTooLargeError | 413 | The response body is too large.
internalError | 500 | The system suffered an internal failure.
notImplementedError | 501 | The request is for a feature that has not yet been implemented.
systemFailureError | 503 | The system is experiencing heavy load or another system failure.

