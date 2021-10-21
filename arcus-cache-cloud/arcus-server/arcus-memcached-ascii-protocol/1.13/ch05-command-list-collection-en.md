# Chapter 5. LIST Command

The commands for List collection are as follows.

- [Create List Collection: lop create](ch05-command-list-collection-en.md#lop-create-create-list-collection)
- Delete List collection: delete (existing key-value item's delete command used as it is)

The commands for List elements are as follows.

- [Insert List Element: lop insert](ch05-command-list-collection-en.md#lop-insert-insert-list-element)
- [Delete List Element: lop delete](ch05-command-list-collection-en.md#lop-delete-delete-list-element)
- [Retrieve List Element: lop get](ch05-command-list-collection-en.md#lop-get-retrieve-list-element)

## lop create (Create List Collection)

Create an empty List collection.

```
lop create <key> <attributes> [noreply]\r\n
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
| “SERVER_ERROR out of memory”           |  Insufficient memory                             |

## lop insert (Insert List Element)

Insert a single element into a List collection.
You can also insert an element while creating a List Collection. 

```
lop insert <key> <index> <bytes> [create <attributes>] [noreply|pipe]\r\n<data>\r\n
* attributes: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<index\> - specifies insert position as 0-based index.
  - 0, 1, 2, ... : indicates each element's position starting from the top of the list.
   - -1, -2, -3, ... : indicates each element's position starting from the end of the list.
- \<bytes\> - data length to insert (excluding trailing characters "\r\n")
- create \<attributes\> - when list collection does not exist, request creation of list.Please refer to [Item Attribute Description](ch03-item-attributes-en.md) for further details.
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply or pipe - if state this, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- \<data\> - data to insert. For the maximum size please refer to the [basic constraints](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch01-arcus-basic-concept-en.md#basic-constraints).

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "STORED"                               | Successfully stored                            |
| “CREATED_STORED”                       | Successfully stored (Create a collection and insert an element)                        
| “NOT_FOUND”                            | Key Miss
| “TYPE_MISMATCH”                        | Given item is not a List collection
| “OVERFLOWED”                           | Overflow occurred
| “OUT_OF_RANG"                          | Position of an index is beyond the element index range of a List, e.g. there are 10 elements and the insert position is 20.
| "NOT_SUPPORTED"                        | Not supported
| “CLIENT_ERROR bad command line format” | Incorrect protocol syntax
| “CLIENT_ERROR too large value”         | Data to insert is larger than the maximum size of the element value
| “CLIENT_ERROR bad data chunk”          | Data length to insert is different from \<bytes\> or does not end with "\r\n".
| “SERVER_ERROR out of memory”           | Insufficient memory

## lop delete (Delete List Element) 

Deletes a single element from a position of index
or multiple elements within the index range of a List.
  
```
lop delete <key> <index or "index range"> [drop] [noreply|pipe]\r\n
lop delete 명령에서 각 인자의 설명은 아래와 같다. 
```  

- \<key\> - key string of target item
- \<index or "index range"\> - index or index range of element to delete. 
  As introduced in the "lop insert" command element index specifies delete position as 0-based index
   index range is specified in the form of index1..index2
  - 0..-1: from the first element until the last element (sequence to forward)
  - 2..-2: from the third element in the beginning until the second element in the back (sequence to forward)
  - -3..-1: from the third element until the first element in the back (sequence to forward)
  - 4..2 : from the fifth element until the third element in the beginning (sequence to backward)
  - -1..0: from the last element until the first element (sequence to backward)
- drop - specifies to drop the list after the element is deleted and list becomes empty
- noreply or pipe - response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
  
Response strings and their meaning are as follows.
  
| **String**                               | **Description**                                |  
| ---------------------------------------- | ---------------------------------------------- |  
| "DELETED"                                |  Successfully deleted (only element deleted)   |
| “DELETED_DROPPED”                        |  Successfully deleted (if the List becomes empty, drop(delete) that list too |
| “NOT_FOUND"                              |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                      |  Element miss (no element corresponding to single index or index range)      |
| “TYPE_MISMATCH”                          |  Given item is not a List collection           |
| "NOT_SUPPORTED"                          |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”   |  Incorrect protocol syntax                     |

 
## lop get (Retrieve List Element)
  
Retrieve element(s) corresponding to an index or index range from the List collection.  
  
```
lop get <key> <index or "index range"> [delete|drop]\r\n  
```  

- \<key\> - key string of target item
- \<index or "index range"\> - index or index range of element to retieve. for further detailes check "lop delete" command
- delete or drop - specifies whether to delete element when retrieving it and drop the list when it becomes an empty list. 
  
 When succeeded, the response string is as follows.
- \<count\> of the VALUE line indicates the number of elements retrieved.
- in the last line, if you receive on the END, DELETED, DELETED_DROPPED, it means respectively:
  - element(s) has been retrieved
  - element retrieved and deleted
  - element retrieved and deleted, the list also dropped
  
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
| “NOT_FOUND_ELEMENT”                                  |  Element miss (no element corresponding to single index or index range)
| “TYPE_MISMATCH”                                      |  Given item is not a List collection           |
| “UNREADABLE”                                         |  Given item is unreadable                      | 
| "NOT_SUPPORTED"                                      |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”               |  Incorrect protocol syntax                     | 
| “SERVER_ERROR out of memory [writing get response]”  |  Insufficient memory                           |
  
  
 
