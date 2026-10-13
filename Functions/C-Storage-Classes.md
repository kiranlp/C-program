# Storage Classes in C (auto & static)

> Notes by Kiran Kumar | Embedded Systems & VLSI | 12/10/26

## Table of Contents
1. [Overview](#1-overview)
2. [auto storage class](#2-auto-storage-class)
3. [Stack frame](#3-stack-frame)
4. [Scope of auto variables](#4-scope-of-auto-variables)
5. [Dangling pointer](#5-dangling-pointer)
6. [static storage class](#6-static-storage-class)
7. [auto vs static summary](#7-auto-vs-static-summary)

---

## 1. Overview

C has four storage classes:

1. `auto`
2. `static`
3. `extern`
4. `register`

Storage classes are applicable with two things in a program:

1. **Variable** (data type)
2. **Function**

All four storage classes are applicable on a variable, but only two (`static` and `extern`) are applicable on a function.

### Where can we declare a variable?
1. Inside a function → **local variable**
2. Outside a function → **global variable**

### What does the storage class of a variable determine?

| # | Property | Meaning |
|---|----------|---------|
| 1 | **Scope** | Section of code that can use the variable (variable visibility domain) |
| 2 | **Linkage** | How a variable can be used in a multiple source file program |
| 3 | **Life (storage duration)** | How long the variable persists in memory |
| 4 | **Default value** | Value taken initially if not defined |
| 5 | **Memory section** | Specifies how and where to store the subsequent variable |

---

## 2. auto storage class

When we declare any **local variable** and no storage class is provided, then by default the variable is an **auto variable**.

- All auto variables are local variables, **but** all local variables may not be auto (e.g. `static` locals).

| Property | auto |
|----------|------|
| Default value | Garbage / indeterminate |
| Memory section | Stack section |
| Scope | Within function / block |
| Lifetime | Starts when function execution starts (function stack frame created); ends when function execution ends (stack frame deleted) |
| Linkage | None (auto variables don't have a linkage property) |

### Example: which declarations are auto?

```c
#include <stdio.h>

auto int k;          // invalid -> Error (auto not allowed at global scope)

int main()
{
    int i;           // auto var
    int *p;          // auto var
    int a[5];        // auto var
    auto int j;      // auto var
    static int l;    // static var
}
```

---

## 3. Stack frame

**Q. What is a stack frame?**

When we execute our program, a stack memory (typically 8 MB) is allocated for our program with respect to function calls. The stack memory is divided into small portions, and each portion is called a **stack frame**.

```c
#include <stdio.h>

void abc(void);
void def(void);

int main()
{
    int i = 10, j = 20;                 // (1)
    abc();                              // (2)
    printf("i=%d j=%d\n", i, j);        // (3)
    return 0;                           // (4)
}

void abc(void)
{
    int k = 30, l = 40;                 // (1)
    def();                              // (2)
    printf("k=%d l=%d\n", k, l);        // (3)
}

void def(void)
{
    int m = 50, n = 60;                 // (1)
    printf("m=%d n=%d\n", m, n);        // (2)
}
```

**Output:**
```
m=50 n=60
k=30 l=40
i=10 j=20
```

**Stack layout (8 MB):**

```
| def  : m=50, n=60  |   <- top (created last, deleted first)
| abc  : k=30, l=40  |
| main : i=10, j=20  |   <- bottom (created first, deleted last)
```

Each frame is created when the function is called and deleted after the function finishes executing.

---

## 4. Scope of auto variables

Scope of an auto variable is **within the block only**.

### Example 1: Nested block (valid)

```c
#include <stdio.h>

int main()
{
    int i = 10;
    {
        int i = 20;                 // inner block, new variable
        printf("i=%d\n", i);        // 20
    }
    printf("i=%d\n", i);            // 10
}
```
**Result:** Valid. The inner `i` hides the outer `i` inside its block.

### Example 1b: Same variable defined twice in the same block (error)

```c
int main()
{
    int i = 10;
    int i = 20;                     // Error: redefinition / redeclaration
    printf("i=%d\n", i);
}
```
**Result:** Error (definition / redeclaration).

### Example 2: Loop variable scope

```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i <= 2; i++)
        printf("in loop... %d\n", i);
    printf("out of loop... %d\n", i);
    return 0;
}
```
**Output:**
```
in loop... 0
in loop... 1
in loop... 2
out of loop... 3
```

If `i` is declared inside the `for` statement (`for (int i = 0; ...)`) and used after the loop, it is an **Error**, because `i` exists only inside the loop block.

### Example 3: Returning address of an auto variable

```c
#include <stdio.h>

int *abc(void);

int main()
{
    int *p;
    p = abc();
    printf("%d\n", *p);
    return 0;
}

int *abc(void)
{
    int i = 10;
    return &i;                      // we can't return address of auto var
}
```
**Result:** Error / segmentation fault (undefined behavior). `i` dies when `abc()` returns.

---

## 5. Dangling pointer

When a pointer is holding (pointing to) the address of a variable whose **life has ended**, the pointer is called a **dangling pointer**.

Example: `p` in Example 3 above, after `abc()` returns, is dangling.

---

## 6. static storage class

A static variable can be **local as well as global** both.

**Q. When a local and a global static variable have the same name, what will be the behavior of the compiler?**
**A.** The **local variable gets priority**.

| Property | static |
|----------|--------|
| Default value | **Zero** |
| Memory section | **Data section** |
| Lifetime | Starts when program execution starts; ends when program execution ends |
| Scope (local static) | Within function / block |
| Scope (global static) | Within the entire program file |
| Linkage (local static) | None |
| Linkage (global static) | **Internal linkage** |

### Example 1: Global and local static

```c
#include <stdio.h>

static int i = 10;

int main()
{
    static int j = 20;
    printf("i=%d j=%d\n", i, j);
    return 0;
}
```
**Output:** `i=10 j=20`

### Example 2: Local priority

```c
#include <stdio.h>

static int i = 10;

int main()
{
    static int i = 20;
    printf("i=%d\n", i);            // local var priority
    return 0;
}
```
**Output:** `i=20`

### Example 3: auto stack memory and static memory are unrelated

```c
#include <stdio.h>

void abc(void);
void def(void);

int main()
{
    int i = 10, j = 20;
    static int a = 100;
    abc();
    printf("i=%d j=%d\n", i, j);
    return 0;
}

void abc(void)
{
    int k = 30, l = 40;
    def();
    printf("%d %d\n", k, l);
}

void def(void)
{
    int m = 50, n = 60;
    static int b = 200;
    printf("%d %d\n", m, n);
}
```
**Note:** There is **no relation** between auto (stack) memory and static memory. `a = 100` and `b = 200` stay allocated in the data section for the whole program, even when their functions are not running.

### Example 4: Returning address of a static variable (valid)

```c
#include <stdio.h>

int *abc(void);

int main()
{
    int *p;
    p = abc();
    printf("%d\n", *p);
    return 0;
}

int *abc(void)
{
    static int i = 10;
    return &i;                      // we can return address of static var
}
```
**Output:** `10`

### Example 5: Global static vs local static in another function

```c
#include <stdio.h>

static int i = 20;                  // global static

void abc(void);

int main()
{
    static int i = 10;              // local static (priority inside main)
    abc();
    printf("in main: %d\n", i);
    return 0;
}

void abc(void)
{
    printf("in abc: %d\n", i);      // uses global i
}
```
**Output:**
```
in abc: 20
in main: 10
```
If the global `static int i` is declared **after** `abc()`'s definition, then `i` is not visible inside `abc()` -> **compile-time error**.

### Example 6: Global static modified in a function

```c
#include <stdio.h>

static int i = 20;
void abc(void);

int main()
{
    // static int i = 10;           // commented out
    abc();
    printf("in main = %d\n", i);
    return 0;
}

void abc(void)
{
    printf("in abc = %d\n", i);
    i = 50;
}
```
**Output:**
```
in abc = 20
in main = 50
```

### Example 7: static is initialized only once

```c
#include <stdio.h>

void abc(void);

int main()
{
    abc();
    abc();
    abc();
    return 0;
}

void abc(void)
{
    // int i = 10;                  // auto var can be re-initialized (prints 11 11 11)
    static int i = 10;              // static var cannot be re-initialized
    i++;
    printf("%d\n", i);
}
```
**Output (static):**
```
11
12
13
```

---

## 7. auto vs static summary

| Property | auto | static |
|----------|------|--------|
| Declared | Inside function/block only | Inside or outside function |
| Default value | Garbage | Zero |
| Memory section | Stack | Data section |
| Lifetime | Function/block execution | Entire program execution |
| Scope | Within function / block | Local: function/block, Global: whole file |
| Linkage | None | Local: none, Global: internal |
| Re-initialized on each call? | Yes | No (initialized only once) |
| Safe to return its address? | No (dangling pointer) | Yes |

---

*Next topics: `extern` and `register` storage classes.*
