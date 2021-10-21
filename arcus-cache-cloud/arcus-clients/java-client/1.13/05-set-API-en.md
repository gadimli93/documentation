# Set Item

Set item has a set of unique values for a single key. It is mainly useful on membership checking.

#### Constraints

- Max number of elements can be stored: 4000 by default (expandable with attribute settings up to 50,000).
- Max size of value per element: 16KB.
- It does not allow duplication of element values.

The basic operations that can be performed on the Set item are as follows.

- [Create Set Item](05-set-API-en.md#create-set-item) (Set item can be deleted by delete function of key-value item)
- [Insert Set Element](05-set-API-en.md#insert-set-element)
- [Delete Set Element](05-set-API-en.md#delete-set-element)
- [Existence of Set Element](05-set-API-en.md#existence-of-set-element)
- [Retrieve Set Element](05-set-API-en.md#retrieve-set-element)

The bulk operation for multiple set elements to perform at once is as follows.

- [Insert Set Elements in Bulk](05-set-API-en.md#insert-set-elements-in-bulk)
- [Existence of Set Elements in Bulk](05-set-API-en.md#existence-of-set-elements-in-bulk)


## Create Set Item

Create a new empty set item.

```java
CollectionFuture<Boolean> asyncSopCreate(String key, ElementValueType valueType, CollectionAttributes attributes)
```

- key: key of a set to create
- valueType: specifies type of a value to store in the Set. There are the following types:
  - ElementValueType.STRING
  - ElementValueType.LONG
  - ElementValueType.INTEGER
  - ElementValueType.BOOLEAN
  - ElementValueType.DATE
  - ElementValueType.BYTE
  - ElementValueType.FLOAT
  - ElementValueType.DOUBLE
  - ElementValueType.BYTEARRAY
  - ElementValueType.OTHERS
- attributes: specifies the attributes of the set item.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | ----------------------------------------- | -------
True         | CollectionResponse.CREATED                | successfully created
False        | CollectionResponse.EXISTS                 | same key already exists

An example of creating a Set item is as follows.

```java
String key = "Sample:EmptySet";
CollectionFuture<Boolean> future = null;
CollectionAttributes attribute = new CollectionAttributes();
try {
	future = client.asyncSopCreate(key, ElementValueType.OTHERS,
			attribute); // (1)
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

1. When creating an empty set, it is necessary to specify in advance the type of element to store in a set. In the above sample, through the `ElementValueType` empty set was created with a type of OTHERS where it can store data of any type except for the types that can be specified.
2. `timeout` is set to 1 second. If the empty set is successfully created, `future` returns `true`. If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Detailed information about the creation result can be retrieved through the `future.getOperationStatus().getResponse()`.

## Insert Set Element

A function to insert an element into a Set.

```java
CollectionFuture<Boolean> asyncSopInsert(String key, Object value, CollectionAttributes attributesForCreate)
```

- key: key of a target set to insert.
- value: value of an element to insert.
- attributesForCreate: specifies what to do when the target set does not exist.
   - null: does not insert an element.
   - attributes: create an empty set item with the given attributes and then insert the element 

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | ----------------------------------------- | -------
True         | CollectionResponse.STORED                 | Set collection already exists and only element(s) inserted
True         | CollectionResponse.CREATED_STORED         | Set collection created and then element inserted
False        | CollectionResponse.NOT_FOUND              | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH          | The given item is not a Set
False        | CollectionResponse.OVERFLOWED             | State of overflow
False        | CollectionResponse.ELEMENT_EXISTS         | Element with the same value already exists in Set

An example of inserting an element into a Set is as follows.

```java
String key = "Sample:Set";
String value = "This is a value.";
CollectionAttributes attributesForCreate = new CollectionAttributes();
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncSopInsert(key, value, attributesForCreate); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result); // (3)
    System.out.println(future.getOperationStatus().getResponse()); // (4)
} catch (TimeoutException e) {
    future.cancel(true);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```
1. If a value of `attributesForCreate` is not `null` and a key does not exist create a 
   new set with a `attributesForCreate` property and then insert an element.
   If a value of `attributesForCreate` is `null` and a key does not exist, an element will not be inserted.
   - using `CollectionAttributes` without special settings, default expiration time means it does not expire at 0.
   - note that even if the value is saved while the key already exists, the expiration time set to the key does not change. In other words, when value is added, expiration time will not be changed or extended.
2. `timeout` is set to 1 second. If the insertion is successful, `future` returns `true`. If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Set does not allow duplicate values. Depending on the presence or absence of duplicate values, the return value varies as follows.
   - Return `false` if the dublicate value already exists in Set.
   - Return `true` if it does not exist.
5. Detailed information on the result of insertion can be retrieved through the `future.getOperationStatus().getResponse()`.

## Delete Set Element

A function that deletes a single element of a given value from  a Set.

```java
CollectionFuture<Boolean> asyncSopDelete(String key, Object value, boolean dropIfEmpty)
```
- key : key of a target set to delete.
- value: value of element to delete
- dropIfEmpty: specifies whether to delete a Set itself after deleting element(s) results in an empty set.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | ----------------------------------------- | -------
True         | CollectionResponse.DELETED                | Delete only an element from a Set
True         | CollectionResponse.DELETED_DROPPED        | Delete an element from a Set, if the Set becomes an empty set, then delete that Set too.
False        | CollectionResponse.NOT_FOUND              | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH          | The given key is not a Set
False        | CollectionResponse.NOT_FOUND_ELEMENT      | Key exist but no element matches the condition

Sample of deleting an element from a Set.

```java
String key = "Sample:Set";
String value = "This is a value.";
boolean dropIfEmpty = false;
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncSopDelete(key, value, dropIfEmpty); // (1)
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

1. If a value of `dropIfEmpty` is `true` after an element is deleted, if the Set becomes empty, delete that key too.
2. `timeout` is set to 1 second. If the delete is successful, `future` returns `true`. If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Returns `true` if the element is deleted successfully. Check `future.getOperationStatus().getResponse()` for detailed result of return value of delete.

## Existence of Set Element

Checks if an element exists for a given value in Set.

```java
CollectionFuture<Boolean> asyncSopExist(String key, Object value)
```
- key: key of the target set to lookup
- value: value to check existence

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
True         | CollectionResponse.EXIST               | Element exists
True         | CollectionResponse.NOT_EXIST           | Element does not exist
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given key is not a Set
False        | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

Sample of checking if an element exists in Set.

```java
String key = "Sample:Set";
String value = "This is a value.";
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncSopExist(key, value); // (1)
} catch (IllegalStateException e) {
     // handle exception
}

if (future == null)
    return;

try {
    Boolean result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result); // (2)

    CollectionResponse response = future.getOperationStatus().getResponse(); // (3)
    System.out.println(response);

    if (response.equals(CollectionResponse.NOT_FOUND)) {
        System.out.println("Key is missing. (There is no set stored in Key.");
    } else if (response.equals(CollectionResponse.NOT_EXIST)) {
        System.out.println("Key exists, but none of the values stored in the Set meet the condition.");
    }
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Checks existence of the value in the set stored in in key.
2. `timeout` is set to 1 second. If the delete is successful, `future` returns `true`. If the result does not show at the specified time
   or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Detailed information on the result of retrieval can be checked through the `future.getOperationStatus().getResponse()`.

## Retrieve Set Element

A function that retrieves an element from Set. This function retrieves any `count` element.

```java
CollectionFuture<Set<Object>> asyncSopGet(String key, int count, boolean withDelete, boolean dropIfEmpty)
```

- key: key of a target set to retrieve
- count: number of elements to retrieve
- withDelete: specifies whether to delete an element with an element retrieval
- dropIfEmpty: specifies whether to delete a Set itself after deleting element(s) that results in an empty set
 
The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
not null     | CollectionResponse.END                 | Retrieved element only
not null     | CollectionResponse.DELETED             | Element retrieved and deleted
not null     | CollectionResponse.DELETED_DROPPED     | Element retrieved and deleted, Set also dropped(deleted)
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found, Set is empty.
null         | CollectionResponse.TYPE_MISMATCH       | The given key is not a Set.
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

Sample of retrieving an element from Set.

```java
String key = "Sample:Set";
int count = 10;
boolean withDelete = false;
boolean dropIfEmpty = false;

CollectionFuture<Set<Object>> future = null;

try {
    future = client.asyncSopGet(key, count, withDelete, dropIfEmpty); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Set<Object> result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
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

1. Retrieves `count` number of elements from Set collection.
   - if the value of `withDelete` is `true`, delete the element from a Set Collection with its retrieval.
   - if the value of `dropIfEmpty` is `true`, after the element is deleted, if the Set collection becomes empty, delete the Set too.
2. `timeout` is set to 1 second. If the delete is successful, `future` returns `true`. If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Detailed information on the result of retrieval can be checked through the `future.getOperationStatus().getResponse()`.

## Insert Set Elements in Bulk

It provides two types of bulk insert functions for inserting multiple elements into Set.

1. First, a function that inserts multiple elements into a Set that a single key refers to.

```java
CollectionFuture <Map<Integer, CollectionOperationStatus>>
asyncSopPipedInsertBulk(String key, List<Object> valueList, CollectionAttributes attributesForCreate)
```

- key: key of a target set to insert.
- valueList: value list of elements to insert.
- attributesForCreate: specifies what to do when the target set does not exist
  - null: does not insert an element
  - attributes: create an empty set item with the given attributes and then insert the element.

2. Second, a function that inserts one element into each Sets referred by several keys

```java
Future<Map<String, CollectionOperationStatus>>
asyncSopInsertBulk(List<String> keyList, Object value, CollectionAttributes attributesForCreate)
```

- key: key list of a target set to insert.
- value: value of elements to insert.
- attributesForCreate: specifies what to do when the target set does not exist
  - null: does not insert an element
  - attributes: create an empty set item with the given attributes and then insert the element.

Below is the sample code for bulk insert of multiple elements into a Set and checking the insert results for each item.

```java
String key = "Sample:SetBulk";
List<Object> bulkData = getBulkData();
CollectionAttributes attributesForCreate = new CollectionAttributes();
if (bulkData.size() > client.getMaxPipedItemCount()) { // (1)
    System.out.println("The number of inserted items cannot exceed client.getMaxPipedItemCount.");
    return;
}

CollectionFuture<Map<Integer, CollectionOperationStatus>> future = null;

try {
    future = client.asyncSopPipedInsertBulk(key, bulkData, attributesForCreate); // (2)
} catch (Exception e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Integer, CollectionOperationStatus> result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)
    if (!result.isEmpty()) { // (4)
        System.out.println("Some items failed to be inserted");
        for (Entry<Integer, CollectionOperationStatus> entry : result.entrySet()) { // (5)
            System.out.print("Failed item=" + bulkData.get(entry.getKey()));
            System.out.println(", cause of failure=" + entry.getValue().getResponse()); // (6)
        }
    } else {
        System.out.println("All inserts were completed successfully.");
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
2. Insert `bulkData` at once into the Set stored in the key and then return the `future` object that contains the result.
   From the `future` object checked whether each item successfully inserted or failed to insert.
   Before bulk insertion, `attributesForCreate` value is specified, to create a key and insert the element, in case there is no key initially.
3. `timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue
   fails to handle it due to overload of JVM `TimeoutException` occurs.
4. Return empty map if all items successfully inserted.
   - returned `Map's Key=` value of index when iterates inserted value(`bulkData`)
   - returned `Map's Value=` cause of failed insert
5. To check the cause of failed items, retrieve the `Map` result depending on the order of the iteration of value(`bulkData`) used to insert.
6. Because, Key of the Map obtained from Future is an index of inserted value(`bulkData`) 
   the cause of the failure can be checked in the same manner as mentioned before.

## Existence of Set Elements in Bulk

A Function to check multiple elements' existence at once in Set.

```java
CollectionFuture<Map<Object, Boolean>> asyncSopPipedExistBulk(String key, List<Object> values)
```

- key: key of a target Set to retrieve
- values:  value list to check existence

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
not null     | CollectionResponse.EXIST               | Element exists
not null     | CollectionResponse.NOT_EXIST           | Element does not exist
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.TYPE_MISMATCH       | The given key is not a Set
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

The following information can be obtained through the result of `(Map<Object, Boolean>)` object.

Method of result object | Data Type | Description
------------------------|-----------|----------
getKey()                | Object    | value
getValue()              | Boolean   | existing value presented as

The sample code below determines the existence of values from `VALUE1` to `VALUE4` in the Set.
The key of resulting value in the form of `Map<Object, Boolean>` is the value for determining whether the value(s) exist(s) or not.
If the value exists, the value of the map returns `true`.

```java
String key = "Sample:Set";
List<Object> valueList = new ArrayList<Object>();
valueList.add("value1");
valueList.add("value2");
valueList.add("value3");
valueList.add("value4");
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncSopPipedExistBulk(key, valueList); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Object, Boolean> result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    for(Entry<Object, Boolean> entry : result.entrySet()) {
        System.out.println("Object=" + entry.getKey() + ", exists=" + entry.getValue());
    }

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

1. Check whether each value included in the value list exists in the Set. 
2. The result is returned in `Map<Object, Boolean>` form.
  - key of the Map entry determines whether value exists or not.
  - value of Map entry is a `boolean` indicating whether it exists or not. `true` if value exists, `false` if it does not.
3.  Detailed information on the result of bulk exists can be checked through the `future.getOperationStatus().getResponse()`.

