# Chapter 2. Collection Concept

## Collection Structure and Features
The type of collection, its structure, and its characteristics are as follows.

**List** - linked list

> It has a double-linked list structure of elements.
  While maintaining information of head and tail elements, starting from head/tail
  elements located at the specific position can be accessed in the forward/backward directions.
  There is a performance issue when approaching any element located in the middle position in a List
  with many elements, therefore it's recommended to use the List as a *queue* concept. 

**Set** - unordered set of unique value

> Set data structure is suitable for membership checking.
  Internally hash table structure is used to store an unordered set of unique values.
  To dynamically adjust the entire size of the hash table in proportion to the number of elements in a Set,
  similar to a general tree structure, Set has a hash table structure consisting of many depths. 
  
**Map** - unordered set of \<field, value\>
  
> Map data structure stores pairs of \<field, value\>.
  By ensuring the uniqueness of the field value and based on field hash table structure 
  is used to speed up the search of corresponding elements.
  To dynamically adjust the entire size of hash table in proportion to the number of elements in a Map,
  similar to a general tree structure, Map has a hash table structure consisting of many depths. 

**B+tree** - sorted map based on b+tree key

> Each element has a unique key and based on these unique keys a sorted set of elements is stored in
  a b+tree structure and a range scan of forward/backward directions is provided.
  Memory usage is minimized by using a b+tree structure that dynamically adjusts depth to the proportion of
  the number of elements. In addition, nonleaf node of b+tree contains information on the number of elements
  stored in sub-tree centered on each sub-node, so it can provide retrieval of element's position and
  position-based element retrieval functions for specific elements.

Collection items have a structure of a \<key, "collection meta info"\>. Collection meta info has the attribute 
information of the collection type and information to promptly access the elements of the collection.
For example, this applies to the head/tail element's address of a List, highest hash table address of a Set,
the top hash table structure of a Map, root node address of a b+tree. 

## Element Structure

The element structure according to the collection type is as follows.

- List/Set element : \<data\> 
  - each element has only one data
- Map element : \<field(map element key), data\>
  - in map a field for distinguishing each element is required and it does not allow duplication.
- B+tree element : \<bkey(b+tree key), eflag(element flag), data\>
  - in b+tree, to sort elements by some criteria a bkey is required, optionally, when performing bkey based scan 
    it can have an eflag for filtering a specific element, and data that is dependent on bkey and used for simple store/retrieve purposes.

 

## BKey (B+Tree Key)

There are two types of available bkey data in the B+tree collection.

- 8 bytes unsigned integer
  - values can be specified in the range of 0 ~ 18446744073709551615. Since this type is advantageous over the
    hexadecimal type in terms of performance and memory space, we recommend using this type of bkey.
 
- hexadecimal
  - specifies even number of hexadecimal strings starting with "0x" and all case letters are available.
    ARCUS Cache Server stores two hexadecimal characters as 1 byte and stores them as a variable 
    length byte array of 1~31 lengths.

The reasons for the number of bytes stored when the hexadecimal representation is correct and
when it's wrong are as follows  
  
 hexadecimal value | storage bytes | incorrect reason
 ----------------- | ------------- | ----------------
 0x34F40056        | 4 bytes       |
 0xabcd00778899    | 6 bytes       |
 34F40056          |               | no "0x" on the front
 0x34F40           |               | odd number of hexadecimal strings
 0x34F40G          |               | 'G' is not a hexadecimal character

  
Comparison operation of bkey's size(large/small): 
- if it's a value of 8 bytes unsigned integer type, it performs a simple comparison operation of two integer values,
- if it's a value of hexadecimal type, it performs comparison operation of two values with lexicographical order as follows:
  - Starting from the first byte of two hexadecimal, compare the entire size one by one in bytes,
    if there is a difference, terminate the comparison of size.
  - Between the two hexadecimal, if the two values are the same in comparison of small lengths,
    then it's considered that the hexadecimal with a longer length has a larger value.
  - if both of two hexadecimal have the same length and the values of each byte are the same,
    then it's considered that two hexadecimal values are the same.
  
## EFlag (Element Flag)
  
Eflag is a feature that currently exists only in b+tree elements.
The eflag data type can only be hexadecimal, and it follows bkey's hexadecimal expression and storage method.   

### EFlag Filter
  
Filter conditions for eflag are expressed as follows.
- (1) `compare` operation of the partial/full and specific value of eflag or
- (2) `compare` operation of the after chosen bitwise operation result that performed as some `operand` about partial/full 
       value of eflag and specific value.   
  
```java
eflag_filter: <fwhere> [<bitwop> <foperand>] <compop> <fvalue>
```

- \<fwhere\>
  - In the eflag value, the start offset to take the bitwise/compare operation is expressed in bytes.
    The length of the data to take the bitwise/compare operation is set as a length of \<fvalue\>.
    For example, if you select the entire eflag data, \<fwhere\> should be 0, and the length of \<fvalue\> 
    should be the same as the length of the entire eflag data.
  
- [\<bitwop\> \<foperand\>]
  - It specifies the bitwise operation about eflag and can be omitted.
  - If a bitwise operation is specified, the result of this operation is subject to a compare operation,
    if omitted, the eflag value itself will become the subject of the compare operation.
  - \<bitwop\> can be specified in one of the following: "&"(bitwise and), "|"(bitwise or), "^"(bitwise xor).
  - \<foperand\> is expressed in hexadecimal as an operand to take the bitwise operation.
    The length of \<foperand\> should be the same as the length of \<fvalue\> that took the compare operation.
  
- \<compop\> \<fvalue\>
  - Specifies the compare operation about the eflag.
  - \<compop\> can specify compare operation as one of the "EQ", "NE", "LT", "LE", "GT", and "GE".
    \<fvalue\> is similarly expressed in hexadecimal as a subject value to take the compare operation.
  - Can specify IN or NOT IN conditions. 
    - **IN** condition can be specified with "EQ" operation and comma-separated hexadecimal values. 
    - **NOT IN** condition can be specified with "NE" operation and comma-separated hexadecimal values. 
      In this case, the maximum number of the hexadecimal values which is separetad with a comma is only supported up to 100.
  
It is recommended to use the same length element flag for elements of one b+tree. However, if the application requires
eflag can be omitted or have eflags of different lengths for elements belonging to a b+tree. In this case,
eflag filtering can become uncertain as follows. In this situation, if the comparison of filter conditions is "NE"
it's determined as `true` and other comparison operations are determined by `false`.
  
- An element can get **eflag_filter** condition even it doesn't have the eflag.
- An element can have eflag, but may not have data of offset and length specified in the **eflag_filter** condition.
  For example, in a situation where eflag is 4 bytes, (1) there may be a case where the offset of the 
  **eflag_filter** condition is 5 or (2) the offset of the eflag_filter condition is 3 and the length is 4.
  
### EFlag Update
  
Update operation is available for eflag's partial/full value and the syntax is as follows.
Update of full eflag is replacing it with a new eflag value, update of partial eflag is 
replacing it with the result of the bitwise operation on the partial data of eflag.
  
```java
eflag_update: [<fwhere> <bitwop>] <fvalue>  
```  

- [\<fwhere\> \<bitwop\>]
  - Specifies only when eflag is partially updated.
  - \<fwhere\> represented in bytes the start offset of the data to be partially updated in the eflag.
    in this case, the length of the data to be partially updated is set by the length of specified
    \<fvalue\> at the end.
  - \<bitwop\> is a bitwise operation to be taken on data to partially update and it may be specified as
    one of the following: "&"(bitwise and), "|"(bitwise or), "^"(bitwise xor). 
  
- \<fvalue\>
  - Specifies new value to be update.
  - when omitting \<fwhere\> and \<bitwop\>, you can update entire data with \<fvalue\>.
    When <\fwhere\> and \<bitwop\> are specified for partial update, \<fvalue\> is used as an operand to take 
    bitwise operation for eflag partial data and the result of the bitwise operation is updated
    as a new value of eflag.
  
The existing eflag value can be deleted and changed to the state of no eflag.
In order to this this, omit \<fwhere\> and \<bitwop\> and set \<fvalue\>'s value to 0.
  
  
  
  
  
  
  
  
  
