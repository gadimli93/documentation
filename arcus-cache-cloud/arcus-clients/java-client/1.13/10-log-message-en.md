# Java Client Log Messages

The logs lefts by Java Client and their meanings are as follows.
Log messages stored in a location set by a service.
Please refer to [arcus client settings](02-arcus-java-client.md#arcus-client-settings) for changing log level and logger settings.

## Initialize ARCUS Client

Following log message is shown when the ARCUS Client is successfully initialized.

```java
INFO net.spy.memcached.CacheManager: CacheManager started. ([mailto:dev@dev.arcuscloud.nhncorp.com:17288 dev@dev.arcuscloud.nhncorp.com:17288])

WARN net.spy.memcached.CacheMonitor: Cache list has been changed : From=null, To=[127.0.0.1:11211-hostname, xxx.xxx.xxx.xxx:xxxx-hostname] : [serviceCode=dev]

INFO net.spy.memcached.MemcachedConnection: new memcached node added {QA sa=/127.0.0.1:11211, #Rops=0, #Wops=0, #iq=0, topRop=null, topWop=null, toWrite=0, interested=0} to connect queue

INFO net.spy.memcached.MemcachedConnection: new memcached node added {QA sa=/127.0.0.1:11211, #Rops=0, #Wops=0, #iq=0, topRop=null, topWop=null, toWrite=0, interested=0} to connect queue

INFO net.spy.memcached.MemcachedConnection: Connection state changed for sun.nio.ch.SelectionKeyImpl@388ee016

INFO net.spy.memcached.MemcachedConnection: Connection state changed for sun.nio.ch.SelectionKeyImpl@2e5bbd6

2021-09-06 17:49:54.055 WARN net.spy.memcached.CacheManager: All arcus connections are established.
```

## Invalid ARCUS Admin Address

Following log message is shown when the ARCUS admin address is incorrectly specified or the server is unresponsive.

```java
FATAL net.spy.memcached.CacheManager: Unexpected exception. contact to arcus administrator

INFO net.spy.memcached.CacheManager: Close ZooKeeper client.
```

## Conncetion Lost with ARCUS Admin

Following log message is shown when the connection is lost with ARCUS admin due to a network problem and 
retry to connect with ARCUS admin.

```java
WARN net.spy.memcached.CacheMonitor: Disconnected from the Arcus admin. Trying to reconnect : [serviceCode=dev]
```

## ARCUS Admin Session Expiration

Following log message is shown when the session has expired due to `heart beat` is not working properly because of the connection issue with admin.

```java
WARN net.spy.memcached.CacheMonitor: Session expired. Trying to reconnect to the Arcus admin : [serviceCode=dev]
```

When the session expires, the ARCUS client attempts to reconnect again.

```java
INFO net.spy.memcached.CacheMonitor: Shutting down the CacheMonitor : [serviceCode=dev]

WARN net.spy.memcached.CacheManager: Unexpected disconnection from Arcus admin. Trying to reconnect to Arcus admin.

INFO net.spy.memcached.CacheManager: Close ZooKeeper client.
```

Following log message is shown when the connection is successful established.

```java
WARN net.spy.memcached.CacheMonitor: Reconnected to the Arcus admin : [serviceCode=dev]

ERROR net.spy.memcached.CacheMonitor: Cache list has been changed : From=null, To=[10.0.0.1:11211-arcus01.companyname.com, 10.0.0.2:11211-arcus02.companyname.com] : [serviceCode=dev]
```

## Object serialization Problem

Following log message is shown if the stored value is not serializable or if it is a `null`.

If you want to store a `null`.

```java
Can’t serialize null
```

If you want to store a value that is not serializable.

```java
java.lang.IllegalArgumentException: Non-serializable object, cause=`Reason`
```

Here in `Reason` shows the class name that causes a serialization failure.

## Out of Memory Storing Object

If you set the expire time to -1,when storing an item then `out of memory storing object` error may occur.
The reason is when running the ARCUS server, storage area of sticky item specified in percentage is full.
The same error message will appear even when Sticky item's possible proportion percentage is not specified.

```java
[ERROR](StoreOperationImpl :? ) Error: SERVER_ERROR out of memory storing object
[INFO ](MemcachedConnection :? ) Reconnection due to exception handling a memcached operation on {QA sa=/ 127.0.0.1:11211, #Rops=2, #Wops=0, #iq=0, topRop=net.spy.memcached.protocol.ascii.StoreOperationImpl@250d593e, topWop=null, toWrite=0, interested=1}. This may be due to an authentication failure.
OperationException: SERVER: SERVER_ERROR out of memory storing object
at net.spy.memcached.protocol.BaseOperationImpl.handleError(BaseOperationImpl.java:127)
at net.spy.memcached.protocol.ascii.OperationImpl.readFromBuffer(OperationImpl.java:131)
at net.spy.memcached.MemcachedConnection.handleReads(MemcachedConnection.java:457)
at net.spy.memcached.MemcachedConnection.handleIO(MemcachedConnection.java:389)
at net.spy.memcached.MemcachedConnection.handleIO(MemcachedConnection.java:182)
at net.spy.memcached.MemcachedClient.run(MemcachedClient.java:1630)
```

## Operation Timeout

Following log message is shown when a user's requested result value cannot be returned within `timeout` time and `TimeoutException` occurs.

```java
net.spy.memcached.internal.CheckedOperationTimeoutException: Timed out waiting for operation. > 300 - failing node: /127.0.0.1:11211 [WRITING] [#iq=13 #Wops=7 #Rops=10 #CT=13]
```

Meaning of the Log messages are as follows.

| Message                                      | Description |
| -------------------------------------------- | ----------- |
| GET operation timed out (>300 MILLISECONDS ) | Timeout is set to 300ms. if it takes more than 300ms to return the result of GET request then timeout occurs.  |
| Failing node: /127.0.0.1:11211               | Corresponding request performs in 127.0.0.1:11211                                                          |
| [WRITING]	                                   | Corresponding request is pending for socket write.                                                         |
| [READING]                                    | Corresponding request was sent to server. Either waiting for a result to return or reading a result value. |
| #iq	                                         | Number of request pending in the corresponding ARCUS Node Input Queue.                                     |
| #Wops	                                       | Number of request in the state of writing.                                                                 |
| #Rops                                        | Number of request in the state of reading.                                                                 |
| #CT	                                         | Beyond continuous timeout, number of continuous timeouts on the corresponding ARCUS node, timeout threshold value specified by the connection factory builder, the client disconnects from the server and reconnects. |


In other words, this means ARCUS node send request to 127.0.0.1:11211 but the result could not be returned within 300ms, thus causing a failure.
This is a message most likely to occur for various reasons. The cause of the failure can be as follows.

- When the JVM Full GC time value is greater than the operation timeout value
  - Measure the full GC time of the WAS and set the timeout value greater than that.
  - Check if Operation timeout is set to a very small value (most of the time causes are similar to these).
- Network problem between Client and ARCUS Server
  - Check if there are any network(switch, AS, DS etc.) problems between the Service and the ARCUS Server.
  - Check if there is any problem with the connection to the external server that connects to the service.
    If there is no external connection, check the logs to see if there is any connection problem with the Arcus Admin Server.
    Even if there were network-related problems, ARCUS timeout could occur. 
    For example, if ARCUS timeout is set to 1 second and the timeout with DB is 2 seconds.
    If the network is disconnected for 1.5 seconds, only the ARCUS timeout message will appear.
    If a network disconnection occurred for 3 seconds, Arcus |- timeout will occur first, it will then appear that DB timeout occurs.
    Most of the time, the thread of WAS increases when this kind of problems occur.
    If the request process thread fails to respond within timeout, an additional thread is generated to receive additional incoming requests.
- ARCUS Server
  - Check hardware of server and Hubble monitoring tool to find the cause.
  - Check if there is more than one ARCUS host where timeout occurred.
-  Limitation  
  - There is a limit to processing with one client. Consider using a pool.    







