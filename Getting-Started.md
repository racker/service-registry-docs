# Getting Started

## Getting Started with Rackspace Service Registry

The quickest way to try out the Service Registry is to use one of the
client libraries to create a session and a service, and then use the
command line client to list them and view the corresponding events.

To see how to create sessions and services using the Twisted Python or
Node.js libraries, take a look at the [Integration Instructions](integration-instructions).

Now that you've created a session and a service, you can list them
using the command line tool:

```shell
raxsr sessions list --user=username --api-key=key
```

### Service Registry CLI Sessions List

```
+------------+-------------------+-----------+-----------+
| Session ID | Heartbeat Timeout | Last Seen | Metadata  |
+------------+-------------------+-----------+-----------+
| seDMRMcCQh |                30 |           |           |
+------------+-------------------+-----------+-----------+

```

```shell
raxsr services list --user=username --api-key=key
```

### Service Registry CLI Services List

```
+------------+------------+------+-------------+
| Service ID | Session ID | Tags | Metadata    |
+------------+------------+------+-------------+
| service id | seDMRMcCQh | www  | region: dfw |
+------------+------------+------+-------------+

```

You can also inspect the events feed which should include a `service.join`
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
|                                      |                     |                  | tags: www              |
|                                      |                     |                  |                        |
+--------------------------------------+---------------------+------------------+------------------------+

```

You can find more in-depth examples in the
[Integration Instructions](integration-instructions) chapter.
