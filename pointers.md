# Pointers

## Introduction and Basics
### Initialization
Pointer variables store memory addresses. They can be initialized in the following ways:
```c
int n, m;

int *p;         // a pointer to an int
p = &m;         // points p to m 

int *q = &n;    // declares a pointer to n

int *i, *j, *k; // declares three pointers, all to ints
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
You 

### The NULL Pointer
The `NULL` pointer is a special pointer with an address of 0. Since 0 is a falsey value,
we can check for `NULL` pointers easily:
```c
int *p = NULL;  // a null pointer
if (!p) {
    // do stuff
    // this code will run because p is null
}
```