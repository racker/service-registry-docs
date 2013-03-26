# Service API Operations

## Account

### Get Limits

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
    "resource": {
        "session": 100,
        "service": 200,
        "configuration_value": 1000
    },
    "rate": {
        "/.*": {
            "limit": 500000,
            "used": 0,
            "window": "24.0 hours"
        }
    }
}
```

## Services

A service represents an instance of a long running process on your server.
The client is responsible for sending heartbeats to indicate that the service is
still alive.

### Attributes

Name | Description | Validation
---- | --- | -----------
heartbeat_timeout | Maximum time between heartbeats | Integer, Value (3..120)
id | Service id | Immutable, String between 3 and 65 characters long, String matching the regex /^[a-z0-9_\-\.]{3,65}$/i
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
            "tags": [],
            "metadata": {},
            "heartbeat_timeout": 3,
            "last_seen": null
        },
        {
            "id": "dfw1-db1",
            "tags": [
                "database",
                "mysql"
            ],
            "metadata": {
                "region": "dfw",
                "port": "3306",
                "ip": "127.0.0.1",
                "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
            },
            "heartbeat_timeout": 3,
            "last_seen": null
        }
    ],
    "metadata": {
        "count": 2,
        "limit": 100,
        "marker": null,
        "next_marker": null,
        "next_href": null
    }
}
```

#### List Services Response (filtering on a tag - /services?tag=database)

```javascript
{
    "values": [
        {
            "id": "dfw1-db1",
            "tags": [
                "database",
                "mysql"
            ],
            "metadata": {
                "region": "dfw",
                "port": "3306",
                "ip": "127.0.0.1",
                "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
            },
            "heartbeat_timeout": 3,
            "last_seen": null
        }
    ],
    "metadata": {
        "count": 1,
        "limit": 100,
        "marker": null,
        "next_marker": null,
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
    "tags": [
        "database",
        "mysql"
    ],
    "metadata": {
        "region": "dfw",
        "port": "3306",
        "ip": "127.0.0.1",
        "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
    },
    "heartbeat_timeout": 3,
    "last_seen": null
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
        "database",
        "mysql"
    ],
    "metadata": {
        "region": "dfw",
        "port": "3306",
        "ip": "127.0.0.1",
        "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
    },
    "id": "dfw1-db1",
    "heartbeat_timeout": 3
}
```

#### Service Create Response

```javascript
{
    "token": "36865510-f7da-11e1-b732-793f90dd0c35"
}
```

### Update Service

Verb | URI | Description
---- | --- | -----------
PUT | /services/serviceId | Updates an existing service.

Normal Response Code: (204) This code contains no content with an empty
response body.

Error Response Codes: 400, 401, 403, 404, 500, 503

### Heartbeat a Service

Heartbeating is used for telling our service that the service is still
alive. The interval in which you need to hearbeat the service is dictated
by the `heartbeat_timeout` attribute on the service object.

For example, if you use a `heartbeat_timeout` of `30` seconds, this means
that you need to heartbeat the service every 30 seconds or more often.
Because of possible network delay and other factors, you are advised to
heartbeat your service at least 3 seconds sooner than the
`heartbeat_timeout`. In this example it would be every `27` seconds.

When you heartbeat a service, you are advised to use an
[HTTP/1.1 persistent connection](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html)
instead of opening a new connection for every heartbeat request.

All of the official client libraries listed on the
[Client Libraries and Tools](client-libraries-and-tools-client-libraries) page
re-use the same HTTP connection when heartbeating a service.

__Heartbeating a service multiple times using the same token has an
undefined behavior.__

Verb | URI | Description
---- | --- | -----------
POST | /services/serviceId/heartbeat | Heartbeat a service.

Normal Response Code: (200), Response body contains a token which must be
used on the next heartbeat.

Error Response Codes: 400, 401, 403, 404, 500, 503

#### Service Heartbeat Request

```javascript
{
    "token": "ea9b7110-9628-11e2-96d5-4d31c0f15a00"
}
```

If this is a first heartbeat request for a service , body must include an
initial heartbeat token which has been returned when you created a service.
Otherwise you must include a token which was included in the previous
heartbeat response body.

#### Service Heartbeat Response

```javascript
{
    "token": "36865510-f7da-11e1-b732-793f90dd0c35"
}
```

Response body contains a token which you mean include next time you heartbeat
this service.

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
id | Configuration value id | Immutable, String between 3 and 500 characters long, String matching the regex /^(\/[a-z0-9_\-]{3,50}\/?){0,10}([a-z0-9_\-\.]{3,170})?$/i


### List Configuration Values

Verb | URI | Description
---- | --- | -----------
GET | /configuration | Returns configuration values for this account.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### List Configuration Values

```javascript
{
    "values": [
        {
            "id": "/production/cassandra/listen_ip",
            "value": "value for /production/cassandra/listen_ip"
        },
        {
            "id": "/production/cassandra/listen_port",
            "value": "value for /production/cassandra/listen_port"
        },
        {
            "id": "/production/cassandra/rpc_server/timeout",
            "value": "value for /production/cassandra/rpc_server/timeout"
        },
        {
            "id": "/production/cassandra/rpc_server/type",
            "value": "value for /production/cassandra/rpc_server/type"
        },
        {
            "id": "/production/zookeeper/listen_ip",
            "value": "value for /production/zookeeper/listen_ip"
        },
        {
            "id": "/production/zookeeper/listen_port",
            "value": "value for /production/zookeeper/listen_port"
        },
        {
            "id": "configId1",
            "value": "test value 123456"
        },
        {
            "id": "configId2",
            "value": "test value 123456"
        }
    ],
    "metadata": {
        "count": 8,
        "limit": 100,
        "marker": null,
        "next_marker": null,
        "next_href": null
    }
}
```

### List Configuration Values Under a Namespace

Verb | URI | Description
---- | --- | -----------
GET | /configuration/<namespace 1>/<namespace n>/ | Returns configuration values for the provided namespace and all the sub namespaces.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

__When retrieving all the configuration values under a namespace you need to
include a trailing slash after the last namespace. If you don't do that it will
be treated like you are retrieving a single configuration value.__

#### List Configuration Values Under a Namespace Response (GET /configuration/production/cassandra/)

```javascript
{
    "values": [
        {
            "id": "/production/cassandra/listen_ip",
            "value": "value for /production/cassandra/listen_ip"
        },
        {
            "id": "/production/cassandra/listen_port",
            "value": "value for /production/cassandra/listen_port"
        },
        {
            "id": "/production/cassandra/rpc_server/timeout",
            "value": "value for /production/cassandra/rpc_server/timeout"
        },
        {
            "id": "/production/cassandra/rpc_server/type",
            "value": "value for /production/cassandra/rpc_server/type"
        }
    ],
    "metadata": {
        "count": 4,
        "limit": 100,
        "marker": null,
        "next_marker": null,
        "next_href": null
    }
}
```

### Get Configuration Value

Verb | URI | Description
---- | --- | -----------
GET | /configuration/configurationId | Retrieves a single configuration value.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

#### Get Configuration Value

```javascript
{
    "id": "configId",
    "value": "test value 123456"
}
```

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
of your account. Every time a service is created, a configuration
value is updated or removed, or a service times out, an event is inserted.

### Event Object Attributes

Name | Description | Validation
---- | --- | -----------
id | Event id | Immutable, String between 3 and 255 characters long
timestamp | Event timestamp | Integer
type | Event type | String
payload | Event payload. | Optional, Optional, Hash [String,String between 1 and 255 characters long:String,String between 1 and 255 characters long]


### Event Types

#### service.join

This event represents a new service joining the registry. The payload contains
a service object.

##### service.join Example Event Object

```javascript
{
    "id": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20",
    "timestamp": 1346967146370,
    "type": "service.join",
    "payload": {
        "id": "dfw1-api",
        "heartbeat_timeout": 3,
        "tags": [],
        "metadata": {}
    }
}
```

#### service.remove

This event is inserted when a user deletes a service using an API call. The 
payload contains a deleted service object.

##### service.remove Example Event Object

```javascript
{
    "type": "service.remove",
    "payload": {
        "id": "dfw1-db1",
        "heartbeat_timeout": 3,
        "tags": [
            "database",
            "mysql"
        ],
        "metadata": {
            "region": "dfw",
            "port": "3306",
            "ip": "127.0.0.1",
            "version": "5.5.24-0ubuntu0.12.04.1 (Ubuntu)"
        }
    }
}
```

#### service.timeout

This event represents a service timeout, which occurs when a client doesn't
heartbeat a service within the defined timeout. The payload contains a service
object.

##### service.timeout Example Event Object

```javascript
{
    "type": "service.timeout",
    "payload": {
        "id": "dfw1-api",
        "heartbeat_timeout": 3,
        "tags": [],
        "metadata": {}
    }
}
```

#### configuration_value.update

This event represents a configuration value being updated. The payload
contains the configuration value id, the old value, and the new value. If
this is the first time the value has been set, the old value will be `null`.

##### configuration_value.update Example Event Object

```javascript
{
    "type": "configuration_value.update",
    "payload": {
        "old_value": null,
        "new_value": "test value 123456",
        "configuration_value_id": "configId2"
    }
}
```

#### configuration_value.remove

This event represents a configuration value being removed. The payload
contains the configuration value id and the old value.

##### configuration_value.remove Event Object

```javascript
{
    "type": "configuration_value.remove",
    "payload": {
        "old_value": "test value 123456",
        "configuration_value_id": "configId"
    }
}
```

### List Events

Verb | URI | Description
---- | --- | -----------
GET | /events | Returns a list of events for this account.

There are no parameters for this call.

Normal Response Code: 200

Error Response Codes: 401, 403, 500, 503

A client can specify the optional `marker` query parameter. If the marker is
provided, only the events with ids that are newer or equal to the  marker are
returned. If no marker is provided, events from the last hour are
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
                "heartbeat_timeout": 3,
                "tags": [],
                "metadata": {}
            }
        },
        {
            "type": "service.join",
            "payload": {
                "id": "dfw1-db1",
                "heartbeat_timeout": 3,
                "tags": [
                    "database",
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
            "type": "service.remove",
            "payload": {
                "id": "dfw1-db1",
                "heartbeat_timeout": 3,
                "tags": [
                    "database",
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
                "configuration_value_id": "configId2"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "test value 123456",
                "configuration_value_id": "configId"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "test value 123456",
                "configuration_value_id": "configId1"
            }
        },
        {
            "type": "configuration_value.remove",
            "payload": {
                "old_value": "test value 123456",
                "configuration_value_id": "configId"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/zookeeper/listen_ip",
                "configuration_value_id": "/production/zookeeper/listen_ip"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/cassandra/rpc_server/timeout",
                "configuration_value_id": "/production/cassandra/rpc_server/timeout"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/cassandra/rpc_server/type",
                "configuration_value_id": "/production/cassandra/rpc_server/type"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/cassandra/listen_port",
                "configuration_value_id": "/production/cassandra/listen_port"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/cassandra/listen_ip",
                "configuration_value_id": "/production/cassandra/listen_ip"
            }
        },
        {
            "type": "configuration_value.update",
            "payload": {
                "old_value": null,
                "new_value": "value for /production/zookeeper/listen_port",
                "configuration_value_id": "/production/zookeeper/listen_port"
            }
        },
        {
            "type": "service.timeout",
            "payload": {
                "id": "dfw1-api",
                "heartbeat_timeout": 3,
                "tags": [],
                "metadata": {}
            }
        }
    ],
    "metadata": {
        "count": 14,
        "limit": 100,
        "marker": null,
        "next_marker": null,
        "next_href": null
    }
}
```
