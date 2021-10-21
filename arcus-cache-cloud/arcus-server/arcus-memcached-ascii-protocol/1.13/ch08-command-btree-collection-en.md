# Chapter 8. B+Tree Command

The commands for B+tree collection are as follows.

- [Create B+tree Collection: bop create](ch08-command-btree-collection-en.md#bop-create-create-btree-collection)
- Delete B+tree collection: delete (existing key-value item's delete command used as it is)

The commands for B+tree elements are as follows.

- [Insert/Replace B+tree Element: bop insert/upsert](ch08-command-btree-collection-en.md#bop-insertupsert-insertreplace-btree-element)
- [Update B+tree Element: bop update](ch08-command-btree-collection-en.md#bop-update-update-btree-element)
- [Delete B+tree Element: bop delete](ch08-command-btree-collection-en.md#bop-delete-delete-btree-element)
- [Retrieve B+tree Element: bop get](ch08-command-btree-collection-en.md#bop-get-retrieve-btree-element)
- [B+Tree Element Count: bop count](ch08-command-btree-collection-en.md#bop-count-btree-element-count)
- [B+Tree Element Value Increment/Decrement: bop incr/decr](ch08-command-btree-collection-en.md#bop-incrdecr-btree-element-value-incrementdecrement)

The ARCUS Cache Server specifically supports retrieval functions for multiple b+trees, which are as follows.

- [The function to perform a retrieval of multiple b+trees at once with one command: bop mget](ch08-command-btree-collection-en.md#bop-mget-btree-multiple-get)
- [The function to obtain the final result by sort-merging elements several b+trees satisfying the retrieval condition in: bop smget](ch08-command-btree-collection-en.md#bop-smget-btree-sort-merge-get)

Besides the bkey-based element retrieval function, ARCUS Cache Server supports b+tree position-based element retrieval function.
Position of a particular element is referred to location information of that element in the b+tree and
based on alignment(ASC or DESC) criteria of bkeys, it indicates the number of position of located elements.
B+tree position is expressed by a 0-based index. For example, if there are _N_ elements in the b+tree, 
then the index is expressed from 0 to _N-1_.

The b+tree position-related commands provided by the ARCUS Cache Server are as follows.

- [A function to retrieve a specific position of bkey in b+tree: bop position](ch08-command-btree-collection-en.md#bop-position-retrieve-btree-position)
- [A function to retrieve element(s) corresponding to the position or position range in a b+tree: bop gbp(get by position)](ch08-command-btree-collection-en.md#bop-gbp-btree-get-by-position)
- [A function to retrieve the position and element of a specific bkey along with its peripheral(front/back position) elements in b+tree: bop pwg(position with get)](ch08-command-btree-collection-en.md#bop-pwg-btree-find-position-with-get-version-180)

One such example that requires b+tree position-based retrieval is a ranking system.
In the ranking system specific score is set as bkey to store the corresponding element
and based on the highest/lowest score retrieval there are many cases that find elements corresponding 
to the number of positions or the range of positions.

## bop create (Create B+tree Collection)

Create an empty b+tree collection.

```
bop create <key> <attributes> [noreply]\r\n
* attributes: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<attributes\> - item attributes to be set. Please refer to [Item Attribute Description](ch03-item-attributes-en.md).
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply - if state this, response string will not be delivered.

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "CREATED"                              |  Successfully created                          |
| "EXISTS"                               |  Item with the same key string already exists  |
| "NOT_SUPPORTED"                        |  Not supported                                 |
| “CLIENT_ERROR bad command line format” |  Incorrect protocol syntax                     |
| “SERVER_ERROR out of memory”           |  Insufficient memory                           | 

## bop insert/upsert (Insert/Replace B+tree Element)

As a command to add one element to the B+tree collection,
(1) a `bop insert` command to insert a single element and 
(2) if there is no element with the bkey that is currently inserted, insert the current element and
if there is an element with the current bkey, with a `bop upsert` command replace it with the current element.
With these commands, you can add an element while creating a b+tree collection.

```
bop insert <key> <bkey> [<eflag>] <bytes> [create <attributes>] [noreply|pipe|getrim]\r\n<data>\r\n
bop upsert <key> <bkey> [<eflag>] <bytes> [create <attributes>] [noreply|pipe|getrim]\r\n<data>\r\n
* attributes: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<bkey\> - bkey of element to insert
- \<eflag\> - optional flag of element to insert.
- \<bytes\> and \<data\> - data and data length of element to insert. For the maximum size please refer to the [basic constraints](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch01-arcus-basic-concept-en.md#basic-constraints).
- create \<attributes\> - when b+tree collection does not exist, request creation of b+tree. Please refer to [Item Attribute Description](ch03-item-attributes-en.md) for further details.
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply or pipe - if state this, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- getrim - gets trimmed element information when adding a new element causes _overflow trim_ due to the maxcount constraints.

In case of trimmed element information is returned, the response string is as follows.

```
VALUE <flags> <count>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
END\r\n
```

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "STORED"                               | Successfully stored  (only element inserted)
| "CREATED_STORED"                       | Successfully stored (create a collection and insert an element) 
| "REPLACED"                             | Successfully replaced (element replaced)
| "NOT_FOUND"                            | Key Miss
| "TYPE_MISMATCH"                        | Given item is not a B+tree collection
| "BKEY_MISMATCH"                        | bkey type to insert is different from bkey type of target b+tree collection 
| "OVERFLOWED"                           | Overflow occurred
| "OUT_OF_RANGE"                         | Insert is failed due to the new element insertion violates maxcount or maxbkeyrange constraints and bkey value of that element is automatically deleted by overflowaction. (e.g. in the smallest_trim situation while bkey value of the newly inserted element is less than the smallest bkey of b+tree, _maxcount_ number of elements already exist or it's out of maxbkeyrange.)
| "ELEMENT_EXISTS"                       | Element with the same bkey already exists
| "NOT_SUPPORTED"                        | Not supported
| "CLIENT_ERROR bad command line format" | Incorrect protocol syntax
| "CLIENT_ERROR too large value"         | Data to insert is larger than the maximum size of the element value
| "CLIENT_ERROR bad data chunk"          | Data length to insert is different from \<bytes\> or does not end with "\r\n".
| "SERVER_ERROR out of memory"           | Insufficient memory 

## bop Update (Update B+tree Element)

Update eflag and/or data of an element in B+tree collection. Currently, update operation for multiple elements is not supported.

```
bop update <key> <bkey> [<eflag_update>] <bytes> [noreply|pipe]\r\n[<data>\r\n]
* eflag_update : [<fwhere> <bitwop>] <fvalue>
```

- \<key\> - key string of target item
- \<bkey\> - bkey of a target element
- \<eflag_update\> - specifies eflag update. Please refer to the eflag update in [Collection Basic Concept](ch02-collection-items-en.md).
-  \<bytes\> and \<data\> - data and data length of element to update. If you dont want to update data, set \<bytes\> to `-1` and omit the \<data\>.
- noreply or pipe - if state this, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "UPDATED"                              | Successfully updated                     
| vNOT_FOUND"                            | Key Miss
| "NOT_FOUND_ELEMENT"                    | Element Miss (no element fount to update) 
| "TYPE_MISMATCH"                        | Given item is not a B+tree collection
| "BKEY_MISMATCH"                        | bkey type given as the command factor is different from bkey type of target b+tree collection
| "EFLAG_MISMATCH"                       | \<eflag_update\> cannot be applied to the eflag value of the corresponding element (e.g. eflag to update does not exist or even if it does, it does not have the data of the specified under the \<eflag_update\> condition)
| "NOTHING_TO_UPDATE"                    | Neither the eflag update nor data update is specified
| "NOT_SUPPORTED"                        | Not supported
| "CLIENT_ERROR bad command line format" | Incorrect protocol syntax
| "CLIENT_ERROR too large value"         | Data to update is larger than the maximum size of the element value
| "CLIENT_ERROR bad data chunk"          | Data length to update is different from \<bytes\> or does not end with "\r\n".
| "SERVER_ERROR out of memory"           | Insufficient memory 

## bop delete (Delete B+tree Element)

Deletes _N_ number of elements that meet a bkey or bkey range condition and eflag filter condition from B+tree collection. 

```
b+tree collection에서 하나의 bkey 또는 bkey range 조건과 eflag filter 조건을 만족하는 N 개의 elements를 삭제한다.
```

- \<key\> - key string of target item.
- \<bkey or "bkey range"\> - retrieval condition of bkey or bkey range. bkey range is expressed in the form of "bkey1..bkey2"
- \<eflag_filter\> - eflag filter condition.  Please refer to eflag filter in [Collection Basic Concept](ch02-collection-items-en.md).
- \<count\> - specifies number of elements to delete
- drop - specifies whether to drop the b+tree after the element is deleted and b+tree becomes empty
- noreply or pipe - response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).  

Response strings and their meaning are as follows.
  
| **String**                               | **Description**                                |  
| ---------------------------------------- | ---------------------------------------------- |  
| "DELETED"                                | Successfully deleted  (only element deleted) 
| "DELETED_DROPPED"                        | Successfully deleted (if the b+tree becomes empty, drop collection too
| "NOT_FOUND"                              | Key Miss                                      
| "NOT_FOUND_ELEMENT"                      | Element Miss (no element found to delete) 
| "TYPE_MISMATCH"                          | Given item is not a b+tree collection
| "BKEY_MISMATCH"                          | bkey type given as the command factor is different from bkey type of target b+tree collection
| "NOT_SUPPORTED"                          |  Not Supported                                 |
| "CLIENT_ERROR bad command line format"   |  Incorrect protocol syntax                     |

## bop get (Retrieve B+tree Element)

After skipping the offset in the elements retrieve _count_ number of elements that meet a bkey or bkey range condition
and eflag filter condition in B+tree collection.

```
bop get <key> <bkey or "bkey range"> [<eflag_filter>] [[<offset>] <count>] [delete|drop]\r\n
* <eflag_filter> : <fwhere> [<bitwop> <foperand>] <compop> <fvalue>
```

- \<key\> - key string of target item.
- \<bkey or "bkey range"\> - retrieval condition of a bkey or bkey range. bkey range is expressed in the form of "bkey1..bkey2"
- \<eflag_filter\> - eflag filter condition.  Please refer to eflag filter in [Collection Basic Concept](ch02-collection-items-en.md).
- [\<offset\>] \<count\> - number of elements to skip and actual number of elements retrieve that meets the retrieval condition.
- delete or drop - specifies whether to delete the element after its retrieval and if b+tree becomes empty after deletion whether to drop b+tree. 

When succeeded, the response string is as follows.

```
VALUE <flags> <count>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
…
END|TRIMMED|DELETED|DELETED_DROPPED\r\n
```

Description of the above response string is as follows

- \<count\> of the VALUE line indicates the number of retrieved elements. Starting from the following lines retrieve   
  bkey, flag, data of each element.
- In the last line, as a retrieval reference you will receive one of the END, TRIMMED, DELETED, DELETED_DROPPED, and their meaning are as follows:
  - END : refers to each element's retrieval only.
  - DELEETED : refers to the element's retrieval and its deletion.
  - DELEETD_DROPPED : refers to the element's retrieval and its deletion, and drop of b+tree collection if it's empty.
  - TRIMMED : has a special indication. It refers that while retrieving an element,
    the element retrieval condition overlapped with trimmed bkey area by overflowaction of b+tree. 
    According to said, this lets the application know that although elements meet the retrieval condition, 
    they got overflow trim and won't be returned.
    
If required, the application can retrieve again the remaining elements that haven't been retrieved from back-end storage.
Please be advised that, if `smallest_silent_trim` or `largest_silent_trim` is used as an overflow action,
then whether trim occurred or not, inside the b+tree collection it does not maintain and it will not inform 
the trim occurrence with TRIMMED. In this case, the inspection of trim occurrence must be performed from the application.     

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                                |  
| ---------------------------------------------------- | ---------------------------------------------- |  
| "NOT_FOUND"                                          | Key Miss                                     
| "NOT_FOUND_ELEMENT"                                  | Element Miss  (there is no element that meets the retrieval condition)|
| "OUT_OF_RANGE"                                       | There is no element that meets the retrieval conditions and also it indicates that the given bkey range has overlapped with the trimmed bkey area due to the overflowaction of b+tree.
| "TYPE_MISMATCH"                                      | Given item is not a b+tree collection          
| "BKEY_MISMATCH"                                      | bkey type given as the command factor is different from bkey type of target b+tree collection
| "UNREADABLE"                                         | Given item is unreadable                      
| "NOT_SUPPORTED"                                      | Not Supported                                 
| "CLIENT_ERROR bad command line format"               | Incorrect protocol syntax                     
| "SERVER_ERROR out of memory [writing get response]"  | Insufficient memory                            

## bop count (B+Tree Element Count)

Retrieve number of elements that meet a bkey or bkey range condition and eflag filter condition in B+tree collection

```
bop count <key> <bkey or "bkey range"> [<eflag_filter>]\r\n
* <eflag_filter> : <fwhere> [<bitwop> <foperand>] <compop> <fvalue>
```

- \<key\> - key string of target item.
- \<bkey or "bkey range"\> - retrieval condition of a bkey or bkey range. bkey range is expressed in the form of "bkey1..bkey2"
- \<eflag_filter\> - eflag filter condition.  Please refer to eflag filter in [Collection Basic Concept](ch02-collection-items-en.md).

When succeeded, the response string is as follows.

```
COUNT=<count>
```

Response strings when failed and their meaning are as follows.

| **String**                               | **Description**                           |  
| ---------------------------------------- | ------------------------------------------|  
| "NOT_FOUND"                              | Key Miss                                  |
| "TYPE_MISMATCH"                          | Given item is not a B+tree collection     |
| "BKEY_MISMATCH"                          | bkey type given as the command factor and bkey type of the target b+tree are different |
| "UNREADABLE"                             | Given item is unreadable                  | 
| "NOT_SUPPORTED"                          | Not Supported                             |
| "CLIENT_ERROR bad command line format"   | Incorrect protocol syntax                 | 


## bop incr/decr (B+Tree Element Value Increment/Decrement)

Increases or decreases data of a specific element in B+tree collection and returns the increased or decreased data.
This command is similar to the incr/decr command of key-value items, to perform this command data of the b+tree element
should be numeric data that can be increased or decreased.

```
bop incr <key> <bkey> <delta> [<initial> [<eflag>]] [noreply|pipe]\r\n
bop decr <key> <bkey> <delta> [<initial> [<eflag>]] [noreply|pipe]\r\n
```

- \<key\> - key string of target item
- \<bkey\> - bkey of a target element
- \<delta\> - delta value to increment/decrement, must have a value greater than 0
  - when 64bit unsigned integer overflow with the increment operation, it is wrapped around and set to the residual value
  - when 64bit unsigned integer underflow with the decrement operation, a new value is unconditionally set to zero.
- \<initial\> - when there is no target element, a new element is created and set as an initial value
  - \<eflag\> - can be specified when eflag value is given to a new element

When succeeded, the response string is as follows.

```
<value>\r\n
```

Response strings when failed and their meaning are as follows.

| **String**                                                     | **Description**                           |  
| -------------------------------------------------------------- | ------------------------------------------|  
| "NOT_FOUND"                                                    | Key Miss                                  |
| "NOT_FOUND_ELEMENT"                                            | Element Miss                              |
| "TYPE_MISMATCH"                                                | Given item is not a B+tree collection     |
| "BKEY_MISMATCH"                                                | bkey type given as the command factor and bkey type of the target b+tree are different |
| "OVERFLOWED"                                                   | Overflow occurred
| "OUT_OF_RANGE"                                                 | Insert is failed due to the new element insertion violating maxcount or maxbkeyrange constraints and bkey value of that element is automatically deleted by overflowaction. (e.g. in the smallest_trim situation while bkey value of the newly inserted element is less than the smallest bkey of b+tree, _maxcount_ number of elements already exist or it's out of maxbkeyrange.)
| "NOT_SUPPORTED"                                                | Not Supported                             |
| "CLIENT_ERROR cannot increment or decrement non-numeric value" | Data of the corresponding element is not numeric type.
| "CLIENT_ERROR bad command line format"                         | Incorrect protocol syntax                 | 
| "SERVER_ERROR out of memory [writing get response]"            | Insufficient memory                       |  

## bop mget (B+Tree Multiple Get)

Retrieve elements all at once with uniform retrieval conditions(bkey range and eflag filter) for multiple b+trees. 
Since the uniform retrieval condition is used for multiple b+trees, all target b+trees must have the same bkey type.
Additionally, it is recommended to use data of the same nature for the eflags.

```
bop mget <lenkeys> <numkeys> <bkey or "bkey range"> [<eflag_filter>] [<offset>] <count>\r\n
<”space separated keys”>\r\n
* <eflag_filter> : <fwhere> [<bitwop> <foperand>] <compop> <fvalue>
```

- \<"space separated keys"\> -  a key list of target b+trees seperated by space('')
  - for lower compatibilities (less than 1.10.X version) comma(,) si also supported but it's not recommended.
- \<lenkeys\> and \<numkeys\> - length and key count of key list string
- \<bkey or "bkey range"\> - retrieval condition of a bkey or bkey range. bkey range is expressed in the form of "bkey1..bkey2".
- \<eflag_filter\> - eflag filter condition.  Please refer to eflag filter in [Collection Basic Concept](ch02-collection-items-en.md).
- [\<offset\>] \<count\> - number of elements to skip and actual number of elements retrieve that meets the retrieval condition.

bop `mget` command has the following constraints for the O(small N) execution rule. 
- maximum number of the keys can be specified in the key list: 200
- maximum value of count: 50

When succeeded, the response string is as follows.

```
VALUE <key> <status> [<flags> <ecount>]\r\n
[ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n
 ...
 ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n]
VALUE <key> <status> [<flags> <ecount>]\r\n
[ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n
 ...
 ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n]

...

VALUE <key> <status> [<flags> <ecount>]\r\n
[ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n
 ...
 ELEMENT <bkey> [<eflag>] <bytes> <data>\r\n]
END\r\n
```

There is a VALUE line per each target key to retrieve and it reveals the target key string and retrieval status.
Retrieval status is one of the following below. 
Please check the meaning of each in the bop get command's response string.

- OK : Normal retrieval
- TRIMMED : normal retrieval but, there is trimmed element
- NOT_FOUND
- NOT_FOUND_ELEMENT
- OUT_OF_RANGE
- TYPE_MISMATCH
- BKEY_MISMATCH
- UNREADABLE

If the retrieval status is normal "OK" and "TRIMMED",  
it specifies a value of flags set to that key and retrieval element count,
and starting from the following lines bkey optional flag of each retrieval element, data length and data itself specified.
Regarding to the remaining retrieval statut, in the case of the failed retrieval of an element from the given key,
retrieved element information is omitted included flags and counts details.

Response strings when failed and their meaning are as follows.

| **String**                                            | **Description**                           |  
| ----------------------------------------------------- | ------------------------------------------|  
| "NOT_SUPPORTED                                        | Not supported                             |
| "CLIENT_ERROR bad command line format"                | Incorrect protocol syntax                 |
| "CLIENT_ERROR bad data chunk"                         | Data length to insert is different from \<bytes\> or does not end with "\r\n".
| "CLIENT_ERROR bad value"                              | Violation of the constraints of bop mget command
| "SERVER_ERROR out of memory [writing get response]"   | Insufficient memory                       |

## bop smget (B+Tree Sort Merge Get)

Retrieve elements that meet both bkey range condition and eflag filter condition in a form of sort-merge and
return `count` number of elements from multiple b+trees. In other words, think about as if multiple b+trees composes one large b+tree,
which is the same as the element retrieval function.

`smget` operation is a process of overlapping the range query with the trim area of any b+tree, with below shown two modes operations.

1. Existing `smget` Operation (works on lower 1.8.X versions)
   - Even if any trimmed b+tree exists the first element that meets the `smget` retrieval condition, it sends OUT_OF_RANGE response.
   - In the absence of OUT_OF_RANGE, if the second subsequent element meets trimmed b+tree that satisfies the retrieval condition while
    performing `smget`, elements retrieved up to the point that is sent as final elements results and the `smget` performance
    status is TRIMMED. In this case, the application retrieves the elements of the trim area in the DB for all keys
    and reflected in the result of the smget.

2. New `smget` Operation (works on above 1.9.0 versions)
   - The b+tree corresponding to the existing OUT_OF_RANGE is classified as missed keys and `smget` is continued on the remaining b+trees.
    Therefore, in the application, elements can be retrieved from the DB, only for missed keys, and reflected in the final `smget` results.
   - Even if a second subsequent element that meets the `smget` lookup criteria exists, `smget` is not stopped at that point, but rather
    continues until such b+trees are classified as trimmed keys and the desired number of elements are found. 
    Therefore, the application can only retrieve trim elements in the DB and reflect them in the final `smget` results.
   - It supports a unique lookup function for bkey key. In addition to duplicated queries that allow duplicate bkeys to be viewed, 
    it also removes duplicate bkeys and supports unique bkeys that only query unique bkeys.
   - Removed the offset feature in the retrieval condition.

It is recommended that the offset value is always used at 0, even if you use the existing `smget` operation.
Missed keys exist in `smget` with a positive offset, and the reason is if the retrieval of DB for missed keys has skipped elements
by offset then the application couldn't be able to process offset accurately.
If you want to additionally retrieve following the previous retrieval results, 
then based on previously retrieved bkey values, bkey range can be re-adjusted and used.

```
bop smget <lenkeys> <numkeys> <bkey or "bkey range"> [<eflag_filter>] <count> [duplicate|unique]\r\n
<"space separated keys">\r\n
* <eflag_filter> : <fwhere> [<bitwop> <foperand>] <compop> <fvalue>
```

- \<"space separated keys"\> -  a key list of target b+trees seperated by space('')
  - for lower compatibilities (lower than 1.10.X version) comma(,) is also supported but it's not recommended.
- \<lenkeys\> and \<numkeys\> - length and key count of key list string
- \<bkey or "bkey range"\> - retrieval condition of a bkey or bkey range. bkey range is expressed in the form of "bkey1..bkey2".
- \<eflag_filter\> - eflag filter condition.  Please refer to eflag filter in [Collection Basic Concept](ch02-collection-items-en.md).
- \<count\> - number of elements to retrieve
- [duplicate | unique] - specifies `smget` operation method
  - if omitted this, then the previous `smget` operation setting is used
  - if specified, then a new `smget` operation is executed. 
    - Duplicate allows duplication of bkey, 
    - Unique removed duplicated bkey. 

bop `smget` command has the following constraints for the O(small N) execution rule. 
- maximum number of the keys can be specified in the key list: 10,000
- maximum value of count: 2,000

When succeeded in the existing `smget` operation, the response string is as follows.

```
VALUE <ecount>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
...
MISSED_KEYS <kcount>\r\n
<key>\r\n
<key>\r\n
…
END|DUPLICATED|TRIMMED|DUPLICATRED_TRIMMED\r\n
```

Description of the above response string is as follows

- VALUE: represents retrieved elements
  - Element information consists of a key string of b+tree to which retrieved element belongs, flags information, bkey, optional eflag, and data of that element.
  - Element information is sorted based on bkey and elements with the same bkey are sorted based on key string.
- MISSED_KEYS: uninvolved key list in the `smget` retrieval and its cause
  - \<key\>: key string that was not involved in the `smget` operation.
- Last line indicates the end of the `smget` operation's response string.
  - END : no duplicated bkey on the retrieval result
  - DUPLICATED: there is a dublicated bkey on the retrieval result
  - TRIMMED: detect the b+tree where query range overlapped with the trim area
  - DUPLICATED_TRIMMED: represents both DUPLICATED and TRIMMED states.

When succeeded in the new `smget` operation, response string is as follows.

```
ELEMENTS <ecount>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
<key> <flags> <bkey> [<eflag>] <bytes> <data>\r\n
...
MISSED_KEYS <kcount>\r\n
<key> <cause>\r\n
<key> <cause>\r\n
…
TRIMMED_KEYS <kcount>\r\n
<key> <bkey>\r\n
<key> <bkey>\r\n
…
END|DUPLICATED\r\n

* <cause> = NOT_FOUND | UNREADABLE | OUT_OF_RANGE
```

Description of above response string is as follows

- ELEMENTS: retrieved element
  - element information consists of a key string of b+tree to which retrieved element belongs, flags information, bkey, optional eflag, and data of that element.
  - element information is sorted based on bkey and elements with the same bkey are sorted based on key string.
- MISSED_KEYS: uninvolved key list in the `smget` retrieval and its cause
  - \<key\>: key string that was not involved in the `smget` operation.
  - \<cause\>: cause of uninvolvement in `smget` operation
    - NOT_FOUND: that key does not exist in the cache
    - UNREADABLE: that key is unreadable
    - OUT_OF_RANGE: beginning of the bkey range overlaps with the trim area of same key
- TRIMMED_KEYS: key list in which trim occured at the end of the `smget` query range
  - \<key\>: key string that trim occured.
  - \<bkey\>: last bkey right before trim.
  - trimmed keys information sorted based on bkey.
- Last line indicates the end of the `smget` operation's response string.
  - END: no duplicated bkey in the retrieval result.
  - DUPLICATED: there is duplicated bkey in the retrieval result.

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                           |  
| ---------------------------------------------------- | ------------------------------------------|  
| "TYPE_MISMATCH"                                      | Given item is not a B+tree collection     |
| "BKEY_MISMATCH"                                      | bkey types of b+tree in the `smget` operation are different from each other
| "OUT_OF_RANGE"                                       | Failure response string that can only occur in the existing `smget` operation
| "NOT_SUPPORTED"                                      | Not Supported                             |
| "CLIENT_ERROR bad command line format"               | Incorrect protocol syntax                 | 
| "CLIENT_ERROR bad data chunk"                        | There is duplicated key in the given key list or the length of \<lenkeys\>  is different from length of the given key list or does not end with "\r\n".
| "CLIENT_ERROR bad value"                             | Violation of the constraints of bop mget command
| "SERVER_ERROR out of memory [writing get response]"  | Insufficient memory                       |

## bop position (Retrieve B+Tree Position)

Retrieve a specific element position in the b+tree collection.
As  a piece of position information in the b+tree, based on bkey alignment(ASC or DESC) element position 
represents a number of the position of a located element in the index from 0 to _N-1_.  

```
bop position <key> <bkey> <order>\r\n
* <order> = asc | desc
```

- \<key\> - key string of target item.
- \<bkey\> - bkey of a target element
- \<order\> - indicates what position to get based on certain bkey alignment

When succeeded, the response string is as follows.

```
POSITION=<position>\r\n
```

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                           |  
| ---------------------------------------------------- | ------------------------------------------|  
| "NOT_FOUND"                                          | Key Miss                                  |
| "NOT_FOUND_ELEMENT"                                  | Element Miss                              |
| "TYPE_MISMATCH"                                      | Given item is not a B+tree collection     |
| "UNREADABLE"                                         | Given item is unreadable                      
| "NOT_SUPPORTED"                                      | Not Supported                                 
| "CLIENT_ERROR bad command line format"               | Incorrect protocol syntax                     
| "SERVER_ERROR out of memory [writing get response]"  | Insufficient memory 

## bop gbp (B+Tree Get By Position)

Retrieve position-based elements from a b+tree collection.

```
bop gbp <key> <order> <position or "position range">\r\n
* <order> = asc | desc
```

- \<key\> - key string of target item.
- \<order\> - indicates what position to get based on certain bkey alignment
- \<position or "position range"\> - a position or position range  of elements to retrieve. Position range is expressed in the form of "position1..position2"

When succeeded, response string is as follows. Please check the response string of bop get for a detailed description.

```
VALUE <flags> <count>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
…
END\r\n

```

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                           |  
| ---------------------------------------------------- | ------------------------------------------|  
| "NOT_FOUND"                                          | Key Miss                                  |
| "NOT_FOUND_ELEMENT"                                  | Element Miss                              |
| "TYPE_MISMATCH"                                      | Given item is not a B+tree collection     |
| "UNREADABLE"                                         | Given item is unreadable                      
| "NOT_SUPPORTED"                                      | Not Supported                                 
| "CLIENT_ERROR bad command line format"               | Incorrect protocol syntax                     
| "SERVER_ERROR out of memory [writing get response]"  | Insufficient memory 

## bop pwg (B+Tree Find Position with Get [version 1.8.0])

While retrieving the position of a specific bkey in the b+tree collection,
retrieve _N_ number of elements at once from front and back (peripheral) positions including elements belonging to that bkey.

```
bop pwg <key> <bkey> <order> [<count>]\r\n
* <order> = asc | desc
```

- \<key\> - key string of target item
- \<bkey\> - bkey of a target element
- \<order\> - indicates what position to get based on certain bkey alignment
- \<count\> - indicates how many elements to retrieve from retrieved position's front and back respectively (**maximum value is limited to 100**)
- 0: return only element on the retrieved position
- if it is positive, besides to the element of the retrieved position, from the front and back of that position retrieve this number of elements respectively.

When succeeded, the response string is as follows.

```
VALUE <position> <flags> <count> <index>\r\n
<bkey> [<eflag>] <bytes> <data>\r\n
...
<bkey> [<eflag>] <bytes> <data>\r\n
END\r\n
```

The meaning of each value in the VALUE line is as follows.
In the below lines the expression of element value is same as in in the case of bop get.

- \<position\>: position of a given bkey
- \<flags\>: property value of b+tree item flags
- \<count\>: total element count to retrieve
- \<index\>: element position (0-based index) of given bkey in the entire element list
  - if only retrieve the position of given bkey and element, then count becomes 1 and its index will be 0. 
  - addition to the position of given bkey and element, if retrieve 10 elements in bidirectional ways, and if there are 5 elements 
    in front of that position and 10 elements at the end of it, then count becomes (5+1+10)=16 and index will be 5.

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                           |  
| ---------------------------------------------------- | ------------------------------------------|  
| "NOT_FOUND"                                          | Key Miss                                  |
| "NOT_FOUND_ELEMENT"                                  | Element Miss                              |
| "TYPE_MISMATCH"                                      | Given item is not a B+tree collection     |
| "BKEY_MISMATCH"                                      | bkey type given as the command factor and bkey type of the target b+tree are different |
| "UNREADABLE"                                         | Given item is unreadable                  | 
| "NOT_SUPPORTED"                                      | Not Supported                             |
| "CLIENT_ERROR bad command line format"               | Incorrect protocol syntax                 | 
| "SERVER_ERROR out of memory [writing get response]"  | Insufficient memory                       |  




