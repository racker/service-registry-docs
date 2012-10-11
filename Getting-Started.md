# Getting Started

## Getting Started with the Rackspace Service Registry

The quickest way to try out the Service Registry is to use one of the
client libraries to create a session and a service, and then use the
command line client to list them and view the corresponding events.

To see how to create sessions and services using the Twisted Python or
Node.js libraries, take a look at the [Integration Guide](integration-guide).

Now that you've created a session and a service, you can list them
using the command line tool.

```shell
raxsr sessions list --user=username --api-key=key
```

### Service Registry CLI Sessions List

```
+------------+-------------------+-----------+
| Session ID | Heartbeat Timeout | Last Seen |
+------------+-------------------+-----------+
| seDMRMcCQh |                30 | None      |
+------------+-------------------+-----------+

```

```shell
raxsr services list --user=username --api-key=key
```

### Service Registry CLI Services List

```
+------------+------------+------+
| Service ID | Session ID | Tags |
+------------+------------+------+
| service id | seDMRMcCQh |      |
+------------+------------+------+

```

You can also inspect the events feed and there should be a `service.join`
event:

```shell
raxsr events list --user=username --api-key=key
```

### Service Registry CLI Events List

```
+--------------------------------------+---------------------+------------------+------------------------+
| Event ID                             | Timestamp           | Event Type       | Payload                |
+--------------------------------------+---------------------+------------------+------------------------+
| fdf898e0-1244-11e2-8e15-0295b0845fc6 | 2012-10-09 12:12:30 | service.join     | metadata:              |
|                                      |                     |                  | id: service id         |
|                                      |                     |                  | session_id: seDMRMcCQh |
|                                      |                     |                  | tags: []               |
|                                      |                     |                  |                        |
+--------------------------------------+---------------------+------------------+------------------------+

```

For more in-depth examples, please see the
[Integration Guide](integration-guide).
