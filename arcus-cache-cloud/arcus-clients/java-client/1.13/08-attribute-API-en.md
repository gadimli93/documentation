# Item Attributes

Item attributes represent the metadata of each cache item.
Please refer the [Item Attributes of ARCUS Cache Server](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch03-item-attributes-en.md) for the basics of Item attributes.

Functions for update or retrieve operations of item attributes are as follows.

- [Update Attribute](08-attribute-API-en.md#update-attribute)
- [Retrieve Attribute](08-attribute-API-en.md#retrieve-attribute)

## Update Attribute 

A function to update the attributes of a given key.

```java
CollectionFuture <Boolean> asyncSetAttr(String key, Attributes attr)
```

Until you insert all the elements into a collection,
you  should not retrieve the elements of the collection on several other threads.
To this end, below is a sample, where the READABLE property is set to `false`, thus 
making the collection UNREADABLE. After all elements have been inserted change property of READABLE to `true`.

```java
// Create unreadable list.
CollectionAttributes attribute = new CollectionAttributes();
attribute.setReadable(false);

CollectionFuture<Boolean> createFuture = mc.asyncLopCreate(KEY, ElementValueType.STRING, attribute);

try {
    createFuture.get(300L, TimeUnit.MILLISECONDS);
} catch (Exception e) {
    createFuture.cancel(true);
    // throw an exception or logging.
}

// Updating the list here. 
// At this sate the collection cannot be READ. It can only be written, editted, or deleted.

// List를 Readable상태로 만든다.
CollectionAttributes attrs = new CollectionAttributes();
attrs.setReadable(true);

CollectionFuture<Boolean> setAttrFuture = mc.asyncSetAttr(KEY, attrs);
try {
    setAttrFuture.get(300L, TimeUnit.MILLISECONDS);
} catch (Exception e) {
    setAttrFuture.cancel(true);
    // throw an exception or logging.
}

// Now I can READ the collection.
```

Below example updates expire time property.

```java
String key = "Sample:Object";

CollectionFuture<Boolean> future = null;

try {
    Attributes attrs = new Attributes(); // (1)
    attrs.setExpireTime(1);
    future = client.asyncSetAttr(key, attrs); // (2)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)
    System.out.println(result);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Created Atrtribute object and set expire time to 1 second. If the result does not show at the specified time or
   if the operation queue fails to handle it due to overload of JVM, `TimeoutException` occurs.
2. Use the `asyncSetAttr` method to update a key's attribute. The key's expire time will be resetted 
   after the specified 1 second in the attribute.
   In conclusion, with `asyncSetAttr` method the expire time of a key will be reset after specified 1 second in the Attribute.
3. Return `true` is attribute successfully applied in the key.

## Retrieve Attribute 

A function to retrieve the attributes of a given key.

```java
CollectionFuture <CollectionAttributes> asyncGetAttr(String key)
```

Sample of retrieval of number of elements that stored in a Collection.

```java
String key = "Sample:List";
CollectionFuture<CollectionAttributes> future = null;

try {
    future = client.asyncGetAttr(key); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null) {
    return;
}

try {
    CollectionAttributes result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)

    if (result == null) { // (3)
        System.out.println("There is no key.");
        return;
    }

    long totalItemCountOfBTree = result.getCount(); // (4)
    System.out.println("Item count=" + totalItemCountOfBTree);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Retrieve the attributed of the key.
2. `timeout` is set to 1 second. If the result does not show at the specified time or
   if the operation queue fails to handle it due to overload of JVM, `TimeoutException` occurs.
3. Return `null` if the key does not exist.
4. Lookup `count` value from the retrieved Attribute object. The retrieved count value is the total number of elements
   that the collection has in the key
 





