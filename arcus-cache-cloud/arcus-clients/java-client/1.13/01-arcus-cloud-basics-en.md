# ARCUS Cloud Basics

ARCUS supports an extended Key-Value data model.
In addition to simple key-value data type, where one key has only one data, 
ARCUS supports a collection type in which a single key stores multiple data in structured form. 

Constraints of ARCUS Cache server's key value model are as follows:
- Simple Key-Value:
  - maximum size of `key`: 4000 characters;
  - maximum size of `value`: 1MB;

- Collection:
  - maximum number of elements of a collection: 50,000;
  - maximum size of `value` of a each collection element: 16KB; 

Here are the main fundamentals of ARCUS Cloud to being with:
- [Service Code](01-arcus-cloud-basics-en.md#service-code)
- [ARCUS Admin](01-arcus-cloud-basics-en.md#arcus-admin)
- [Cache Key](01-arcus-cloud-basics-en.md#cache-key)
- [Cache Item](01-arcus-cloud-basics-en.md#cache-item)
- [Expiration, Eviction and Sticky Item](01-arcus-cloud-basics-en.md#expiration-eviction-and-sticky-item)

## Service code

Service Code of ARCUS is a code that distinguishes cache cloud(clusters) from each other. The term  of 'service code' comes from 
a sense of delivering the ARCUS Cache Cloud service to applications.

In a single application you can deploy and use one or more ARCUS Cache Clouds. 
ARCUS Java Client object has only one ARCUS service code and only one ARCUS Cache Cloud is accessible. 
If the application needs to access more than one ARCUS Cache Cloud, then for each ARCUS Cache Cloud a Java Client object with a service code
must be created and used separately.

## ARCUS Admin

ARCUS Admin uses ZooKeeper to manage the ARCUS Cache Cloud corresponding to each service code.
When ZooKeeper manages the specific service code of a cache server list, 
for any possible updates, like adding or removing cache servers to/from list, it keeps the list up to date, 
and sends the latest cache server list information about the service code to the arcus client.
In order to keep ARCUS Admin `highly available` all the time, 
several ZooKeeper servers together compose a ZooKeeper Ensemble.

## Cache Key

Cache Key identifies only Cache Item that are stored in ARCUS Cache. The Cache Key syntax is shown below.

```
  Cache Key : [<prefix>:]<subkey>
```

- `<prefix>` is a namespace preceding the cache key.
  -  in prefix units you can group keys stored in a cache server to Flush them out or view Stats.
  -  prefix can be omitted but it's recommended to be used as much as possible.
-  by a default delimiter(':') prefix and subkey are separated.
- `<subkey>` is a key that is generally used in applications.

To name the Prefix and Subkey here are rules to follow:
- `prefix` can only consist of Uppercase letters, numbers, underbars (_), hyphens (-), plus (+), and dots (.) characters.
  Hyphens (-) cannot used as the first character.
- `subkey` cannot contain spaces, and by default, it is recommended to use only alphanumeric characters.

## Cache Item

In addition to simple key-value, ARCUS Cache has various collection item types.
- key-value item: a simple key-value item that has a single value.
- Collection item
  - list item: an item that has a double-linked list of data elements;
  - set item: an item that has an unordered dataset of unique data elements;
  - map item: an item that has an unordered dataset of <mkey, value> pair;
  - b+tree item: an item that has an ordered dataset based on b+tree key; 

## Expiration, Eviction and Sticky Item

Each cache item has an `expiration time` attribute. The setting of this value allows to specify automatic expiration 
or set the cache item to not expire.

ARCUS Cache is an in-memory cache, which uses a certain amount of memory space to cache data.
If a request to save a new cache item comes when there is no space in the memory, 
ARCUS Cache either will print *"out of memory"* error or `evict` a cache item that has not been accessed for 
a long time based on LRU(Least Recently Used) algorithm to free up an available memory space before saving the new cache item.

Some applications might not want any cache items to be subject of expiration or eviction.
These kinds of cache items are called `sticky items` and their expiration time is set to `-1`.
One thing to be cautious about is removal of the sticky item process. When deleting sticky items, the entire process should be managed by the application.
The sticky item is also stored in memory, it will disappear when the server is restarted. 

