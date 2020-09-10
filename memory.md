# Memory
## Functions
The following list contains useful functions relevant to memory.
* `sizeof(x)` returns the size *in bytes* of a type or variable.

* `malloc(size_t n)` allocates `n` bytes for a pointer to point to. It must be
typecasted to the same type as the pointer.
    ```c
    int *ptr;
    ptr = (int *) malloc(sizeof(int));
    // ptr now points to an int-sized space in memory
    ```
    * `malloc` is rarely used to allocate a single variable; oftentimes,
    `sizeof(int)` is multiplied by `n` to create an array of size `n`.
    * Note that the allocated memory is initially garbage, so should not be
    used until values have been set for it.

* `calloc(size_t n, size_t s)` allocates memory for `n` items of size `s` similar to `malloc`, but
zeros the allocated memory as well.

* `realloc(void *ptr, size_t n)` reallocates the block previously allocated at
`ptr` to a new size and returns the address of the new memory block. It
must be typecasted to the same type as the pointer.
    * If `n` is 0, this function behaves like `free` (listed below).

* `free(void *ptr)` deallocates memory after it is done being used. This
should always be used to clean up allocating memory.
    * `free`ing unallocated memory will cause errors or bugs


`malloc`, `realloc`, and `calloc` will return a null pointer if they fail to find
adequate memory. **ALWAYS** check for a null pointer after using these functions!

## Memory Locations
C has three pools of memory, each which is a part of a program's *address space*.
* **Stack**: Stores information declared inside procedures. Memory is freed when the procedure ends. Grows downwards in 
the address space.
    * Composed of "frames" including return addresses, parameters, and local variables
    * Stored contiguously in LIFO order with a pointer to the newest frame
* **Heap**: Where dynamically allocated memory (`malloc`) is stored until deallocated
by the programmer. Grows upwards in the address space.
    * Not allocated contiguously
* **Static storage**: Global variable storage with fixed size. Does not grow or shrink
until the program ends.

The address space also contains code, which is loaded when the program starts, and
which does not change.

## Memory Management
*Fragmentation* refers to when a large amount of free memory is in discontiguous chunks.
Each block of memory is preceded by a header containing the *size* of the block and the *pointer*
to the next block. Free blocks of memory are kept in a *circular linked list.*
