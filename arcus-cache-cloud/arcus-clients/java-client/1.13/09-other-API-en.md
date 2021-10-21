# Other API

This chapter explains below shown the remaining APIs.

- [Flush](09-other-API-en.md#flush)

## Flush

ARCUS provides flush feature by prefix unit.
Among all data that stored in the cache server, all the keys that use a particular prefix can be deleted with once request.
However, Front Cache data will not be flushed.

```java
OperationFuture<Boolean> flush(String prefix)
```

Return `true` if prefix is successfully removed, `false` if a prefix does not exist and removal fails.

**Please be careful about the use of a prefix, due to it can delete all the items in the particular prefix.
Especially, in a public cloud, all items of cache node can be deleted (if you don't enter prefix).**

A sample of flushing a specific prefix.

```java
OperationFuture<Boolean> future = null; 
try { 
    future = client.flush(“myprefix”); 
    boolean result = future.get(1000L, TimeUnit.MILLISECONDS); 
    System.out.println(result); 
} catch (InterruptedException e) { 
    future.cancel(true); 
} catch (TimeoutException e) { 
    future.cancel(true); 
} catch (ExecutionException e) { 
    future.cancel(true); 
}
```

For example, let's suppose the following keys are stored on the ARCUS server.

- mydata:subkey1
- mydata:subkey2
- yourdata:subkey3
- ourdata:subkey4
- theirdata:subkey5

In the sample code above, if you call `client.flush(“myprefix”)` both mydata:subkey1 and mydata:subkey1 keys that use "myprefix"
as a prefix will be removed from ARCUS. And of course `future.get()` would return `true` for successful completion,
if this was a result performed on all the servers that compose ARCUS Cloud.
