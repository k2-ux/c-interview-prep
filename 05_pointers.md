# C Interview Prep — File 5: Pointers — C's Superpower (and Biggest Footgun 🔫)

> "A pointer is a variable that contains the address of a variable."
> — K&R, stating it perfectly in one sentence.

If variables are houses, pointers are addresses. You can write a letter (value) to a house (variable) directly, or you can write down the address on a piece of paper (pointer) and give that to someone who'll go write the letter themselves. 📬

---

## THEORETICAL Q&A

### Q1. What are the `&` and `*` operators?

```c
int x = 1;
int *ip;          /* ip is a pointer to int */

ip = &x;          /* & gives address of x — ip now "points to" x */
int y = *ip;      /* * dereferences ip — y gets value of x (1) */
*ip = 0;          /* sets x to 0 via the pointer */
```

- `&` (address-of): gives the memory address of a variable. Only applies to objects in memory — NOT constants, expressions, or `register` variables.
- `*` (dereference/indirection): gives the value at the address stored in a pointer.

**Declaration syntax reflects usage:**
```c
int *ip;           /* *ip is an int, so ip is pointer-to-int */
double *dp, atof(char *);  /* *dp is double; atof returns double, takes char* */
```

---

### Q2. What is pointer arithmetic? What operations are valid?

Pointer arithmetic is **scaled** — arithmetic on a pointer accounts for the size of the pointed-to type.

```c
int arr[10];
int *pa = &arr[0];   /* or: pa = arr; */

pa + 1;   /* points to arr[1] — NOT 1 byte forward, but sizeof(int) bytes forward */
pa + i;   /* points to arr[i] */
*(pa + i) /* same as arr[i] */
```

**Valid pointer operations:**
1. **Assignment** between same-type pointers: `p = q;`
2. **Adding/subtracting an integer:** `p + n`, `p - n`, `p++`, `p--`, `p += n`
3. **Subtracting two pointers** to elements of the SAME array: `p - q` gives the number of elements between them (type `ptrdiff_t`)
4. **Comparing** pointers to elements of the SAME array: `p < q`, `p == q`, etc.
5. **Comparing or assigning** with `NULL` (zero): `if (p == NULL)`

**INVALID operations:**
- Adding two pointers
- Multiplying, dividing, shifting pointers
- Adding `float` or `double` to pointers
- Pointer arithmetic between different arrays (undefined behavior)

---

### Q3. What is the relationship between arrays and pointers?

This is THE big C concept — nail it! 🎯

```c
int a[10];
int *pa = a;    /* pa = &a[0] — array name is pointer to first element */
```

**These are IDENTICAL:**
```c
a[i]       ←→    *(a + i)
&a[i]      ←→    a + i
pa[i]      ←→    *(pa + i)
```

**Key difference** — an array name is NOT a variable:
```c
int a[10];
int *pa = a;   /* OK */
pa = a;        /* OK — pa can be reassigned */
pa++;          /* OK — pa advances to next element */

a = pa;        /* ERROR — array name is not an lvalue! */
a++;           /* ERROR — illegal */
```

Think of `a` as a **constant pointer** — it always points to `a[0]` and you can't change that.

**When passed to a function, arrays DECAY to pointers:**
```c
void f(int arr[], int n)   /* arr is actually int *arr */
/* sizeof(arr) inside f = sizeof(int*), NOT sizeof the array! */
```

---

### Q4. What is a NULL pointer? What is `void *`?

**NULL pointer:** a pointer guaranteed not to point to any object.
```c
int *p = NULL;   /* or: int *p = 0; */
if (p == NULL)   /* testing if pointer is null */
    ...
```
- `NULL` is defined in `<stdio.h>`, `<stdlib.h>`, etc.
- Dereferencing a null pointer = **undefined behavior** (typically a crash/segfault).
- C guarantees zero is never a valid data address, so 0 means "no object here."

**`void *` — generic pointer:**
```c
void *malloc(size_t n);   /* returns pointer to untyped memory */
```
- A `void *` can hold any pointer type.
- Can be assigned to/from any pointer without a cast (in C, not C++).
- Cannot be dereferenced directly — must cast first.
```c
int *p = malloc(sizeof(int) * 10);   /* void* → int* implicit conversion */
```

---

### Q5. How do you pass pointer arguments to functions? (The `swap` example)

```c
/* WRONG — call by value, doesn't affect caller's variables */
void swap_wrong(int x, int y) {
    int temp = x; x = y; y = temp;   /* swaps LOCAL copies only */
}

/* CORRECT — pass addresses, modify through pointers */
void swap(int *px, int *py) {
    int temp = *px;
    *px = *py;
    *py = temp;
}

int main() {
    int a = 3, b = 7;
    swap(&a, &b);   /* pass addresses */
    /* a = 7, b = 3 now */
}
```

**Pattern for functions that need to "return" multiple values:**
```c
/* scanf uses this pattern! */
int getint(int *pn) {
    /* reads an integer and stores it via pn */
    /* returns EOF, 0 (no number), or positive (got a number) */
    *pn = 42;
    return 1;
}

int main() {
    int val;
    if (getint(&val) > 0)
        printf("Got: %d\n", val);
}
```

---

### Q6. What are pointers to functions? Why are they useful?

A function name, like an array name, is a **pointer to the function**.

```c
int (*cmp)(void *, void *);   /* cmp is a POINTER to a function taking two void* and returning int */
```

Reading complex declarations: start at the name, go right then left:
- `cmp` → is a
- `(...)` → pointer to a function with params `(void*, void*)`
- `int` → returning int

```c
/* Pass a comparison function to qsort */
int numcmp(const char *s1, const char *s2) {
    return atof(s1) - atof(s2) > 0 ? 1 : (atof(s1) == atof(s2) ? 0 : -1);
}

qsort(lineptr, 0, nlines-1, 
      (int (*)(void*, void*)) strcmp);     /* lexicographic */
/* or */
qsort(lineptr, 0, nlines-1,
      (int (*)(void*, void*)) numcmp);     /* numeric */
```

**Why useful:** You can change the behavior of a function at runtime without changing its code — that's **polymorphism in C**!

---

### Q7. What is pointer comparison and pointer subtraction?

```c
int arr[10];
int *p = &arr[2], *q = &arr[7];

q - p;     /* 5 — number of elements between them (ptrdiff_t) */
p < q;     /* 1 (true) — p points to earlier element */
p == q;    /* 0 (false) */
```

**Valid comparisons:**
- Between pointers to elements of **same array** (including one-past-the-end).
- Any pointer vs `NULL`.

**Invalid comparison:** between pointers to different arrays — undefined behavior.

**The one-past-the-end idiom (guaranteed valid):**
```c
int arr[10];
for (int *p = arr; p < arr + 10; p++)   /* arr+10 is valid to compare, NOT to dereference */
    *p = 0;
```

---

### Q8. Explain `*p++` vs `(*p)++` vs `*++p`.

Postfix `++` has higher precedence than `*`, so:

```c
*p++   /* equivalent to *(p++) — fetch *p, then advance p */
(*p)++ /* increment the value pointed to by p */
*++p   /* equivalent to *(++p) — advance p first, then fetch *p */
++*p   /* equivalent to ++(*p) — increment value at p */
```

**The classic push/pop stack idiom:**
```c
*p++ = val;      /* PUSH: store val at *p, advance p (top of stack moves up) */
val = *--p;      /* POP: back p up, fetch value (top of stack moves down) */
```

---

### Q9. What are `const` pointers and pointers to `const`?

```c
int i = 10;

const int *pc = &i;       /* pointer to const int — can't change *pc, but can change pc itself */
*pc = 20;                 /* ERROR — can't modify through pc */
pc = &another_int;        /* OK — pc can point elsewhere */

int * const cp = &i;      /* const pointer to int — can't change cp, but can change *cp */
*cp = 20;                 /* OK — can modify the int */
cp = &another_int;        /* ERROR — cp is locked to &i */

const int * const ccp = &i;   /* const pointer to const int — nothing can change */
```

**Memory trick:** 
- Read right to left: `const int *pc` = "pc is a pointer to (const int)"
- `int *const cp` = "cp is a const pointer to int"

**In function parameters:**
```c
void strcpy(char *s, const char *t)   /* t won't be modified — good practice */
```

---

## EXERCISES WITH SOLUTIONS

### Exercise 5-1. Fix `getint` to push back `+` or `-` not followed by a digit.
```c
#include <ctype.h>

int getch(void);
void ungetch(int);

int getint(int *pn) {
    int c, sign;

    while (isspace(c = getch()))
        ;

    if (!isdigit(c) && c != EOF && c != '+' && c != '-') {
        ungetch(c);
        return 0;
    }

    sign = (c == '-') ? -1 : 1;
    if (c == '+' || c == '-') {
        int next = getch();
        if (!isdigit(next)) {    /* +/- not followed by digit */
            ungetch(next);
            ungetch(c);          /* push back the sign too */
            return 0;
        }
        c = next;
    }

    for (*pn = 0; isdigit(c); c = getch())
        *pn = 10 * *pn + (c - '0');
    *pn *= sign;

    if (c != EOF) ungetch(c);
    return c;
}
```

---

### Exercise 5-3. Pointer version of `strcat`.
```c
void strcat_ptr(char *s, const char *t) {
    while (*s)     /* advance s to end of string */
        s++;
    while ((*s++ = *t++))  /* copy t — includes copying the '\0' */
        ;
}
```

---

### Exercise 5-4. `strend(s, t)` — does t occur at end of s?
```c
int strend(const char *s, const char *t) {
    int ls = strlen(s), lt = strlen(t);
    if (lt > ls) return 0;
    return strcmp(s + ls - lt, t) == 0;
}
```

---

### Exercise 5-5. `strncpy`, `strncat`, `strncmp`.
```c
char *strncpy(char *s, const char *t, int n) {
    char *start = s;
    while (n-- > 0 && (*s++ = *t++))
        ;
    while (n-- > 0)        /* pad with '\0' if t shorter than n */
        *s++ = '\0';
    return start;
}

char *strncat(char *s, const char *t, int n) {
    char *start = s;
    while (*s) s++;        /* find end of s */
    while (n-- > 0 && (*s = *t)) { s++; t++; }
    *s = '\0';
    return start;
}

int strncmp(const char *s, const char *t, int n) {
    for (; n > 0 && *s == *t; s++, t++, n--)
        if (*s == '\0') return 0;
    return (n == 0) ? 0 : (unsigned char)*s - (unsigned char)*t;
}
```

---

### Exercise 5-6. Pointer version of `getline`.
```c
int getline(char *s, int lim) {
    int c;
    char *start = s;

    while (--lim > 0 && (c = getchar()) != EOF && c != '\n')
        *s++ = c;
    if (c == '\n')
        *s++ = c;
    *s = '\0';
    return s - start;
}
```

---

### Exercise 5-7. Pointer version of `reverse`.
```c
void reverse(char *s) {
    char *end = s + strlen(s) - 1;
    while (s < end) {
        char temp = *s;
        *s++ = *end;
        *end-- = temp;
    }
}
```

---

### Key Interview Traps 🪤

**Trap 1: Double free / use-after-free**
```c
int *p = malloc(sizeof(int));
free(p);
*p = 5;   /* USE AFTER FREE — undefined behavior, often a crash or security hole */
free(p);  /* DOUBLE FREE — undefined behavior */
```
After freeing: `p = NULL;` immediately.

**Trap 2: Array vs pointer sizeof**
```c
char arr[] = "hello";   /* arr is an array — sizeof(arr) = 6 */
char *ptr = "hello";    /* ptr is a pointer — sizeof(ptr) = 8 (on 64-bit) */
```

**Trap 3: `char *p = "hello"; p[0] = 'H';`**
String literals are read-only. This is **undefined behavior** (usually a segfault). Use `char p[] = "hello";` for a mutable copy.

**Trap 4: Returning pointer to local stack variable**
```c
int *bad(void) {
    int x = 42;
    return &x;   /* x is gone after function returns — dangling pointer */
}
```

**Trap 5: Pointer to pointer confusion**
```c
void getmem(int **pp, int n) {
    *pp = malloc(n * sizeof(int));   /* need pointer-to-pointer to change caller's pointer */
}
int *arr;
getmem(&arr, 10);   /* pass ADDRESS of the pointer */
```

**Trap 6: The off-by-one in pointer loops**
```c
/* CORRECT — arr + n is one PAST the end, valid for comparison but NOT dereference */
for (int *p = arr; p < arr + n; p++)
    *p = 0;   /* OK — never dereferences arr+n */

/* WRONG */
for (int *p = arr; p <= arr + n; p++)   /* dereferences arr+n — off by one! */
```
