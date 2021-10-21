# Chapter 7. MAP Command

The commands for Set collection are as follows.


- [Create Map Collection: mop create](ch07-command-map-collection-en.md#mop-create-create-map-collection)
- Delete Map collection: delete (existing key-value item's delete command used as it is)

The commands for Map elements are as follows.

- [Insert Map Element: mop insert](ch07-command-map-collection-en.md#mop-insert-insert-map-element)
- [Update Map Element: mop update](ch07-command-map-collection-en.md#mop-update-update-map-element)
- [Delete Map Element: mop delete](ch07-command-map-collection-en.md#mop-delete-delete-map-element)
- [Retrieve Map Element: mop get](ch07-command-map-collection-en.md#mop-get-retrieve-map-element)

## mop create (Create Map Collection)

Create an empty Map collection.

```
mop create <key> <attributes> [noreply]\r\n
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

## mop insert (Insert Map Element)

Insert one field element into a Map collection.
You can also insert an element of \<field, value\> while creating a Map Collection. 

```
mop insert <key> <field> <bytes> [create <attributes>] [noreply|pipe]\r\n<data>\r\n
* <attributes>: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable]
```

- \<key\> - key string of target item.
- \<field\> - field string of element to insert
- \<bytes\> - data length of element to insert (excluding trailing characters "\r\n")
- create \<attributes\> - when map collection does not exist, request creation of map. Please refer to [Item Attribute Description](ch03-item-attributes-en.md) for further details.
  - unreadable - if state this, means the READABLE attribute is set to OFF.
- noreply or pipe - if state it, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- \<data\> - data to insert. For the maximum size please refer to the [basic constraints](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch01-arcus-basic-concept-en.md#basic-constraints).

Response strings and their meaning are as follows.

| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "STORED"                               | Successfully stored  (field, element inserted  |
| "CREATED_STORED"                       | Successfully stored (create a collection and insert an element)                        
| "NOT_FOUND"                            | Key Miss
| "TYPE_MISMATCH"                        | Given item is not a Map collection
| "OVERFLOWED"                           | Overflow occurred
| "ELEMENT_EXISTS"                       | Element with the same data already exists (violation of map field uniqueness)
| "NOT_SUPPORTED"                        | Not supported
| "CLIENT_ERROR bad command line format" | Incorrect protocol syntax
| "CLIENT_ERROR too large value"         | Data to insert is larger than the maximum size of the element value
| "CLIENT_ERROR bad data chunk"          | Data length to insert is different from \<bytes\> or does not end with "\r\n".
| "CLIENT_ERROR invalid prefix name"     | Invalid (non existent) prefix name  
| "SERVER_ERROR out of memory"           | Insufficient memory 

## mop update (Update Map Element)

Update an element of one field in Map collection. Currently, update operation for multiple fields is not supported. 

```
mop insert <key> <field> <bytes> [create <attributes>] [noreply|pipe]\r\n<data>\r\n
* <attributes>: <flags> <exptime> <maxcount> [<ovflaction>] [unreadable] 
```  
  
- \<key\> - key string of target item.
- \<field\> - field string of a target element 
- \<bytes\> - data length to insert (excluding trailing characters "\r\n")
- noreply or pipe - if state it, response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).
- \<data\> - data to insert. For the maximum size please refer to the [basic constraints](https://github.com/jam2in/arcus-docs/blob/master/docs/english-manual/arcus-server/ARCUS-Server-Ascii-Protocol/1.13/ch01-arcus-basic-concept-en.md#basic-constraints).

Response strings and their meaning are as follows.
  
| **String**                             | **Description**                                |  
| -------------------------------------- | ---------------------------------------------- |  
| "UPDATED"                              | Successfully updated  (element updated                      
| “NOT_FOUND”                            | Key Miss
| "NOT_FOUND_ELEMENT"                    | Field Miss 
| “TYPE_MISMATCH”                        | Given item is not a Map collection
| "ELEMENT_EXISTS"                       | Element with the same data already exists (violation of map field uniqueness)
| "NOT_SUPPORTED"                        | Not supported
| “CLIENT_ERROR bad command line format” | Incorrect protocol syntax
| “CLIENT_ERROR too large value”         | Data to update is larger than the maximum size of the element value
| “CLIENT_ERROR bad data chunk”          | Data length to update is different from \<bytes\> or does not end with "\r\n".
| “SERVER_ERROR out of memory”           | Insufficient memory 
  
## mop delete (Delete Map Element) 
  
Deletes corresponding elements of given one or more field names from Map collection.    
  
```  
mop delete <key> <lenfields> <numfields> [drop] [noreply|pipe]\r\n
[<"space separated fields">]\r\n
```  

- "space separated fields" - divided by space(' ') and field list of the target map
  - comma(,) is also supported for lower compatibility (lower 1.10.X versions), but is not recommended.
- \<key\> - key string of target item.
- \<lenfields\> and \<numfields\> - represents length of field list string and number of fields. 0 means all fields, elements.
- drop - specifies whether to drop the map after the field, the element is deleted and the map becomes empty
- noreply or pipe - response string will not be delivered. For the use pipe please refer to [Command Pipelining](ch09-command-pipelining-en.md).  
  
Response strings and their meaning are as follows.
  
| **String**                               | **Description**                                |  
| ---------------------------------------- | ---------------------------------------------- |  
| "DELETED"                                |  Successfully deleted  (deleted all or partial field, elements) 
| “DELETED_DROPPED”                        |  Successfully deleted (if the Map becomes empty, drop collection too
| “NOT_FOUND"                              |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                      |  Field Miss (no field, element found to delete, returned when there is no field to delete) 
| “TYPE_MISMATCH”                          |  Given item is not a Map collection            |
| "NOT_SUPPORTED"                          |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”   |  Incorrect protocol syntax                     |

## mop get (Retrieve Map Element)    
  
Retrieve corresponding elements of given one or more field names from Map collection.    

```
mop get <key> <lenfields> <numfields> [delete|drop]\r\n
[<"space separated fields">]\r\n  
```  
  
- "space separated fields" - divided by space(' ') and field list of the target map
  - comma(,) is also supported for lower compatibility (lower 1.10.X versions), but is not recommended.
- \<key\> - key string of target item.
- \<lenfields\> and \<numfields\> - represents length of field list string and number of fields. 0 means all fields, elements.
- delete or drop - specifies whether to the field,element after retrieval, and if map becomes empty whether to drop it. 
  
When succeeded, the response string is as follows.
- \<count\> of the VALUE line indicates the number of fields retrieved.
- in the last line, you will receive one the END, DELETED, DELETED_DROPPED message and their meaning are as follows respectively:
  - field(s) has been retrieved
  - field retrieved and deleted
  - field retrieved and deleted, the map also dropped
  
```
 VALUE <flags> <count>\r\n
<field> <bytes> <data>\r\n
<field> <bytes> <data>\r\n
<field> <bytes> <data>\r\n
...
END|DELETED|DELETED_DROPPED\r\n
```

Response strings when failed and their meaning are as follows.

| **String**                                           | **Description**                                |  
| ---------------------------------------------------- | ---------------------------------------------- |  
| “NOT_FOUND"                                          |  Key Miss                                      |
| “NOT_FOUND_ELEMENT”                                  |  field miss (there is no element in any of the given field nameset)|
| “TYPE_MISMATCH”                                      |  Given item is not a Map collection            |
| “UNREADABLE”                                         |  Given item is unreadable                      | 
| "NOT_SUPPORTED"                                      |  Not Supported                                 |
| “CLIENT_ERROR bad command line format”               |  Incorrect protocol syntax                     | 
| “SERVER_ERROR out of memory [writing get response]”  |  Insufficient memory                           |  
