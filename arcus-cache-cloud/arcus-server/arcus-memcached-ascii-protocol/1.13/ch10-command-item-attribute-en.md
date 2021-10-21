# Chapter 10. Item Attribute Command

Introducing the `getattr` command to retrieve and the `setattr` command to update item attributes.

Please check [Item Attribute Description](ch03-item-attributes-en.md) to find out what item attributes are provided in ARCUS.

## getattr (Item Attribute Retrieval)

`getattr` command to retrieve item attributes is as follows.

```
getattr <key> [<name> ...]\r\n
```

- \<key\> - key string of target item
- [\<name\> ...] - sequentially specifies attribute names to retrieve, if omit this, all possible retrieval attribute values will be
                   retrieved based on item types.

When succeeded, the response string is as follows.
Returns the pair of names and values in the order of specified attribute names as factors in the `getattr` command

```
ATTR <name>=<value>\r\n
ATTR <name>=<value>\r\n
...
END\r\n
```

Response strings when failed and their meaning are as follows.

| **String**                                  | **Description**                                |  
| ------------------------------------------- | ---------------------------------------------- |  
| "NOT_FOUND"                                 | Key Miss                                       |
| "ATTR_ERROR not found"                      | the attribute specified as a factor does not exist or is not supported by the given item type |
| "CLIENT_ERROR bad command line format"      | Incorrect protocol syntax                      |

## setattr (Update Item Attribute)

`setattr` command to update item attributes is as follows.
Although retrieval of all attributes is possible, attribute update is only possible for some of them.
Updatable attributes are `expiretime`, `maxcount`, `overflowaction`, `readable`, `maxbkeyrange`.

```
setattr <key> <name>=<value> [<name>=<value> ...]\r\n
```

- \<key\> - key string of target item
- \<name\> = \<value\> - at least one pair of name and value of attribute must be specified 

Response strings and their meaning are as follows.

| **String**                              | **Description**                                |  
| --------------------------------------- | ---------------------------------------------- |  
| "OK"                                    | Successfully updated                           |
| "NOT_FOUND"                             | Key Miss                                       |
| "ATTR_ERROR not found"                  | The attribute specified as a factor does not exist or is not supported by the given item type |
| "ATTR_ERROR bad value"                  | New value to update a given attribute is not an allowed value |
| "CLIENT_ERROR bad command line format"  | Incorrect protocol syntax   |




