# Chapter 3. Item Attribute Description

ARCUS Cache Server provides extended key-value item types with Collection support that has List, Set, Map,
B+tree item types. Attributes that can be configured/retrieved are classified depending on each item type.
The overview of attributes is shown in the table below. The table below exhibits the item types to which each
attribute is applied, a brief description of the attribute, allowed values, and default values.

```
|-----------------------------------------------------------------------------------------------------------------|
| Attribute Name | Item Type   | Description           | Allowed Values                 | Default Value           |
|-----------------------------------------------------------------------------------------------------------------|
| flags          | all         | data specific flags   | 4 bytes unsigned integer       | 0                       |
|-----------------------------------------------------------------------------------------------------------------|
| expiretime     | all         | item expiration time  | 4 bytes singed integer         | 0                       |
|                |             |                       |  -1: sticky                    |                         |
|                |             |                       |   0: never expired             |                         |
|                |             |                       |  >0: expired in the future     |                         |
|-----------------------------------------------------------------------------------------------------------------|
| type           | all         | item type             | "kv", "list", "set", "map",    | N/A                     |
|                |             |                       | "b+tree"                       |                         |
|-----------------------------------------------------------------------------------------------------------------|
| count          | collection  | current # of elements | 4 bytes unsigned integer       | N/A                     |
|-----------------------------------------------------------------------------------------------------------------|
| maxcount       | collection  | maximum # of elements | 4 bytes unsigned integer       | 4000                    |
|-----------------------------------------------------------------------------------------------------------------|
| overflowaction | collection  | overflow action       | “error”: all collections       | list: "tail_trim"       |
|                |             |                       | “head_trim”: list              | set: "error"            |
|                |             |                       | “tail_trim”: list              | map: "error"            |
|                |             |                       | “smallest_trim”: b+tree        | b+tree: "smallest_trim" |
|                |             |                       | “largest_trim”: b+tree         |                         |
|                |             |                       | “smallest_silent_trim”: b+tree |                         |
|                |             |                       | “largest_silent_trim”: b+tree  |                         |
|-----------------------------------------------------------------------------------------------------------------|
| readable       | collection  | readable/unreadable   | “on”, “off”                    | "on"                    |
|-----------------------------------------------------------------------------------------------------------------|
| maxbkeyrange   | b+tree only | maximum bkey range    | 8 bytes unsigned integer or    | 0                       |
|                |             |                       | hexadecimal (max 31 bytes)     |                         |
|-----------------------------------------------------------------------------------------------------------------|
```

ARCUS Cache Server provides `getattr` and `setattr` commands for retrieving or updating item attributes.
For a detailed description of these commands please check the [Item Attribute Commands](ch10-command-item-attribute-en.md) chapter.

In order to help you understand the item attributes accurately, attributes requiring further explanation
will be described in detail below.

## flags Attribute

Flags are used for the purpose of storing data-specific information about an item. For example,
when ARCUS Java Client stores some java object in a cache server, create data to be stored by
serialization(or marshalling) depending on the type of java object, using the type information of the java object
as the flags value, sends a request to the ARCUS Cache Server and store it there. When retrieving data from
ARCUS Cache Server get the flag information with the data and depending on the type of java object 
de-serialization(or de-marshalling) data to create a java object.


## expiretime Attribute

With the item's expiretime attribute, the expiration time of the items is set in seconds.

ARCUS Cache Server provides a sticky item feature that is not expired and is not evicted even in low memory cases.
Sticky item also specified with expiretime attribute.

- **-1 :** set as sticky item
- **0 :** set as never expired item, but it can be evicted when memory runs out.
- **X <= (60 * 60 * 24 * 30) :** if value is less than 30 days old, set expiration time with "current time + X(second)"
- **X > (60 * 60 * 24 * 30) :** if value exceeds 30 days, X is acknowledge as an unix time and set the expiration time.
                              Be cautious if X is less than the current time because then it will immediately expire.  

## maxcount Attribute

This attribute is only valid for collection items and defines the maximum number of elements 
that can be stored in a single collection.

The hard limit and default(if this setting is omitted or value is set to 0) values of the maxcount attribute are as follows.

- **hard limit :** 50000
- **default value :** 4000

The reason why the hard limit of maxcount attribute needs to be small is to have the performance cost of O(small N).
Depending on the event-driven processing model, in the case of where one worker thread has to handle multiple client
requests in an asynchronous way, only if the cost of processing a request is as small as possible, 
then it is possible to minimize the effect on the execution latency.

## overflowaction Attribute

Overflow occurs when you exceed maxcount of Collection by adding elements. In this situations
actions to take are specified as follows.

- **"error"**
  - a property that can be set for all collection types.
  - a default overflow action of Set and Map collections.
  - it does not allow adding new elements and returns overflow errors.

- **"head_trim", "tail_trim"**
  - an overflow action that can only be set for List collection.
  - "tail_trim" is a default overflow action of List collection.
  - instead of allowing the addition of new elements, it removes existing elements located in the head or tail of the list.
  - when an overflow trim occurs, a trim flag indicating whether trim has occurred is not internally maintained.

- **"smallest_trim", "largest_trim"**
  - an overflow action that can only be set in B+tree collection.
  - "smallest_trim" is a default overflow action of B+tree collection.
  - instead of allowing the addition of new elements, it removes the existing element of the smallest or the largest bkey.
  - when an overflow trim occurs, a trim flag indicating whether trim has occurred is internally maintained, and
    when retrieving the trim generated bkey area, the result of response includes whether trim occurs or not.

- **"smallest_silent_trim", "largest_silent_trim"**
  - an overflow action that operates same as the "samllest_trim", "largest_trim".
  - the difference is even if an overflow occurs, a trim flag is not maintained internally, and even if the
    bkey area where trim occurred was retrieved, it does not indicate in the returned result.
  - in the application be careful about whether trimmed/not or trimmed data, those should be inspected personally.

For reference, depending on the below described maxbkeyrange attribute, even in the case of removing an element 
overflow action is referenced.

## readable Attribute

ARCUS Cache Server does not provide a command to create a collection with multiple elements atomically.
Instead, the desired collection can be created by repeatedly performing a command to add one element.
In this case, the problem is that before one collection is completed, incomplete collection can be exposed to applications.
For example, let's assume that the SNS friends list information of a user is stored in a cache in the form of a Set collection.  
If you receive a request to check the user's entire friends information, when only some of the friends' information
is actually stored in the set collection, in that case, incomplete friend information will be exposed to the application.
In order to prevent this kind of issue, a function to provide readability for the creation of the collection is required.
ARCUS provides a readable attribute for the implementation of this function.

When creating an empty collection for the first time, set the readable attribute to OFF state, to make sure that
when a retrieval request for a collection comes, it would cause the UNREADABLE error. Only after adding all the elements
to the collection, change the readable attribute back to the ON state, so that the complete collection can be retrieved by the application.

## maxbkeyrange Attribute

Only the b+tree attribute defines the maximum range of the smallest bkey and the largest bkey.
When inserting an element with a new bkey that violates the **maxbkeyrange** set in the b+tree,
depending on b+tree's overflow action policy by making an error or removing elements with a smallest/largest bkey,
it always follows the maxbkeyrange characteristics.

Element removal by the maxbkeyrange attribute is the same as explicit element removal by application request,
so it's not treated the same as trim. Eventually, only overflow trim by the maxcount attribute is treated as trim.

As an example of using the maxbkeyrange, suppose an application stores the data in a b+tree with the data generation
time as bkey and that only the last 2 days' data is maintained in the b+tree. If the time value in seconds is used as the bkey value,
the maxbkeyrange is specified as 172880(2 * 24 * 60 * 60), which corresponds to a value of 2 days match, and
in order to store only recent data, overflow action can be specified as "smallest_trim". With this specification,
each time a new data is added, data that has passed two days in the b+tree is automatically removed by
maxbkeyrange and overflow action. If there is no such function, the application should directly remove old(two days past) data.

The maxbkeyrange setting must be set according to the data type of bkey, 
and the default value when the maxbkeyrange setting is omitted or explicitly set to 0 means
unlimited maxbkeyrange, regardless of the bkey data type





