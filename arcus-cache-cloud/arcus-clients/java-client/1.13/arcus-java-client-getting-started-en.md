# Getting Started with Java Client

This quick start manual is for Java developers who are new to ARCUS.
We assume that you know the concept and basic usage of [Apache Maven](http://maven.apache.org/) and
rather than to explain it here in detail, we can use right jump into discovering how to use ARCUS
through sample codes(for the purpose of learning just Copy & Paste).

## ARCUS

ARCUS is memcached-based an open source key-value cache server which is partially fault-tolerant
in-memory based cache cloud.

- **memcached :** a memory cache server that is used on large scale in Google, Facebook, etc.,
- **cache :** it is a service that reduces requests to a slow storage and allows you to expect 
  faster responsiveness, by placing frequently used data in relatively high-speed storage.
- **in-memory based :** ARCUS stores data only in memory, therefore, all data is volatile and can be deleted at any time.
- **cloud :** each service can configure a dedicated cache cluster when needed, allowing user to
   dynamically add or delete cache servers.(However, some data will be lost.)
- **fault-tolerant :** ARCUS detects an abnormal state of some or all cache servers and
  takes applicable actions.

ARCUS also provides the ability to store not only key-value data model but also data structures 
such as List, Set, Map, and B+Tree.

## Basic Knowledge

- **key**
  - The key of ARCUS consists of prefix and subkey. Prefix and subkey are divided by colons (:).
    e.g. users:user_12345
  - ARCUS collects separate statistical information based on prefix. There is no limit on the 
    number of prefixes, but when collecting statistics, it is recommended to generate them
    at a level(5~10) that is not too much.
  - key cannot exceed 4000 characters, including prefix and subkey, therefore, you have to
    limit the key length in the application.
  
- **value**
  - the value for one key can be stored up to 1MB in the form of a byte stream.
  - in the case of storing a Java object, the object must implement Serializable interface.
  
- **ARCUS Access Information**
  - **ARCUS Admin :** as the ZooKeeper server address, it retrieves the IP and PORT information
    of cache servers and notifies the client when there is a change.
  - **ARCUS Service Code :** code value that distinguishes cache servers assigned to a user or service.


## Hello, ARCUS!

Let's do a basic key-value cache request.
Assuming that the arcus server is already configured, first create an empty Java project as follows.

```java
$ mvn archetype:generate -DgroupId=com.navercorp.arcus -DartifactId=arcus-quick-start -DinteractiveMode=false
$ cd arcus-quick-start
$ mvn eclipse:eclipse // If you are using the Eclipse IDE, run this to create Eclipse project.
```

### pom.xml

When Java project is created, next add the required ARCUS Client dependencies to pom.xml file as shown below.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.navercorp.arcus</groupId>
    <artifactId>arcus-quick-start</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>arcus-quick-start</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- For convenience, change the JUnit version to 4.x. -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.4</version>
            <scope>test</scope>
        </dependency>

        <!-- Add ARCUS client dependence. -->
        <dependency>
            <groupId>com.navercorp.arcus</groupId>
            <artifactId>arcus-java-client</artifactId>
            <version>1.13.2</version>
        </dependency>
        
        <!-- Add logger dependence. -->
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
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.8.2</version>
        </dependency>
    </dependencies>
</project>
```

### HelloArcus.java

Now, create a class that will communicate with ARCUS. Please refer to the sample code below.

- HelloArcus.sayHello(): Stores "Hello, Arcus!" value to the Arcus Cache Server.
- HelloArcus.listenHello(): Reads "Hello, Arcus!" value stored in the Arcus Cache Server.

```java
// HelloArcusTest.java
package com.navercorp.arcus;

import junit.framework.Assert;

import org.junit.Before;
import org.junit.Test;

public class HelloArcusTest {

    HelloArcus helloArcus = new HelloArcus("127.0.0.1:2181", "test");
    
    @Before
    public void sayHello() {
        helloArcus.sayHello();
    }
    
    @Test
    public void listenHello() {
        Assert.assertEquals("Hello, Arcus!", helloArcus.listenHello());
    }  
}
```
```java
// HelloArcus.java
package com.navercorp.arcus;

import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

import net.spy.memcached.ArcusClient;
import net.spy.memcached.ConnectionFactoryBuilder;

public class HelloArcus {

    private String arcusAdmin;
    private String serviceCode;
    private ArcusClient arcusClient;

    public HelloArcus(String arcusAdmin, String serviceCode) {
        this.arcusAdmin = arcusAdmin;
        this.serviceCode = serviceCode;
        
        // Enable log4j logger.
        // You can use the JVM environment variable below without adding it directly to the code.
        //  -Dnet.spy.log.LoggerImpl=net.spy.memcached.compat.log.Log4JLogger
        System.setProperty("net.spy.log.LoggerImpl", "net.spy.memcached.compat.log.Log4JLogger");

        //  Create Arcus client object.
        // - arcusAdmin : address of the admin server (ZooKeeper) that manages groups of ARCUS cache servers.
        // - serviceCode : code value for the set of ARCUS cache servers assigned to the user. 
        // - connectionFactoryBuilder : allows specifying client creation options.
        //
        // In summary, the combination of arcusAdmin and serviceCode allows to obtain and connect unique set of cache servers.
        this.arcusClient = ArcusClient.createArcusClient(arcusAdmin, serviceCode, new ConnectionFactoryBuilder());
    }

    public boolean sayHello() {
        Future<Boolean> future = null;
        boolean setSuccess = false;

        // Store the "Hello, Arcus!" value in the "test:Hello" key of ARCUS.
        // and almost all of ARCUS APIs are supposed to return Future.
        // Unless it's a server specialized in asynchronous processing, you must explicitly perform future.get().
        // Wait for the response to be returned.
        future = this.arcusClient.set("test:hello", 600, "Hello, Arcus!");
        
        try {
            setSuccess = future.get(700L, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            if (future != null) future.cancel(true);
            e.printStackTrace();
        }        
        return setSuccess;
    }
    
    public String listenHello() {
        Future<Object> future = null;
        String result = "Not OK.";
        
        // Retrieve value of "test:hello" key of ARCUS.
        // ARCUS guides you to explicitly specify timeout value for all possible commands.
        // User should use an API that starts with async for all requests except set.
        future = this.arcusClient.asyncGet("test:hello");
        
        try {
            result = (String)future.get(700L, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            if (future != null) future.cancel(true);
            e.printStackTrace();
        }        
        return result;
    }
}
```

### src/test/resources/log4j2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%-5p](%-35c{1}:%-3L) %m%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="WARN">
            <AppenderRef ref="console" />
        </Root>
        <Logger name="net.spy.memcached.StatisticsHandler" level="INFO" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.ArcusClient" level="INFO" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.BTreeGetBulkOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionInsertOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionPipedInsertOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionGetOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.BTreeSortMergeGetOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionDeleteOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionUpdateOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionPipedExistOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.SetAttrOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.StoreOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
        <Logger name="net.spy.memcached.protocol.ascii.CollectionCountOperationImpl" level="DEBUG" additivity="false">
            <AppenderRef ref="console" />
        </Logger>
    </Loggers>
</Configuration>
```

## Test

The above example assumes that ZooKeeper is working on `127.0.0.1:2181` and memcached server
is running. If you're not ready yet, follow the Running Test Cases on the below page.

https://github.com/naver/arcus-java-client/blob/master/README.md

You will receive the following output when the test passes.

```java
$ mvn test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
...
Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.885s
[INFO] Finished at: Mon Dec 17 14:13:22 KST 2012
[INFO] Final Memory: 4M/81M
[INFO] ------------------------------------------------------------------------

```









