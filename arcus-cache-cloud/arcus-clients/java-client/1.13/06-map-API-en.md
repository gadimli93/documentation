# Map Item

Map item has the `key & pair` pair as a set of data based on hash structure for a single key.
It is recommended to use Map as Java's Map data type for storage purposes.

#### Constraints

- Maximum number of elements can be stored: 4000 by default (expandable with attribute settings up to 50,000).
- Maximum size of value per element: 16KB.
- Maximum length of `mkey` is 250 bytes and duplication of `mkey` in a map not allowed.

The basic operations that can be performed on the Map item are as follows.

- [Create Map Item](06-map-API-en.md#create-map-item) 
- [Insert Map Element](06-map-API-en.md#insert-map-element)
- [Update Map Element](06-map-API-en.md#update-map-element)
- [Delete Map Element](06-map-API-en.md#delete-map-element)
- [Retrieve Map Element](06-map-API-en.md#retrieve-map-element)

The bulk operation for multiple map elements to perform at once is as follows.

- [Insert Map Elements in Bulk](06-map-API-en.md#insert-map-elements-in-bulk)
- [Update Map Elements in Bulk](06-map-API-en.md#update-map-elements-in-bulk)

## Create Map Item

Create a new empty map item.
 
```java
CollectionFuture<Boolean> asyncMopCreate(String key, ElementValueType valueType, CollectionAttributes attributes)
```
 
- key: key of map item to create 
- valueType: specifies type of a value to store in the Map. There are the following types.
  - ElementValueType.STRING
  - ElementValueType.LONG
  - ElementValueType.INTEGER
  - ElementValueType.BOOLEAN
  - ElementValueType.DATE
  - ElementValueType.BYTE
  - ElementValueType.FLOAT
  - ElementValueType.DOUBLE
  - ElementValueType.BYTEARRAY
  - ElementValueType.OTHERS : for example, user defined class
- attributes: specifies the attributes of a map item.
 
The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
True         | CollectionResponse.CREATED             | successfully created
False        | CollectionResponse.EXISTS              | same key already exists

An example of creating a Map item is as follows.
 
```java
String key = "Sample:EmptyMap";
CollectionFuture<Boolean> future = null;
CollectionAttributes attribute = new CollectionAttributes(); // (1)
attribute.setExpireTime(60); // (1)

try {
    future = client.asyncMopCreate(key, ElementValueType.STRING, attribute); // (2)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)
    System.out.println(result);
    System.out.println(future.getOperationStatus().getResponse()); // (4)
} catch (TimeoutException e) {
    future.cancel(true);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Map's expiration time is set to 60 seconds. Detailed usage of `CollectionAttributes` is covered in detail 
   in the chapter on [Manual: How to use Attributes](08-attribute-API-en.md)
2. When creating an empty map, it is necessary to specify in advance the type of element to store in a map.
   The reason is the mechanism by which a Java Client encodes/decodes a value.
   The sample above creates an empty sample to store a String type. 
   If you store a value that does not match the element type specified when creating an empty map,
   you will not be able to lookup/retrieve the value, even if it was successfully stored.
3. `timeout` is set to 1 second. If the empty map is successfully created, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle 
   it due to overload of JVM `TimeoutException` occurs.
4. Detailed information about the creation result can be retrieved through the `future.getOperationStatus().getResponse()`.

## Insert Map Element

A function to insert an element into a Map.

```java
 CollectionFuture<Boolean> asyncMopInsert(String key, String mkey, Object value, CollectionAttributes attributesForCreate)
```

- key: key of a target map to insert.
- mkey: mkey of element to insert
- value: value of an element to insert.
- attributesForCreate: specifies what to do when the target map does not exist.
  - null: does not insert an element.
  - attributes: create an empty map item with the given attributes and then insert the element

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse()| Description
------------ | -------------------------------------- | ---------
True         | CollectionResponse.STORED              | Map collection already exists and only element(s) inserted
True         | CollectionResponse.CREATED_STORED      | Map collection created and then element inserted
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a Map
False        | CollectionResponse.ELEMENT_EXISTS      | Element with given mkey already exists
False        | CollectionResponse.OVERFLOWED          | State of overflow (Maximum number of elements already present)
 
A sample code of inserting an element into a Map is as follows.
 
```java
String key = "Prefix:MapKey";
String mkey = "mkey";
String value = "This is a value.";

CollectionAttributes attributesForCreate = new CollectionAttributes();
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncMopInsert(key, mkey, value, attributesForCreate); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result);
    System.out.println(future.getOperationStatus().getResponse()); // (3)
} catch (TimeoutException e) {
    future.cancel(true);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. If a value of `attributesForCreate` is not `null` and a map does not exist, create a new map with  `attributesForCreate` property and then 
   insert an element. If a value of `attributesForCreate` is `null` and a key does not exist, an element will not be inserted.
    - above sample uses default `CollectionAttributes` without special settings,
      default expiration time means it does not expire at 0.
2. `timeout` is set to 1 second. If the insertion is successful, `future` returns `true`.
  If the result does not show at the specified time or if the operation queue fails to handle
  it due to overload of JVM `TimeoutException` occurs.
3. Detailed information on the result of insertion can be retrieved through the `future.getOperationStatus().getResponse()`.

## Update Map Element

A function to update an element in a Map. It changes the value of the element.

```java
 CollectionFuture<Boolean> asyncMopUpdate(String key, String mkey, Object value)
```
 
- key: a new key of a target map to update.
- mkey: key of target element to update.
- value: a new value of an element to update.
 
The result is obtained through the `future` object.
  
future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | ---------
True         | CollectionResponse.UPDATED             | Element is updated
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.NOT_FOUND_ELEMENT   | Element with given mkey does not exist
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a Map

Update the value of a particular element.
  
```java
 CollectionFuture<Boolean> future = mc.asyncMopUpdate(key, mkey, value);
```
 
Detailed information about the update of element result can be retrieved through the `future.getOperationStatus().getResponse()`.
 
## Delete Map Element

There are two types of functions that delete elements from a Map.

First, delete all elements of a given Map.

```java
CollectionFuture<Boolean>
asyncMopDelete(String key, boolean dropIfEmpty)
```
 
Second, delete a given element of `mkey` from a Map.
 
```java
CollectionFuture<Boolean>
asyncMopDelete(String key, String mkey, boolean dropIfEmpty)
```

- key : key of a target map to delete.
- dropIfEmpty: specifies whether to delete a Map itself after deleting element(s) results in an empty map.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | ---------
True         | CollectionResponse.DELETED             | Delete only an element from a Map
True         | CollectionResponse.DELETED_DROPPED     | Delete an element from a Map, if the Map becomes an empty map, then delete that Map too.
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.NOT_FOUND_ELEMENT   | Element with given mkey does not exist
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a Map 
 
A sample code of deleting `mkey` element from a Map where `mkey` is a `mkey1` is as follows.
 
```java
String key = "Prefix:MapKey";
String mkey1 = "mkey1";
boolean dropIfEmpty = true;
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncMopDelete(key, mkey1, dropIfEmpty); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result);
    CollectionResponse response = future.getOperationStatus().getResponse(); // (3)
    System.out.println(response);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Deletes an element of `mkey1` from Map. 
    If a value of `dropIfEmpty` is `true` after an element is deleted, and a map becomes empty, delete that map too.
3. `delete timeout` is set to 1 second. If the delete is successful, `future` returns `true`.
    If the result does not show at the specified time or if the operation queue fails to handle 
    it due to overload of JVM `TimeoutException` occurs.
3. Check `future.getOperationStatus().getResponse()` for detailed information about a result of delete.

## Retrieve Map Element

There are three types of functions that retrieve element from Map.

First, retrieve all elements from a Map.

```java
CollectionFuture<Map<String, Object>>
asyncMopGet(String key, boolean withDelete, boolean dropIfEmpty)
```
 
 Second, retrieve a given `mkey` element from a Map.
 
```java
CollectionFuture<Map<String, Object>>
asyncMopGet(String key, String mkey, boolean withDelete, boolean dropIfEmpty)
```
 
 Third, retrieve an element of a given `mkeyList` from a Map.
 
```java
CollectionFuture<Map<String, Object>>
asyncMopGet(String key, List<String> mkeyList, boolean withDelete, boolean dropIfEmpty)
```
 
- key: key of a map item
- mkey: mkey of element to retrieve
- mkeyList: mkeyLists of element to retrieve
- withDelete: specifies whether to delete an element with an element retrieval
- dropIfEmpty: specifies whether to delete a Map itself after deleting element(s) that results in an empty map

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
not null     | CollectionResponse.END                 | Retrieved element only
not null     | CollectionResponse.DELETED             | Element retrieved and deleted
not null     | CollectionResponse.DELETED_DROPPED     | Element retrieved and deleted, Map also dropped(deleted)
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found, key does not exist in mkey List
null         | CollectionResponse.TYPE_MISMATCH       | The given item is not a Map 
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

Sample of retrieving a Map element.
 
```java
String key = "Prefix:MapKey";
List<String> mkeyList = new ArrayList<String>();
mkeyList.add("mkey1");
mkeyList.add("mkey2");
boolean withDelete = false;
boolean dropIfEmpty = false;
CollectionFuture<Map<String, Object>> future = null;

try {
    future = client.asyncMopGet(key, mkeyList, withDelete, dropIfEmpty); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<String, Object> result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result);

    CollectionResponse response = future.getOperationStatus().getResponse(); // (3)
    if (response.equals(CollectionResponse.NOT_FOUND)) {
        System.out.println("Key is missing. (There is no Map stored in Key.");
    } else if (response.equals(CollectionResponse.NOT_FOUND_ELEMENT)) {
        System.out.println("Key exists, but none of the values stored in the Map meet the condition.");
    }

} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. To retrieve both mkey1 and mkey2 from a Map at once, added them to a List and retrieved a mkeyList.
2. `timeout` is set to 1 second. 
   If the result does not show at the specified time or if the operation queue fails to handle 
   it due to overload of JVM `TimeoutException` occurs.
   The implementation of the returned Map interface is `HashMap`, the result is one of the following:
   - key is missing, return null
   - key exists but no element found matching the condition, return empty map
   - key exists and elements meet the condition, return non-empty map 
3. Detailed information on the result of retrieval can be checked through the `future.getOperationStatus().getResponse()`.

## Insert Map Elements in Bulk

There are two types of bulk insert functions for inserting multiple elements into Map.

First, a function that inserts multiple elements into a Map referred by a single key.

```java
CollectionFuture<Map<Integer, CollectionOperationStatus>>
asyncMopPipedInsertBulk(String key, Map<String, Object> elements, CollectionAttributes attributesForCreate)
 ```
 
- key: key of a target map to insert.
- element: elements to insert
  - Map<String, Object> type.
- attributesForCreate: specifies what to do when the target map does not exist.
   - null: does not insert an element.
   - attributes: create an empty map item with the given attributes and then insert the element.
  
Second, a function that inserts one element into each Map referred by multiple keys.
 
```java
Future<Map<String, CollectionOperationStatus>>
asyncMopInsertBulk(List<String> keyList, String mkey, Object value, CollectionAttributes attributesForCreate)
```
 
- keyList: key list of a target map to insert.
- mkey: mkey of an element to insert
- value: value of element to insert
- attributesForCreate: specifies what to do when the target map does not exist.
   - null: does not insert an element.
   - attributes: create an empty map item with the given attributes and then insert the element.
 
Below is the sample code for bulk insert of multiple elements into a Map and checking the insert results for each item.
 
```java
String key = "Sample:MapBulk";
Map<String, Object> elements = new HashMap<String, Object>();

elements.put("mkey1", "value1");
elements.put("mkey2", "value2");
elements.put("mkey3", "value3");

boolean createKeyIfNotExists = true;

if (elements.size() > mc.getMaxPipedItemCount()) { // (1)
    System.out.println("insert 할 아이템 개수는 mc.getMaxPipedItemCount개를 초과할 수 없다.");
    return;
}

CollectionFuture<Map<Integer, CollectionOperationStatus>> future = null;

try {
    future = mc.asyncMopPipedInsertBulk(key, elements, new CollectionAttributes()); // (2)

} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Integer, CollectionOperationStatus> result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)

    if (!result.isEmpty()) { // (4)
        System.out.println("일부 item이 insert 실패 하였음.");
        
        for (Map.Entry<Integer, CollectionOperationStatus> entry : result.entrySet()) {
            System.out.print("실패한 아이템=" + elements.get(entry.getKey()));
            System.out.println(", 실패원인=" + entry.getValue().getResponse());
        }
    } else {
        System.out.println("모두 insert 성공함.");
    }
} catch (TimeoutException e) {
    future.cancel(true);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. The number of inserted items at once cannot exceed `client.getMaxPipedItemCount()` (default is 500). 
   If the number exceeds, `IllegalArguementException` occurs.
2. Insert `bulkData` at once into the Map stored in the key and then return the `future` object that contains the result. From the `future`
   object checked whether each item was successfully inserted or failed to insert. 
   Before bulk insertion, `attributesForCreate` value is specified, to create a key and insert the element,
   in case of there is no key initially.
3. `delete timeout` is set to 1 second. If the result does not show at the specified time or 
   if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
4. Return empty map if all items are successfully inserted.
   - returned `Map's Key=` value of index when iterates inserted value(`bulkData`)
   - returned `Map's Value=` cause of failed insert
5. To check the cause of some failed items, retrieve the `Map` result depending on the order of the iteration 
   of value(`bulkData`) used to insert.
6. Because the Key of the Map obtained from `Future` is a mapKey of inserted value(`bulkData`) 
   the cause of the failure can be checked in the same manner as mentioned before.

## Update Map Elements in Bulk

Bulk update of all values of given elements in the Map.

```java
CollectionFuture<Map<Integer, CollectionOperationStatus>>
asyncMopPipedUpdateBulk(String key, Map<String, Object>> elements)
```
- key: key of a target map to update
- elements: contains `mkey`, `new value` about a target map to update


