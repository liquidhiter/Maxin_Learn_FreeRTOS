# memmang

- drawback of using C library malloc and free
    - not always available on small embedded systems (bare-metal ?)
    - take up code size
    - **rarely thread-safe**
    - execution time can’t be guaranteed
    - fragmentation issue
    - complicate the linker configuration (?)
    - problematic if heap size goes into memory used by other variables

- MPU wrappers
    - MPU: memory protection unit

```c
/* Defining MPU_WRAPPERS_INCLUDED_FROM_API_FILE prevents task.h from redefining
 * all the API functions to use the MPU wrappers.  That should only be done when
 * task.h is included from an application file. */
#define MPU_WRAPPERS_INCLUDED_FROM_API_FILE
```

- when this macro is defined
    
    [mpu_wrappers.h](mpu_wrappers%20h%20b6636085882c466e85a3656835d5276d.md)
    
    - redefines API functions to be called through a wrapper macro only when the ports are using the MPU
    - MPU wrappers will not define `MPU_` specific macros but use the ones defined in `task.h`
- when including the task.h in an application file
    - API functions declared in `task.h` are used

> summarization
> 
- define `MPU_WRAPPERS_INCLUDED_FROM_API_FILE` to avoid re-define of the macros when the wrapper header file is included in the `task.c` or `queue.c`

[heap_1](memmang%20fb2bf0610ab0465e8eb4de3beec5eb65/heap_1%209322f543901143009b8899e29776cd1b.md)

[heap_2](memmang%20fb2bf0610ab0465e8eb4de3beec5eb65/heap_2%209ecbba8b8e27464ea031312180af31f9.md)

[heap_3](memmang%20fb2bf0610ab0465e8eb4de3beec5eb65/heap_3%204cecec2243cc4700b75f35f6fc1305b5.md)

[heap_4](memmang%20fb2bf0610ab0465e8eb4de3beec5eb65/heap_4%2011e6fdbce98d483598558526bb559c14.md)