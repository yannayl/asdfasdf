# Attacks
Write It


## Structural Attacks

Using the allocator's internal data structures for overwriting arbtirary memory


### Frontlink
* Insert node to a linked list
```C
#define frontlink(prev, p) do { \
    p->fd = prev->fd; \
    prev->fd = p; \
\} while (0);
```
* Control `prev`'s fd => write-pointer-to-what-where
* Controlling prev requires:
    - Insertion to large bin
    - Ability to overwrite the list's head
* Never mitigated!!!!!!!
    - In spite of whatever you have read
Note:
Inserting to a list happens in `free` and `p` points to data the designer may control


### Frontlink - Example
TODO


### Unlink
* Remove node from linked list:
```C
#define unlink(p) do { \
    p->bk->fd = p->fd; \
} while (0)
```
* Control chunk's `bk` and `fd` => write-what-where
* Mitgated (2004): 
```C
if (p->bk->fd != p) malloc_printerr(check_action, ...)
```
* Still useful:
    - Unsorted Bin Attack: un-mitigated unlink
    - House of Cards: overwrite global `check_action` which changes `malloc_printer` to no-op
    - Unlink pointer to attacker controlled data
Note:
`malloc_printerr` changed in glibc 2.27 and always termintes the process


## Functionality Attacks

Overriding allocator's metadata to induce allocation to user in arbitrary memory


### Tcache Poison Attack
* `tcache` is singly linked cache of chunks
```C
static void *tcache_get (size_t tc_idx)
{
  tcache_entry *e = tcache->entries[tc_idx];
  tcache->entries[tc_idx] = e->next;
  return (void *) e;
}
```
* Attacker controls `e->next` (`fd`)
* Malloc returns attacker controlled location 


### Tcache Poison Atttack - Example
TODO


### Malloc Maleficarum et al
* House of Lore
* House of Force
* House of Spirit
* Fast Bin Attack 


## Restricted Functionality Attacks
Restricted overriding metadata to compromise the integrity of other data on the heap

* Poisoned NULL Byte
* Fragment-and-Write
Note:
* bunch of ways to create overlapping chunks or forgotten chunks
* turn liner write to relative write


## Other Cool Tricks
* `munmap` any page
* Allocation of bins
