# Flow Hijacking
Become a Wizard


## GlibC Malloc Hooks
* `__free_hook`, `__malloc_hook`, `realloc_hook`
* Attacker invokes API calls
* Naturally combined with Fast Bin Attack 
* May also be reached from I/O system
Note:
IO system - printf uses malloc/realloc/free internally when sizes are large engouh


## GlibC dlopen Hook
* `_dl_open_hook->dlopen_mode`
* Invoked upon `dlopen`
* Attacker can esily trigger it:
    - design detectable error -> `malloc_printerr` -> print backtrace -> dlopen
* Naturally combined with frontlink attack
Note:
Since GlibC 2.27 `malloc_printerr` simply exits


## GlibC FILE Struct
* `_IO_list_all->_vtable.__overflow()`
* Invoked upon process termination
* Attacker can easily triggrer it similar to `_dlopn_hook`
* Partially mitigated with `IO_validate_vtable` 


## Program's Specific
* Return Addresses
* Vtables
* Function Pointers
