# Pointers

## Basic Concepts
### Declaration
Pointer variables store memory addresses. They can be initialized in the following ways:
```c
int n, m;

int *p;         // a pointer to an int
p = &m;         // points p to m 

int *q = &n;    // declares a pointer to n

int *i, *j, *k; // declares three pointers, all to ints
i = p;          // points i to the same pointee as p
```
As shown above, the `&` symbol can be used to obtain the memory address of variables.
Note that `&p` would return the address where `p` is stored.

Pointers may also point to functions. The code below shows the declaration and usage of
a pointer to a function that accepts an `int *` pointer and a `char`, and returns an `int`.
```c
int (*fn) (int *, char) = &foo;    // points to a function foo
(*fn) (x, y);                      // calls the function
```

### Dereferencing
After initializing a pointer, the `*` symbol can also be used to *dereference* it. This
allows you to access and modify the variable being pointed at.
```c
int n = 0;
int *p = &n;
*p = 5;         // n is now equal to 5
```

### Structures
There are two different ways to access the members of a structure from a pointer.
Generally, arrow notation is preferred.
```c
/* Creates a Point structure and a point, pt. */
typedef struct {
    int x, y;
} Point;          

Point pt;

/* Declares a pointer to pt and accesses its values. */
Point *p = &pt;
(*p).x = 0;         // changes x using parentheses
p->y = 1;           // changes y using arrow notation
```

### The NULL Pointer
The `NULL` pointer is a special pointer with an address of 0. Since 0 is a falsey value,
we can check for `NULL` pointers easily. The `if` statement below will evaluate to `true`:
```c
int *p = NULL;  // a null pointer
if (!p) {
    // do stuff
}
```
### Generic Pointers
Generic poiners are declared using `void *` and can point to data of any type. They
should be used sparingly, as they can produce more bugs and introduce security issues.

## Arrays
Array variables are pointers to the first element of the array. Declaing an array simply
reserves of a block of memory.
```c
int a[5];             // declares an array of 5 ints
int b[] = {1, 2, 3};  // declares and initializes an array of 3 ints
```

### Pointer Arithmetic
Addition and subtraction can be done with pointers to move through memory. Adding 1 or
subtracting `1` to a pointer alters its stored memory address by `sizeof(pointer type)`.
In an array, this is equivalent to indexing the next element.

## Potential Errors
### Declaring Multiple Pointers
An asterisk is needed before each variable name when declaring multiple pointers.
```c
int *p, q, r; // this declares a pointer and two ints
```

### Dangling Pointers
Local variables in C are not initialized; newly declared pointers are *not* initialized to
null. Hence, dereferencing uninitialized pointers will result in unexpected behavior.
```c
/* This code will likely error. */
int *p;
*p = 5;
```

### Stale Pointers

### Returned Pointers
Functions should not return pointers to local variables, because the stack frame is erased
and the memory is deallocated upon reaching `return`. While the pointer will point to the
correct location, this unallocated memory may be overwritten in the future, causing the
values to change unexpectedly.
```c
/* A function that returns a pointer to the local variable y.*/
int *foo() {
    int y = 3;
    return &y;
}

/* The main function. */
main() {
    int *p;
    p = foo();          // calls foo, pointing p to y
    printf("%d", *p)    // correctly prints 3
    printf("%d", *p)    // prints garbage left by printf
}
```