# Getting Started

## Getting Started with Rackspace Service Registry

The quickest way to try out the Service Registry is to use one of the
client libraries to create a service, and then use the command line
client to list the services and view the corresponding events.

First create a service by following the instructions in the
[Integration Instructions](integration-instructions) chapter.

Now that you've created a service, you can list all of the services using
the command line tool:

```shell
raxsr services list --username=username --api-key=key
```

All the commands in the client follow the format described bellow:

```shell
raxsr <resource> <action> [options]
```

Resource can be one of the following:

* account
* services
* events
* configuration

Available actions depend on the resource and the options depend on the resource
and action.

To view available options for a command, run the following command in your
terminal:

```shell
raxsr help <resource> <action>
```

### Service Registry CLI Services List

List all the services.

```shell
raxsr services list --username=username --api-key=key
```

### Service Registry CLI Services List

```
+------------+-------------------+------+-------------+
| Service ID | Heartbeat Timeout | Tags | Metadata    |
+------------+-------------------+------+-------------+
| service id | 30                | www  | region: dfw |
+------------+-------------------+------+-------------+

Limit: 100

```

You can also inspect the events feed which should include a `service.join`
event:

```shell
raxsr events list --username=username --api-key=key
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

Limit: 100

```

You can find more in-depth examples in the
[Integration Instructions](integration-instructions) chapter.
