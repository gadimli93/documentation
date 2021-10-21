# List Item

List Collection maintains multiple values in a double linked list structure for a single key.

#### Constraints

- Max number of elements can be stored: 4000 by default (expandable with attribute settings up to 50,000).
- Max size of value per element: 16KB.
- It is recommended to insert/delete elements from the top and the end of a List.
  Elements can be inserted/deleted at any index location, but currently there is no data structure 
  to quickly find any index location, therefore it can be costly.

The basic operations that can be performed on the List item are as follows.

- [Create List Item](04-list-API-en.md#create-list-item)
- [Insert List Element](04-list-API-en.md#insert-list-element)
- [Delete List Element](04-list-API-en.md#delete-list-element)
- [Retrieve List Element](04-list-API-en.md#retrieve-list-element)

The bulk operation for multiple list elements to perform at once is as follows.

- [Insert List Elements in Bulk](04-list-API-en.md#insert-list-elements-in-bulk)

## Create List Item

Create a new empty list item.

```java
CollectionFuture<Boolean> asyncLopCreate(String key, ElementValueType valueType, CollectionAttributes attributes)
```
- key:  key of a list to create.
- valueType: specifies type of a value to store in the list. There are the following types:
  - ElementValueType.STRING - String
  - ElementValueType.LONG - Long
  - ElementValueType.INTEGER - Integer
  - ElementValueType.BOOLEAN - Boolean
  - ElementValueType.DATE - Date
  - ElementValueType.BYTE - Byte
  - ElementValueType.FLOAT - Float
  - ElementValueType.DOUBLE - Double
  - ElementValueType.BYTEARRAY - Byte array
  - ElementValueType.OTHERS - other types except the above listed ones.
- attribute: specifies the attributes of the list item.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
True         | CollectionResponse.CREATED             | successfully created
False        | CollectionResponse.EXISTS              | same key already exists

An example of creating a list item is as follows.

```java
String key = "Sample:EmptyList";

CollectionFuture<Boolean> future = null;
CollectionAttributes attribute = new CollectionAttributes();

try {
    future = client.asyncLopCreate(key, ElementValueType.OTHERS, attribute); // (1)
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
1. When creating an empty list, it is necessary to specify in advance the type of element to store in a list.
   In the above sample, through the `ElementValueType` empty list created with a type of `OTHERS` where it can store
   data any types except for the types that can be specified. 
2. `timeout` is set to 1 second. If the empty list is successfully created, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM
   `TimeoutException` occurs.
3. Detailed information about the creation result can be retrieved through the `future.getOperationStatus().getResponse()`.

## Insert List Element

A function that inserts a single element into a List.

```java
CollectionFuture<Boolean> asyncLopInsert(String key, int index, Object value, CollectionAttributes attributesForCreate)
```

Inserts a new element into a List.

- key: key of a target list to insert.
- index: specifies insert position as 0-based index.
  - 0, 1, 2, ... : indicates each element's position starting from the top of the list.
  - -1, -2, -3, ... : indicates each element's position starting from the end of the list.
- value: value of an element to insert.
- attributesForCreate: specifies what to do when the target list does not exist.
  - null: does not insert an element.
  - attributes: create an empty list item with the given attributes and then insert the element.    

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
True         | CollectionResponse.STORED              | List collection already exists and only elements are inserted
True         | CollectionResponse.CREATED_STORED      | List collection created and then element inserted
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a List
False        | CollectionResponse.OVERFLOWED          | State of overflow 
False        | CollectionResponse.OUT_OF_RANGE        | Location of an index position is beyond the element index range of a List

An example of inserting an element into a list is as follows.

```java
String key = "Sample:List";
int index = -1;
String value = "This is a value.";
CollectionAttributes attributesForCreate = new CollectionAttributes();
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncLopInsert(key, index, value, attributesForCreate); // (1)
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
 
1. If a `attributesForCreate` value is not `null`, in the case of when a key does not exist create a new list
   with a `attributesForCreate` property and then insert an element. If a value of `attributesForCreate` is null and
   a key does not exist, an element will not be inserted.
   - above sample, uses Collection Attributes without special settings, and default expiration time means it does not expire at 0.
   - note that even if the value is saved while the key already exists, the expiration time set to the key does not change.
     In other words, when value is added, expiration time will not be changed or extended.
2. `timeout` is set to 1 second. If the insertion is successful, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM
   `TimeoutException` occurs.
3.  Detailed information on the result of insertion can be retrieved through the `future.getOperationStatus().getResponse()`.

## Delete List Element

A function that deletes a single element from a position of index or multiple elements within the index range of a List.

```java
CollectionFuture<Boolean> asyncLopDelete(String key, int index, Boolean dropIfEmpty)
CollectionFuture<Boolean> asyncLopDelete(String key, int from, int to, boolean dropIfEmpty)
```

- key : key of a target list to delete.
- index: index range (from, to): specifies delete position as 0-based index
  - 0, 1, 2, ... : indicates each element's position starting from the top of the list
  - -1, -2, -3, ... : indicates each element's position starting from the end of the list.
- dropIfEmpty: specifies whether to delete a list itself after deleting element(s) results in an empty list.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
True         | CollectionResponse.DELETED             | Delete only an element from a List
True         | CollectionResponse.DELETED_DROPPED     | Delete an element from a List, if the List is an empty list delete that List too.
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given key is not a List
False        | CollectionResponse.NOT_FOUND_ELEMENT   | The Lists exist but no element matches the condition

Sample of deleting elements within the range of index from 0 to 10.

```java
String key = "Sample:List";
int from = 0;
int to = 10;
boolean dropIfEmpty = false;
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncLopDelete(key, from, to, dropIfEmpty); // (1)
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

1. In the List, delete elements within `from` and `to` index range.
   If a value of `dropIfEmpty` is true after an element is deleted, if the List becomes empty, delete list too. 
2. `timeout` is set to 1 second. If the delete is successful, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM
   `TimeoutException` occurs.
3. Returns `true` if the element is deleted successfully. Check `future.getOperationStatus().getResponse()` for detailed result of return value of delete. 

## Retrieve List Element

Retrieved a single element from a given index or index range in the List. 
If the element does not exist, return a `null`.

- key: key of a target list to retrieve.
- index or index range (from, to): specifies retrieve position as 0-based index
 - 0, 1, 2, ... : indicates each element's position starting from the top of the list
 - -1, -2, -3, ... : indicates each element's position starting from the end of the list.
- withDelete: specifies whether to delete an element with an element retrieval.
- dropIfEmpty: specifies whether to delete a list itself after retrieving element(s) that results in an empty list.

The result is obtained through the `future` object.

```java
CollectionFuture<List<Object>> asyncLopGet(String key, int index, boolean withDelete, boolean dropIfEmpty)
CollectionFuture<List<Object>> asyncLopGet(String key, int from, int to, boolean withDelete, boolean dropIfEmpty)
```


future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
not null     | CollectionResponse.END                 | Element retrieved only
not null     | CollectionResponse.DELETED             | Element retrieved and deleted
not null     | CollectionResponse.DELETED_DROPPED     | Element retrieved and deleted, a List also dropped(deleted)
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.TYPE_MISMATCH       | The given key is not a List
null         | CollectionResponse.UNREADABLE          | the given key cannot be read (unreadable item)
null         | CollectionResponse.OUT_OF_RANGE        | Index position is beyond the element index range of a list

An example of retrieving a list element is as follows.

```java
String key = "Sample:List";
int from = 0;
int to = 5;
boolean withDelete = false;
boolean dropIfEmpty = false;
CollectionFuture<List<Object>> future = null;

try {
    future = client.asyncLopGet(key, from, to, withDelete, dropIfEmpty); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    List<Object> result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result);
    CollectionResponse response = future.getOperationStatus().getResponse();  // (3)
    System.out.println(response);

    if (response.equals(CollectionResponse.NOT_FOUND)) {
        System.out.println("Key is missing. (There is no List stored in Key.");
    } else if (response.equals(CollectionResponse.NOT_FOUND_ELEMENT)) {
        System.out.println("Key exists, but none of the values stored in the List meet the condition.");
    }

} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. Retrieve elements within the index range of 0 to 5.
   - if the value of `withDelete` is true, delete the element from a List collection with its retrieval.
   - if the value of `dropIfEmpty` is true, after the element is deleted, if the List collection becomes empty, delete the list too.
2. `timeout` is set to 1 second. If the delete is successful, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM
   `TimeoutException` occurs. Results of `future.get()` return are as follows:
   - if the key is missing, return null.
   - return empty list if the key exists but no element found matching the condition(in index).
   - if the key exists and only some elements meet the condition, return only the matched elements that meet the condition.
3. Detailed information on the result of retrieval can be checked through the `future.getOperationStatus().getResponse()`.

## Insert List Elements in Bulk

It provides two types of bulk insert functions.
1. First, a function that inserts multiple elements into a List that a single key refers to.

```java
CollectionFuture <Map<Integer, CollectionOperationStatus>>
asyncLopPipedInsertBulk(String key, int index, List<Object> valueList, CollectionAttributes attributesForCreate)
```

- key: key of a target list to insert.
- index: specifies insert position as 0-based index.
  - -1: insert to the end of a list.
  - 0: insert to the top of a list.
- valueList: valueList of elements to insert.
- attributesForCreate: specifies what to do when the target list does not exist.
  - null: does not insert an element.
  - attributes: create an empty list item with the given attributes and then insert the element.   

2. Second, a function that inserts one element into each Lists referred by several keys

```java
Future <Map<String, CollectionOperationStatus>>
asyncLopInsertBulk(List<String> keyList, int index, Object value, CollectionAttributes attributesForCreate)
```

Insert one element for all keys specified in the keyList.

- key: key list of target lists to insert.
- index: specifies insert position as 0-based index.
  - -1: insert to the end of a list.
  - 0: insert to the top of a list.
- value: value of element to insert.
- attributesForCreate: specifies what to do when the target list does not exist.
  - null: does not insert an element.
  - attributes: create an empty list item with the given attributes and then insert the element.  

Below is the sample code for bulk insert of multiple elements into a single List and checking the insert results for each item.

```java
String key = "Sample:ListBulk";
List<Object> bulkData = getBulkData();
int index = -1; // insert at the end of the List.
CollectionAttributes attributesForCreate = new CollectionAttributes();
if (bulkData.size() > client.getMaxPipedItemCount()) { // (1)
    System.out.println("The number of inserted items cannot exceed client.getMaxPipedItemCount.");
    return;
}

CollectionFuture<Map<Integer, CollectionOperationStatus>> future = null;

try {
    future = client.asyncLopPipedInsertBulk(key, index, bulkData, attributesForCreate); // (2)
} catch (Exception e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Integer, CollectionOperationStatus> result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)
    if (!result.isEmpty()) { // (4)
        System.out.println("Some items failed to be inserted.");
        for (Entry<Integer, CollectionOperationStatus> entry : result.entrySet()) { // (5)
            System.out.print("Failed item=" + bulkData.get(entry.getKey()));
            System.out.println(", Cause of failure=" + entry.getValue().getResponse()); // (6)
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
2. Insert `bulkData` at once into the List stored in the key and then return the `future` object that contains the result.
   From the `future` object checked whether each item successfully inserted or failed to insert.
   Before bulk insertion, `attributesForCreate` value is specified, to create a key and insert the element, in case there is no key initially.
3. delete `timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue 
   fails to handle it due to overload of JVM `TimeoutException` occurs.
4. Return empty map if all items successfully inserted.
   - returned `Map's Key=` value of index when iterates inserted value(`bulkData`)
   - returned `Map's Value=` cause of failed insert
5. To check the cause of some failed items, retrieve the `Map` result depending on the order of the iteration of value(`bulkData`) used to insert.
6. Because, Key of the Map obtained from `Future` is an index of inserted value(`bulkData`) 
   the cause of the failure can be checked in the same manner as mentioned before.
