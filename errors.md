# Potential Errors
## Pointers and Memory
### Declaring Multiple Pointers
An asterisk is needed before each variable name when declaring multiple pointers.
```c
int *p, q, r;   // incorrectly declares a pointer and two ints
int *i, *j, *k; // correctly declares three pointers
```

### Dangling Pointers
Local variables in C are not initialized; hence, newly declared pointers are *not* initialized to
null. Dereferencing and altering uninitialized pointers will result in unexpected behavior.
```c
/* Bad things happen here. */
int *p;
*p = 5;
```

### Stale Pointers
If two pointers are pointing to the same memory address and the data stored is freed or reallocated
(using, for example, `realloc`) using one pointer, the other pointer will not redirect itself to the
correct memory address.
```c
/* Creates a integer array and points p and q to it. */
int *p, *q;
p = malloc(sizeof(int) * 10);
q = p;

/* Reallocates the integer array using p. q may no longer point 
to valid memory. */
p = realloc(sizeof(int) * 20);
```

### Returned Pointers
Functions should not return pointers to local variables, because the stack frame is erased
and the memory is deallocated upon reaching `return`. While the pointer will point to the
correct location, this unallocated memory may be overwritten in the future, causing the
values to change unexpectedly.
```c
/** A function that returns a pointer to the local variable y.*/
int *foo() {
    int y = 3;
    return &y;
}

/** The main function. */
main() {
    int *p;
    p = foo();          // calls foo, pointing p to y
    printf("%d", *p);   // correctly prints 3
    printf("%d", *p);   // prints garbage left by printf
}
```

### Null Pointers
The functions `malloc` and `realloc` will return `NULL` if the program is unable to find
suitable space to allocate memory; attempting the modify the null pointer will result in
an error.
```c
int *p = malloc(sizeof(int) * 99999); // likely impossible
*p = 1;                               // error
```
