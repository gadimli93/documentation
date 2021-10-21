# Chapter 1. ARCUS Basic Concept

In addition to a simple key-value with only one data as a value, the ARCUS Cache Server provides an extended key-value
data model with a Collection that stores multiple data in a structured form as a single value.

### Basic Constraints

The key-value model of the ARCUS Cache Server has the following basic constraints.

 - Constraints of existing Key-Value model
    - the maximum size of the **Key** is 32000 characters (arcus-memcached `1.11` version or higher).
      - in the existing version, the maximum size of the key is 250 characters.
    - the maximum size of **Value** is 1 MB (including "\r\n" trailing characters). 
 - Constraints of Collection
    - the maximum number of elements that can enter a collection is 50,000.
    - maximum size of the value of each element of the collection is 16KB (including "\r\n" trailing characters) which can be changed in settings.
 
## Cache Key

Cache Key identifies the code representing data that are stored in ARCUS Cache Server.
The syntax Cache Key is as follows.

```java
Cache Key : [<prefix>:]<subkey>
```

- **\<prefix\> :** a namespace preceding the cache key
  - in the prefix unit, you can group keys stored in a cache server to Flush them out or view statistical information
  - a prefix can be omitted but it's recommended to be used as much as possible.
- **delimiter :** by a default delimiter is a colon (':') character that separates prefix and subkey.
- **\<subkey\> :** a key that generally used in applications.

In order to name the Prefix and Subkey follow the below listed rules.

- a prefix can only consist of Uppercase letters, numbers, underbars (_), hyphens (-), plus (+), and dots (.) characters.
  Hyphens (-) cannot come as the first character.
- subkey cannot contain spaces, and by default, it is recommended to use only alphanumeric characters.

## Cache Item

In addition to simple key-value, ARCUS Cache Server has various item types with collection support.

- simple key-value item - existing key-value item
- collection item
  - list item: an item that has a value of a double-linked list of data
  - set item: an item that has a value unordered dataset of unique data
  - map item: an item that has a value of unordered dataset of <field, value> pair;
  - b+tree item: an item that has an ordered dataset based on a b+tree key;

## Expiration, Eviction, and Sticky

Each cache item has an **expiration time** attribute, which allows you to specify an item that will not be expired
or an item that will be automatically expired after a specific time. For a detailed description please check the 
[Item Attribute Description](ch03-item-attributes-en.md) chapter.

ARCUS Cache Server is an in-memory cache, which uses a certain amount of memory space to cache data.
In the case of memory space exhaustion when a request to save a new cache item is received, ARCUS Cache Server
either will print the **"out of memory"** error or **evict** a cache item that has not been accessed for a long time based
on LRU(Least Recently Used) algorithm to free up an available memory space before saving the new cache item.
This operation method can be specified with a `-M` running option of the ARCUS Cache Server, and 
as a default, the LRU-based eviction method is used.

Some applications might not want any cache items to be subject of expiration or eviction.
In ARCUS Cache Server these kinds of cache items are called **sticky items** and their expiration time is set to `-1`.
One thing to be cautious about is removal of the sticky item process. When deleting sticky items, 
the entire process should be managed by the application. Because the sticky item is also stored in memory,
it will disappear when the server is restarted.

Sticky items are generally unlikely to be many but to prevent sticky items from taking up the entire memory space of the
ARCUS Server due to some application mistake, ARCUS provides a `-g` (gummed or sticky) running option that enables only a 
portion of the entire memory space to be used by Sticky items.
As a memory ratio of the Sticky items to be used as memory space, it can be designated as a value in the range of 0 to 100.
The default `0` means that sticky items are not allowed, and `100` means that the entire memory can be used for sticky items.

## Memory Allocator

ARCUS Cache Server provides two memory allocators for the purpose of efficiently managing allocation and return of
item memory space.

### Slab Allocator

A slab allocator is a memory allocator that quickly processes the allocation and return of memory space of that size.
It is divided into slab classes to manage memory space by memory size. In each slab class,
*slabs* that are memory spaces of the same size, are managed in the form of a *free list*.
This is the main memory allocator used in the existing memcached.

The maximum slab size is currently 1 MB. The minimum slab size, i.e. the slab size of the first slab class and following
slab size of the slab classes is set as the ARCUS cache Server running option as follows.

- **-n \<bytes\> :** minimum space allocated from key+value+flags (default: 48)
  - determines the slab size of the minimum size.
- **-f \<factor\> :** chunk size growth factor (default: 1.25)
  -  degree of increase in the size of the slab should be specified as a value greater than `1.0` for each slab class.

### Small Memory Allocator 

Thanks to the collection support there have been more requests for allocation and return of small memory spaces.
In order to efficiently manage such a small memory space, a new **small memory allocator** has been developed and used.
A small memory allocator is responsible for less than 8,000bytes of memory space, and an existing slab allocator
is responsible for more than 8,000bytes of memory space.

## LRU list per Slab Class

ARCUS Cache Server maintains an LRU list for each slab class and allows items that have not been 
accessed for a long time to be selected as eviction target items.

Due to the addition of a small memory allocator, there is a change in the LRU list for each slab class.
Especially, if using a small memory allocator instead of the 0th slab class, collection items and key-value items
of small size that allocate memory space from small memory allocators are linked to the 0th LRU list.
Therefore LRU lists of the existing slab class corresponding to the memory space less than 8,000bytes become **empty**.
