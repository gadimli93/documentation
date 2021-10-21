# Arcus Java Client

- [How To Use ARCUS Client](02-arcus-java-client-en.md#how-to-use-arcus-client)
- [Create, Destroy, Manage ARCUS Java Client](02-arcus-java-client-en.md#create-destroy-manage-arcus-java-client)
- [ARCUS Client Settings](02-arcus-java-client-en.md#arcus-client-settings)

## How To Use ARCUS Client

Through the sample code below, let's explore default usage of ARCUS Client.
In the example, we store cache item in ARCUS Cache where key is "sample:testKey" and a value is "testValue". 

```java
package com.navercorp.arcus.example; 

import java.util.concurrent.ExecutionException; 
import java.util.concurrent.Future; 
import java.util.concurrent.TimeUnit; 
import java.util.concurrent.TimeoutException; 
import net.spy.memcached.ArcusClient; 
import net.spy.memcached.ConnectionFactoryBuilder; 

public class HelloArcus { 

    private static final String ARCUS_ADMIN = "10.0.0.1:2181,10.0.0.2:2181,10.0.0.3:2181"; 
    private static final String SERVICE_CODE = "test"; 
    private final ArcusClient arcusClient; 

    public static void main(String[] args) { 
        HelloArcus hello = new HelloArcus(); 
        System.out.printf("hello.setTest() result=%b", hello.setTest()); 
        hello.closeArcusConnection(); 
    } 

    public HelloArcus() { 
        arcusClient = ArcusClient.createArcusClient(ARCUS_ADMIN, SERVICE_CODE, 
                new ConnectionFactoryBuilder()); // (1) 
    } 

    public boolean setTest() { 
        Future<Boolean> future = null; 
        try { 
            future = arcusClient.set("sample:testKey", 10, "testValue"); // (2) 
        } catch (IllegalStateException e) { 
            // exception handling when a request is not registered due to client operation queue. 
        } 

        if (future == null) return false; 

        try { 
            return future.get(500L, TimeUnit.MILLISECONDS); // (3) 
        } catch (TimeoutException te) { // (4) 
            future.cancel(true); 
        } catch (ExecutionException re) { // (5) 
            future.cancel(true); 
        } catch (InterruptedException ie) { // (6) 
            future.cancel(true); 
        } 

        return false; 
    } 

    public void closeArcusConnection() { 
        arcusClient.shutdown(); // (7) 
    } 
} 
```

1. Create an object(client object) of ArcusClient class. Client objects are not created per request but
   made only one in advance and reused it. When connecting to ARCUS, `ConnectionFactoryBuilder` used 
   to change all kind of settings
   - **if an invalide SERVICE_CODE is specified, `NotExistsServiceCodeException` occurs.**
   - **if SERVICE_CODE is correct but if there is no accessible cache server(or node),
     every request results in Exception. When the cache server is running and and becomes accessible,
     it is automatically connected to the corresponding cache server to provide normal service.**
     
2. Call the `set` method of the client object, the result is obtained as the object(future object) of a Future class with Boolean type value.
   - "if (string with a length of 0) is added" as the value to be stored, it's stored as it is.(The corresponding key is not deleted)
   - `null` cannot be specified as the value to be stored.(if you intend to delete  the key, use the `delete` method)
   - values to be stored should be serializable.(for custom classes, serializable interfaces must be implemented)

3. Get the value of Future object to obtain result.(if `set` operation fails, as a result returns `false`)
   - if the key does not exist in the cache server, in other workd, if it's cache miss `null` is returned.
   - The string ("") having a length of 0 is not a cache miss, but a stored "".

4. If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM TimeoutException occurs.
   - for example, timeout is set to 500ms but GC time took 600ms, `TimeoutException` would occur because it exceeded for 100ms
     despite there was no problem communicating with the ARCUS Cache Server.
   - if `TimeoutException` occurs more than n times(default is 10) in a row, then client disconnects with the server and 
     tries reconnect again. The value n can be specified when creating the ConnectionFactoryBuilder.
   - additionly, `future.cancel(true)` must be called in all situations where Exception occurs.

5. When the job waiting in the operation queue of the ArcusClient is canceled, `ExecutionException` occurs.
   `ExecutionException` occurs in the event of a server or network error. 
   
6. `InterruptedException` occurs when the work of the thread is interrupted by another thread. (unlikely to happen)

7. Before to end the process, since no longer ArcusClient will be used, you MUST call the `shutdown` method to disconnect from the server.
   - in WAS same as Tomcat, when Tomcat shuts down, all you have to do is to call `shutdown` method.
   - when managing Spring container, `shutdown` method must be set to be called in destroy-method of bean settings

## Create, Destroy, Manage ARCUS Java Client  

### Create ARCUS Client

One ARCUS Client Object creates connection one by one to all cache servers (or cache nodes) in the ARCUS Cache Cloud.
For each requested cache item's key, using the connection with the cache server where the key is mapped,
a request is sent and a response is returned.

There are two ways to create an ARCUS Client object.
- Create a single ARCUS Client
- Create a pool of ARCUS Clients

1. A method for creating a single ARCUS Client object is as follows.

```java
ArcusClient.createArcusClient(String arcusAdminAddress, String serviceCode, ConnectionFactoryBuilder cfb)
```

- arcusAdminAddress : an address of ARCUS ZooKeeper Ensemble that manages cache cloud to connect
  - IP:port : specifies port list in the form of "ip1:port,ip2:port,ip3:port"  or
  - specifies it in the from of "FQDN:port" (If the domain name for the ZooKeeper's IP list is registered in DNS)
- serviceCode : identifier of cache cloud to connect
- cfb : `ConnectionFactoryBuilder` object for the settings of ARCUS Client's operation

An example of creating a single ArcusClient object connecting to cache cloud corresponding SERVICE_CODE managed by ARCUS_ADMIN server is as follows.

```java
ConnectionFactoryBuilder cfb = new ConnectionFactoryBuilder();
ArcusClient client = ArcusClient.createArcusClient(ARCUS_ADMIN, SERVICE_CODE, cfb);
```

There is a limitation to throughput capacity to process application requests with only one ARCUS Client.
For example, assuming that one request is processed through one connection takes 1ms,
through the connection can only up to 1000 requests/seconds be processed.
Therefore, in the case of an application requiring a large amount of request throughput,
multiple ARCUS Client objects must be created. To this end, an ARCUS Client Pool object can be created using the method below.

```java
ArcusClient.createArcusClientPool(String arcusAdminAddress, String serviceCode, ConnectionFactoryBuilder cfb, int poolSize);
```
In addition to factors when creating a single ARCUS Client object as a factor of the method,
there is a poolSize factor that specifies the number of ARCUS Client objects to enter the pool.
If the pool size is too small, there is a problem that application requests cannot be processed on time, and
if it is too large, unnecessarily many connections are made with the ARCUS Cache server.
The applicable pool size can be obtained by dividing the "request amount of `peak arcus request` of application server" by 
"throughput of one ARCUS Client". Here, the throughput that one ARCUS Client can process may be affected by the type of ARCUS request
by the application server and the network status between the application server and the cache server. Therefore it is recommended
to determine the pool size through the test.

An example of creating a pool of four ARCUS Clients connecting to cache cloud corresponding SERVICE_CODE is as follows.

```java
int poolSize = 4;
ConnectionFactoryBuilder cfb = new ConnectionFactoryBuilder();
ArcusClientPool pool = ArcusClient.createArcusClientPool(ARCUS_ADMIN, SERVICE_CODE, cfb, poolSize);
```

When the ARCUS Client object is succesfully created, you will get the below shown log message.

```java
WARN net.spy.memcached.CacheManager: All arcus connections are established.
```

If it is not connected successfully to the ARCUS Cache Cloud, then below shown log message will appear.
For example, if you try to connect five cache servers, but some of them have not been able to connect, the log below will remain.
And for cache servers that failed to connect, the ARCUS Client automatically attempts to reconnect once a second.

```java
WARN net.spy.memcached.CacheManager: Some arcus connections are not established.
```

### Destroy ARCUS Client

After using ArcusClient or ArcusClientPool, you MUST call the `shutdown()` method to disconnect the client from Admin, and cache server.

```java
client.shutdown();
pool.shutdown();
```

### Manage ARCUS Client Life Cycle

It is not applicable to create and destroy an ARCUS Client object for each request to ARCUS.
When the application server is running, create the ARCUS Client object, and at the end, destroy the ARCUS Client object.

Generlly, it's recommended to create and use ArcusClient wrapper in the application.
This makes easier to manage a life cycle of ArcusClient.
Let's turn the `factory` owning ArcusClient instance for each SERVICE_CODE into singleton and
when WAS is reinitiated, connect with ARCUS Server. When WAS is shutdown, it is most ideal to set ArcusClient also to shutdown together.

### Manage Cache Server List

ARCUS automatically manages the cache server list. when some of the cache servers become unavailable,
ARCUS Admin automatically acknowledges the situation and removes the unavailable server from the cache server list.
Once it's removed, ARCUS Admin notifies each client about the changes, thus every ARCUS client maintains the latest cache server list.
To the contrary, when new cache server is added, similarly ARCUS Admin updates mapping of cache key and cache server,
notifies the clients, thus ARCUS clients maintain the latest cache server list.
Hence when you're using ARCUS Client, you don't need to pay extra attention to the defense logic of cache server count change.

## ARCUS Client Settings

### Data Compression Settings in Key-Value

ARCUS Client has the feature of compressing and decompressing data of key-value items.
If data is larger than a certain size, the data can be compressed and then stored in the cache server,
and if the data obtained from the cache server is compressed, it is decompressed and delivered to the application.

ARCUS Client is configured to compress value size over 16KB and store in a cache server.
The data compression threshold can be set with `setTranscoder` method of `ConnectionFactoryBuilder`.

Following is the sample of setting all data over 4KB to be compressed.

```java
ConnectionFactoryBuilder cfb = new ConnectionFactoryBuilder();

SerializingTranscoder trans = new SerializingTranscoder();
trans.setCharset(“UTF-8”);
trans.setCompressionThreshold(4096);

cfb.setTranscoder(trans);

ArcusClient client = ArcusClient.createArcusClient(SERVICE_CODE, cfb);
```

### Logger Settings

ARCUS Client allows using four types of Logger, such as `default(DefaultLogger)` `log4j(Log4JLogger)` `slf4j(SLF4JLogger)`
`jdk(SunLogger)` etcetera. If you do not specify Logger to use, then by default ArcusClient uses `DefaultLogger`. 
`DefaultLogger` outputs a log of INFO level or higher ones as `stderr(System.err)`. (Unable to Change)

In order to manage logs of ArcusClient using `log4j`, add the below option to the WAS or Java process options
to specify the **system property** when JVM is running.

```java
-Dnet.spy.log.LoggerImpl=net.spy.memcached.compat.log.Log4JLogger
```

Alternatively, prior to using ArcusClient/ArcusClientPool the system property may be directly set and used
in the source code. (programmatic configuration)

```java
System.setProperty("net.spy.log.LoggerImpl", "net.spy.memcached.compat.log.Log4JLogger");
...
ConnectionFactoryBuilder cfb = new ConnectionFactoryBuilder();
ArcusClient client = ArcusClient.createArcusClient(SERVICE_CODE, cfb);
```

In the ARCUS Java Client when recording Log, Logger is used separately based on the name(`clazz.getName()`) of Class,
and if there's no Log matching the name of the class, it uses the upper Logger on the logger tree.

In the example below, level of the `root` logger is set to `WARN` and logs above WARN level are always recorded,
only logs of `net.spy.memcached.protocol.ascii.CollectionUpdateOperationImpl` class are recorded at the DEBUG level or higher.

```xml
<Root level="WARN">
    <AppenderRef ref="console" />
</Root>
<Logger name="net.spy.memcached.protocol.ascii.CollectionUpdateOperationImpl" additivity="false" level="DEBUG">
    <AppenderRef ref="console" />
</Logger>
```

When the application needs to be debugged, there are times when you are curious about the ascii protocol string
that the ARCUS client transmits to the Arcus server. To view the transmitted protocol from the ARCUS Java Client
to the ARCUS Server as a log, set the logger as follows. It is convenient to set only the logger corresponding 
to the necessary request, because if you set all the loggers listed in the sample, all logs for 
each request((get, set, etc.,) remains unused. For more information on ASCII Protocol, please refer to the 
[ARCUS Server Command Protocol](/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch00-arcus-ascii-protocol-en.md) documentation.

```xml
<!-- collection update -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionUpdateOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- collection piped exist -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionPipedExistOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- set attributes -->
<Logger name="net.spy.memcached.protocol.ascii.SetAttrOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- collection insert -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionInsertOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- collection get -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionGetOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- collection update -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionUpdateOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>

<!-- collection count -->
<Logger name="net.spy.memcached.protocol.ascii.CollectionCountOperationImpl" level="DEBUG" additivity="false">
    <AppenderRef ref="console" />
</Logger>
```

For further details on log4j setting method, please check the [log4j configuration](http://logging.apache.org/log4j/2.x/manual/configuration.html).

### Usage Precautions of Log4JLogger 

There is a security vulnerability in log4j `1.2` and below versions, therefore from the ARCUS Client version 1.11.5
in order to use to use `Log4JLogger` a log4j2 library is required.

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.8.2</version>
</dependency>
```

If the following exception occurs, the log4j2 library does not exist in the class path.
Check that the log4j2 library is properly added to the application dependence.

```java
Warning:  net.spy.memcached.compat.log.Log4JLogger not found while initializing net.spy.compat.log.LoggerFactory
java.lang.NoClassDefFoundError: org/apache/logging/log4j/spi/ExtendedLogger
    at java.base/java.lang.Class.forName0(Native Method)
    at java.base/java.lang.Class.forName(Class.java:315)
    at net.spy.memcached.compat.log.LoggerFactory.getConstructor(LoggerFactory.java:134)
    at net.spy.memcached.compat.log.LoggerFactory.getNewInstance(LoggerFactory.java:119)
    at net.spy.memcached.compat.log.LoggerFactory.internalGetLogger(LoggerFactory.java:100)
    at net.spy.memcached.compat.log.LoggerFactory.getLogger(LoggerFactory.java:89)
    at net.spy.memcached.ArcusClient.<clinit>(ArcusClient.java:183)
    at Main.main(Main.java:10)
```

### Usage Precautions of SLF4JLogger

In the case of slf4j the `SLF4JLogger` class of the ARCUS Client will be used.
To use this class, a logging library that implements `slf4j` must be added to the application dependence.
Otherwise, the following exception message will occurs.

```java
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

Java's log libraries such as log4j, logback, provide implementation libraries that implemented slf4j API.
When using this library, add it to the application dependence as shown follows.
For more information, please check the [slf4j](http://www.slf4j.org/manual.html#swapping) documentation.

```xml
<!-- slf4j + log4j 사용시 -->
<dependency>
    <groupId>com.navercorp.arcus</groupId>
    <artifactId>arcus-java-client</artifactId>
    <version>${arcus-java-client.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>${log4j.version}</version>
</dependency>
```

```xml
<!-- slf4j + logback 사용시 -->
<dependency>
    <groupId>com.navercorp.arcus</groupId>
    <artifactId>arcus-java-client</artifactId>
    <version>${arcus-java-client.version}</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>${logback.version}</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>${logback.version}</version>
</dependency>
```

In addition, if two or more slf4j implementation libraries (log4j-slf4j-impl, logback-classic, ...)
exist in the same class path, a [multiple binding error](http://www.slf4j.org/codes.html#multiple_bindings) occurs in SLF4J, therefore it is essential to use the **exclusion** keyword to ensure that only one library for implementation of slf4j exists.


```java
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
```

### Usage of Transparent Front Cache

ARCUS is a remote cache system, due to that is a disadvantage in whenever a response to a request is received
data must be objectified. This will eventually put a burden on JVM's Garbage Collector.
Therefore, if the actual data is rarely changed, even if there's a change, 
if it doesn't matter if you'll get the previous data in a very short time, the it is worth considering using Front Cache.

When there is a Hit in Remote cache, in order to use Front Cache,
you have to store records in separate Front Cache repositories but then code tends to get chaotic.
Additionally, when you retrieving data, it first checks front cache as shown in the below image,
and after then checks remote cache, this is also complicates the program.

<img src="https://raw.githubusercontent.com/jam2in/arcus-docs/master/docs/english-manual/arcus-clients/java-client/images/java_client_encache.png" height="350" width="500"></img>

Therefore, in a transparent way, if library could activate Front Cache on its own, JVM can store items for a given
period of time, then it will be possible to develop convenient and faster applications. ARCUS has added Ehcache
as a plug-in local cache, enabling the use of Front Cache immediately without complicated program work. 
Users can automatically perform tasks 2 and 3(shown in the above image) within the library by simple setting options.

The following ones apply when creating a `ConnectionFactoryBuilder` class as a method for using Front Cache.

- `setMaxFrontCacheElements(int to)` (Required)
   - The value applied here indicates the maximum number of items to be used in Front Cache.
     The default value is 0, which means that Front Cache is not used.
     Therefore, to use Front Cache, a positive integer value must be specified.
     If the maximum number of items exceeds, with LRU algorithm the least recently used items will be removed and new item will be registered.
     
- `setFrontCacheExpireTime(int to)` (Optional, default 5)
    - Expiretime of Front Cache item. Front cache does not set the expire time for each item,
      in contrary it sets the same expiretime to all registered items.
      The default value is 5 and the unit is second. If not set the expiretime, then the default value is used as it is,
      items automatically will disappear after 5 seconds of registration.
      
- `setFrontCacheCopyOnRead(boolean copyOnRead)` (Optional, default false)
    - A setting to activate the `copy on read` option of the Copy Cache feature in Front Cache,
      and default value is `false`.
- `setFrontCacheCopyOnWrite(boolean copyOnWrite)` (Optional, default false)
    - A setting to activate the `copy on write` option of the Copy Cache feature in Front Cache,
      and default value is `false`.

Please refer to the [EhCache's Copy Cache feature](https://www.ehcache.org/documentation/2.8/get-started/getting-started.html) documentation for more information.

Warnings for using Front Cache are as follows.
- Transparent Front Cache is currently only applicable to `get/set` of Key-Value.
- Front Cache is mainly suitable for catching *read-only* data because it does not synchronize well with remote ARCUS.
  In addition, expire time of Front Cache should be identified and set based on the period of time when the 
  synchronization does not fit well according to the `remote cache entry update`.
- Front Cache data cannot be flush with the Flush command.
 
 Below is a sample code for using Front Cache. If `setMaxFrontCacheElements` is set to a value greater than 0,
 Front Cache is activated. (It's recommeded to set `setFrontCacheExpireTime` also to explicit values for the purpose of use.)

```java
ConnectionFactoryBuilder factory = new ConnectionFactoryBuilder();

/* Required to use transparent front cache */
factory.setMaxFrontCacheElements(10000);

/* Optional settings */
factory.setFrontCacheExpireTime(5);
factory.setFrontCacheCopyOnRead(true);
factory.setFrontCacheCopyOnRead(true);

ArcusClient client = new ArcusClient(SERVICE_CODE, factory);
```

If the application is divided into the parts where the Front Cache should be used for ARCUS and where it should not,
then ARCUS Client objects suitable for each purposes should be created and used separately.

### Key Methods of the ConnectionFactoryBuilder Class

- setFailureMode(FailureMode fm)

  Updates the Failure Mode. There are three FailureMode: Cancel, Redistrubute, Retry; each of their meaning is as follows.

  - **Cancel**: when the node is *down* automatically cancels all operations request going to that node.
  - **Redistribute**: in the case of multiple nodes are registered and if the request fails,
      the corresponding request send again to the next node. This way, it's performed until `timeout` occurs.
      If there is only one node, then the next node becomes itself again.
  - **Retry**: if a request fails until the `timeout`, it attempt to continues sending a request to the current node.
 
  ARCUS uses *Cancel* mode as a default. If *Redistribute* or *Retry* is used, to avoid repetitive requests and 
  overloads on application server, use of these two methods is prohibited.

- setOpTimeout(long t)

  `SpyThread` sets the operation `timeout` in milliseconds while receiving a response from the ARCUS Cache Server.
  The default value is 1,000 milliseconds. For the reference, until application gets a `Callback`,
  configured `timeout` is different from timeout contains "Operation timeout + command creation time + command registration time".

- setProtocol(ConnectionFactoryBuilder.Protocol prot)

  Specifies the protocol to be used between the ARCUS Client and Server.
  There are two protocols, `Text` and `Binary`, but only **Text** protocol should be used in ARCUS.

- setMaxReconnectDelay(long to)

  If the connection with ARCUS is lost, in order to connect again specify the maximum waiting time in seconds.
  ARCUS as a default uses 1 second.

- setOpQueueFactory(OperationQueueFactory q)

  Create an operation queue containing the contents of the command. As a dafult 16,384 sized queue is used.
  If you want to change the size of the queue to 1000, you can updated it 
  to `setOpQueueFactory(new ArrayOperationQueueFactory(1000))`.

- setTranscoder(Transcoder t)
  
  Sets the `character set` and `compression criteria` about the data area of the cache.
  GZip compression is used, and default values are **UTF-8 and 16,384 bytes**.
  In other words, data area of all requests is encoded/decoded to UTF-8 and
  when the size of data area is 16,384 or more, it is compressed to be able communicate with ARCUS.  
  
  Below is a sample of updating character set to EUC-KR and the compression criteria to 4,096 bytes.

```java
SerializingTranscoder trans = new SerializingTranscoder();
trans.setCharset(“EUC-KR”);
trans.setCompressionThreshold(4096);
setTranscoder(trans);
```

- setShouldOptimize(boolean o)

  Determines whether to use the optimization logic. Optimization logic performs get operations in sequence
  in the Operation Queue in combination of `get` operation with `multi-get`.
  **ARCUS does not recommend use optimize logic.**
  
- setReadBufferSize(int to)

  Sets used entire ByteBuffer size when communicating with ARCUS Server Socket
  (even tough its name *Read*, but the size of the *read/write buffer* follows this value). If the data exceeds 
  the ByteBuffer size, in order to increase re-usability, after processing as much as ByteBuffer size holds,
  contents of ByteBuffer are removed and reused again.

- setDaemon(boolean d)

  The default value is `true`.

- setTimeoutExceptionThreshold(int to)

  When timeout occurs continuously, then a problem is in the connection, therefore disconnect the connection and
  try to connect again. The default continuous `timeout limit` used by ARCUS is 10.

- setTimeoutRatioThreshold(int to)

  If for some reason the client request is not processed for a long time, ARCUS client detects this using the 
  continuous timeout method and sends a quick failure response to the application.
  
  If the client request is not processed for a long time or its processing speed is very slow, some requests
  can get an operation timeout where other requests may just processed normally. In this case, client request does not
  processed normally, but continuous timeout may not occur. In order to detect these kind of conditions 
  calculate the timeout ratio for the last 100 requests, if it is above a specific threshold, 
  this is a function of attempting to disconnect and reconnect the current connection. Accordingly, 
  depending on the nature of the failed request, application decides whether send request to DB or re-request from ARCUS.
  
  The *Timeout Ratio Threshold* is disabled by default value 0. To set to operate the timeout ratio threshold 
  give a value between 1~99. 
  
- setOpQueueMaxBlockTime(long t)

  Specifies maximum waiting time when asynchronously register in the Operation Queue and request an Operation. 
  This option is used only when Queue is full and a request for job is in order
  The unit is millisecond, and the default value is 10000ms.









