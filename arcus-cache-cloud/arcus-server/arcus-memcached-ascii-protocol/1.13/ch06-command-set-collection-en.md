## Chapter 6. SET Command

The commands for Set collection are as follows.

- [Create Set Collection: sop create](ch06-command-set-collection-en.md#sop-create-create-set-collection)
- Delete Set collection: delete (existing key-value item's delete command used as it is)

The commands for Set elements are as follows.

- [Insert Set Element: sop insert](ch06-command-set-collection-en.md#sop-insert-insert-set-element)
- [Delete Set Element: sop delete](ch06-command-set-collection-en.md#sop-delete-delete-set-element)
- [Retrieve Set Element: sop get](ch06-command-set-collection-en.md#sop-get-retrieve-set-element)
- [Check Existence of Set Element: sop exist](ch06-command-set-collection-en.md#sop-exist-check-existence-of-set-element)

## sop create (Create Set Collection)

Create an empty Set collection.

```
sop create <key> <attributes> [noreply]\r\n
* <attributes>: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<attributes\> - item attributes to be set. Please refer to [Item Attribute Description](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch03-item-attributes-en.md).
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply - if state it, response string will not be delivered.

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "CREATED"                              |  Successfully created                          |
| "EXISTS"                               |  Item with the same key string already exists  |
| "NOT_SUPPORTED"                        |  Not supported                                 |
| “CLIENT_ERROR bad command line format” |  Incorrect protocol syntax                     |
| “SERVER_ERROR out of memory”           |  Insufficient memory                           |

## sop insert (Insert Set Element)

Insert a single element into a Set collection.
You can also insert an element while creating a Set Collection. 

```
sop insert <key> <bytes> [create <attributes>] [noreply|pipe]\r\n<data>\r\n
* <attributes>: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<bytes\> - data length to insert (excluding trailing characters "\r\n")
- create \<attributes\> - when set collection does not exist, request creation of set. Please refer to [Item Attribute Description](ch03-item-attributes-en.md) for further details.
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply or pipe - if state it, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- \<data\> - data to insert. For the maximum size please refer to the [basic constraints](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch01-arcus-basic-concept-en.md#basic-constraints).

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "STORED"                               | Successfully stored                            |
| “CREATED_STORED”                       | Successfully stored (Create a collection and insert an element)                        
| “NOT_FOUND”                            | Key Miss
| “TYPE_MISMATCH”                        | Given item is not a Set collection
| “OVERFLOWED”                           | Overflow occurred
| "ELEMENT_EXISTS"                       | Element with the same data already exists (violation of set uniqueness)
| "NOT_SUPPORTED"                        | Not supported
| “CLIENT_ERROR bad command line format” | Incorrect protocol syntax
| “CLIENT_ERROR too large value”         | Data to insert is larger than the maximum size of the element value
| “CLIENT_ERROR bad data chunk”          | Data length to insert is different from \<bytes\> or does not end with "\r\n".
| “SERVER_ERROR out of memory”           | Insufficient memory
  
## sop delete (Delete Set Element) 
  
Deletes a single element from Set collection.  
  
```
sop delete <key> <bytes> [drop] [noreply|pipe]\r\n<data>\r\n  
```  

- \<key\> - key string of target item.
- \<bytes\> - data length to insert (excluding trailing characters "\r\n")
- drop - specifies to drop the set after the element is deleted and set becomes empty
- noreply or pipe - response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- \<data\> - data to delete.  
  
Response strings and their meaning are as follows.
  
| **String**                               | **Description**                                |  
| ---------------------------------------- | ---------------------------------------------- |  
| "DELETED"                                |  Successfully deleted                          |
| “DELETED_DROPPED”                        |  Successfully deleted (if the Set becomes empty, drop(delete) that set too |
| “NOT_FOUND"                              |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                      |  Element miss (no element corresponding to single index or index range)      |
| “TYPE_MISMATCH”                          |  Given item is not a Set collection            |
| "NOT_SUPPORTED"                          |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”   |  Incorrect protocol syntax                     |
| “CLIENT_ERROR too large value”           |  Data to delete is larger than the maximum size of the element value
| “CLIENT_ERROR bad data chunk”            |  Data length to delete is different from \<bytes\> or does not end with "\r\n".  
  
## sop get (Retrieve Set Element)  
  
Retrieve *N* number of elements from Set collection.

```
sop get <key> <count> [delete|drop]\r\n  
```  

- \<key\> - key string of target item
- \<count\> - specifies number of elements to retrieve. 0 means all elements.
- delete or drop - specifies whether to delete element when retrieving it and drop the set when it becomes an empty set.

When succeeded, the response string is as follows.
- \<count\> of the VALUE line indicates the number of elements retrieved.
- in the last line, if you receive on the END, DELETED, DELETED_DROPPED, it means respectively:
  - element(s) has been retrieved
  - element retrieved and deleted
  - element retrieved and deleted, the set also dropped

```
VALUE <flags> <count>\r\n
<bytes> <data>\r\n
<bytes> <data>\r\n
<bytes> <data>\r\n
...
END|DELETED|DELETED_DROPPED\r\n
```  

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                                |  
| ---------------------------------------------------- | ---------------------------------------------- |  
| “NOT_FOUND"                                          |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                                  |  Element miss (element does not exist)         |
| “TYPE_MISMATCH”                                      |  Given item is not a Set collection            |
| “UNREADABLE”                                         |  Given item is unreadable                      | 
| "NOT_SUPPORTED"                                      |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”               |  Incorrect protocol syntax                     | 
| “SERVER_ERROR out of memory [writing get response]”  |  Insufficient memory                           |

## sop exist (Check Existence of Set Element)

Checks if the specific element exists in Set collection.
  
- \<key\> - key string of target item.
- \<bytes\> - data length to insert (excluding trailing characters "\r\n")
- pipe - if state it, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
 
  
```
sop exist <key> <bytes> [pipe]\r\n<data>\r\n
```

Response strings when failed and their meaning are as follows. 
  
| **String**                               | **Description**                                |  
| ---------------------------------------- | ---------------------------------------------- |  
| “EXIST"                                  |  Success, given data exists in the set         |
| "NOT_EXIST"                              |  Success, given data does not exist in the set |
| “NOT_FOUND"                              |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                      |  Element miss (element does not exist)         |
| “TYPE_MISMATCH”                          |  Given item is not a Set collection            |
| “UNREADABLE”                             |  Given item is unreadable                      | 
| "NOT_SUPPORTED"                          |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”   |  Incorrect protocol syntax                     | 
| “CLIENT_ERROR too large value”           |  Data to delete is larger than the maximum size of the element value
| “CLIENT_ERROR bad data chunk”            |  Data length to delete is different from \<bytes\> or does not end with "\r\n"
  
  
  
  
  

