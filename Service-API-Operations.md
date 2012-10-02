# Service API Operations

## Account

### Get limits

This endpoint returns resource and rate limits that apply to your account.

Verb | URI | Description
---- | --- | -----------
GET | /limits | Returns account limits.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### Get Limits Response

```javascript
{
    "resource": {},
    "rate": {
        "/.*": {
            "limit": 500000,
            "used": 0,
            "window": "24.0 hours"
        }
    }
}
```

## Sessions

Sessions enable clients to create persistent sessions on the server that
are used as a context for other operations. The client is responsible for
sending heartbeats to maintain the session.

### Attributes

Name | Description | Validation
---- | --- | -----------
heartbeat_timeout | Maximum time between heartbeats | Integer, Value (3..30)
metadata | Arbitrary key/value pairs. | Optional, Hash [String,String between 1 and 255 characters long:String,String between 1 and 255 characters long], Array or object with number of items between 0 and 20


### List Sessions

Verb | URI | Description
---- | --- | -----------
GET | /sessions | Returns sessions for this account.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### List Sessions Response

```javascript
{
    "values": [
        {
            "id": "sessionId",
            "metadata": {
                "region": "dfw"
            },
            "heartbeat_timeout": 30,
            "last_seen": null
        }
    ],
    "metadata": {
        "count": 1,
        "limit": 100,
        "marker": null,
        "next_href": null
    }
}
```

### Get Session

Verb | URI | Description
---- | --- | -----------
GET | /sessions/sessionId | Retrieves a single session.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### Get Session Response

```javascript
{
    "id": "sessionId",
    "metadata": {
        "region": "dfw"
    },
    "heartbeat_timeout": 30,
    "last_seen": null
}
```

### Create Session

Verb | URI | Description
---- | --- | -----------
POST | /sessions | Creates a new session.

Normal Response Code: (201) 'Location' header contains a link to the newly
created session and the body contains the token which should be used
when initially heartbeating this session.

Error Response Codes: 400, 401, 403, 500, 503

#### Session Create Request

```javascript
{
    "metadata": {
        "region": "dfw"
    },
    "heartbeat_timeout": 30
}
```

#### Session Create Response

```javascript
{
    "token": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20"
}
```

### Update Session

Verb | URI | Description
---- | --- | -----------
PUT | /sessions/sessionId | Updates an existing session.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

### Heartbeat a Session

Heartbeating is used for telling our service that the session is still
alive. The interval in which you need to hearbeat the session is dictated
by the `heartbeat_timeout` attribute on the session object.

For example, if you use a `heartbeat_timeout` of `30` seconds, this means
that you need to heartbeat the session every 30 seconds or more often.
Because of possible network delay and other factors, you are advised to
heartbeat your session at least 3 seconds sooner than the
`heartbeat_timeout`. In this example it would be every `27` seconds.

When you heartbeat a session, you are advised to use an
[HTTP/1.1 persistent connection](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html)
instead of opening a new connection for every heartbeat request.

All of the official client libraries listed on the
[Client Libraries and Tools](client-libraries-and-tools-client-libraries) page
re-use the same HTTP connection when heartbeating a session.

Note: Heartbeating a session multiple times using the same token has an
undefined behavior.

Verb | URI | Description
---- | --- | -----------
POST | /sessions/sessionId/heartbeat | Heartbeat a session.

Normal Response Code: (200), Response body contains a token which must be
used on the next heartbeat.

Error Response Codes: 400, 401, 403, 404, 500, 503

#### Session Heartbeat Response

```javascript
{
    "token": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20"
}
```

## Services

A service represents an instance of a long running process on your server.
Each service belongs to a single session which must be periodically
heartbeated to indicate that the session and its services are still alive.

### Attributes

Name | Description | Validation
---- | --- | -----------
session_id | Session id. | Immutable, String between 1 and 255 characters long
id | Service id | Immutable, String between 3 and 55 characters long, String matching the regex /[a-z0-9_-]{3,55}/i
metadata | Arbitrary key/value pairs. | Optional, Hash [String,String between 1 and 255 characters long:String,String between 1 and 255 characters long], Array or object with number of items between 0 and 20
tags | Service tags. | Optional, Array [String,String between 1 and 55 characters long], Array or object with number of items between 0 and 10


`tag` field allows you to logically group services (e.g. tagging all the API
service instances with `www`) and retrieve a list of services for a particular
tag.

`metadata` field allows you to store arbitrary key-value pairs on a service
object. Common fields include:

* service instance version (e.g. git revision hash)
* service instance IP address
* service instance port
* service instance region
* status (e.g. enabled or disabled)

### List Services

Verb | URI | Description
---- | --- | -----------
GET | /services | Returns services for this account.

This endpoint returns a list of services on your account.

There are no required parameters for this call. You may filter returned
services by `tag` with the optional query parameter `tag`.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### List Services Response (no filters)

```javascript
{
    "values": [
        {
            "id": "dfw1-api",
            "session_id": "sessionId",
            "tags": [],
            "metadata": {}
        },
        {
            "id": "dfw1-db1",
            "session_id": "sessionId",
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
        "count": 2,
        "limit": 100,
        "marker": null,
        "next_href": null
    }
}
```

#### List Services Response (filtering on a tag - /services?tag=db)

```javascript
{
    "values": [
        {
            "id": "dfw1-db1",
            "session_id": "sessionId",
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
        "marker": null,
        "next_href": null
    }
}
```

### Get Service

Verb | URI | Description
---- | --- | -----------
GET | /services/serviceId | Retrieves a single service.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### Get Service Response

```javascript
{
    "id": "dfw1-db1",
    "session_id": "sessionId",
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
```

### Create Service

Verb | URI | Description
---- | --- | -----------
POST | /services | Creates a new service.

Normal Response Code: (201) 'Location' header contains a link to the newly
created service.

Error Response Codes: 400, 401, 403, 500, 503

#### Service Create Request

```javascript
{
    "tags": [
        "db",
        "mysql"
    ],
    "metadata": {
        "region": "dfw",
        "port": "3306",
        "ip": "127.0.0.1",
        "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
    },
    "id": "dfw1-db1",
    "session_id": "sessionId"
}
```

### Update Service

Verb | URI | Description
---- | --- | -----------
PUT | /services/serviceId | Updates an existing service.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

### Delete Service

Verb | URI | Description
---- | --- | -----------
DELETE | /services/serviceId | Deletes an existing service.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

## Configuration

Configuration enables clients to store arbitrary key/value pairs on the
server and get notified via an event feed when a value is updated or
deleted.

A configuration value is an opaque string and is treated as such in our
system.

### Attributes

Name | Description | Validation
---- | --- | -----------
value | Configuration value | String between 1 and 1024 characters long
id | Configuration value id | Immutable, String between 3 and 32 characters long, String matching the regex /[a-z0-9_-]{3,32}/i


### List Configuration Values

Verb | URI | Description
---- | --- | -----------
GET | /configuration | Returns configuration values for this account.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

### Get Configuration Value

Verb | URI | Description
---- | --- | -----------
GET | /configuration/configurationId | Retrieves a single configuration value.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

### Set Configuration Value

Verb | URI | Description
---- | --- | -----------
PUT | /configuration/configurationId | Sets a configuration value.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

#### Configuration Value Set Request

```javascript
{
    "value": "test value 123456"
}
```

### Delete Configuration Value

Verb | URI | Description
---- | --- | -----------
DELETE | /configuration/configurationId | Deletes a configuration value.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

## Events

Events feed contains a list of events which occurred during the life-time
of your account. Every time a service joins a session, a configuration
value is updated or removed, or a session times out, an event is inserted.

### Event Object Attributes

Name | Description | Validation
---- | --- | -----------
timestamp | Event timestamp | Integer
type | Event type | String
id | Event id | Immutable, String between 3 and 255 characters long
payload | Event payload. | Optional, Optional, Hash [String,String between 1 and 255 characters long:String,String between 1 and 255 characters long]


### Event Types

#### service.join

This event represents a new service joining a session. The payload contains
a service object.

#### services.timeout

This event represents a session timeout, which occurs when a client doesn't
heartbeat a session within the defined timeout. The payload contains a list
of services associated with the session.

#### configuration_value.update

This event represents a configuration value being updated. The payload
contains the configuration value id, the old value, and the new value. If
this is the first time the value has been set, the old value will be `null`.

#### configuration_value.remove

This event represents a configuration value being removed. The payload
contains the configuration value id and the old value.

### List events

Verb | URI | Description
---- | --- | -----------
GET | /events | Returns a list of events for this account.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

A client can specify the optional `marker` query parameter. If the marker is
provided, only the events with ids that are newer than the  marker are
returned. If no marker is provided, events from the last 24 hours are
returned.

#### List Events Response

```javascript
{
    "values": [
        {
            "id": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20",
            "timestamp": 1346967146370,
            "type": "service.join",
            "payload": {
                "id": "dfw1-api",
                "session_id": "sessionId",
                "tags": [],
                "metadata": {}
            }
        },
        {
            "type": "service.join",
            "payload": {
                "id": "dfw1-db1",
                "session_id": "sessionId",
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
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "test value 123456",
                "configuration_value_id": "configId"
            }
        }
    ],
    "metadata": {
        "count": 3,
        "limit": 100,
        "marker": null,
        "next_href": null
    }
}
```

## Views

Views contain a combination of data that usually includes multiple different
objects. The primary purpose of a view is to save API calls and make data
retrieval more efficient. Instead of doing multiple API calls and then
combining the result yourself, you can perform a single API call against the
view endpoint.

### Get Overview

Verb | URI | Description
---- | --- | -----------
GET | /views/overview | Returns the overview view for this account.

There are no required parameters for this call.

Normal Response Code: 200

Error Response Codes: 400, 401, 403, 404, 500, 503

This view includes a list of sessions on your account and each session's
child service objects.

```javascript
{
    "values": [
        {
            "session": {
                "id": "seId0",
                "metadata": {
                    "region": "dfw"
                },
                "heartbeat_timeout": 30,
                "last_seen": null
            },
            "services": [
                {
                    "id": "dfw1-api",
                    "session_id": "seId0",
                    "tags": [],
                    "metadata": {}
                },
                {
                    "id": "dfw1-db1",
                    "session_id": "seId0",
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
            ]
        }
    ],
    "metadata": {
        "count": 1,
        "limit": 100,
        "marker": null,
        "next_href": null
    }
}
```
