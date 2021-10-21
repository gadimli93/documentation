# Future - get, getSome

- [get](12-arcus-future-get-API-en.md#get)
- [getSome](12-arcus-future-get-API-en.md#getsome)

ARCUS Java Client provides a return type Future implementations as a return value for asynchronous operations.
These implementations not only return asynchronous computational values, but also it supports additional functions.
Among them there is a difference in how `get()` and `getSome()` APIs obtain computed data.

## get

Generally, `get()` of Future without timeout waits until operation is competed and returns the result.
In ARCUS Java Client, default timeout is set and when the preconfigured time is out it throws away 
all the processed data and returns null(all or nothing) with `OperationTimeoutException`.

```java
V get()
V get(long timeout, TimeUnit unit)
```

- `get()`'s default timeout is 700ms. If there is no response within 700ms of waiting time then `OperationTimeoutException` occurs.
- `get(long timeout, TimeUnit unit)` method is the same as the `get()` method's operation but timeout may be explicitly specified.
- default timeout of `get()` operation can be changed using the `setOpTimeout()` method of `ConnectionFactoryBuilder`. 
- During the operation processing if the corresponding operation is interrupted then `InterruptedException` occurs.
- During the operation processing if an exception other than the above-mentioned occurs then it causes `ExecutionException`.

## getSome

`BulkGetFuture`returned through `asyncBulkGet()`, `asyncBulkGet()` API operations supports `getSome()` API.
When timeout occurs `getSome()` API does not throws data like `get()` API and returns null, 
in contrary it returns the processed result(partial) up to that(timeout) point.
Therefore, the returned data can be used in the application and only the data that has not been returned,
needs to be retrieved from the backend database, thereby reducing waste of resources.

```java
V getSome(long timeout, TimeUnit unit)
```

Below is sample code for execution of `get()` and `getSome()`.
Through the sample, you can understand the difference between `get()` and `getSome()`.
After a specified time passes, `get()` `getSome()` `TimeoutException` and
`getSome()` returns processed result during a specified time.

```java
List<String> keyList = new ArrayList<String>();
for(int i = 1; i < 300; i++) {
  client.set("key" + i, 5 * 60, "value");
  keyList.add("key"+i);
}

BulkFuture<Map<String, Object>> future = client.asyncGetBulk(keyList);

Map<String, Object> map = null;
try {
  future.get(6, TimeUnit.MILLISECONDS);
} catch (ExecutionException e) {
  e.printStackTrace();
} catch (TimeoutException e) {
  e.printStackTrace();
}

System.out.println(map);

Map<String, Object> map2 = null;
BulkFuture<Map<String, Object>> future2 = client.asyncGetBulk(keyList);
try {
  map2 = future2.getSome(6, TimeUnit.MILLISECONDS);
} catch (ExecutionException e) {
  e.printStackTrace();
}

System.out.println(map2);
```

The results are as follows.

```java
# get()
null

# getSome()
{
key118=value, key117=value, key114=value, key234=value, key113=value, key237=value, key115=value, key231=value, key230=value,
key42=value, key111=value, key199=value, key232=value, key37=value, key36=value, key35=value, key34=value, key190=value, 
key38=value, key191=value, key129=value, key128=value, key249=value, key51=value, key125=value, key124=value, key245=value, 
key248=value, key126=value, key247=value, key55=value, key121=value, key242=value, key54=value, key53=value, key243=value,
key47=value, key46=value, key217=value, key216=value, key219=value, key62=value, key61=value, key179=value, key60=value, 
key215=value, key297=value, key65=value, key175=value, key211=value, key299=value, key177=value, key58=value, key171=value, 
key292=value, key174=value, key295=value, key56=value, key170=value, key107=value, key228=value, key106=value, key109=value, 
key103=value, key224=value, key102=value, key71=value, key105=value, key226=value, key225=value, key187=value, key76=value, 
key189=value, key74=value, key100=value, key221=value, key183=value, key68=value, key185=value, key67=value, key180=value
}
```

- `get()` returned `null` by timeout. (all or nothing)
- `getSome()` returned the result up to point of timeout. (partial)

