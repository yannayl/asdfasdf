# Attacks
Write It


## Structural Attacks

Using the allocator's internal data structures for overwriting arbtirary memory


### [Frontlink][Vudo Malloc Tricks]

* Insert node to a linked list
    ```C
    void frontlink(mchunkptr prev, mchunkptr p) {
        p->fd = prev->fd;
        prev->fd = p;
    }
    ```
* Control `prev`'s fd => write-pointer-to-what-where
* Controlling prev requires:
    - Ability to [overwrite the list's head](http://blog.frizn.fr/glibc/glibc-heap-to-rip)
    - Insertion to large bin
* Never mitigated!!!!!!!
* Articles:
    - PoC||GTFO 0x18 House of Fun
    - [Vudo Malloc Tricks]

[Vudo Malloc Tricks]: http://phrack.org/issues/57/8.html
Note: Inserting to a list happens in `free` and `p` points to data the designer may control


### Frontlink - Example
TODO


### [Unlink][Vudo Malloc Tricks]
* Remove node from linked list:
    ```C
    void unlink(mchunkptr p) {
        p->bk->fd = p->fd;
    }
    ```
* Control chunk's `bk` and `fd` => write-what-where
* Mitgated (2004): 
```C
if (p->bk->fd != p) malloc_printerr(check_action, ...)
```
* Still useful:
    - [Unsorted Bin Attack]: un-mitigated unlink
    - [House of Cards]: overwrite global `check_action` which changes `malloc_printer` to no-op
    - [Unlink pointer] to attacker controlled data

[Vudo Malloc Tricks]: http://phrack.org/issues/57/8.html
[Unsorted Bin Attack]: https://github.com/shellphish/how2heap/blob/master/unsorted_bin_attack.c
[House of Cards]: http://tukan.farm/2016/09/04/fastbin-fever/
[Unlink pointer]: https://heap-exploitation.dhavalkapil.com/attacks/unlink_exploit.html 

Note:
`malloc_printerr` changed in glibc 2.27 and always termintes the process


## Functionality Attacks

Overriding allocator's metadata to induce allocation to user in arbitrary memory


### [Tcache Poison Attack]
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

[Tcache Poison Attack]: http://tukan.farm/2017/07/08/tcache/


### Tcache Poison Atttack - Example
TODO


### [Malloc Maleficarum] et al
* [House of Lore]
* [House of Force]
* [House of Spirit]
* [Fast Bin Attack] 

[Malloc Maleficarum]: https://dl.packetstormsecurity.net/papers/attack/MallocMaleficarum.txt
[House of Lore]: https://github.com/shellphish/how2heap/blob/master/house_of_lore.c
[House of Force]: https://github.com/shellphish/how2heap/blob/master/house_of_force.c
[House of Spirit]: https://github.com/shellphish/how2heap/blob/master/house_of_spirit.c
[Fast Bin Attack]: http://uaf.io/exploitation/2017/03/19/0ctf-Quals-2017-BabyHeap2017.html 


## Restricted Functionality Attacks
Restricted overriding metadata to compromise the integrity of other data on the heap

* [Poisoned NULL Byte](https://googleprojectzero.blogspot.lu/2014/08/the-poisoned-nul-byte-2014-edition.html)
* [Fragment-and-Write](https://www.alchemistowl.org/pocorgtfo/pocorgtfo16.pdf "Adventure of the Fragmented Chunks")

Note:
bunch of ways to create overlapping chunks or forgotten chunks
turn liner write to relative write


## Other Cool Tricks
* [`munmap` any page](http://tukan.farm/2016/07/27/munmap-madness/)
* [Allocation of bins](http://blog.frizn.fr/glibc/glibc-heap-to-rip)
