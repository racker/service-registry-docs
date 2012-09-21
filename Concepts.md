# Concepts

## Session

Sessions enable clients to create persistent sessions on the server that
are used as a context for other operations. The client is responsible for
sending heartbeats to maintain the session.

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

A configuration value is an opaque string and is treated as such in our
system.

## Events Feed

The events feed contains a list of events which occurred during the life-
time of your account. Every time a service joins a session, a configuration
value is updated or removed, or a session times out, an event is inserted.

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
