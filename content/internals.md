# Internals
Brief version of [GlibC Malloc Internals][MallocInternals]
[MallocInternals]: https://sourceware.org/glibc/wiki/MallocInternals


## API


## API
* __malloc__ - Allocates a chunk* at least size bytes and returns
```C
p = malloc(size);
```

* __free__ - De-allocates chunk* pointed by p
```C
free(p);
```

* __realloc__ - Increases the size of a chunk* while preserving it's content
```C
p1 = realloc(p0, size);
```


## Structs


### Chunk
* The basic memory unit managed by the allocator
* Two possible states: Allocated or Freed
* Content when allocated:
    ```C
    -------------------------------
    | size | user-data........... |
    -------------------------------
    ```
* Content when freed:
    ```C
    -------------------------------
    | size | fd | bk | ... | size |
    -------------------------------
    ```
* Chunk's min size - 0x20, granularity - 0x10
* The three LSB of size are flags
    - bit 0 - previous chunk in-use (`prev_inuse`)
* Really big chunks are mmaped - out of scope
Note:
* malloc returns pointer to user data


### `malloc_chunk` struct
Internally, use this struct
```C
----------------------------------------------------------
| prev_size | size | fd | bk | fd_nextsize | bk_nextsize |
----------------------------------------------------------
```
* `malloc_chunk` is not aligned with the `chunk` it describes
    ```C
 ------------------------------------------------------
 | prev_sz | sz | fd | bk | fd_nextsize | bk_nextsize |
 -----------------------------------------------------------------
            | sz | fd | bk | fd_nextsize | bk_nextsize | ... | sz |
            -------------------------------------------------------
    ```
* `prev_size` valid iff `prev_inuse` bit is off
* `size` in the end coincides with next `prev_size`


### Bins
Lists of freed chunks

| Type | Size |  List Type | Notes|
|------|------|------------|------|
| Fast | Each size in [0x20, 0x80] | Singly Linked | next chunk's `prev_inuse` is on |
| Unsorted | All Sizes | Doubly Linked | Chunks before sorting 
| Small | Each sizes [0x20, 0x400) | Doubly Linked | |
| Large | Range of sizes | Two-levels doubly-linked | |


### Arena (`malloc_state` struct)
```C
--------------------------------------------------------------------------
| mutex | flags | fastbins[] | top | remainder |       bins[]        |...|
--------------------------------------------------------------------------
                                              /                       \
                                              --------------------------
                                              |unsorted|small[]|large[]|
                                              --------------------------
```
* Contains bins and other heap info
* `top` - points to "wilderness" 
* Secondary Arenas - out of scope for today

Note:
* top points to the wilderness which is a special chunk not in bins, never allocated
* `main_arena` is resides in glibc whereas the chunks reside in the heap


### Structs Summary
![arena and bins](content/arena_and_bins.png)
Shamelessly copied from [GlibC Malloc Internals][MallocInternals]
[MallocInternals]: https://sourceware.org/glibc/wiki/MallocInternals


## Algorithms


### Free
1. If fast - Put in fastbin
2. Else - Coalesce and put in unsorted bin


### Alloc
1. Look in the exact fit bins (fast/small)
2. Consolidate, coalesce and sort
    1. Not necessarily in that order
    2. Consolidate - move from fastbins to unsorted bin
    3. Coalesce - merge with adjacent freed chunks
3. Search through the best fit bins
4. Split the top


## tcache (2.26+)
* Thread local storage caching
* Singly linked list in chunk's `fd` (sim. to FastBins)
* All sizes
* Major performance enhancement 
* Not even trying to be secured :/
