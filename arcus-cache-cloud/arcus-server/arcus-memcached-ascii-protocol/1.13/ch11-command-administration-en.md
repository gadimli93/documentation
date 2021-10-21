## Chapter 11. Admin & Monitoring Command

- [FLUSH Command](ch11-command-administration-en.md#flush-command)
- [SCRUB Command](ch11-command-administration-en.md#scrub-command)
- [STATS Command](ch11-command-administration-en.md#stats-command)
- [CONFIG Command](ch11-command-administration-en.md#config-command)
- [CMDLOG Command](ch11-command-administration-en.md#command-logging-command)
- [LQDETECT Command](ch11-command-administration-en.md#long-query-detect-command)
- [KEY DUMP Command](ch11-command-administration-en.md#key-dump-command)
- [ZKENSEMBLE Command](ch11-command-administration-en.md#zkensemble-command)
- [HELP Command](ch11-command-administration-en.md#help-command)

## FLUSH Command

ARCUS Cache Server supports two flush commands in order to invalidate the items.

- **flush_all** : flush all items
- **flush_prefix** : flush items only with the specific prefix. 

Although the flush operation invalidates the items, it does not immediately return the occupied memory space of those items.
Instead, by recording the information at the point of flush execution time as global information of ARCUS Cache Server,
it makes it possible to discover the items that existed before that point are invalidated.
Therefore, each time an item is accessed, there is an inconvenience of checking whether the item is invalidated or not.
However, the flush operation itself is completed at O(1) time.

`flush_all` command only records the flush execution time information and leaves out the statistical information of all 
prefixes as it is. Therefore, even though items are flushed by `flush_all` command, their statistical information are 
intact and retrievable. When all items belonging to the prefix are removed, statistical information of that
prefix is also removed together. On the other hand, the difference of `flush_prefix` command from the `flush_all`command is that,
`flush_prefix` command records the flush execution time information for the corresponding prefix, and 
removes all the statistical information of that prefix by reset. Therefore, after execution of the `flush_prefix` command,
it is not possible to retrieve statistical information about the corresponding prefix.

Syntaxes of the two flush commands are as follows.

```
flush_all [<delay>] [noreply]\r\n
flush_prefix <prefix> [<delay>] [noreply]\r\n
```

- \<prefix> - a prefix string. when "\<null\>" is used, then items without a prefix string are invalidated.
- \<delay\> - specified at the time of delayed invalidation request and its delay period is specified in SECONDS.
- noreply - if state this, response string will be omitted.

Response strings and their meaning are as follows.

String                                 | Description                    |
-------------------------------------- | -------------------------------| 
"OK"                                   | Successful
"NOT_FOUND”                            | Prefix miss (for in the case flush_prefix command) 
"CLIENT_ERROR bad command line format" | Incorrect protocol syntax
 
## SCRUB Command

There might be items that occupy memory even if they are not available in the ARCUS Cache Server.
These items are divided into the following two types.

- If items expired in the ARCUS Cache Server, those items are not immediately removed. 
  Although those items are invalidated with the flush command, they are not removed instantly.
  These items continue to exist and occupy memory inside the ARCUS Cache Server.
  For some reason when there is an attempt to access these items, 
  ARCUS Cache Server discovers that they are in the state of expired/flushed and 
  by removing those items, it returns the occupied memory space.
- In the environment that forms the cache cloud and using key-to-node mapping of consistent hashing
  In that cache cloud, key-to-node-remapping occurs when adding or deleting a particular node.
  When such key-to-node-remapping occurs, no matter in which node, that existing items will no longer be used.
  These items are called stale items. These stale items can eventually expire, but there is a possibility for
  stale items to carry some old data and then to convert back to a valid item by key-to-node remapping.
  Therefore, these stale items must be removed whenever the node list of the cache cloud is updated.

Scrub feature refers to:
(1) a function to explicitly remove _expired items_ and similar to flushed items _invalidated items_,
(2) a function to explicitly remove _stale items_ generated from key-to-node remapping in the cache cloud.

This scrub feature is performed by daemon thread as a background operation, and 
only one scrub operation can be performed at a time. 
Therefore, it is not possible to request a new scrub operation while the other scrub operation is already in progress.

```
scrub [stale]\r\n
```

- **stale** - if not specified, it removes the invalidated items, if it's specified then removes the stale items.

Response strings and their meaning are as follows.

String                                 | Description                    |
-------------------------------------- | -------------------------------| 
"OK"                                   | Successful
"BUSY"                                 | Unable to request a new scrub operation due to the scrub operation is already in progress.
"NOT_SUPPORTED"                        | Unsupported scrub command
"CLIENT_ERROR bad command line format" | Incorrect protocol syntax

Please be aware, since the scrub command is implemented as an extension feature of ASCII command when starting ARCUS Cache Server,
scrub command can be used only by giving the run option to dynamically link the `ascii_scrub.so` file. 

## STATS Command

Retrieves or resets the various statistical information of ARCUS Cache Server.

```
stats [<args>]\r\n
```

The behavior of stats operation depending on if \<args\> is omitted or given a certain value varies as follows.

```
 <args>             | Behaviors of stats command                 |
 ------------------ | -------------------------------------------|
                    | Retrieve General Purpose Statistical Information
 settings           | Retrieve Settings Information
 items              | Retrieve Item Statistical Information
 slabs              | Retrieve Slab Statistical Information
 prefixes           | Retrieve item Statistical Information Per Prefix 
 detail on|off|dump | Retrieve and Control Execution Command Statistical Information Per Prefix
 zookeeper          | Retrieve Zookeeper Information
 scrub              | Retrieve Status of Scrub Execution
 cachedump          | Cache Key Dump Per Slab Class
 reset              | Reset All Statistical Information
 persistence        | Retrieve Persistence Information
```

It is recommended to perform the `stats` command at least once, to grasp a concept.
Additional required explanations are described below.

- [General Purpose Information](ch11-command-administration-en.md#general-purpose-information)
- [Settings Statistical Information](ch11-command-administration-en.md#settings-statistical-information)
- [Items Statistical Information](ch11-command-administration-en.md#item-statistical-information)
- [Slabs Statistical Information](ch11-command-administration-en.md#slabs-statistical-information)
- [Prefix Statistical Information](ch11-command-administration-en.md#prefix-statistical-information)
- [ZooKeeper Status Information](ch11-command-administration-en.md#zookeeper-status-information)
- [Scrub Performance Status](ch11-command-administration-en.md#scrub-performance-status)
- [Cache Key Dump for Each Slab Class](ch11-command-administration-en.md#cache-key-dump-for-each-slab-class)
- [Retrieve Persistence Information](ch11-command-administration-en.md#retrieve-persistence-information)

### General Purpose Information

A command to check general-purpose statistics that are not limited to a specific classification.
A sample result of the `stats` command is as follows.

```
STAT pid 3553
STAT uptime 6910
STAT time 1584942539
STAT version 1.11.7
STAT libevent 2.1.11-stable
STAT pointer_size 64
STAT rusage_user 1.241010
STAT rusage_system 2.843840
STAT daemon_connections 2
STAT curr_connections 1
STAT quit_connections 0
STAT reject_connections 0
STAT total_connections 3
STAT connection_structures 3
STAT cmd_get 0
STAT cmd_set 0
STAT cmd_incr 0
STAT cmd_decr 0
STAT cmd_delete 0
STAT cmd_flush 0
STAT cmd_flush_prefix 0
STAT cmd_cas 0
STAT cmd_lop_create 0
STAT cmd_lop_insert 0
STAT cmd_lop_delete 0
STAT cmd_lop_get 0
STAT cmd_sop_create 0
STAT cmd_sop_insert 0
STAT cmd_sop_delete 0
STAT cmd_sop_get 0
STAT cmd_sop_exist 0
STAT cmd_mop_create 0
STAT cmd_mop_insert 0
STAT cmd_mop_update 0
STAT cmd_mop_delete 0
STAT cmd_mop_get 0
STAT cmd_bop_create 0
STAT cmd_bop_insert 0
STAT cmd_bop_update 0
STAT cmd_bop_delete 0
STAT cmd_bop_get 0
STAT cmd_bop_count 0
STAT cmd_bop_position 0
STAT cmd_bop_pwg 0
STAT cmd_bop_gbp 0
STAT cmd_bop_mget 0
STAT cmd_bop_smget 0
STAT cmd_bop_incr 0
STAT cmd_bop_decr 0
STAT cmd_getattr 0
STAT cmd_setattr 0
STAT auth_cmds 0
STAT auth_errors 0
STAT get_hits 0
STAT get_misses 0
STAT delete_misses 0
STAT delete_hits 0
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT lop_create_oks 0
STAT lop_insert_misses 0
STAT lop_insert_hits 0
STAT lop_delete_misses 0
STAT lop_delete_elem_hits 0
STAT lop_delete_none_hits 0
STAT lop_get_misses 0
STAT lop_get_elem_hits 0
STAT lop_get_none_hits 0
STAT sop_create_oks 0
STAT sop_insert_misses 0
STAT sop_insert_hits 0
STAT sop_delete_misses 0
STAT sop_delete_elem_hits 0
STAT sop_delete_none_hits 0
STAT sop_get_misses 0
STAT sop_get_elem_hits 0
STAT sop_get_none_hits 0
STAT sop_exist_misses 0
STAT sop_exist_hits 0
STAT mop_create_oks 0
STAT mop_insert_misses 0
STAT mop_insert_hits 0
STAT mop_update_misses 0
STAT mop_update_elem_hits 0
STAT mop_update_none_hits 0
STAT mop_delete_misses 0
STAT mop_delete_elem_hits 0
STAT mop_delete_none_hits 0
STAT mop_get_misses 0
STAT mop_get_elem_hits 0
STAT mop_get_none_hits 0
STAT bop_create_oks 0
STAT bop_insert_misses 0
STAT bop_insert_hits 0
STAT bop_update_misses 0
STAT bop_update_elem_hits 0
STAT bop_update_none_hits 0
STAT bop_delete_misses 0
STAT bop_delete_elem_hits 0
STAT bop_delete_none_hits 0
STAT bop_get_misses 0
STAT bop_get_elem_hits 0
STAT bop_get_none_hits 0
STAT bop_count_misses 0
STAT bop_count_hits 0
STAT bop_position_misses 0
STAT bop_position_elem_hits 0
STAT bop_position_none_hits 0
STAT bop_pwg_misses 0
STAT bop_pwg_elem_hits 0
STAT bop_pwg_none_hits 0
STAT bop_gbp_misses 0
STAT bop_gbp_elem_hits 0
STAT bop_gbp_none_hits 0
STAT bop_mget_oks 0
STAT bop_smget_oks 0
STAT bop_incr_elem_hits 0
STAT bop_incr_none_hits 0
STAT bop_incr_misses 0
STAT bop_decr_elem_hits 0
STAT bop_decr_none_hits 0
STAT bop_decr_misses 0
STAT getattr_misses 0
STAT getattr_hits 0
STAT setattr_misses 0
STAT setattr_hits 0
STAT stat_prefixes 0
STAT bytes_read 23
STAT bytes_written 771
STAT limit_maxbytes 8589934592
STAT threads 6
STAT conn_yields 0
STAT curr_prefixes 0
STAT reclaimed 0
STAT evictions 0
STAT outofmemorys 0
STAT sticky_items 0
STAT curr_items 0
STAT total_items 0
STAT sticky_bytes 0
STAT bytes 0
STAT sticky_limit 0
STAT engine_maxbytes 8589934592
END
```

The main statistics for each command are summarized as follows. 

- **cmd_\<command_name\>**: the number of times the corresponding command is executed
- **\<command_name\>_hits**: the number of key hits of the corresponding command
- **\<command_name\>_misses**: the number of key misses of the corresponding command
- the number of key hits of the collection command is not provided separately, and it can be calculated as the below total number of times  
  - **\<collection_name\>_\<command_name\>_elem_hits**: key hit of collection command and the number of element hits
  - **\<collection_name\>_\<command_name\>_none_hits**: key hit of collection command and the number of element misses

Below are the other individual statistics.

```
| stats                 | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| pid                   | Process ID of Cache Node                                     
| uptime                | Running Time(SECONDS) of Cache Server                                  
| time                  | Current Time (UNIX TIME)                                        
| version               | Current Version of ARCUS-MEMCACHED                                    
| libevent              | Currently Used LIBEVENT's Version                                      
| pointer_size          | Size(BITS) of Pointer                                      
| rusage_user           | Accumulated USER TIME of the Process                                
| rusage_system         | Accumulated SYSTEM TIME of the Process                                
| daemon_connections    | Number of the DEAMON CONNECTIONS used by the server                       
| curr_connections      | Number of CONNECTIONS currently open                              
| quit_connections      | Number of times a client disconnected using the QUIT command            
| reject_connections    | Number of times a connection with a client is REJECTED                         
| total_connections     | Total number of accumulated CONNECTIONS after starting the server                          
| connection_structures | Number of CONNECTION STRUCTURES allocated by the server                         
| auth_cmds             | Number of SASL authentication                                           
| auth_errors           | Number of FAILED SASL authentication                                         
| cas_badval            | Number of requests that found the key but did not match the CAS value              
| bytes_read            | Total amount(BYTES) of data READ by the server on the network           
| bytes_written         | Total amount(BYTES) of data WRITTEN by the server on the network                  
| limit_maxbytes        | Maximum MEMORY CAPACITY(BYTES) allowed on the server                      
| threads               | Number of WORKER THREADS                                        
| conn_yields           | LIMIT on the maximum number of requests per event                               
| curr_prefixes         | Number of PREFIXES currently stored                                     
| reclaimed             | Number of times stored new items using the space of EXPIRED items
| evictions             | Number of EVICTIONS                                              
| outofmemorys          | Number of times OUTOFMEMORY occurred (in the state of insufficient memory, eviction is not allowed or failed)
| sticky_items          | Current number of STICKY items                                    
| curr_items            | Number of ITEMS currently stored in the server                             
| total_items           | Total number of accumulated ITEMS stored after starting the server                       
| sticky_bytes          | Memory capacity(BYTES) occupied by STICKY items          
| bytes                 | Currently used MEMORY CAPACITY(BYTES)                            
| sticky_limit          | Maximum memory capacity(BYTES) to store STICKY items        
| engine_maxbytes       | Maximum storage capacity allowed for the ENGINE                               
```

### Settings Statistical Information

A command to view statistical information on various setting values.
A sample result of the `stats settings` execution is as follows.

```
STAT maxbytes 8589934592
STAT maxconns 3000
STAT tcpport 11911
STAT udpport 0
STAT sticky_limit 0
STAT inter NULL
STAT verbosity 1
STAT oldest 0
STAT evictions on
STAT domain_socket NULL
STAT umask 700
STAT growth_factor 1.25
STAT chunk_size 48
STAT num_threads 6
STAT stat_key_prefix :
STAT detail_enabled yes
STAT allow_detailed yes
STAT reqs_per_event 5
STAT cas_enabled yes
STAT tcp_backlog 8192
STAT binding_protocol auto-negotiate
STAT auth_enabled_sasl no
STAT auth_sasl_engine none
STAT auth_required_sasl no
STAT item_size_max 1048576
STAT max_list_size 50000
STAT max_set_size 50000
STAT max_map_size 50000
STAT max_btree_size 50000
STAT max_element_bytes 16384
STAT scrub_count 96
STAT topkeys 0
STAT logger syslog
STAT ascii_extension scrub
END
```

| stats              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| maxbytes           | Maximum STORAGE CAPACITY(BYTES) of a cache server                           
| maxconns           | Maximum number of CLIENTS to connect                        
| tcpport            | TCP PORTS that listening on                                     
| udpport            | UDP PORTS that listening on                                    
| sticky_limit       | Maximum memory capacity(BYTES) to store STICKY items      
| inter              | Interface of LISTEN                                             
| verbosity          | Current VERBOSITY level(0~3)                                     
| oldest             | The stored time of OLDEST stored item                     
| evictions          | Whether or not the EVICTION is allowed                                        
| domain_socket      | Path of the DOMAIN SOCKET                                        
| umask              | UMASK of the domain socket                                       
| growth_factor      | GROWTH FACTOR of chuck size of slab class                           
| detail_enabled     | Whether to collect DETAILED STATS(statistics per prefix)                       
| allow_detailed     | Whether or not the STAT DETAIL command is allowed                                  
| reqs_per_event     | Maximum number of IO operations can be handled in the IO event                
| cas_enabled        | Whether or not the CAS operation is allowed                                          
| tcp_backlog        | Size of the QUEUE of TCP backlog                                       
| binding_protocol   | PROTOCOL(ASCII, Binary or Auto(negotiating)) in use  
| auth_enabled_sasl  | Whether or not to use the SASL authentication                                     
| auth_sasl_engine   | ENGINE to use for SASL authentication                                
| auth_required_sasl | Whether or not SASL authentication is MANDATORY                                          
| item_size_max      | Maximum SIZE of the item                                         
| max_list_size      | Maximum number of elements in the LIST collection                          
| max_set_size       | Maximum number of elements in the SET collection                         
| max_map_size       | Maximum number of elements in the MAP collection                          
| max_btree_size     | Maximum number of elements in the B+TREE collection                        
| max_element_bytes  | Maximum size of COLLECTION ELEMENT DATA                       
| topkeys            | Number of tracking TOPKEYS                                    
| logger             | LOGGER EXTENSION  in use                                  
| ascii_extension    | ASCII PROTOCOL EXTENTION in use                          

### Items Statistical Information

A command to retrieves statistical information about an item per slab class.
A sample result of the `stats items` execution is as follows.

```
STAT items:0:number 2000002
STAT items:0:sticky 0
STAT items:0:age 5401
STAT items:0:evicted 0
STAT items:0:evicted_nonzero 0
STAT items:0:evicted_time 0
STAT items:0:outofmemory 0
STAT items:0:tailrepairs 0
STAT items:0:reclaimed 0
END
```

The number next to `items:` is the slab class id.
The description of the statistical information is as follows.

| Stats           | Description                                                        |
| --------------- | ------------------------------------------------------------------ |
| number          | Number of ITEMS stored in the corresponding class                          
| sticky          | Number of items set as STICKY. Please refer to [ARCUS Basic Concept](ch01-arcus-basic-concept-en.md#expiration-eviction-and-sticky) 
| age             | Time(SECONDS) after the oldest item was created in the LRU chain
| evicted         | Number of EVICTED items                                        
| evicted_nonzero | Among the evicted items, the number of ITEMS those expired time was set to an explicit positive value.
| evicted_time    | The past time(SECONDS) from the last time access of the recently evicted item
| out_of_memory   | Number of times FAILED to store an item due to memory insufficiency          
| tailrepairs     | Number of times the SLAB ALLOCATOR was recovered from refcount leak            
| reclaimed       | Number of times stored new items using the space of EXPIRED items

### Slabs Statistical Information

A command to retrieves statistical information of each slab class and meta information for the entire class.
A sample result of the `stats slabs` execution is as follows.

```
STAT SM:free_min_classid 709
STAT SM:free_max_classid -1
STAT SM:used_total_space 472
STAT SM:used_01pct_space 288
STAT SM:free_small_space 0
STAT SM:free_bslot_space 261640
STAT SM:free_avail_space 261640
STAT SM:free_chunk_space 0
STAT SM:free_limit_space 0
STAT SM:space_shortage_level 0
STAT 0:chunk_size 262144
STAT 0:chunks_per_page 4
STAT 0:reserved_pages 0
STAT 0:total_pages 1
STAT 0:total_chunks 4
STAT 0:used_chunks 1
STAT 0:free_chunks 0
STAT 0:free_chunks_end 3
STAT 0:mem_requested 262144
STAT active_slabs 1
STAT memory_limit 8589934592
STAT total_malloced 1048576
END
```

The character in front of the colon (:) is the slab class number.
`SM` is a **small manager class** that handles small-sized data.

The description of the statistical information in the general class is as follows.

| stats           | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| chunk_size      | Total size(BYTES) of memory spaces used by each chuck           
| chunks_per_page | Number of chunks per page                                      
| reserved_pages  | Number of reserved pages to allocate in the corresponding class               
| total_pages     | Number of pages allocated in the corresponding class                            
| total_chunks    | Number of chunks allocated in the corresponding class                            
| used_chunks     | Number of chunks already allocated to the item                          
| free_chunks     | Number of chunks that have not yet been allocated or are free after the deletion operations
| free_chunks_end | Number of remaining chunks at the end of the last allocated page            
| mem_requested   | Total size(BYTES) of memory spaces requested in the corresponding class           

The description of the small manager class's statistical information is as follows.

| stats                | Description                                                |
| -------------------- | ---------------------------------------------------------- |
| free_min_classid     | Minimum value among the slot free IDs                                  
| free_max_classid     | Maximum value among the slot free IDs(ID of last slot - big free slot is excluded)  
| used_total_space     | Total size(BYTES) of the used space                              
| used_01pct_space     | Total size(BYTES) of the spaces used by slots in the top 1% in size
| free_small_space     | Total size(BYTES) of the memory spaces that are not allocated and are less likely to be allocated
| free_bslot_space     | Total size(BYTES) of the remaining spaces of the big free slot                
| free_avail_space     | Total size(BYTES) of the memory spaces that are not allocated and highly likely to be allocated
| free_chunk_space     | Total size(BYTES) of the spaces to allocate memory block(chunk)  
| free_limit_space     | Minimum amount(BYTES) of the free space that must always be left empty
| space_shortage_level | Levels that are digitized from 0 to 100 to specify the lack of space              

If the `space_shortage_level` goes up above 10, then to secure memory space separate threads are executed on the background to evicts 
items. Items are deleted as much as from the end of the LRU chain to the `space_shortage_level` (if `ssl` is 10, then delete 10 items).

Other meta-statistics are as follows.

| stats          | Description                                    |
| -------------- | ---------------------------------------------- |
| active_slabs   | Total number of allocated slab class                    
| memory_limit   | Maximum capacity(bytes) of cache server                    
| total_malloced | Total size(bytes) of allocated memory space in slap page

### Prefix Statistical Information

All prefixes' item statistical information is retrieved with the `stats prefixes` command.
All prefixes' operation statistical information are retrieved with the `stats detail dump` command.
In addition, only in the operation statistical information of prefixes can be specified with ON or OFF
in order to whether or not to collect statistical information.

A sample result of item statistical information of all prefixes is as follows.
\<null\> prefix statistic indicates to items statistics that do not have prefixes.

```
PREFIX <null> itm 2 kitm 1 litm 1 sitm 0 mitm 0 bitm 0 tsz 144 ktsz 64 ltsz 80 stsz 0 mtsz 0 btsz 0 time 20121105152422
PREFIX a itm 5 kitm 5 litm 0 sitm 0 mitm 0 bitm 0 tsz 376 ktsz 376 ltsz 0 stsz 0 mtsz 0 btsz 0 time 20121105152422
PREFIX b itm 2 kitm 2 litm 0 sitm 0 mitm 0 bitm 0 tsz 144 ktsz 144 ltsz 0 stsz 0 mtsz 0 btsz 0 time 20121105152422
END
```

**itm** is total item count in the item statistical information for each prefix, where
**kitm**, **litm**, **sitm**, **mitm** and **bitm** are counts of the kv, list, set, map, b+tree respectively.
**tsz**(total size) is the size of the space occupied by the total items, and 
**ktsz**, **ltsz**, **stsz**, **mtsz**, **btsz** are the sizes of the spaces occupied by kv, list, set, map, b+tree respectively.
**time** is the prefix creation time.

Operation statistical information's result of all prefixes is as follows.
Although in reality each PREFIX line is shown as a single line, however in this document for ease the understanding
it is displayed in several lines.

```
PREFIX <null> get 2 hit 2 set 2 del 0
       lcs 0 lis 0 lih 0 lds 0 ldh 0 lgs 0 lgh 0
       scs 0 sis 0 sih 0 sds 0 sdh 0 sgs 0 sgh 0 ses 0 seh 0
       mcs 0 mis 0 mih 0 mus 0 muh 0 mds 0 mdh 0 mgs 0 mgh 0
       bcs 0 bis 0 bih 0 bus 0 buh 0 bds 0 bdh 0 bps 0 bph 0 bms 0 bmh 0 bgs 0 bgh 0 bns 0 bnh 0
       pfs 0 pfh 0 pgs 0 pgh 0
       gas 0 sas 0
PREFIX a get 5 hit 5 set 5 del 0 
       lcs 0 lis 0 lih 0 lds 0 ldh 0 lgs 0 lgh 0
       scs 0 sis 0 sih 0 sds 0 sdh 0 sgs 0 sgh 0 ses 0 seh 0
       mcs 0 mis 0 mih 0 mus 0 muh 0 mds 0 mdh 0 mgs 0 mgh 0
       bcs 0 bis 0 bih 0 bus 0 buh 0 bds 0 bdh 0 bps 0 bph 0 bms 0 bmh 0 bgs 0 bgh 0 bns 0 bnh 0
       pfs 0 pfh 0 pgs 0 pgh 0
       gas 0 sas 0
PREFIX b get 2 hit 2 set 2 del 0 
       lcs 0 lis 0 lih 0 lds 0 ldh 0 lgs 0 lgh 0
       scs 0 sis 0 sih 0 sds 0 sdh 0 sgs 0 sgh 0 ses 0 seh 0
       mcs 0 mis 0 mih 0 mus 0 muh 0 mds 0 mdh 0 mgs 0 mgh 0
       bcs 0 bis 0 bih 0 bus 0 buh 0 bds 0 bdh 0 bps 0 bph 0 bms 0 bmh 0 bgs 0 bgh 0 bns 0 bnh 0
       pfs 0 pfh 0 pgs 0 pgh 0
       gas 0 sas 0
END
```

The **get**, **hit**, **set**, **del** are operation statistics for items of KV type in the operation statistics
information of each prefix and the three characters starting with 'l', 's', 'm', 'b' are operation statistics for each item 
of the list, set, map, b+tree, and in addition, three characters starting with 'p' are statistics of position operation for b+tree.
**gas** and **sas** are statistics of the item attribute operation.

The meanings of the each three characters in the operation statistics are as follows.

- list ooperation statistics
  - lcs - number of **lop create** executions
  - lis, lih - number of **lop insert** executions and hit count 
  - lds, ldh – number of **lop delete** executions and hit count 
  - lgs, lgh – number of **lop get** executions and hit count 
- set operation statistics
  - scs - number of **sop create** executions
  - sis, sih - number of **sop insert** executions and hit count 
  - sds, sdh – number of **sop delete** executions and hit count 
  - sgs, sgh – number of **sop get** executions and hit count 
  - ses, seh - number of **sop exist** executions and hit count
- map operation statistics
  - mcs - number of **mop create** executions
  - mis, mih - number of **mop insert** executions and hit count
  - mus, muh – number of **mop update** executions and hit count
  - mds, mdh – number of **mop delete** executions and hit count
  - mgs, mgh – number of **mop get** executions and hit count
- b+tree operation statistics
  - bcs – number of **bop create** executions
  - bis, bih – number of **bop insert/upsert** executions and hit count
  - bus, buh – number of **bop update** executions and count hit
  - bps, bph – number of **bop incr(plus)** executions and count hit
  - bms, bmh - number of **bop decr(minus)** executions and count hit
  - bds, bdh – number of **bop delete** executions and count hit
  - bgs, bgh – number of **bop get** executions and count hit
  - bns, bnh – number of **bop count**  executions and count hit
- b+tree position operation statistics
  - pfs, pfh - number of **bop position** executions and hit count
  - pgs, pgh - number of **bop gbp** executions and hit count
- item attribute operation statistics
  - gas - number of **getattr** execution 
  - sas - number of **setattr** execution

### ZooKeeper Status Information

A command to check the ZooKeeper status information.
A sample of the execution result of `stats zookeeper` is as follows.

```
STAT zk_connected true
STAT zk_failstop on
STAT zk_timeout 30000
STAT zk_reconfig on
STAT zk_reconfig_version 100000000
END
```

| stats               | Description                                               |
| ------------------- | ----------------------------------------------------------|
| zk_connected        | ZooKeeper is connected                                         
| zk_failstop         | Specifies whether the server automatically shuts down when the ZooKeeper session is expired                    
| zk_timeout          | ZooKeeper session timeout(msec)                                   
| zk_reconfig         | Specifies whether to use or not ZooKeeper dynamic reconfiguration feature             
| zk_reconfig_version | ZooKeeper dynamic reconfiguration mirror version (16bits)        

### Scrub Performance Status

A sample of the retrieval result of the scrub performance status is as follows.

```
STAT scrubber: status stopped
STAT scrubber: last_run 0
STAT scrubber: visited 0
STAT scrubber: cleaned 0
END
```

- status : specifies whether the scrub operation is currently RUNNING or STOPPED.
- last_run : specifies required time in seconds for previously completed scrub operation.
- visited : specifies the number of items accessed in currently executing or previously executed scrub operations.
- cleaned : specifies the number of deleted items in currently executing or previously executed scrub operations.

### Cache Key Dump for Each Slab Class

Per slab class `stats cachedump` command is provided to dump cache keys of items that depend on LRU.

```
stats cachedump <slab_clsid> <limit> [ forward | backward [sticky] ]\r\n
```

- **\<slab_clsid\>** : slab class ID to specify a dump target LRU
- **\<limit\>** : number of items to dump can only be specified within the range of 0 ~ 200.
           if it's 0, then it's specified as default to 50, and if it exceeds 200, then dumped only 200.
           Starting with the head or tail of LRU dump the `limit` number of cache keys of items.
- **forward or backward** : specifies from where to start dump, the head or tail of LRU.
           If _forward_ then starts from the head, if _backward_ then it starts from the tail.
           If not specified, then the default is _forward_. 
- **sticky** : the LRU lists of non-sticky items and sticky items are maintained separately in a single slab class.
           dump in sticky LRU, if sticky is specified, if not specified then dump in the non-sticky LRU.

A sample result of the Cachedump is as follows.

```
ITEM a: bkey2
ITEM a: bkey1
ITEM b: bkey3
ITEM b: bkey1
ITEM b: bkey2
ITEM c: bkey1
ITEM c: bkey2
END
```

### Retrieve Persistence Information

Sample of the retrieval result of execution and configuration details of Persistence is as follows.

```
STAT use_persistence on
STAT data_path /home/test/arcus/data/ARCUS-DB
STAT logs_path /home/test/arcus/data/ARCUS-DB
STAT async_logging false
STAT chkpt_interval_pct_snapshot 100
STAT chkpt_interval_min_logsize 256
STAT recovery_elapsed_time_sec 5
STAT last_chkpt_in_progress true
STAT last_chkpt_failure_count 0
STAT last_chkpt_start_time 20201222182102
STAT last_chkpt_elapsed_time_sec 8
STAT last_chkpt_snapshot_filesize_bytes 423333
STAT current_command_log_filesize_bytes 65555
END
```

- use_persistence : specifies whether the current Persistence mode is ON or OFF.
- data_path : specifies the path through which the snapshot file is created.
- logs_path : specifies the path through which the command log file is created.
- async_logging : specifies the operation mode of command logging, FALSE if it's synchronous logging mode, TRUE if it's asynchronous logging mode.
- chkpt_interval_pct_snapshot : specifies the additionally increased size ratio of the command log file compared to the snapshot file size by the setting of checkpoint execution cycle.
- chkpt_interval_min_logsize : specifies the minimum size of the command log file to perform the checkpoint by the setting of checkpoint execution cycle.
- recovery_elapsed_time_sec : specifies the taken time to recover data after restarting the ARCUS instance (unit : seconds) 
- last_chkpt_in_progress : specifies whether the current checkpoint is executed or not.
- last_chkpt_failure_count : specifies how many times the current checkpoint has failed until it is successfully executed.
- last_chkpt_start_time : specifies starting time of the previous checkpoint when its conditions are met. (unit : absolute timestamp)
- last_chkpt_elapsed_time_sec : specifies taken time to perform the previous checkpoint. (unit : seconds)
- last_chkpt_snapshot_filesize_bytes : specifies size of the snapshot file of the previous checkpoint. (unit : bytes)
- current_command_log_filesize_bytes : specifies size of the current command log file. (unit : bytes)

## CONFIG Command

ARCUS Cache Server supports a function to dynamically update or retrieve the current value of a specific configuration.
Dynamical updates are currently supported only in the below configurations, except (*) ones.

- [verbosity](ch11-command-administration-en.md#config-verbosity)
- [memlimit](ch11-command-administration-en.md#config-memlimit)
- [zkfailstop](ch11-command-administration-en.md#config-zkfailstop)
- [*hbtimeout](ch11-command-administration-en.md#config-hbtimeout)
- [*hbfailstop](ch11-command-administration-en.md#config-hbfailstop)
- [maxconns](ch11-command-administration-en.md#config-maxconns)
- [max_collection_size](ch11-command-administration-en.md#config-max_collection_size)
- [max_element_bytes](ch11-command-administration-en.md#config-max_element_bytes)
- [scrub_count](ch11-command-administration-en.md#config-scrub_count)

### config verbosity

Dynamically(without restart) updates/retrieves `verbose log level` of ARCUS Cache Server.

```
config verbosity [<verbose>]\r\n
```

As a value of verbose log level to specify \<verbose\>'s acceptable range is 0~2.
If this factor is omitted, then retrieve the value of the current set verbose.

### config memlimit

Dynamically(without restart) updates/retrieves memory limit, set with `-m` option while ARCUS Cache Server is running.

```
config memlimit [<memsize>]\r\n
```

As a memory limit to specify \<memsize\> is set in megabyte(MB) units and
can only be set to a value greater than the `total_malloced` memory size that ARCUS Cache Server is currently using.
If this factor is omitted, then retrieve the value of the current set memory limit. 

### config zkfailstop

Turns ON or OFF automatic failstop feature of ARCUS Cache Server.

```
config zkfailstop [on|off]\r\n
```

In the network failure state, if there is a cache server that cannot perform normal service in the cache cloud, then
all requests to the data range that the corresponding cache server is responsible for will fail, 
and in consequence, it will put a great burden on DB. Additionally, even if it reconnects to the ZooKeeper again,
there is a possibility that it will still have the old data which can cause malfunctioning in the application.
Therefore, to solve this issue, ARCUS Cache Server provides the `automatic failstop` feature that
automatically removes failed cache servers from the cache cloud when ZooKeeper session timeout occurs.

### config hbtimeout

Retrieves/Updates a value of `hbtimeout`.

```
config hbtimeout [<hbtimeout>]\r\n
```

ARCUS Cache Server has a `heartbeat` operation that periodically checks whether nodes operate normally.
`hbtimeout` represent the timeout of the `heartbeat` operation. If the `heartbeat` operation does not work, 
even after a time set to `hbtimeout` is exceeded, then the corresponding `heartbeat` is considered as a **timeout**.
A default value of `hbtimeout` is 10_000ms and it can be set from a minimum of 50ms to a maximum of 10_000ms.

### config hbfailstop

Retrieves/Updates a value of `hbfailstop`.

```
config hbfailstop [hbfailstop]\r\n
```

ARCUS Cache Server can force shutdown a server if a `heartbeat` delay continues.
Every time a continuous timeout occurs, a value of `hbtimeout` accumulates and adds up, 
and if the accumulated value exceeds the `hbtimeout` value then a failstop is executed.
For example, let's say a `hbfailstop` is 30 seconds, and a `hbtimeout` is 10 seconds,
if `hbtimeout` continuously occurs 3 times in a row, then failstop will be initiated.
A default value of `hbfailstop` is 60_000ms and it can be set from a minimum of 3_000ms to a maximum of 300_000ms.

### config maxconns

Dynamically(without restart) updates/retrieves maximum number of connections, set with `-c` option while ARCUS Cache Server is running.

```
config maxconns [<maxconn>]\r\n
```

As a number of maximum connections to specify \<maxconn\> can only be set to a value greater than 10% of the current number
of connections. If this factor is omitted, then retrieve the value of the currently set maximum number of connections.

### config max_collection_size

Retrieves/Updates maximum number of elements of the collection item.

```
config max_<collection>_size [<max_size>]\r\n
* <collection>: list|set|btree|map
```

A default setting is 50_000 and it can be set from a minimum of 10_000 to a maximum of 1_000_000.
It cannot be set smaller than the existing value.
If retrieve multiple elements at once, after setting it to a value greater than the default value, then 
it not only slows down the response speed of retrieve, but also critically affects the response speed of other operations.   
Therefore, as a precaution, even if the maximum number of elements increased,
the application must be implemented as retrieval requests of a small number of elements in a repetitive form at once.

### config max_element_bytes

The maximum size of the collection element's value is set in bytes.
A default setting is 16KB and it can be changed from 1 ~ 32 KB.

```
config max_element_bytes [<maxbytes>]\r\n
```

### config scrub_count

ARCUS Cache Server supports a `scrub` command to delete items in bulk that are no longer valid.
When daemon thread executes a `scrub` command `config scrub_count` command set/retrieves how many items 
will be deleted for each operation. A default value is 96 and it can be set from a minimum of 16 to a maximum of 320.

```
config scrub_count [<scrub_count>]\r\n
```

## Command Logging Command

Logging input commands into ARCUS Cache Server. 
Records all commands from the start of the `start` command until the logging is completed. 
However, for performance maintenance, there might be some skipped commands and their count can be checked with the `stats` command.
When ten of 10MB log files are used and exceeded, then they will be automatically terminated.

```
cmdlog [start [<log_file_path>] | stop | stats]\r\n
```

\<log_file_path\> is a path of a file to store logging information.

- path can be omitted, and if it is, then it is specified as a `default`.
  - if it is automatically specified with default, then the log file is created in the memcached running location / `command_log` directory.
  - `command_log` directory is not created automatically, therefore must be created at the location where the memcached process is running. 
  - `command_port_bgndate_bgntime_{n}.log` is a file name of the created log file.
- it is possible to specify the path directly as an absolute path and a relative path. The directory where the final file will be created must also be specified.

As a result of the `start` command content printed on the log file is as follows.

```
---------------------------------------
format : <time> <client_ip> <command>\n
---------------------------------------

19:14:45.530198 127.0.0.1 bop insert arcustest-Collection_Btree:vuRYyfqyeP0Egg8daGF72 0x626B65795F6279746541727279323239 0x45464C4147 80 create 0 600 4000
19:14:45.530387 127.0.0.1 lop insert arcustest-Collection_List:pGhEn6DFv5MixbYObBgp1 -1 64 create 0 600 4000
19:14:45.530221 127.0.0.1 lop insert arcustest-Collection_List:hhSAED2pFBH9xGqEgAeW1 -1 80 create 0 600 4000
19:14:45.530334 127.0.0.1 bop insert arcustest-Collection_Btree:RGSXLACxWpKwLPdC86qn0 0x626B65795F6279746541727279303331 0x45464C4147 80 create 0 600 4000
19:14:45.530385 127.0.0.1 lop insert arcustest-Collection_List:PwFTiFSEWlenireHcxNb2 -1 80 create 0 600 4000
19:14:45.530407 127.0.0.1 bop insert arcustest-Collection_Btree:P1lfJrJyVFyP0ogrw27h1 0x626B65795F6279746541727279313238 0x45464C4147 101 create 0 600 4000
19:14:45.530537 127.0.0.1 sop exist arcustest-Collection_Set:gTx8KDPBiufiGN9ArtgG3 81 pipe
19:14:45.530757 127.0.0.1 sop exist arcustest-Collection_Set:gTx8KDPBiufiGN9ArtgG3 81
```

`stop` command can be used to stop logging before it is completed.

`stats` command retrieves the status of the most recently executed(or in-progress) command logging
and its result is as follows.

```
Command logging stats : running                                      //Not started | stopped by causes(request or overflow or error) | running
The last running time : 20160126_192729 ~ 20160126_192742            //bgndate_bgntime ~ enddate_endtime
The number of entered commands : 146783                              //entered_commands
The number of skipped commands : 0                                   //skipped_commands
The number of log files : 1                                          //file_count
The log file name: /Users/temp/command_11211_20160126_192729_{n}.log //path/file_name
```

## Long Query Detect Command

In the ARCUS Cache Server among the requests for the collection items, there are some requests with a long processing time.
In order to detect them  `lqdetect` command is provided. Starting with the `start` command until the detection is completed, 
during the command processing for the commands with long query potential, 
extract commands that exceed the specific standard for a number of accessed elements, and store 20 of them as samples.
It automatically ends when storing 20 samples for all target commands of a long query.
Stored samples can be checked via the `show` command.

Target commands of long query detection are as follows.

```
1. sop get
2. lop insert
3. lop delete
4. lop get
5. bop delete
6. bop get
7. bop count
8. bop gbp
```

`lqdetect` command syntax is as follows.

```
lqdetect [start [<detect_standard>] | stop | show | stats]\r\n
```

\<detect_standard\> represents number of accessed elements in the given request that classified as long query,
in some requests, request that accesses more elements than the detection criteria are classified as long query.
If omitted, the default standard is 4000.

`start` command starts detection.

`stop` command can be used to stop detection before it is completed.

`show` command prints the stored commands sample, and the result is as follows.

```
-----------------------------------------------------------
format : <time> <client_ip> <count> <command> <arguments>\n
-----------------------------------------------------------

sop get command entered count : 0

lop insert command entered count : 0

lop delete command entered count : 0

lop get command entered count : 92
17:56:33.276847 127.0.0.1 <46> lop get arcustest-Collection_List:YN8UCtNaoD4hHnMMwMJq1 0..44
17:56:33.278116 127.0.0.1 <43> lop get arcustest-Collection_List:orjTteJo7F0bWdXDDGcP0 0..41
17:56:33.279856 127.0.0.1 <48> lop get arcustest-Collection_List:r7ERYr3IdiD3RO8hLNvI3 0..46
17:56:33.304063 127.0.0.1 <45> lop get arcustest-Collection_List:0OWKNF3Z17NaTSaDTZG61 0..43

bop delete command entered count : 0

bop get command entered count : 81
17:56:33.142590 127.0.0.1 <47> bop get arcustest-Collection_Btree:0X6mqSiwBx6fEZVLuwKF0 0x626B65795F62797465417272793030..0x626B65795F6279746541727279303530 efilter 0 47
17:56:33.142762 127.0.0.1 <49> bop get arcustest-Collection_Btree:PiX8strLCv7iWywd1ZuE0 0x626B65795F62797465417272793030..0x626B65795F6279746541727279303530 efilter 0 49
17:56:33.143326 127.0.0.1 <46> bop get arcustest-Collection_Btree:PiX8strLCv7iWywd1ZuE1 0x626B65795F62797465417272793130..0x626B65795F6279746541727279313530 efilter 0 48

bop count command entered count : 0

bop gbp command entered count : 0
```

`stats` command retrieves the status of the most recently executed(or in-progress) long query detection
and its result is as follows.

```
Long query detection stats : running              //stopped by causes(request or overflow) | running
The last running time : 20160126_175629 ~ 0_0     //bgndata_bgntime ~ enddate_endtime
The number of total long query commands : 1152    //detected_commands 
The detection standard : 43                       //standard
```

## KEY DUMP Command

Dumps keys of ARCUS Cache Server.
Dump ASCII command is as follows.

```
dump start key [<prefix>] filepath\r\n
dump stop\r\n
stats dump\r\n
```

dump start command

- the first factor is always a "key"
  - currently dump only key string
  - in the future, you can select and dump a key or an item. If it is an item, then you can dump an entire content of an item.
- the second factor is \<prefix\> of a cache key, and if not needed it can be omitted.
  - if omitted, all key strings are dumped
  - if \<null\> is given, key strings without \<prefix\> are dumped
  - if a particular prefix is given, key strings of that particular prefix are dumped
- the third factor is \<file path\>
  - MUST be specified
  - it can be assigned as an absolute path and a relative path.
  - if it is a relative path then it is a relative path of a location where memcached process is running

The `dump stop` command can be used to terminate the dump operation if its execution time is too long. 

`stats dump` command prints information about currently running or most recently performed dump operation.

Dump files are always must be made into one file. The file content format is as follows.

```
<type> <key_string> <exptime>\n
...
<type> <key_string> <exptime>\n
DUMP SUMMARY: { prefix=<prefix>, count=<count>, total=<total> elapsed=<elapsed> }\n
```

The meaning of the above result message is as follows.

- key dump result
  - \<type\> : represents item type in 1 character
    - "K" : kv
    - "L" : list
    - "S" : set
    - "M" : map
    - "B" : b+tree
  - \<key_string\> : key string of cache server
  - \<exptime\> : expire time of key. It has one of the values below.
    - 0 : exptime = 0 
    - -1 : sticky item (exptime = -1)
    - timestamp (exptime \> 0) : time since the Epoch (00:00:00 UTC, January 1, 1970) measured in seconds

- DUMP SUMMARY
  - \<prefix\> :  prefix name
    - "\<all\>" :  all key dump 
    - "\<null\>" : key dump without prefix
    - if it is a key dump of a specific prefix, then specify the that prefix name 
  - \<count\> : number of dumped keys
  - \<total\> : total number of keys in cache
  - \<elapsed\> : taking time amount to dump (unit: seconds)

## ZKensemble Command

Provides commands for setting ZooKeeper Ensemble to which ARCUS Cache Server is connected.

```
zkensemble set <ensemble_list>\r\n
zkensemble get\r\n
zkensemble rejoin\r\n
```

`set` command updates the ZK ensemble address.
ensemble_list is specified in a form of a list of ZK servers such as \<ip:port\>,...,\<ip:port\> and
in a form of a domain address of ZK ensemble. 

`get` command retrieves the ZK ensemble address. Result is retrieved in a form of \<ip:port\>,...,\<ip:port\>.

`rejoin` is a command that disconnects the ZK ensemble and connects the cache servers waiting out of the cache cloud to the ZK ensemble again.
Cache server exits the cache cloud in the following cases:
- when ZooKeeper session timeout occurs in the state of failstop OFF, 
- when ephemeral znode of cache server registered in the cache_list is deleted by operator's mistake.

## HELP Command

Retrieves the ASCII command syntax of the ARCUS Cache Server.

```
help [<subcommand>]\r\n
```

\<subcommand\> includes kv, lop, sop, mop, bop, stats, flush, config, etc.
and if \<subcommand\> is omitted, the available subcommand list to use can be printed with the `help` command.
