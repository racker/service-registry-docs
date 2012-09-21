# Client Libraries and Tools

This section describes the client libraries and tools that you can use to
interact with the API.

## Client Libraries

### Node.js

The official Node.js library which allows you to interact with the API can
be found at [https://github.com/racker/node-service-registry-client](https://github.com/racker/node-service-registry-client).

#### Installation

```shell
npm install rackspace-cloud-registry-client
```

### Twisted Python

The official Twisted Python library which allows you to interact with the
API can be found at [https://github.com/racker/python-twisted-service-registry-client](https://github.com/racker/python-twisted-service-registry-client).

#### Installation

```shell
pip install rackspace-cloud-registry-client
```

### Java

The official Java library which allows you to interact with the API can be
found at [https://github.com/racker/java-service-registry-client](https://github.com/racker/java-service-registry-client).

#### Installation

TODO

## Best Practices For Writing Client Libraries

This section describes best practices which you should follow if you are
building a custom client library.

### Use Persistent connections

HTTP/1.1 defines persistent connections which you should use when talking with
the API.

This is especially important when you are sending heartbeats and polling the
events feed. Instead of opening a new TCP connection when you are sending a
heartbeat or polling the events feed, re-use the existing connection. This is
more efficient for both the client and the server.

### Make sure to re-authenticate with the Auth API

When our API returns 401, you should try to re-authenticate with the Auth API
and retrieve a new token. A 401 can either mean that you supplied an invalid
token or that the token has expired.

The Auth API returns a token expiration time with the "obtain token" response,
but some times, for various reasons, tokens are purged before the actual
expiration time.

This is especially important because of the nature of this service. Code
hitting our API won't be located in a run-once-script but rather inside
long-running processes constantly hitting our API.

### Cache a List of Available Services

To avoid interruptions in your service because of minor hiccups on your or our
side you should cache the list services response in your client.

This will allow you to retrieve a (potentially stale) list of services from the
cache in case a minor service interruption occurs.

## Tools

### Long Running Process Wrapper

This tool written in Node.js allows you to wrap and register arbitrary
long-running process inside the service registry. It works by execing a new
process for the long-running process you want to wrap and regsitering it in the
service registry.

If the wrapped process dies, the wrapper itself also exits which will cause
the session to eventually expire (how long it takes depends on the heartbeat
interval).

#### Installation

```shell
npm install todo
```

#### Usage

```shell
todo
```
