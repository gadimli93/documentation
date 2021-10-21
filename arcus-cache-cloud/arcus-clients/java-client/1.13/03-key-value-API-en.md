# Key-Value Item

A key-value item is an item that stores only one value for one key.

#### Constraints:
- max size of key is 4000 character;
- max size of cache item is 1MB.

The operations that can be performed for the key-value item are as follows:
- [Key-Value Item Storing](03-key-value-API-en.md#key-value-item-storing)
- [Key-Value Item Retrieval](03-key-value-API-en.md#key-value-item-retrieval)
- [Key-Value Item INCR/DECR](03-key-value-API-en.md#key-value-item-increment-decrement)
- [Key-Value Item Delete](03-key-value-API-en.md#key-value-item-delete)

## Key-Value Item Storing

Key-Value Item provides set, add, replace APIs to store an item.

```java
OperationFuture<Boolean> set(String key, int exp, Object obj)
OperationFuture<Boolean> add(String key, int exp, Object obj)
OperationFuture<Boolean> replace(String key, int exp, Object obj)
```

- Stores a key-value item of <key, obj>.
- Depending on the existence of a key in Cache, each API behaves as follows:
  - `set` saves <key, obj> item unconditionally. If the key already exists, it replaces and saves it.
  - `add` stores <key, obj> item only if the key does not exist.
  - `replace` the <key, obj> item and save it only if the key already exists.
- a stored key-value item is deleted after `exp` seconds.

The result is obtained through the `future` object.

future.getStatus().getStatusCode()          | Description
--------------------------------------------| ---------
StatusCode.SUCCESS                          | Successfully stored
StatusCode.ERR_NOT_STORED                   | Failed to store (add: key already exists, replace: item does not exist in the given key)

It provides a 'prepend', 'append' as an API to add a value given to the key-value item.

```java
OperationFuture<Boolean> prepend(long cas, String key, Object val)
OperationFuture<Boolean> append(long cas, String key, Object val)
```

- In key-value items, the location of add value depends on the API.
  - prepend is added at the beginning of the value section of the item.
  - append is added at the end of the value section of the item.
- The first factor, cas is not currently available, so you can give it any value. 
  Initially, it was needed to perform CAS(compare-and-set) operation.

The result is obtained through the `future` object.

future.getStatus().getStatusCode()          | Description
--------------------------------------------| ---------
StatusCode.SUCCESS                          | Successfully stored
StatusCode.ERR_NOT_STORED                   | Failed to store (item does not exist in the given key )

It provides a bulk API that sets multiple key-value items in a single API call.

```java
Future<Map<String, OperationStatus>> asyncStoreBulk(StoreType type, List<String> key, int exp, Object obj)
Future<Map<String, OperationStatus>> asyncStoreBulk(StoreType type, Map<String, Object> map, int exp)

```
- Stores multiple key-value items at once.
  - Succeeding API performs a save operation with the same `obj` for all keys in the key list at once.
  - Preceding API performs a save operation for all `<key, obj>` in the map at once.
- All stored key-value items are deleted after `exp` seconds.
- `StoreType` specifies the storage type of the operation, shown as below.
  - StoreType.set
  - StoreType.add
  - StoreType.replace

`expiration` - enters the time (number of seconds) from the current time until the key will expire.
If the time exceeds 30 days, enter the `Unix time` to be expire.
In addition, to prevent expiration the values below can be specified.

- 0: sets the key not to expire. However, if the ARCUS Cache server runs out of memory, it can be deleted at any time by the LRU algorithm.
- -1: make the key a sticky item. Sticky item never expires and it's not deleted by the LRU algorithm.

The key that failed to save and the cause of the failure can be retrieved in a Map form with `future` object.

future.get(key).getStatusCode()   | Description
----------------------------------| ---------
StatusCode.ERR_NOT_FOUND          | Key miss (no item found for given key)
StatusCode.ERR_EXISTS             | Same key already exists

## Key-Value Item Retrieval

Provides an API that retrieves a value stored in a cache item with a single key.

```java
GetFuture<Object> asyncGet(String key)
```

- Returns the stored value of a given key.

The result is obtained through the `future` object.

future.get(key).getStatusCode() | Description
--------------------------------| ---------
StatusCode.SUCCESS              | Successfully retrieved (even the item does not exist for given key)

Provides a bulk API that retrieves values of multiple keys at once.

```java
BulkFuture<Map<String, Object>> asyncGetBulk(Collection<String> keys)
BulkFuture<Map<String, Object>> asyncGetBulk(String... keys)
```

- Returns the stored values of multiple keys in `Map<String, Object>` form.
- Keys can be either a collection of String type or cataloged a list of keys of String type.

Provides an API that retrieves `CASValue` stored in a cache item with a single key.

```java
GetFuture<CASValue<Object>> asyncGets(String key)
```

- Returns the stored `CASValue(cas, value)` of a given key.

The result is obtained through the `future` object.

future.get(key).getStatusCode() | Description
--------------------------------| ---------
StatusCode.SUCCESS              | Successfully retrieved (even the item does not exist for given key)

Provides a bulk API that retrieves `CASValue` of multiple keys at once.

```java
BulkFuture<Map<String, CASValue<Object>>> asyncGetsBulk(Collection<String> keys)
BulkFuture<Map<String, CASValue<Object>>> asyncGetsBulk(String... keys)
```

- Returns the stored `CASValue` of multiple keys in `Map<String, CASValue<Object>` form. 
- Keys can be either a collection of String type or cataloged a list of keys of String type.

## Key-Value Item Increment-Decrement

An operation that increases or decreases a value of a key-value item.
[WARNING] In order to use an increase or decrease operation, the value must be a numeric value of a String type.

```java
OperationFuture<Long> asyncIncr(String key, int by)
OperationFuture<Long> asyncDecr(String key, int by)
```

- Increase/Decrease stored value of integer type data in a key as much as`by`.
  If the key does not exist in the cache, neither increase nor decrease operations will be performed.
- Returns increased or decreased value.
  
```java
OperationFuture<Long> asyncIncr(String key, int by, long def, int exp)
OperationFuture<Long> asyncDecr(String key, int by, long def, int exp)
```

- Increase/Decrease stored value of integer type data in a key as much as`by`.
  If the key does not exist in the cache, its add `<key, def>` item and delete it after `exp` seconds.
- Returns increased or decreased value.  
  
The result is obtained through the `future` object.
  
future.getStatus().getStatusCode()          | Description
--------------------------------------------| ---------
StatusCode.SUCCESS                          | Successfully increase/decrease
StatusCode.ERR_NOT_FOUND                    | Failed to increase/decrease (Key miss, item does not exist in the given key ) 

## Key-Value Item Delete

Provides an API that deletes an item for one key and a bulk API that deletes items for multiple keys at once.

```java
OperationFuture<Boolean> delete(String key)
```
- Deletes an item from a cache for a given key.
  
The result is obtained through the `future` object.

future.getStatus().getStatusCode()          | Description
--------------------------------------------| ---------
StatusCode.SUCCESS                          | Successfully deleted
StatusCode.ERR_NOT_FOUND                    | Failed to delete (Key miss, no item found for the given key)

```java
Future<Map<String, OperationStatus>> asyncDeleteBulk(List<String> key)
Future<Map<String, OperationStatus>> asyncDeleteBulk(String... key)
```
- Deletes multiple key-value items at once.
- Keys can be either a String type `List` or cataloged a list of keys of String type.

The key that failed to delete and the cause of the failure can be retrieved in a Map form with `future` object.

future.get().get(key).getStatusCode()  | Description
---------------------------------------| ---------
StatusCode.ERR_NOT_FOUND               | Failed to delete (Key miss, no item found for the given key)



