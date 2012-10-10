# Integration Guide

This document will explain how to integrate Rackspace Service Registry clients,
into your applications and also contains an example on how to integrate
the Twisted Python client into your Twisted-powered application.

## General Flow

The general flow when registering a service with Rackspace Service Registry is:

* Create a session.
* Add service(s) to the session.
* Heartbeat the session to maintain it.

Once you've done those things, you'll have a session in Cloud Service
Registry with a service attached to it, and you will be able to utilize
features such as the Events feed, which contains events for when a service
has joined, when a session has timed out, and more.

### Create a session

The first thing you will want to do when using the Rackspace Service Registry
is create a session.

As described in the Session section of the Concepts document,
sessions enable clients to create persistent sessions on the server that
are used as a context for other operations. Clients should first create a
session and heartbeat it to maintain the session.

To create a session, POST to /sessions, replacing "1234" with your account
ID number, and using the auth token returned by the auth API. The request
body must contain `heartbeat_timeout` in the range of 3-30 seconds, and may
contain optional metadata (key/value pairs).

```shell
POST /v1.0/1234/sessions HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

#### Create Session Request Body

```javascript
{
    "metadata": {
        "region": "dfw"
    },
    "heartbeat_timeout": 30
}
```

The location header of the response should look something like this:

https://dfw.registry.api.rackspacecloud.com/v1.0/1234/sessions/seMkzI0mxC

The last part of the location header, seMkzI0mxC, is the session ID.

#### Create Session Response

```javascript
{
    "token": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20"
}
```

It contains the initial token which you can use to start heartbeating the
session.

### Add Services

Next, you'll want to add services to the session. Service IDs are user
provided, and must be unique across the whole account. Let's say our
service name is 'dfw1-db1'. The way to create a service is to POST to
/services with a body containing the service ID, the session ID that the
service should belong to, optional metadata, and optional tags:

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

Let's say you also added a service called 'dfw1-api' without any tags
or metadata. If you GET /services, you should see this:

#### GET /services Response

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

You can also GET services by tag:

```shell
GET /v1.0/1234/services?tag=db HTTP/1.1
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

To change the dfw1-db1 service (for example, update its metadata),
you can do an HTTP PUT request to /services/dfw1-db1 with a body that
contains the new metadata that you'd like the service to have.

### Heartbeat the Session

Heartbeating is as simple as POSTing to /sessions/[session ID]/heartbeat
with the token as the body.

```shell
POST /v1.0/1234/sessions/seMkzI0mxC/heartbeat HTTP/1.1
Host: dfw.registry.api.rackspacecloud.com
Accept: application/json
X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb
```

The request body would look like this:

#### Heartbeat Sesssion Request Body

```javascript
{
    "token": "6bc8d050-f86a-11e1-a89e-ca2ffe480b20"
}
```

And the response body would look the same, except that the token would be
different. You could then use the new token to heartbeat once again.

There are client libraries that abstract away heartbeating so that when you
create a session, you get an object back that heartbeats for you
automatically. We will see that in the Twisted Python client in the next
section.

## Using the Twisted Python client

All of the Service Registry functionality has been abstracted away in various
clients for popular programming languages such as Java, Node.js, and Python. In
this section, we'll see how to use the Twisted Python client to go through
the same flow of creating a session, adding services to it, and
heartbeating the session.

### Installing the client

The client is available in the Python Package Index. To install, you can do:

```shell
pip install txServiceRegistry
```

### Create a session (Python)

In order to create a session using the Twisted Python client, we first have
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
Rackspace Service Registry API. Creating a session is straightforward:

#### Create Session

```python
def cb(result):
    token = result[0]['token']
    sessionId = result[1]
    heartbeater = result[2]

d = client.sessions.create(30)
d.addCallback(cb)

reactor.run()

```

### Add Services

We also now have the session ID (let's say it's 'seMkzI0mxC'), so we can
start adding services to the session:

#### Register Service (Python)

```python
def cb(result):
    serviceId = result

d = client.services.register('seMkzI0mxC', 'serviceId', {'tags': ['tag1', 'tag2', 'tag3']})
d.addCallback(cb)

reactor.run()

```

### Heartbeat the Session

When creating a session using the Twisted client, the result contains the
response body (which contains the initial token required for heartbeating
the session), the session ID, and a HeartBeater object. The HeartBeater
object allows us to automatically heartbeat the session by calling the
start() method:

```python
heartbeater.start()
```

This causes the HeartBeater object to start heartbeating automatically,
using the initial token. It will heartbeat the session, get the next token,
and heartbeat the session again continuously until the stop() method is
called.

You may also heartbeat the session manually like so:

#### Heartbeat Session Manually (Python)

```python
client.sessions.heartbeat('seMkzI0mxC', 'token')

```

### Integration Example

Here is a short example of a web server that registers with the Cloud
Service Registry on startup, and uses the HeartBeater object while it is
running in order to maintain the session:

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


def cbSession(result):
    global heartBeater
    sessionId = result[1]
    heartBeater = result[2]
    heartBeater.start()

    def cbService(result):
        print result

    d = client.services.register(sessionId,
                                 'http-service',
                                 {'tags': ['web']})
    d.addCallback(cbService)

d = client.sessions.create(30)
d.addCallback(cbSession)

site = server.Site(Simple())
reactor.listenTCP(8080, site)
reactor.run()

```

The code above is a simple web server that responds with "<html>Hello,
world!</html> on every GET request. The code that interacts with the Cloud
Service Registry can be explained as follows:

First, the server creates a session with a heartbeat interval of 30. Since
client.sessions.create() returns a Twisted Deferred, a callback must be
added to it in order to use the result. This is done here:

```python
d = client.sessions.create(30)
d.addCallback(cbSession)
```

The cbSesssion function takes the result of client.sessions.create() as an
argument, and grabs the session ID as well as starts the HeartBeater. It
then creates a service named 'http-service' and attaches it to the session.

## Using the Node.js client

### Installing the client

The client is available in npm. To install, you can do:

```shell
npm install service-registry-client
```

### Create a session

In order to create a session using the Node.js client, we first have
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
Rackspace Service Registry API. Creating a session is straightforward:

#### Create Session (Javascript)

```javascript
client.sessions.create(30, {}, function(err, seId, resp, heartbeater) {});

```

### Add Services

We also now have the session ID (let's say it's 'seMkzI0mxC'), so we can
start adding services to the session:

#### Register Service (Javascript)

```javascript
client.services.register('seMkzI0mxC',
                         'serviceId',
                         {'tags': ['tag1', 'tag2', 'tag3']},
                         null, function(err, resp) {});

```

### Heartbeat the Session

When creating a session using the Node.js client, the result contains the
session ID, response body (which contains the initial token required for
heartbeating the session), and a HeartBeater object. The HeartBeater
object allows us to automatically heartbeat the session by calling the
start() method:

```Javascript
heartbeater.start();
```

This causes the HeartBeater object to start heartbeating automatically,
using the initial token. It will heartbeat the session, get the next token,
and heartbeat the session again continuously until the stop() method is
called.

You may also heartbeat the session manually like so:

#### Heartbeat Session Manually (Javascript)

```javascript
client.sessions.heartbeat('seMkzI0mxC', 'token', function(err, resp) {});

```

### Integration Example

Here is a short example of a web server that registers with the Cloud
Service Registry on startup, and uses the HeartBeater object while it is
running in order to maintain the session:

#### A Web Server That Uses Service Registry (Javascript)

```javascript
var async = require('async');
var Client = require('service-registry-client/lib/client').Client

var username = ''; // your username here
var key = ''; // your API key here

var client = new Client(username, key, 'us', null);

var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello, world!');
}).listen(9000, '127.0.0.1');

async.waterfall([
  function createSession(callback) {
    client.sessions.create(30, {}, function(err, seId, resp, hb) {
      hb.start();
      callback(null, seId);
    });
  },
  function createService(seId, callback) {
    client.services.register(seId, 'webService', {}, function(err, resp) {
      callback();
    });
  }
  ],
  function(err) {
    if (err) {
      console.log('An error has occurred.');
      console.log(err);
    }
  }
);

```

The code above is a simple web server that responds with "Hello, world!"
The code that interacts with the Rackspace Service Registry can be
explained as follows:

First, the server creates a session with a heartbeat interval of 30.

#### Create Session in Web Server
```javascript
function createSession(callback) {
  client.sessions.create(30, {}, function(err, seId, resp, hb) {
    hb.start();
    callback(null, seId);
  });
},

```

This function also calls start() on the HeartBeater object that is returned
when creating a session.

Next, a service called 'webService' is created under the session ID
returned from the API:

#### Register Service in Web Server
```javascript
function createService(seId, callback) {
  client.services.create(seId, 'webService', {}, function(err, resp) {
    callback();
  });
}

```
