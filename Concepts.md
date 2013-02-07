# Concepts

## Session

Sessions give clients the ability to manage persistent context for one or more
services. This context is then used for other operations. The client is
responsible for sending heartbeats to maintain the session.

If a session is not heartbeated in the defined timeout interval, the
session is considered dead. All of its child Service objects are deleted,
and a `services.timeout` event is inserted into the events feed.

## Service

A service represents an instance of a long running process on your server.
Each service belongs to a single session which must be periodically
heartbeated to indicate that the session and its services are still alive.

Some example services include:

* API server instance
* Apache instance
* MySQL instance
* Java application instance
* ZNC instance

Usually a single session will have a single service associated with it, but
in some cases you may want to have multiple services associated with a
single session. An example of this would be a Java application with 2
threads:

* thread 1 - exposes a HTTP interface and listens on port 8000
* thread 2 - exposes a Thrift interface and listens in port 9000

In this scenario, when a whole process dies, both of the threads go away,
which means that you can model it by having a single session with two
services associated with it (one per thread).

## Configuration

Configuration enables clients to store arbitrary key/value pairs on the
server and get notified via an event feed when a value is updated or
deleted.

Each configuration key can contain a namespace. Namespace allows you to organize
different (related) configuration values together in a hierarchical manner.
They also make retrieving a subset of configuration values easier and more 
efficient.

If a key contains a namespace, you need to refer to the configuration using a
fully qualified name which includes a namespace. For example, if a namespace is
`/production/cassandra/` and the key name is `listen_ip`, fully qualified key
for this value is `/production/cassandra/listen_ip`.

Example of fully qualified configuration keys:

* cassandra_listen_ip (no namespace)
* cassandra_listen_port (no namespace)
* /production/cassandra/listen_ip (namespace is /production/cassandra)
* /production/cassandra/listen_port (namespace is /production/cassandra)

![Configuration Namespaces visualized using a tree](https://www.lucidchart.com/publicSegments/view/5112ccdb-ee10-4808-abe2-6fa50a00093e/image.png)

Namespaces are also ephemeral which means they only exist if at least one
configuration value is stored under it.

When retrieving values from the API you differentiate between a configuration
value based on the presence of a trailing forward slash. If a trailing slash is
present it's treated as a namespace otherwise it's treated as a configuration
key.

For example:

* GET /configuration/production/casssandra/ - cassandra is treated as a
sub-namespace under namespace production.
* GET /configuration/production/cassandra - cassandra is treated as a key under
production namespace.

A configuration value is an opaque string and is treated as such in our
system.

## Events Feed

The events feed contains a list of events which occurred during the life-
time of your account. An event is inserted each time one of the following
incidents occur:

* A service joins a session
* A configuration value is updated
* A configuration value is removed
* A session times out

A typical automation use case would be periodically polling the event feed
and acting on the returned events.

Note: When you long poll the event feed you are advised to use an HTTP/1.1
persistent connection instead of opening a new connection for every request.

## Event

An event represents a single event in the events feed. Each event object
has the following attributes:

* id
* timestamp
* type
* payload

## Id

All objects in the system are identified by a unique id. You'll use an
object's id when you want to perform certain operations on it. For example,
when you want to create a service and associate it with a session, you'll
need to know the session id.
