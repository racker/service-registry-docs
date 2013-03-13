# Integration Instructions

This document will explain how to integrate Rackspace Service Registry clients,
into your applications and also contains an example on how to integrate
the Twisted Python client into your Twisted-powered application.

## General Flow

The general flow when registering a service with Rackspace Service Registry is:

* Create a service.
* Heartbeat the service to maintain it.

Once you've done those things, you'll have a service in Rackspace Service
Registry  and you will be able to utilize features such as the Events feed,
which contains events for when a service has joined, when it has timed out,
and more.

### Create a Service

The first thing you will want to do when using the Rackspace Service Registry
is create a service.

As described in the Service section of the [Concepts](concepts) chapter, service
represents an arbitrary long running process on your server.

To create a service, POST to /services, replacing "1234" with your account
ID number, and using the auth token returned by the auth API. The request
body must contain `heartbeat_timeout` in the range of 3-120 seconds, and may
contain optional tags and metadata (key/value pairs) attribute.

```shell
POST /v1.0/1234/services HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

#### Create Service Request Body

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

#### Create Service Response

```javascript
{
    "token": "36865510-f7da-11e1-b732-793f90dd0c35"
}
```

It contains the initial token which you can use to start heartbeating it.

Let's say you also added a service called 'dfw1-api' without any tags
or metadata. If you GET /services, you should see this:

#### GET /services Response

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

You can also GET services by tag:

```shell
GET /v1.0/1234/services?tag=database HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

and the response would look like this:

#### Get Services By Tag Response

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

To change the dfw1-db1 service (for example, update its metadata),
you can do an HTTP PUT request to /services/dfw1-db1 with a body that
contains the new metadata that you'd like the service to have.

### Heartbeat the Service

Heartbeating is as simple as POSTing to /services/<service ID>/heartbeat
with the token as the body.

```shell
POST /v1.0/1234/services/dfw1-api/heartbeat HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

The request body would look like this:

#### Heartbeat Service Request Body

```javascript
{
    "token": "36865510-f7da-11e1-b732-793f90dd0c35"
}
```

And the response body would look the same, except that the token would be
different. You could then use the new token to heartbeat once again.

There are client libraries that abstract away heartbeating so that when you
create a service, you get an object back that heartbeats for you
automatically. We will see that in the Twisted Python client in the next
section.

## Using the Twisted Python Client

All of the Service Registry functionality has been abstracted away in various
clients for popular programming languages such as Java, Node.js, and Python. In
this section, we'll see how to use the Twisted Python client to go through
the same flow of creating a service and heartbeating it.

### Installing the Client

The client is available in the
[Python Package Index](https://pypi.python.org/pypi). To install, you can do:

```shell
pip install txServiceRegistry
```

### Create a Service (Python)

In order to create a service using the Twisted Python client, we first have
to instantiate a client to interact with the Rackspace Service Registry:

#### Instantiate the Client (Python)

```python
from txServiceRegistry.client import Client
from twisted.internet import reactor

RACKSPACE_USERNAME = '' # your username here
RACKSPACE_KEY = '' # your API key here
SERVICE_REGISTRY_URL = 'https://dfw.registry.api.rackspace.com/v1.0/'

client = Client(username=RACKSPACE_USERNAME,
                apiKey=RACKSPACE_KEY,
                baseUrl=SERVICE_REGISTRY_URL,
                region='us')

```

The region keyword argument above determines which Rackspace authentication
URL the client will use to authenticate. You can specify either 'us' or
'uk'.

Now that we've created a Client object, we can use it to work with the
Rackspace Service Registry API. Creating a service is straightforward:

#### Register Service (Python)

```python
def cb(result):
    print('Service has been created')

heartbeatTimeout = 15
payload = {'tags': ['tag1', 'tag2', 'tag3']}
d = client.services.register('serviceId', heartbeatTimeout, payload)
d.addCallback(cb)

reactor.run()

```

### Heartbeat the Service (Python)

When creating a service using the Twisted client, the result contains the
response body (which contains the initial token required for heartbeating it),
and a HeartBeater object. The HeartBeater object allows us to automatically
heartbeat the service by calling the start() method:

#### Heartbeat Service Using Heartbeater (Python)

```python
heartbeater.start()
```

This causes the HeartBeater object to start heartbeating automatically,
using the initial token. It will heartbeat the service, get the next token,
and heartbeat the service again continuously until the stop() method is
called.

You may also heartbeat the service manually like so:

#### Heartbeat Service Manually (Python)

```python
client.service.heartbeat('serviceId', 'token')

```

### Integration Example

Here is a short example of a web server that registers with the Cloud
Service Registry on startup, and uses the HeartBeater object while it is
running in order to maintain the service:

#### A Web Server That Uses Service Registry (Python)

```python
from twisted.web import server, resource
from twisted.internet import reactor

from txServiceRegistry import Client

RACKSPACE_USERNAME = '' # your Rackspace username here
RACKSPACE_KEY = '' # your Rackspace API key here

client = Client(RACKSPACE_USERNAME,
                RACKSPACE_KEY,
                'us')

class Simple(resource.Resource):
    isLeaf = True
    def render_GET(self, request):
        return "<html>Hello, world!</html>"


def cbService(result):
    global heartBeater
    heartBeater = result[1]
    heartBeater.start()

payload = {'tags': ['api']}

d = client.services.register('web-service-api0', 30, payload)
d.addCallback(cbService)

site = server.Site(Simple())
reactor.listenTCP(8080, site)
reactor.run()

```

The code above is a simple web server that responds with "<html>Hello,
world!</html> on every GET request. The code that interacts with the Cloud
Service Registry can be explained as follows:

First, the server creates a servicewith a heartbeat interval of 30. Since
client.services.create() returns a Twisted Deferred, a callback must be
added to it in order to use the result. This is done here:

#### Create a Service (Python)

```python
payload = {'tags': ['api']}

d = client.services.register('web-service-api0', 30, payload)
d.addCallback(cbService)
```

The cbService function takes the result of client.services.create() as an
argument and starts the HeartBeater.

## Using the Node.js Client

### Installing the Client

The client is available in [npm](https://npmjs.org/). To install, you can do:

```shell
npm install service-registry-client
```

### Create a Service (Javascript)

In order to create a service using the Node.js client, we first have
to instantiate a client to interact with the Rackspace Service Registry:

#### Instantiate the Client (Javascript)

```javascript
var Client = require('service-registry-client').Client;

var username = ''; // your username here
var key = ''; // your API key here
var service_registry_url = 'https://dfw.registry.api.rackspace.com/v1.0/';

var client = new Client(username, key, 'us', {'url': service_registry_url});

```

The region keyword argument above determines which Rackspace authentication
URL the client will use to authenticate. You can specify either 'us' or
'uk'.

Now that we've created a Client object, we can use it to work with the
Rackspace Service Registry API. Creating a service is straightforward:

#### Register Service (Javascript)

```javascript
var heartbeatTimeout = 15;
var payload = {'tags': ['tag1', 'tag2', 'tag3']};

client.services.register('serviceId', heartbeatTimeout, payload,
                         null, function(err, resp) {});

```

### Heartbeat the Service (Javascript)

When creating a service using the Node.js client, the result contains the
response body (which contains the initial token required for heartbeating the
service), and a HeartBeater object. The HeartBeater object allows us to
automatically heartbeat the service by calling the start() method:

#### Heartbeat Service Using Heartbeater (Javascript)

```Javascript
heartbeater.start();
```

This causes the HeartBeater object to start heartbeating automatically,
using the initial token. It will heartbeat the service, get the next token,
and heartbeat the sevice again continuously until the stop() method is
called.

You may also heartbeat the service manually like so:

#### Heartbeat Service Manually (Javascript)

```javascript
client.services.heartbeat('serviceId', 'token', function(err, resp) {});

```

### Integration Example

Here is a short example of a web server that registers with the Cloud
Service Registry on startup, and uses the HeartBeater object while it is
running in order to maintain the service:

#### A Web Server That Uses Service Registry (Javascript)

```javascript
var http = require('http');

var async = require('async');
var Client = require('service-registry-client/lib/client').Client

var username = ''; // your username here
var key = ''; // your API key here

function main() {
  var client = new Client(username, key, 'us', null);

  async.waterfall([
    function startHttpServer(callback) {
      var server = http.createServer(function (req, res) {
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Hello, world!');
      };

      server.listen(8080, '127.0.0.1', callback);
    },

    function createService(callback) {
      var payload = {'tags': ['api']};

      client.services.register('web-service-api0', 30, payload, function(err, resp, hb) {
        callback(null, hb);
      });
    },

    function startHeartbeating(hb, callback) {
      hb.start();
      callback();
    }
  ],

  function(err) {
    if (err) {
      console.log('An error has occurred.');
      console.log(err);
    }
  });
}

main();

```

The code above is a simple web server that responds with "Hello, world!"
The code that interacts with the Rackspace Service Registry can be
explained as follows:

First, the script creates a service with a heartbeat interval of 30.

#### Create Service in Web Server

```javascript
function createService(callback) {
  client.services.create('web-service-api0', 30, {}, function(err, resp, hb) {
    callback();
  });
}

```

This function also calls start() on the HeartBeater object that is returned
when creating a service.
