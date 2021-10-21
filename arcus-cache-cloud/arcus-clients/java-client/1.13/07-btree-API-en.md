# B+tree Item

B+tree item has a set of data sorted by b+tree key(kbey) based on b+tree structure for a single key.

#### Constraints

- Max number of elements can be stored: 4000 by default (expandable with attribute settings up to 50,000).
- Max size of value per element: 16KB.
- All elements must have same bkey data types within a single b+tree. 
  In other words, `long` bkey and `byte array` bkey types cannot co-exist/mixed.
  
Please check the [ARCUS Server ASCII Protocol](ch02-collection-item-en.md) documentation first for basic B+teee item structure and basic features.

Before proceeding with B+tree item operations, first lets understand b+tree objects used in retrieval and update operations.

- [Bkey(B+tree Key) and EFlag(Element Flag)](07-btree-API-en.md#bkeybtree-key-and-eflagelement-flag)
- [Element Flag Filter Object](07-btree-API-en.md#element-flag-filter-object)
- [Element Flag Update Object](07-btree-API-en.md#element-flag-update-object)

The basic operations that can be performed on the B+tree item are as follows.

- [Create B+Tree Item](07-btree-API-en.md#create-btree-item)
- [Insert B+Tree Element](07-btree-API-en.md#insert-btree-element)
- [Upsert B+Tree Element](07-btree-API-en.md#upsert-btree-element)
- [Update B+Tree Element](07-btree-API-en.md#update-btree-element)
- [Delete B+Tree Element](07-btree-API-en.md#delete-btree-element)
- [B+Tree Element Value INCR/DECR](07-btree-API-en.md#btree-element-value-increment-decrement)
- [B+Tree Element Count](07-btree-API-en.md#btree-element-count)
- [Retrieve B+Tree Element](07-btree-API-en.md#retrieve-btree-element)

The bulk operations for multiple b+tree elements to perform at once are as follows.

- [Insert B+Tree Elements in Bulk](07-btree-API-en.md#insert-btree-elements-in-bulk)
- [Update B+Tree Elements in Bulk](07-btree-API-en.md#update-btree-elements-in-bulk)
- [Retrieve B+Tree Elements in Bulk](07-btree-API-en.md#retrieve-btree-elements-in-bulk)

The bulk operation for multiple b+tree elements to perform  sort-merge retrieval operation is as follows.

- [Sort-Merge Retrieval of B+Tree Element](07-btree-API-en.md#sort-merge-retrieval-of-btree-element)

B+tree position related opertions are as follows.

- [Retrieve B+Tree Position](07-btree-API-en.md#retrieve-btree-position)
- [Retrieve Position-Based B+Tree Element](07-btree-API-en.md#retrieve-position-based-btree-element)
- [Retrieve B+Tree Position and Element](07-btree-API-en.md#retrieve-btree-position-and-element)

## Bkey(B+tree Key) and EFlag(Element Flag)

There are two kind of data types used in b+tree.

- long type
- byte[1-31] type: byte array size can be used from 1 up to 31.

An example of creating bkey of a byte array type is as follows.
If byte array size exceeds more then 31, `IllegalArguementException` occurs.

```java
// use 0x00000001 as the Bkey.
byte[] bkey = new byte[] { 0, 0, 0, 1 }
```

The eflag is a field that currently exists only in b+tree elements.
eflag's data type can only be byte[1-31] and has the same usage method with byte array of bkey. 

## Element Flag Filter Object

eflag(element flag) filter condition can be used when retrieve, update and delete element.
The eflag filter conditions are expressed as follows.

- By default, eflag's full/partial value is computed by `compare value` and `compare operation`.
  - `compare value` is specified in the eflag filter as `operand` with `compare` operation for eflag value.
- Optionally, before performing a `compare` operation with `bitwise value` for the eflag's full/partial value 
   you can select a `bitwise operation` first.
  -  bitwise value is specified as operand that selects bitwise operation for the eflag value in the eflag filter.
  - **currently, there is a constraint that requires  both lengths of bitwise value and compare value must be same.** 

Select the eflag's full/partial value for which the compute/bitwise operations to 
perform in the eflag filter condition is as follows.

- target value of compare operation in eflag's full value is specifies as **compare offset** and **compare length** 
  - **compare offset** is 0 by default, but its value can be changed in the eflag filter.
  - **compare length** is automatically set to the length of **compare value** specified in the eflag filter.
- target value of bitwise operation in eflag's full value is specifies as **bitwise offset** and **bitwise length**
  - **bitwise offset** is not specified separately, but uses the compare offset as it is.
  - **bitwise length** is automatically set to the length of **bitwise value** specified in the eflag filter.

Provided `compare` operators are as follows.

Compare Operators                             | Description
--------------------------------------------- | ----
ElementFlagFilter.CompOperands.Equal	      | Equal
ElementFlagFilter.CompOperands.NotEqual	      | Not Equal
ElementFlagFilter.CompOperands.LessThan	      | Less than
ElementFlagFilter.CompOperands.LessOrEqual    | Less than or Equal to
ElementFlagFilter.CompOperands.GreaterThan    | Greater than
ElementFlagFilter.CompOperands.GreaterOrEqual | Greater than or Equat to

Provided `bitwise` operators are as follows.

Bitwise Operators                             | Description
--------------------------------------------- | ----
ElementFlagFilter.BitwiseOperands.AND	      | AND operation
ElementFlagFilter.BitwiseOperands.OR	      | OR operation
ElementFlagFilter.BitwiseOperands.XOR	      | XOR operation

### Element Flag Filter Method

A constructor function of a `ElementFlagFilter` class is shown below.
Create `ElementFlagFilter` object by specifying **compare operator** and **compare value**.
**compare offset** is set to 0 by default, there is no additional settings for bitwise operation.

```java
ElementFlagFilter(CompOperands compOperand, byte[] compValue)
```

Below method can be use in updating `compare offset` of `ElementFlagFilter` object or to set a `bitwise` operation.

```java
ElementFlagFilter setCompareOffset(int offset)
ElementFlagFilter setBitOperand(BitWiseOperands bitOp, byte[] bitCompValue)
```

### Example of Usage of Element Flag Filter

First sample retrieves total number of all the elements in the b+tree that matches the eflag value of 0x0102. 

```java
ElementFlagFilter filter = new ElementFlagFilter(CompOperands.Equal, new byte[] { 1, 2 }); // (1) 
CollectionFuture<Integer> future = mc.asyncBopGetItemCount(KEY, MIN_BKEY, MAX_BKEY, filter); 
Integer count = future.get(); 
```

1. Create a filter that determines whether eflag matches 0x0102. compare offset is set to 0, compare length is 2.

Second sample retrieves total number of element of all the elements in the b+tree that matches the value of
0x01 - second byte of eflag and 0x01 - results of AND operation.

```java
ElementFlagFilter filter = new ElementFlagFilter(CompOperands.Equal, new byte[] { 1 }); // (1) 
filter.setBitOperand(BitWiseOperands.AND, new byte[] { 1 }); // (2) 
filter.setCompareOffset(1); // (3) 

CollectionFuture<Integer> future = mc.asyncBopGetItemCount(KEY, MIN_BKEY, MAX_BKEY, filter); 
Integer count = future.get(); 
```

1. Create a filter that determines whether eflag matches 0x01.
2. Set a filter to perform 0x01 and bitwise AND operation before execuuting compare operation for eflag. 
3. Set a filter from the second byte of eflag.

If you do not want to use a filter condition for query, specify ` ElementFlagFilter.DO_NOT_FILTER` as the filter value.

Third sample retrieves all the elements in the b+tree without eflag setting.
To retrieve elements without eflag setting, select the 0x00 as first byte of eflag and bitwise AND operation
and on the result look for the elements other than 0x00.
If it's an element with an eflag, it's possible to retrieve elements that do not have eflag, 
if to retrieve a result of bitwise operation that is not 0x00 since it has to be 0x00 in any events  
when selecting 0x00 as the first byte and bitwise AND operation.

```java
ElementFlagFilter filter = new ElementFlagFilter(CompOperands.NotEqual, new byte[] { 0 });  // (1)
filter.setBitOperand(BitWiseOperands.AND, new byte[] { 0 }); // (2)
Map<Long, Object> map = mc.asyncBopGet(KEY, BKEY, BKEY + 100, 0, 100, false, false, filter).get(); // (3)
```

1. Create filter that retrieves elements where eflag is not 0x00
2. eflag that the filter compares to is the 0x00 bitwise AND operation and eflag set in element
3. You can retrieve elements without eflag settings by using filters created in step (1) and (2)

### Element Multi Flags Filter

A filter to perform following IN and NOT IN operations.

- IN Operation : checks if eflag value is the same with one of the several compare values
- Not in Operation : checks if eflag value is different from all the compare values

`ElementMultiFlagsFilter` can specify multiple compare values. It supports below two compare operators.

- ElementFlagFilter.CompOperands.Equal
- ElementFlagFilter.CompOperands.NotEqual

`ElementMultiFlagsFilter` usage sample below, retrieves the number of elements that matches 
0x0102 or 0x0104 - value of eflag among all the elements of the b+tree.
In other word, performing filter of IN operation.

```java
mentMultiFlagsFilter filter = new ElementMultiFlagsFilter(CompOperands.Equal); // (1) 
filter.addCompValue(new byte[] { 1, 2 }); // (2)
filter.addCompValue(new byte[] { 1, 4 }); // (3)
CollectionFuture<Integer> future = mc.asyncBopGetItemCount(KEY, MIN_BKEY, MAX_BKEY, filter); 
Integer count = future.get(); 
```

1. Create a filter
2. Register 0x0102 value to determine a match
3. Register 0x0104 value to determine a match

Up to 100 compare values can be specified with `ElementMultiFlagsFilter` and can only used in 
asyncBopGet, asyncBopCount, asyncBopDelete, asyncBopSortMergeGet.

## Element Flag Update Object

You can change the full or partial value of the eflag.
In order to do this, `ElementFlagUpdate` constructor function is explained below.

- ElementFlagUpdate(byte[] elementFlag
  -  replace the entire value of eflag with new elementFlag.
- ElementFlagUpdate(int elementFlagOffset, BitWiseOperands bitOp, byte[] elementFlag)
  - replace the partial value of the eflag with the result of taken bitwise operation.
  - In eflag the partial value of taken bitwise operation's offset and length determined by
    `elementFlagOffset` and `length of elementFlag value` respectively.   

An example of replacing whole Eflag value is as follows.

```java
byte[] flag = new byte[] { 1, 0, 1, 0 };
ElementFlagUpdate eflagUpdate = new ElementFlagUpdate(flag);

CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, eflagUpdate, null);
```

An example of replacing some Eflag values is as follows.

```java
int eFlagOffset = 1;
BitWiseOperands bitOp = BitWiseOperands.AND;
byte[] flag = new byte[] { 1 };

ElementFlagUpdate eflagUpdate = new ElementFlagUpdate(eFlagOffset, bitOp, flag);
CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, eflagUpdate, null);
```

## Create B+Tree Item

Create a new empty b+tree item.

```java
CollectionFuture<Boolean> asyncBopCreate(String key, ElementValueType valueType, CollectionAttributes attributes)
```

- key: key of b+tree item to create 
- valueType: specifies type of a value to store in the B+tree. There are the following types
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
- attributes: specifies the attributes of a b+tree item

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
True         | CollectionResponse.CREATED             | successfully created
False        | CollectionResponse.EXISTS              | same key already exists

An example of creating a B+tree item is as follows.

```java
String key = "Sample:EmptyBTree";
CollectionFuture<Boolean> future = null;
CollectionAttributes attribute = new CollectionAttributes(); // (1)
attribute.setExpireTime(60); // (1)

try {
    future = client.asyncBopCreate(key, ElementValueType.STRING, attribute); // (2)
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

1. Expire time of b+tree is set to to 60 seconds. Detailed usage of `CollectionAttributes` is covered in detail 
   in the chapter on [Manual: Usage of Java Client/Attribute](08-attribute-API-en.md)
2. When creating an empty b+tree, it is necessary to specify in advance the type of element to store in a b+tree.
   The reason is the mechanism by which a Java Client encodes/decodes a value.
   The sample above creates an empty sample to store String type. If you store a value that does not match the element type
   specified when creating an empty b+tree, you will not be able to lookup/retrieve the value, even if it was successfully stored.
3. `timeout` is set to 1 second. If the empty b+tree is successfully created, `future` returns `true`.
    If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
4. Detailed information about the creation result can be retrieved through the `future.getOperationStatus().getResponse()`.

## Insert B+Tree Element

A function to insert an element into a B+tree.
First one, uses bkey of `long` type, and the second one uses bkey of `byte array` type up to 31 sizes.

```java
CollectionFuture<Boolean>
asyncBopInsert(String key, long bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
CollectionFuture<Boolean>
asyncBopInsert(String key, byte[] bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
```

- key: key of a target b+tree to insert
- bkey: bkey(b+tree key) of element to insert 
- eflag: eflag(element flag) of element to insert [*optional*] 
- value: value of an element to insert.
- attributesForCreate: specifies what to do when the target b+tree does not exist.
  - null: does not insert an element. 
  - attributes: create an empty b+tree item with the given attributes and then insert the element

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | ---------
True         | CollectionResponse.STORED              | Element(s) inserted
True         | CollectionResponse.CREATED_STORED      | B+tree collection created and then element inserted
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
False        | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
False        | CollectionResponse.ELEMENT_EXISTS      | Element with given bkey already exists
False        | CollectionResponse.OVERFLOWED          | State of overflow (Maximum number of elements already present)
False        | CollectionResponse.OUT_OF_RANGE        | The given bkey's position is in trimmed b+tree area

An example of inserting an element into a b+tree is as follows.

```java
String key = "Prefix:BTreeKey";
long bkey = 1L;
String value = "This is a value.";
byte[] eflag = new byte[] { 1, 1, 1, 1 };

CollectionAttributes attributesForCreate = new CollectionAttributes();
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncBopInsert(key, bkey, eflag, value, attributesForCreate); // (1)
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

1. If a value of `attributesForCreate` is not `null` and a b+tree does not exist, create a new b+tree with a `attributesForCreate` property
  and then insert an element. If a value of `attributesForCreate` is `null` and a key does not exist, an element will not be inserted.
    - above sample uses default `CollectionAttributes` without special settings, default expire time means it does not expire at 0.
2. `timeout` is set to 1 second. If the insertion is successful, `future` returns `true`.
  If the result does not show at the specified time or if the operation queue fails to handle it due to overload of JVM `TimeoutException` occurs.
3. Detailed information on the result of insertion can be retrieved through the `future.getOperationStatus().getResponse()`.

In ARCUS B+tree has a limitation for maximum number of eleemnts available.
Within this limitations, a user can direcly specify a b+tree size(maxcount),
and because of this constraint, if you add a new element into a full b+tree,
depending on the settings, some of the existing elements might be deleted.
Therefore, b+tree provides a function that can obtain element(s) that implicitly deleted at the time of input (insert, upsert).

```java
BTreeStoreAndGetFuture<Boolean, Object>
asyncBopInsertAndGetTrimmed(String key, long bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)

BTreeStoreAndGetFuture<Boolean, Object>
asyncBopInsertAndGetTrimmed(String key, byte[] bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
```
It lookups/retrieves value of the element(s) in the b+tree that was implicitly deleted(trim) when inserting or upserting new element corresponding to the bkey in the B+tree

- key:  key of a b+tree item
- bkey: bkey(b+tree key) of element to insert
  - as a key to element bkey can use either `long` or `byte` [1-31] types
  - value can be specified only greater than zero, even if the bkey and value are stored in the existed key,
    an expire time set in the key does not change
- eflag: eflag(element flag) of element to insert.
- value: value of an element to insert.
- attributesForCreate: specifies what to do when the target b+tree does not exist.
  - null: does not insert an element. 
  - attributes: create an empty b+tree item with the given attributes and then insert the element

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | ---------
True         | CollectionResponse.STORED              | Element(s) inserted
True         | CollectionResponse.CREATED_STORED      | B+tree collection created and then element inserted
True         | CollectionResponse.REPLACED            | Element replaced
True         | CollectionResponse.TRIMMED             | Element inserted and trimmed element retrieved as inserted
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
False        | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
False        | CollectionResponse.ELEMENT_EXISTS      | Element with given bkey already exists
False        | CollectionResponse.OVERFLOWED          | State of overflow (Maximum number of elements already present)
False        | CollectionResponse.OUT_OF_RANGE        | The given bkey's position is in trimmed b+tree area

Check ` future.getElement()` object for more information about elements that are deleted (trim).

Method of future.getElement() object | Data Type | Description
-------------------------------------|-----------|------
getValue()                           | Object    | Value of element
getByteArrayBkey()                   | byte[]    | Value(byte[]) of bkey element
getLongBkey()                        | long      | Value(long) of bkey element 
isByteArrayBkey()                    | boolean   | Value(or byte array) of bkey element
getFlag()                            | byte[]    | Value(byte[]) of flag element 

A sample code of retrieving implicitly trim element while inserting element into b+tree is as follows.

```java
private String key = "BopStoreAndGetTest";
private long[] longBkeys = { 10L, 11L, 12L, 13L, 14L, 15L, 16L, 17L, 18L,

public void testInsertAndGetTrimmedLongBKey() throws Exception {
	// insert test data
	CollectionAttributes attrs = new CollectionAttributes();
	attrs.setMaxCount(10);
	attrs.setOverflowAction(CollectionOverflowAction.smallest_trim);
	for (long each : longBkeys) {
		mc.asyncBopInsert(key, each, null, "val", attrs).get();
	}

	// cause an overflow
	assertTrue(mc.asyncBopInsert(key, 1000, null, "val", null).get());
	
	// expecting that bkey 10 was trimmed out and the first bkey is 11 
	Map<Integer, Element<Object>> posMap = mc.asyncBopGetByPosition(key, BTreeOrder.ASC, 0).get();
	assertNotNull(posMap);
	assertNotNull(posMap.get(0)); // the first element
	assertEquals(11L, posMap.get(0).getLongBkey());

	// then cause an overflow again and get a trimmed object
	// it would be a bkey(11)
	BTreeStoreAndGetFuture<Boolean, Object> f = mc.asyncBopInsertAndGetTrimmed(key, 2000, null, "val", null);
	boolean succeeded = f.get();
	Element<Object> element = f.getElement();
	assertTrue(succeeded);
	assertNotNull(element);
	assertEquals(11L, element.getLongBkey());
	System.out.println("The insertion was succeeded and an element " + f.getElement() + " was trimmed out");
	
	// finally check the first bkey which is expected to be 12 
	posMap = mc.asyncBopGetByPosition(key, BTreeOrder.ASC, 0).get();
	assertNotNull(posMap);
	assertNotNull(posMap.get(0)); // the first element
	assertEquals(12L, posMap.get(0).getLongBkey());
}
```

## Upsert B+Tree Element

A function that upsert a single element in b+tree.
Upsert function is an operation inserts an element if it is not exists, and if there is, it updates an element.
First function uses long bkey, and second one uses up to 31 size a byte array bkey.

```java
CollectionFuture<Boolean>
asyncBopUpsert(String key, long bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
CollectionFuture<Boolean>
asyncBopUpsert(String key, byte[] bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
```

- key: key of a target b+tree to upsert
- bkey: bkey(b+tree key) of element to upsert
- eflag: eflag(element flag) of element to upsert [*optional*] 
- value: value of an element to upsert
- attributesForCreate:  specifies what to do when the target b+tree does not exist
  - null: does not upsert an element
  - attributes: create an empty b+tree item with the given attributes and then insert the element

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | ---------
True         | CollectionResponse.STORED              | Only element inserted
True         | CollectionResponse.CREATED_STORED      | B+tree collection created and then element inserted
True         | CollectionResponse.REPLACED            | Element replaced
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
False        | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
False        | CollectionResponse.OVERFLOWED          | State of overflow (Maximum number of elements already present)
False        | CollectionResponse.OUT_OF_RANGE        | The given bkey's position is in trimmed b+tree area

Sample code of upsert of b+tree element is as follows.

```java
String key = "Prefix:BTreeKey";
long bkey = 1L;
String value = "This is a value.";
byte[] eflag = new byte[] { 1, 1, 1, 1 };

CollectionAttributes attributesForCreate = new CollectionAttributes();
CollectionFuture<Boolean> future = null;


try {
    future = client.asyncBopUpsert(key, bkey, eflag, value, attributesForCreate); // (1)
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

1. Update or insert elements into the B+tree. If a value of `attributesForCreate` is not `null` and a b+tree does not exist,
   create a new b+tree with a `attributesForCreate` property and then insert an element.
   If a value of `attributesForCreate` is `null` and a key does not exist, an element will not be inserted.
   If you do not specify expire time in `attributesForCreate` when a key created,
   default value is set to 0, meaning b+tree does not expire.  
2.  `timeout` is set to 1 second. If the upsert is successful, `future` returns `true`.
   If the result does not show at the specified time or if the operation queue fails to handle it due to overload of 
   JVM `TimeoutException` occurs.
3. Detailed information on the result of upsert can be retrieved through the
   `future.getOperationStatus().getResponse()`.


## Update B+Tree Element

A function to update an element in a B+tree. It also updates eflag and/or value of elements.
First one, uses bkey of `long` type, and the second one uses bkey of `byte array` type up to 31 sizes.

```java
CollectionFuture<Boolean> asyncBopUpdate(String key, long bkey, ElementFlagUpdate eFlagUpdate, Object value)
CollectionFuture<Boolean> asyncBopUpdate(String key, byte[] bkey, ElementFlagUpdate eFlagUpdate, Object value)
```

- key:  key of a target b+tree to update
- bkey: bkey(b+tree key) of a target element to update
- eFlagUpdate: eflag content of element to update
  - specify `null` if do not want to change eflag
  - specify `ElementFlagUpdate.RESET_FLAG` to delete eflag
- value: new value of element
  - `null` if do not want to change value.

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse()| Description 
------------ | -------------------------------------- | ---------
True         | CollectionResponse.UPDATED             | Element is updated
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key))
False        | CollectionResponse.NOT_FOUND_ELEMENT   | Element with given bkey does not exist
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
False        | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
False        | CollectionResponse.EFLAG_MISMATCH      | The given eFlagUpdate does not match with existing element's eflag data

Updates only a value of a particular element without changing its eflag.

```java
CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, null, value);
```

Updates only eflag to 0x01000100 of a particular element without changing its value.

```java
byte[] flag = new byte[] { 1, 0, 1, 0 };
ElementFlagUpdate eflagUpdate = new ElementFlagUpdate(flag);
CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, eflagUpdate, null);
```

Delete only eflag of a particular element without changing its value.

```java
CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, ElementFlagUpdate.RESET_FLAG, null);
```

Below is an example of updating a specific byte by bitwise operationin eflag of a particular element.
In the sample code eflag's second byte is updated by the result of 0x01 and AND operation.
Update operation will fail if the eflag does not exist in a given element or a byte referring to the offset does not exist. 

```java
int eFlagOffset = 1;
BitWiseOperands bitOp = BitWiseOperands.AND;
byte[] flag = new byte[] { 1 };

ElementFlagUpdate eflagUpdate = new ElementFlagUpdate(eFlagOffset, bitOp, flag);
CollectionFuture<Boolean> future = mc.asyncBopUpdate(KEY, BKEY, eflagUpdate, null);
```

Detailed information on the result of update can be retrieved through the `future.getOperationStatus().getResponse()`.

## Delete B+Tree Element

There are two types of functions that deletes elements from b+tree. 

First, delete an element from b+tree that meets the `eFlagFilter` condition of a given bkey.

```java
CollectionFuture<Boolean>
asyncBopDelete(String key, long bkey, ElementFlagFilter eFlagFilter, boolean dropIfEmpty)
CollectionFuture<Boolean>
asyncBopDelete(String key, byte[] bkey, ElementFlagFilter eFlagFilter, boolean dropIfEmpty)
```

Second, delete elements from b+tree within `from` and `to` range query that meets the `eFlagFilter` condition.
  - if the `count` is 0, delete all elements that meets the `eFlagFilter` condition
  - if the `count` is greaterm that 0, then delete only `count` elements.

```java
CollectionFuture<Boolean>
asyncBopDelete(String key, long from, long to, ElementFlagFilter eFlagFilter, int count, boolean dropIfEmpty)
CollectionFuture<Boolean>
asyncBopDelete(String key, byte[] from, byte[] to, ElementFlagFilter eFlagFilter, int count, boolean dropIfEmpty)
```

- key: target key of b+tree to delete
- bkey or \<from, to\>: bkey(b+tree key) or bkey range of element to delete
- eFlagFilter: `filter` condition for eflag 
- count: specify number of elements to delete
   -  0 : delete all elements that meets the condition
- dropIfEmpty: specifies whether to delete a b+tree itself after deleting element(s) that results in an empty b+tree

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | ---------
True         | CollectionResponse.DELETED             | Delete only an element
True         | CollectionResponse.DELETED_DROPPED     | Delete an element from a b+tree, if the it becomes an empty then delete that b+tree too.
False        | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
False        | CollectionResponse.NOT_FOUND_ELEMENT   | Element with given bkey does not exist
False        | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
False        | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type

A sample code of deleting element from a b+tree where `bkey` is `1` is as follows.

```java
String key = "Prefix:BTreeKey";
long bkey = 1L;
boolean dropIfEmpty = true;
CollectionFuture<Boolean> future = null;

try {
    future = client.asyncBopDelete(key, bkey, ElementFlagFilter.DO_NOT_FILTER, dropIfEmpty); // (1)
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

1. Deletes the elements from the b+tree corresponding to the bkey.
   If a value of `dropIfEmpty` is `true` after an element is deleted, and a b+tree becomes empty, delete that b+tree too. 
   In the example, the filter's condition is specified as "do not filter".
2. `delete timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue fails
    to handle it due to overload of JVM `TimeoutException` occurs.
3. Detailed information on the result of delete can be retrieved through the
   `future.getOperationStatus().getResponse()`.

## B+Tree Element Value Increment-Decrement

A function that increases or decreases a value of a b+tree element. The value must be a numeric value of a String type.

```java
CollectionFuture<Long> asyncBopIncr(String key, long bkey, int by)
CollectionFuture<Long> asyncBopDecr(String key, long bkey, int by)

CollectionFuture<Long> asyncBopIncr(String key, Byte[] bkey, int by)
CollectionFuture<Long> asyncBopDecr(String key, Byte[] bkey, int by)

CollectionFuture<Long> asyncBopIncr(String key, long subkey, int by, long initial, byte[] eFlag);
CollectionFuture<Long> asyncBopDecr(String key, long subkey, int by, long initial, byte[] eFlag);

CollectionFuture<Long> asyncBopIncr(String key, byte[] subkey, int by, long initial, byte[] eFlag);
CollectionFuture<Long> asyncBopDecr(String key, byte[] subkey, int by, long initial, byte[] eFlag);
```

- key: key of b+tree item
- bkey: bkey of a target element
- by: value of how many time increase or decrease
  - value must be greater than 1
  - if decrease value is greater than value of element, result value of decrease will stored as 0.

[*Optional*] Below are new inserted values if a target element does not exist. 

- initial: value of a element to insert. (value must be greater than 0 [64bit unsigned integer]) 
- eflag: eflag(element flag) of element to insert 

The result is obtained through the `future` object.

future.get()     | future.getOperationStatus().getResponse() | Description
---------------- | -------------------------------------- | ---------
element value    | CollectionResponse.END                 | Successfully increased and[or] decreased
null             | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null             | CollectionResponse.NOT_FOUND_ELEMENT   | Element with given bkey does not exist
null             | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null             | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null             | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)
null             | CollectionResponse.OVERFLOWED          | State of overflow (Maximum number of elements already present)
null             | CollectionResponse.OUT_OF_RANGE        | The given bkey's position is in trimmed b+tree area

A Sample code for increasing value of b+tree element is as follows.

```java
String key = "Prefix:BTree";
long bkey = 0L;
CollectionFuture<Long> future = null;

try {
    future = mc.asyncBopIncr(key, bkey, (int) 2); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Long result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(future.getOperationStatus().getResponse()); // (3)
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. In above example incremented value of a element stored in b+tree by 2.
2. `timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue fails to handle
   it due to overload of JVM, `TimeoutException` occurs.
3. Detailed information on the result of element increment can be retrieved through the `future.getOperationStatus().getResponse()`.

## B+Tree Element Count

Retrives number of elements(`count`) from a range of [`from` `to`] of a given bkey in b+tree that meets the `eflagFilter` condition.

```java
CollectionFuture<Integer>
asyncBopGetItemCount(String key, long from, long to, ElementFlagFilter eFlagFilter)
CollectionFuture<Integer>
asyncBopGetItemCount(String key, byte[] from, byte[] to, ElementFlagFilter eFlagFilter)
```

- key: key of b+tree item
- \<from, to\>: indicates a bkey range query of a element
- eFlagFilter: `filter ` condition for eflag

The result is obtained through the `future` object.

future.get()   | future.getOperationStatus().getResponse() | Description
---------------| -------------------------------------- | -------
element count  | CollectionResponse.END                 | Successfully retrived element count
null           | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null           | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null           | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null           | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

A sample code of checking number of b+tree elements is as follows.

```java
String key = "Prefix:BTree";
long bkeyFrom = 0L;
long bkeyTo = 100L;

CollectionFuture<Integer> future = null;

try {
    future = mc.asyncBopGetItemCount(key, bkeyFrom, bkeyTo, ElementFlagFilter.DO_NOT_FILTER); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Integer result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(future.getOperationStatus().getResponse()); // (3)
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. In above example, bkey retrieved number of elements from `bkeyFrom` to `bkeyTo` among the stored elements in b+tree.
2. `timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue fails to handle
   it due to overload of JVM, `TimeoutException` occurs.
3. Detailed information on the result of element count can be retrieved through the `future.getOperationStatus().getResponse()`.

## Retrieve B+Tree Element

There are two types of functions that retrieves element from b+tree.

First, retrieve a given bkey element from a b+tree that meets `eFlagFilter` condition.

```java
CollectionFuture<Map<Long, Element<Object>>>
asyncBopGet(String key, long bkey, ElementFlagFilter eFlagFilter, boolean withDelete, Boolean dropIfEmpty)
CollectionFuture<Map<ByteArrayBKey, Element<Object>>>
asyncBopGet(String key, byte[] bkey, ElementFlagFilter eFlagFilter, boolean withDelete, Boolean dropIfEmpty)
```

Second, retrive number of elements (`count`) from a range of `from` `to`  of a given bkey in b+tree that meets the `eflagFilter` condition starting with the first offset element.

```java
CollectionFuture<Map<Long, Element<Object>>>
asyncBopGet(String key, long from, long to, ElementFlagFilter eFlagFilter, int offset, int count, boolean withDelete, boolean dropIfEmpty)
CollectionFuture<Map<ByteArrayBKey, Element<Object>>>
asyncBopGet(String key, byte[] from, byte[] to, ElementFlagFilter eFlagFilter, int offset, int count, boolean withDelete, Boolean dropIfEmpty)
```

- key: key of b+tree item
- bkey or \<from, to\>: bkey(b+tree key) or bkey range of element to retrieve
- eFlagFilter: `filter` condition for eflag
- withDelete: specifies whether to delete an element with an element retrieval
- dropIfempty: specifies whether to delete a b+tree itself after deleting element(s) that results in an empty b+tree

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description
------------ | -------------------------------------- | -------
not null     | CollectionResponse.END                 | Retrieve element only, no b+tree trim area in range query
not null     | CollectionResponse.TRIMMED             | Retrieve element only, b+tree trim area in range query
not null     | CollectionResponse.DELETED             | Retrieve element and delete
not null     | CollectionResponse.DELETED_DROPPED     | Retrieve element and delete, b+tree also dropped(deleted)
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found, no b+tree trim area in range query
null         | CollectionResponse.OUT_OF_RANGE        | Element not found, given bkey's position is in trimmed b+tree area
null         | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null         | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

The result is obtained through the `result(Map<Long, Element<Object>>)` object is as follows.

Method of result object       | Data type | Description
------------------------------|-----------|-------------
getKey()                      | Long      | Position in the b+tree
getValue().getValue()         | Object    | Value of Element
getValue().getByteArrayBkey() | byte[]    | Value(byte[]) of bkey element 
getValue().getLongBkey()      | long      | Value(long) of bkey element
getValue().isByteArrayBkey()  | boolean   | Value(or byte array) of bkey element 
getValue().getFlag()          | byte[]    | Value(byte[]) of flag element 

A sample code of retrieving element from b+tree is as follows.

```java
String key = "Prefix:BTreeKey";
long from = 1L;
long to = 6L;
int offset = 2;
int count = 3;
boolean withDelete = false;
boolean dropIfEmpty = false;
ElementFlagFilter filter = new ElementFlagFilter(CompOperands.Equal, new byte[] { 1, 1 });
CollectionFuture<Map<Long, Element<Object>>> future = null;

try {
    future = client.asyncBopGet(key, from, to, filter, offset, count, withDelete, dropIfEmpty); // (1)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Long, Element<Object>> result = future.get(1000L, TimeUnit.MILLISECONDS); // (2)
    System.out.println(result);

    CollectionResponse response = future.getOperationStatus().getResponse(); // (3)
    if (response.equals(CollectionResponse.NOT_FOUND)) {
        System.out.println("There is no key. (no B+tree stored in the key)");
    } else if (response.equals(CollectionResponse.NOT_FOUND_ELEMENT)) {
        System.out.println("There is a B+tree in the key, but none of the stored values meet the conditions.");
    }

} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. bkey is between 1 and 6, and among the elements that meet the filter condition, retrieve three values starting from third position.
   In above example, eflag filter condition checks whether the eflag value is equal to 0x0101.
2. `timeout` is set to 1 second. If the result does not show at the specified time
   or if the operation queue fails to handle it due to overload of JVM, `TimeoutException` occurs.
   The implementation of the returned Map interface is TreeMap, and its result will be one of the following:
   - key is missing, return null
   - key exists but no element found matching the condition, return empty map
   - key exists and element meets the condition, return non-empty map
3. Detailed information on the result of retrieval can be checked through the `future.getOperationStatus().getResponse()`.

## Insert B+Tree Elements in Bulk

There are two types of bulk insert functions for inserting multiple elements into b+tree.

First,  a function that inserts multiple elements into a b+tree referred by a single key.

```java
CollectionFuture<Map<Integer, CollectionOperationStatus>>
asyncBopPipedInsertBulk(String key, List<Element<Object>> elements, CollectionAttributes attributesForCreate)
CollectionFuture<Map<Integer, CollectionOperationStatus>>
asyncBopPipedInsertBulk(String key, Map<Long, Object> elements, CollectionAttributes attributesForCreate)
```

- key: key of target b+tree to insert
- elements: elements to insert
  - List\<Element\<Object\>\> type
  - Map\<Long, Object\> type
- attributesForCreate: specifies what to do when the target b+tree does not exist
  - null: does not insert an element
  - create an empty b+tree item with the given attributes and then insert the element

Second, a function that inserts one element into each b+tree referred by multiple keys.

```java
Future<Map<String, CollectionOperationStatus>>
asyncBopInsertBulk(List<String> keyList, long bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
Future<Map<String, CollectionOperationStatus>>
asyncBopInsertBulk(List<String> keyList, byte[] bkey, byte[] eFlag, Object value, CollectionAttributes attributesForCreate)
```

- keyList: key list of a target b+tree to insert.
- bkey: bkey (b+tree key) of an element to insert
- eflag: eflag(element flag) of an element to insert
- value: value of element to insert
- attributesForCreate: specifies what to do when the target b+tree does not exist
  - null: does not insert an element
  - create an empty b+tree item with the given attributes and then insert the element

Below is a sample code for bulk insert of multiple elements into a b+tree and checking results of insert for each item.

```java
String key = "Sample:BTreeBulk";
List<Element<Object>> elements = new ArrayList<Element<Object>>();

elements.add(new Element<Object>(1L, "VALUE1", new byte[] { 1, 1 }));
elements.add(new Element<Object>(2L, "VALUE2", new byte[] { 1, 1 }));
elements.add(new Element<Object>(3L, "VALUE3", new byte[] { 1, 1 }));

boolean createKeyIfNotExists = true;

if (elements.size() > mc.getMaxPipedItemCount()) { // (1)
    System.out.println("The number of items to insert cannot exceed mc.getMaxPipedItemCount.");
    return;
}

CollectionFuture<Map<Integer, CollectionOperationStatus>> future = null;

try {
    future = mc.asyncBopPipedInsertBulk(key, elements, new CollectionAttributes()); // (2)

} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    Map<Integer, CollectionOperationStatus> result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)

    if (!result.isEmpty()) { // (4)
        System.out.println("Some items failed to insert.");
        
        for (Map.Entry<Integer, CollectionOperationStatus> entry : result.entrySet()) {
            System.out.print("failed item=" + elements.get(entry.getKey()));
            System.out.println(", failure cause=" + entry.getValue().getResponse());
        }
    } else {
        System.out.println("All successfully inserted.");
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
2. Insert `bulkData` at once into a b+tree stored in the key and then return the `future` object that contains the result. 
   From the `future` object checked whether each item was successfully inserted or failed to insert. 
   Before bulk insertion, `attributesForCreate` value is specified, to create a key and insert the element, in case of there is no key initially.
3. `delete timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue fails
    to handle it due to overload of JVM `TimeoutException` occurs.
4. Return empty map if all items are successfully inserted.
	- returned `Map's Key= value` of index when iterates inserted value(bulkData)
	- returned `Map's Value= cause` of failed insert
5. To check the cause of some failed items, retrieve the `Map` result depending on the order of the iteration of value(`bulkData`) used to insert.
6. Because the Key of the Map obtained from `Future` is a mapKey of inserted value(`bulkData`) the cause of the 
   failure can be checked in the same manner as mentioned before.

## Update B+Tree Elements in Bulk

Bulk update all corresponding values of a given element AND/OR their element flag too.

```java
CollectionFuture<Map<Integer, CollectionOperationStatus>>
asyncBopPipedUpdateBulk(String key, List<Element<Object>> elements)
```

- key: key of a target b+tree to update
- elements: contains bkey, eFlagUpdate and new value about a target element to update

## Retrieve B+Tree Elements in Bulk

Retrive number of elements (`count`) from a range of `from` `to`  of a given bkey in each b+tree that meets the `eflagFilter` 
condition starting with the first offset element.

```java
 CollectionGetBulkFuture<Map<String, BTreeGetResult<Long, Object>>>
asyncBopGetBulk(List<String> keyList, long from, long to, ElementFlagFilter eFlagFilter, int offset, int count)
 CollectionGetBulkFuture<Map<String, BTreeGetResult<ByteArrayBKey, Object>>>
asyncBopGetBulk(List<String> keyList, byte[] from, byte[] to, ElementFlagFilter eFlagFilter, int offset, int count)
```

- keyList: key list of b+tree item
- bkey or \<from, to\>: target bkey(b+tree key) or bkey range of element to retrieve
- eFlagFilter: `filter` condition for eflag
   - if eflag filter condition does not required, add `ElementFlagFilter.DO_NOT_FILTER`
- offset, count: specifies offset and count of elements to retrieve that meet `bkey range` and `eflag filter` conditions

The result of operation `Map<Stirng, BTreeGetResult<Bkey, Object>>` obtained through the `future` object, where `Map` is
an object that contains object result `BTreeGetResult` of query retrieval from b+tree and the key of a each b+tree items.

Each queries' result can be retrieve through the `BTreeGetResult` object is as follows.

BTreeGetResult.getElements() |  BtreeGetResult.getCollectionResponse() | Description 
---------------------------- | --------------------------------------- | -------
not null                     | CollectionResponse.OK                   | Retrieve element, no b+tree trim area in range query
not null                     | CollectionResponse.TRIMMED              | Retrieve element, b+tree trim area in range query
null                         | CollectionResponse.NOT_FOUND            | Key miss (no item found for a given key)
null                         | CollectionResponse.NOT_FOUND_ELEMENT    | Element not found, no b+tree trim area in range query
null                         | CollectionResponse.OUT_OF_RANGE         | Element not found, given bkey's position is in trimmed b+tree area
null                         | CollectionResponse.TYPE_MISMATCH        | The given item is not a b+tree
null                         | CollectionResponse.BKEY_MISMATCH        | The given bkey type is different from existing bkey type
null                         | CollectionResponse.UNREADABLE           | The given key is unreadable (unreadable item) 

You can retrieve bkey, eflag, value of each element from `BTreeElement` object retrieved by `BTreeGetResult.getElements()`.

Method if BTreeElement object | Data Type        | Description
----------------------------- | ---------------- | ----
getKey()                      | long or byte[]   | bkey of element
getEFlag()                    | byte[]           | Element flag
getValue()                    | Object           | Value of element

Below is a sample code for bulk retrieval of b+tree elements.

```java
final List<String> keyList = new ArrayList<String>() {
    {
        add("Prefix:BTree1");
        add("Prefix:BTree2");
        add("Prefix:BTree3");
    }
};

ElementFlagFilter filter = ElementFlagFilter.DO_NOT_FILTER;
long bkeyFrom = 0L;
long bkeyTo = 100L;
int queryCount = 10;
int offset = 0;

CollectionGetBulkFuture<Map<String, BTreeGetResult<Long, Object>>> future = null;
Map<String, BTreeGetResult<Long, Object>> results = null;

try {
    future = mc.asyncBopGetBulk(keyList, from, to, filter, offset, count); // (1)
    results = future.get(1000L, TimeUnit.MILLISECONDS);
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}

if (results == null) return;

for(Entry<String, BTreeGetResult<Long, Object>> entry : results.entrySet()) { // (2)
    System.out.println("key=" + entry.getKey());
    System.out.println("result code=" + entry.getValue().getCollectionResponse().getMessage()); // (3)

    if (entry.getValue().getElements() != null) { // (4)
        for(Entry<Long, BTreeElement<Long, Object>> el : entry.getValue().getElements().entrySet()) {
            System.out.println("bkey=" + el.getKey());
            System.out.println("eflag=" + Arrays.toString(el.getValue().getEflag());
            System.out.println("value=" + el.getValue().getValue());
        }
    }
}
```

1. bkey is between `from` and `to` among the elements stored in the b+tree given as keyList,
   and between the elements that meets the eflag filter, retrieved the `count` of elements starting from offset 
2. Result of retrieval returned as a Map, Key becomes the key of b+tree and the value of the Map are the elements stored in each key.
3. In the query result Map, `BTreeGetResult` object that retrieved by per key has query result code and the 
   retrieved elements from b+tree. Depending on the result code a result of `BTreeGetResult.getElements()` can be `null`.
4. You can retrieve the bkey, eflag, and value of the elements from retrieved BTreeElement object by `BTreeGetResult.getElements()` 

## Sort-Merge Retrieval of B+Tree Element

A function of performing element retrieval on many b+trees in a sort-merge manner. Although physically comprised of
several b+trees, it is a function that logically assumes that they are one huge b+tree,
and performs element queries on these b+trees.

`smget` operation is a process of overlapping the range query with the trim area of any b+tree, with below shown two modes operations.

1. Existing Sort-Merge Retrieval (works on lower 1.8.X versions)
   - The first element that meets the `smget` query condition, even if any trimmed b+tree exists, it sends OUT_OF_RANGE response.
   - In the absence of OUT_OF_RANGE, if the second subsequent element meets trimmed b+tree that satisfies 
     the query condition while performing `smget`, elements retrieved up to the point that are sent as final elements results and
     the `smget` performance status is TRIMMED. In this case, the application retrieves the elements of the trim area 
     in the DB for all keys and reflected in the result of the `smget`.
     
2. New Sort-Merge Retrieval (works on above 1.9.0 versions)
   - The b+tree corresponding to the existing OUT_OF_RANGE is classified as missed keys and `smget` is continued on the
     remaining b+trees. Therefore, in the application, elements can be retrieved from the DB, only for missed keys, 
     and reflected in the final `smget` results.
   - Even if a second subsequent element that meets the `smget` lookup criteria exists, `smget` is not stopped at that point,
     but rather continues until such b+trees are classified as trimmed keys and the desired number of elements 
     are found. Therefore, the application can only query trim elements in the DB and reflect them in the final `smget` results.
   - It supports unique lookup function for bkey key. In addition to duplicated queries that allow duplicate bkeys 
     to be viewed, it also removes duplicate bkeys and supports unique bkeys that only query unique bkeys.  
   - Removed the offset feature in the query condition.  

It is recommended that the offset value is always used at 0, even if you use existing `smget` operation.
Missed keys exist in `smget` with a positive offset, and the reason is if retrieval of DB for missed keys has skipped elements 
by offset then application couldn't be able to process offset accurately. 
If you want to additionally retrieve  following the previous retrieval results, then based on previously retrieved bkey values,
bkey range can be re-adjusted and used.

A function that performs sort-merge get on several b+trees is as follows.  
Starting from a number of b+trees having bkey of `from` `to` you can sort-merge them and retrieve `count` number of elements
that meet the eflag filter conditions.

```java
SMGetFuture<List<SMGetElement<Object>>>
asyncBopSortMergeGet(List<String> keyList, long from, long to, ElementFlagFilter eFlagFilter, int count, SMGetMode smgetMode)
SMGetFuture<List<SMGetElement<Object>>>
asyncBopSortMergeGet(List<String> keyList, byte[] from, byte[] to, ElementFlagFilter eFlagFilter, int count, SMGetMode smgetMode)
```

- keyList: key list of b+tree item
- \<from, to\>: bkey range of element to retrieve
- eFlagFilter: filter conditionfor eflag
  - if eflag filter condition does not required, add `ElementFlagFilter.DO_NOT_FILTER`
- count: specifies count of elements to retrieve that meet bkey range and eflag filter conditions
  - **constains: item count must not exceed 1000**
  - reason is to reduce a load of `sort-merge get` operation
- smgetMode: a flag to specify mode of `smget`
  - specifies either query is unique or duplicated

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | -------
not null     | CollectionResponse.END                 | Retrieve element, there is no duplicate bkey
not null     | CollectionResponse.DUPLICATED          | Retrieve element, there is a duplicate bkey
null         | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null         | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null         | CollectionResponse.ATTR_MISMATCH       | The properties of b+tree in sort-merge get are different from each other. Attribute constraints no longer apply after arcus-memcached 1.11.3 version

A sample code of sort-merge retrival of element in b+tree is as follows.

```java
List<String> keyList = new ArrayList<String>() {{
    add("Prefix:KeyA");
    add("Prefix:KeyB");
    add("Prefix:KeyC");
}};

long bkeyFrom = 0L; // (1)
long bkeyTo = 100L;
int queryCount = 10;

SMGetMode smgetMode = SMGetMode.DUPLICATE;
SMGetFuture<List<SMGetElement<Object>>> future = null;

try {
    future = mc.asyncBopSortMergeGet(keyList, bkeyFrom, bkeyTo, ElementFlagFilter.DO_NOT_FILTER, queryCount, smgetMode); // (2)
} catch (IllegalStateException e) {
    // handle exception
}

if (future == null)
    return;

try {
    List<SMGetElement<Object>> result = future.get(1000L, TimeUnit.MILLISECONDS); // (3)
    for (SMGetElement<Object> element : result) { // (4)
        System.out.println(element.getKey());
        System.out.println(element.getBkey());
        System.out.println(element.getValue());
    }

    for (Map.Entry<String, CollectionOperationStatus> m : future.getMissedKeys().entrySet()) {  // (5)
        System.out.print("Missed key : " + m.getKey());
        System.out.println(", response : " + m.getValue().getResponse());
    }
    
    for (SMGetTrimKey e : future.getTrimmedKeys()) { // (6)
        System.out.println("Trimmed key : " + e.getKey() + ", bkey : " + e.getBkey());
    }
} catch (InterruptedException e) {
    future.cancel(true);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (ExecutionException e) {
    future.cancel(true);
}
```

1. In above sample, bkey retrieves 10 elements from 0 to 100 that stored in "KeyA", "KeyB", "KeyC". 
   -  please pay attaention, that all the attribute settings of a b+tree given by the key must be the same.
      Otherwise, an error will occur.
2. `ElementFlagFilter` is a condition in which eflag specified in bkey only retrieves only elements that meet the 
    conditions specifies by `elementFlagFIlter`. In example, eflag filter was disabled.
3. `timeout` is set to 1 second. If the result does not show at the specified time or if the operation queue fails
   to handle it due to overload of JVM, `TimeoutException` occurs.
4. Retrieved value is returned as a from of `List<SMGetElement>`. From here on out you can lookup for retrieved elements.
   If the same bkey exists in the retrieved results, then based on key it's sorted returned. 
5. During the retrieval, among the specified keys that not included `smget` can be retrieved by `future.getMissedKeys()`
   in a form of Map with its cause of failure.
   - cause of failure is one of the: cache miss(NOT_FOUND),unreadable item(UNREADABLE), 
    first bkey that satisfies the bkey range query is trimmed(OUT_OF_RANGE).
   - application will restrieve for elements from the DB(back-end storage) under same query conditions for these keys,
     and reflect it in the sort-merge results.   
6. Although initially bkey exists in bkey range query, you can retrieve the bkey that trimmed and the last bkey
   in the cache before trim.
    - applicatrion will retrieve trimmed bkeys right before last bkey trimmed from the DB(back-end storage) for these keys,
     and reflect it in the sort-merge results.   
7. Final result of `sort-merge get` can be retrieved through the `future.getOperationStatus().getResponse()`.

## Retrieve B+Tree Position

As a search condition of b+tree, based on a position of a particular element you can retrieve its 
peripheral(front/back position) elements as well. The position in b+tree is the index for each element arranged
in a row through the bkey, ranged from 0 to count-1. Ascending (ASC) and descending (DESC) are supported as criteria for order.

A function to retrieve position of element according to a given order that corresponds to a given bkey in b+tree is as follows. 

```java
CollectionFuture<Integer> asyncBopFindPosition(String key, long bkey, BTreeOrder order)
CollectionFuture<Integer> asyncBopFindPosition(String key, byte[] bkey, BTreeOrder order)
```

- key: key of b+tree item
- bkey: bkey (b+tree key) of an element to retrieve
- order: defines the criteria of order in which results will be obtained.
   - ascending order: BTreeOrder.ASC
   - descending order: BTreeOrder.DESC    

The result is obtained through the `future` object.

future.get()     | future.getOperationStatus().getResponse() | Description
---------------- | -------------------------------------- | ---------
element position | CollectionResponse.OK                  | Position of element successfully retrieved
null             | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null             | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found
null             | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null             | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null             | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

A sample code to retrieve position of b+tree.

```java
String key = "BopFindPositionTest";
long[] longBkeys = { 10L, 11L, 12L, 13L, 14L, 15L, 16L, 17L, 18L, 19L };

public void testLongBKeyAsc() throws Exception {
	// insert
	CollectionAttributes attrs = new CollectionAttributes();
	for (long each : longBkeys) {
		arcusClient.asyncBopInsert(key, each, null, "val", attrs).get();
	}
	
	// bop position
	for (int i=0; i<longBkeys.length; i++) {
		CollectionFuture<Integer> f = arcusClient.asyncBopFindPosition(key, longBkeys[i], BTreeOrder.ASC);
		Integer position = f.get();
		assertNotNull(position);
		assertEquals(CollectionResponse.OK, f.getOperationStatus().getResponse());
		assertEquals(i, position.intValue());
	}
}

public void testLongBKeyDesc() throws Exception {
	// insert
	CollectionAttributes attrs = new CollectionAttributes();
	for (long each : longBkeys) {
		arcusClient.asyncBopInsert(key, each, null, "val", attrs).get();
	}
	
	// bop position
	for (int i=0; i<longBkeys.length; i++) {
		CollectionFuture<Integer> f = arcusClient.asyncBopFindPosition(key, longBkeys[i], BTreeOrder.DESC);
		Integer position = f.get();
		assertNotNull(position);
		assertEquals(CollectionResponse.OK, f.getOperationStatus().getResponse());
		assertEquals("invalid position", longBkeys.length-i-1, position.intValue());
	}
}
```

## Retrieve Position-Based B+Tree Element

A function to retrieve element(s) corresponding to a position or position range in a b+tree.

```java
CollectionFuture<Map<Integer, Element<Object>>>
asyncBopGetByPosition(String key, BTreeOrder order, int pos)
CollectionFuture<Map<Integer, Element<Object>>>
asyncBopGetByPosition(String key, BTreeOrder order, int from, int to)
```

- key: key of b+tree item
- order: defines the criteria of order in which results will be obtained.
   - ascending order: BTreeOrder.ASC
   - descending order: BTreeOrder.DESC 
- pos or \<from, to\>: specifies either a position or a range query of element(s)

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Description 
------------ | -------------------------------------- | ---------
not null     | CollectionResponse.END                 | Element successfully retrieved
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found
null         | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

A sample code to retrieve position-based b+tree element.

```java
String key = "BopGetByPositionTest";
long[] longBkeys = { 10L, 11L, 12L, 13L, 14L, 15L, 16L, 17L, 18L, 19L };

public void testLongBKeyMultiple() throws Exception {
	// Insert 10 units of test data.
	CollectionAttributes attrs = new CollectionAttributes();
	for (long each : longBkeys) {
		arcusClient.asyncBopInsert(key, each, null, "val", attrs).get();
	}
	
	// Test: retrieve elements from the position of 5 to 8.
	int posFrom = 5;
	int posTo = 8;
	CollectionFuture<Map<Integer, Element<Object>>> f = arcusClient
			.asyncBopGetByPosition(key, BTreeOrder.ASC, posFrom, posTo);
	Map<Integer, Element<Object>> result = f.get(1000,
			TimeUnit.MILLISECONDS);

	assertEquals(4, result.size());
	assertEquals(CollectionResponse.END, f.getOperationStatus().getResponse());

	int count = 0;
	for (Entry<Integer, Element<Object>> each : result.entrySet()) {
		int currPos = posFrom + count++;
		assertEquals("invalid index", currPos, each.getKey().intValue());
		assertEquals("invalid bkey", longBkeys[currPos], each.getValue().getLongBkey());
		assertEquals("invalid value", "val", each.getValue().getValue());
	}
}
```

## Retrieve B+Tree Position and Element

As a search condition of b+tree, based on a position of a particular element you can retrieve its peripheral(front/back position) elements as well.
The position in b+tree is the index for each element arranged in a row through the bkey, ranged from 0 to count-1.
Ascending (ASC) and descending (DESC) are supported as criteria for order.

A function to retrieve as many elements as `count` according to a given order that based on the position corresponding to bkey in b+tree.

```java
CollectionFuture<Map<Integer, Element<Object>>>
asyncBopFindPositionWithGet(String key, long longBKey, BTreeOrder order, int count)
CollectionFuture<Map<Integer, Element<Object>>>
asyncBopFindPositionWithGet(String key, byte[] byteArrayBKey, BTreeOrder order, int count)
```

- key: key of b+tree item
- longBkey: bkey(b+tree key) of element to retrieve
- order: defines the criteria of order in which results will be obtained based on a position of a given longBkey
  - ascending order: BTreeOrder.ASC
  - descending order: BTreeOrder.DESC
- count: specifies the number of peripheral(front/back positions) retrieval elements based on a position of a given longBkey

The result is obtained through the `future` object.

future.get() | future.getOperationStatus().getResponse() | Decription
------------ | -------------------------------------- | ---------
not null     | CollectionResponse.END                 | Element successfully retrieved
null         | CollectionResponse.NOT_FOUND           | Key miss (no item found for a given key)
null         | CollectionResponse.NOT_FOUND_ELEMENT   | Element not found
null         | CollectionResponse.TYPE_MISMATCH       | The given item is not a b+tree
null         | CollectionResponse.BKEY_MISMATCH       | The given bkey type is different from existing bkey type
null         | CollectionResponse.UNREADABLE          | The given key is unreadable (unreadable item)

The result is obtained through the `result(Map<Long, Element<Object>>)` object is as follows.

Method of result object           | Data Type         | Description
----------------------------------|-------------------|---------------
getKey()                          | integer           | Position in b+tree
getValue().getValue()             | Object            | Value of element
getValue().getByteArrayBkey()     | byte[]            | Value(byte[]) of bkey element
getValue().getLongBkey()          | long              | Value(long) of bkey element
getValue().isByteArrayBkey()      | boolean           | Value(or byte array) of bkey element
getValue().getFlag()              | byte[]            | Value(byte[]) of element flag

A sample code of retrieving `position` and `element` itself at the same time is as follows.

```java
String key = "BopFindPositionWithGetTest";

public void testLongBKey() throws Exception {
        long longBkey, resultBkey;
        int  totCount = 100;
        int  pwgCount = 10;
        int  rstCount;
        int  position, i;

        // totCount개의 테스트 데이터를 insert한다.
        CollectionAttributes attrs = new CollectionAttributes();
        for (i = 0; i < totCount; i++) {
                longBkey = (long)i;
                arcusClient.asyncBopInsert(key, longBkey, null, "val", attrs).get();
        }

        for (i = 0; i < totCount; i++) {
                // Based on the position of the element with longBkey as bkey
		// retrieve the surrounding (front/back position) elements as much as pwgCount.
                longBkey = (long)i;
                CollectionFuture<Map<Integer, Element<Object>>> f = arcusClient
                                .asyncBopFindPositionWithGet(key, longBkey, BTreeOrder.ASC, pwgCount);
                Map<Integer, Element<Object>> result = f.get(1000, TimeUnit.MILLISECONDS);

                if (i >= pwgCount && i < (totCount-pwgCount)) {
                        rstCount = pwgCount + 1 + pwgCount;
                } else {
                        if (i < pwgCount)
                        rstCount = i + 1 + pwgCount;
                        else
                        rstCount = pwgCount + 1 + ((totCount-1)-i);
                }
                assertEquals(rstCount, result.size());
                assertEquals(CollectionResponse.END, f.getOperationStatus().getResponse());

                if (i < pwgCount) {
                        position = 0;
                } else {
                        position = i - pwgCount;
                }
                resultBkey = position;
                for (Entry<Integer, Element<Object>> each : result.entrySet()) {
                        assertEquals("invalid position", position, each.getKey().intValue());
                        assertEquals("invalid bkey", resultBkey, each.getValue().getLongBkey());
                        assertEquals("invalid value", "val", each.getValue().getValue());
                        position++; resultBkey++;
                }
        }
}
```




