# Java Client Warnings

## Counter Usage

When using counter in ARCUS, the counter key must be registered by intially using set or add commands.
Let's store it as follows.

```java
client.set("nhn_counter", 10000, 100); // NEVER use it this way!
```

You'll get a message that it was successfully stored. Now let's increment the value by using `incr` counter.

```java
Future<Long> f = client.asyncIncr("nhn_counter", 1);
Object o = f.get(1000, TimeUnit.MILLISECONDS);
```

But this time you'll receive an error message “cannot increment or decrement non-numeric value”.
If you lookup the counter details from the cache server you can see that there stored "d".
Why did you get error message? There are two reasons for that:

1. Since the data type itself has not been sent, the byte value was stored as it is when you stored the data in the cache server.
  For the reference a bit value of 100 is '01100100' and as an ASCII code value is 'd'.
  Then you might assume that you could just change the value into integer from counter but there is another reason for that why you cannot.
  
2. Cache servers must support multi-platform, so the data can be stored regardless of `byte-ordering(endianness)`.
  More specificly, counter of a cache server is supposed to receive only enteger values, but
  the only way to operate regardless of platform, you have no choice but to enter the value in String.
  If cache server supported a primitive values, namely, if byte-ordering uses different clients, then counter would not be
  operating the way it designed to be. Therefore, as mentioned earlier, the value 'd' could not be changed into integer value. 
  
ARCUS forces that initial value of data to be used as a counter must be always stored as String.

```java
client.set("nhn_counter", 10000, "100"); // MUST be used this way!
```

On the other side, if your initially stored value is 52, the you won't receive an error message when using counter, because
ASCII code value of 52 is 4. [lacks the explation]

Additionally, in ARCUS Client although input value of incr/decr command can receive a primitive or numeric wrapper value,
internally it is supposed to be converted into String and delivered it cache server.

## Operation Queue Block Timeout Settings

In the case of timeout is set, when registering operation in queue using `setOpQueueMaxBlockTime(long t)` function,
all operations must catch `IllegalStateException` as shown in the below code.

```java
Future<Boolean> future = null;
try {
    future = client.set(key, 60 * 60 * 24, value);
} catch (IllegalStateException ise) { // operation queue failed to register Operation within timeout when it's full
    System.out.println("illegal state exception");
}

if (future != null) {
    try {
        boolean result = future.get(1000, TimeUnit.MILLISECONDS);
    } catch (TimeoutException e) {
        f.cancel(true);
        System.out.println("timeout exception");
    } catch (ExecutionException e) {
        f.cancel(true);
        System.out.println("execution exception");
    } catch (InterruptedException e) {
        f.cancel(true);
        System.out.println("interrupted exception");
    }
}
```

If the `operation queue block timeout` option is not set, client functions that request job can all be blocked.
When actually performing a `future.get`, registered job into operation queue canceled by normally performed exception
will continuously exit thus steadily there will be open space to register new job in the operation queue.
However, if the query speed of `request` is much faster than processing speed of the registered job in the operation queue,
rather than querying that `request`, it should be treated as a failure immediately and 
sending the request to back-end datastores can be a better option.

## Expiretime Settings

Expiretime by the number of seconds for specified time changed and stored in Unix Time, which is the time of the future.
However if expiretime exceeds 30 days, it will be changed to "Unix time" of 1970.
For example, if register the expiretime in the same way as `1000 * 60 * 60`, whihc approximately is 40 days, 
it will recognized as an unix time of 1970 and immediately expires due to it will recognize as it's been a long time.
Therefore, if you sure that you have stored into client and performed retrieval command(get,gets), there can be a phenomenon
that cache data never will be returned.

## Operation Timeout Settings

Timeout can be specified when calling all asynchronous methods in the ARCUS Client.
It is recommended to specify first and then use these timeout values.

```java
Future<Boolean> setResult = client.set("sample:testKey", 10, "testValue");
boolean result = setResult.get(300L, TimeUnit.MILLISECONDS);
```

 The sample code specifies a timeout value as 300ms when storing "testValue" in the ARCUS cache server.

1. Full GC (garbage collection) occurred at the time when this code was executed,
and if the full GC time was 500ms, this request would be timeout.
**Therefore, the timeout value should be set considering the JVM full GC time.
In most cases, dozens of timeouts occur because of full GC time is long.**

2. Packet retransmission may occur between the cache client and the cache server for reasons
such as burst traffic, small packet buffer size etc. In Linux environments, the minimum retransmission
timeout is 200ms, followed by 400ms, 800ms, ... it will be twice as long.
Packet transmission is quite common, therefore it is recommended to set the timeout tolerable for such a packet retransmissions.
300ms and 700ms are recommended timeout values.
