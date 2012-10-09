# Getting Started

The quickest way to try out the Service Registry is to use one of the
client libraries to create a session and a service, and then use the
command line client to list them and view the corresponding events.

To see how to create sessions and services using the Twisted Python or
Node.js libraries, take a look at the Integration Guide.

Now that you've created a session and a service, you can list them
using the CLI:

```shell
raxsr sessions list --user='username' --api-key='api key'
```

### Service Registry CLI Sessions List
```shell
+------------+-------------------+-----------+
| Session ID | Heartbeat Timeout | Last Seen |
+------------+-------------------+-----------+
| seDMRMcCQh |                30 | None      |
+------------+-------------------+-----------+

```

```shell
raxsr services list --user='username' --api-key='api key'
```

### Service Registry CLI Services List
```shell
+------------+------------+------+
| Service ID | Session ID | Tags |
+------------+------------+------+
| service id | seDMRMcCQh |      |
+------------+------------+------+

```
