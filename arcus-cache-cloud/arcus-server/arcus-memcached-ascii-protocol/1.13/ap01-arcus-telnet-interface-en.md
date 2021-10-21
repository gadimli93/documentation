# ARCUS Telnet Interface

You can use a telnet interface as a method of simply checking the operation of the ARCUS Cache Server.

## How to use Telnet

Execute the telnet command on the OS prompt as follows.
The factor of the telnet command gives the IP and port number of memcached which is the ARCUS Cache Server you want to connect to. 

```
$ telnet {memcached-ip} {memcached-port}
```

## Connect to Telnet

Let's assume that memcached is running in port number 11211 of localhost.
In order to connect to the memcached with the telnet command, perform the following command on the OS prompt.

```
$ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
```

After connecting to a memcached with the telnet command, you can directly perform the ARCUS ASCII commands.
Examples are given below. Please refer to [ARCUS Cache Server ASCII Protocol](ch00-arcus-ascii-protocol-en.md)
for a detailed description of the ARCUS ASCII commands.

### Example 1. get/set

In order to store <"foo" and "fooval"> as a single key-value item, enter a *set* command as follows.

```
set foo 0 0 6
fooval
```

As a result of performed *set* command, return the string that the key-value item has been successfully stored.

```
STORED
```

To retrieve the stored foo item enter a *get* command as follows.

```
get foo
```

The retrieval result with the *get* command for foo item is as follows.

```
VALUE foo 0 6
fooval
END
```

### Example 2. B+tree

To add 5 elements while creating one b+tree item, perform the following 5 *bop insert* commands one after another.

```
bop insert bkey1 90 6 create 11 0 0
datum9
bop insert bkey1 70 6
datum7
bop insert bkey1 50 6
datum5
bop insert bkey1 30 6
datum3
bop insert bkey1 10 6
datum1
```

The result of the execution of the 5 *bop insert* commands are as follows.

```
CREATED_STORED
STORED
STORED
STORED
STORED
```

In order to search for elements from 30 to 80 belonging to the bkey(b+tree key) range  in the b+tree,
enter the *bop get* command as follows.

```
bop get bkey1 30..80
```

The retrieval result with the *bop get* command is as follows.

```
VALUE 11 3
30 6 datum3
50 6 datum5
70 6 datum7
END
```

## Terminate Telnet

To terminate the current telnet connection, enter the quit command.

```
quit
```



