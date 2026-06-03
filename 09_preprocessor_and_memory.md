# C Interview Prep — File 9: Preprocessor, Memory Management & Advanced Topics

> The C preprocessor runs before the compiler and does pure text substitution. It's like a find-and-replace tool on steroids 💪 — powerful and dangerous. And `malloc`/`free` are like renting memory — you are responsible for returning it, or you get a memory leak bill. 💸

---

## THEORETICAL Q&A

### Q1. What is the C preprocessor and what does it do?

The preprocessor is a **separate first step** before compilation. It operates on text, not C syntax. It:
1. Replaces trigraph sequences (`??=` → `#`)
2. Splices lines ending with `\`
3. Handles `#include`, `#define`, `#ifdef`, etc.
4. Replaces macro names with their expansions
5. Strips comments

**Key point:** The compiler never sees the original source — it sees the preprocessed output.

```bash
# See preprocessor output:
gcc -E myfile.c -o myfile.i
```

---

### Q2. How does `#define` work? What are the rules?

**Simple (object-like) macros:**
```c
#define PI         3.14159265
#define MAXBUF     1024
#define FOREVER    for(;;)
#define DEBUG      1

/* Usage */
double area = PI * r * r;
char buf[MAXBUF];
FOREVER { ... }   /* expands to: for(;;) { ... } */
```

**Rules:**
- Scope: from `#define` to end of file (or `#undef`).
- No semicolon at end.
- Substitution is **textual** — happens before parsing.
- No substitution inside string literals or comments.
- Recursive macros are not expanded again.

**Undefine:**
```c
#undef getchar
int getchar(void) { ... }   /* redefine as real function */
```

---

### Q3. What are function-like macros? What are their pitfalls?

```c
#define max(A, B)    ((A) > (B) ? (A) : (B))
#define min(A, B)    ((A) < (B) ? (A) : (B))
#define square(x)    ((x) * (x))
#define ABS(x)       ((x) < 0 ? -(x) : (x))
```

**Why all those parentheses?** Without them, operator precedence bites you:
```c
#define square(x) x * x
square(3+1)   /* → 3+1 * 3+1 = 3+3+1 = 7 — WRONG! */
              /* should be (3+1)*(3+1) = 16 */

#define max(A, B) A > B ? A : B
x = max(a, b) + 1;  /* → x = a > b ? a : b + 1  — WRONG! */
```

**Rule:** Always wrap each argument in parens, and the whole macro in parens.

**The double-evaluation problem:**
```c
#define max(A, B)  ((A) > (B) ? (A) : (B))

int i = 5, j = 3;
int m = max(i++, j++);  /* WRONG — i++ or j++ evaluated TWICE! */
/* expands to: ((i++) > (j++) ? (i++) : (j++)) */
```
With function: `i++` evaluated once. With macro: may be evaluated twice.

**The multiline macro:**
```c
#define SWAP(t, a, b)  do { t _tmp = (a); (a) = (b); (b) = _tmp; } while(0)
/* do-while(0) trick: makes it safe to use in if/else without braces */

if (x > y)
    SWAP(int, x, y);   /* expands to a single statement */
else
    ...                /* no dangling else problem */
```

---

### Q4. What are the `#` and `##` operators in macros?

**`#` (stringification):** converts a macro argument to a string literal
```c
#define STRINGIFY(x)  #x
STRINGIFY(hello)      /* → "hello" */
STRINGIFY(3+1)        /* → "3+1" */

/* Debug print macro */
#define PRINT_VAR(x)  printf(#x " = %d\n", (x))
PRINT_VAR(count);     /* → printf("count" " = %d\n", count); */
                      /* → printf("count = %d\n", count); */
```

**`##` (token pasting/concatenation):** concatenates two tokens
```c
#define PASTE(front, back)  front ## back
PASTE(var, 1)               /* → var1 */

/* Generate function names: */
#define MAKE_FUNC(type) type##_print
MAKE_FUNC(int)              /* → int_print */
```

---

### Q5. What are the conditional compilation directives?

```c
#if constant-expression     /* if expression is non-zero */
#elif constant-expression   /* else-if */
#else                       /* else */
#endif                      /* required closing */

#ifdef NAME                 /* equivalent to: #if defined(NAME) */
#ifndef NAME                /* equivalent to: #if !defined(NAME) */
```

**Include guard — prevent double inclusion (essential in header files!):**
```c
/* myheader.h */
#ifndef MYHEADER_H
#define MYHEADER_H

/* ... all header content ... */

#endif  /* MYHEADER_H */
```

**Platform-specific code:**
```c
#if defined(_WIN32)
    #include <windows.h>
    #define SLEEP(ms) Sleep(ms)
#elif defined(__unix__)
    #include <unistd.h>
    #define SLEEP(ms) usleep((ms)*1000)
#else
    #error "Unsupported platform"
#endif
```

**Debug builds:**
```c
#define DEBUG 1

#ifdef DEBUG
    #define DPRINT(fmt, ...) fprintf(stderr, "[DEBUG] " fmt, ##__VA_ARGS__)
#else
    #define DPRINT(fmt, ...)   /* empty — compiled away in release */
#endif
```

---

### Q6. What are predefined macros?

```c
__FILE__    /* current source filename as string literal */
__LINE__    /* current line number as integer constant */
__DATE__    /* compilation date: "Mmm dd yyyy" */
__TIME__    /* compilation time: "hh:mm:ss" */
__STDC__    /* 1 if standard conforming implementation */
```

**Practical use — better error messages:**
```c
#define ASSERT(cond)  do { \
    if (!(cond)) { \
        fprintf(stderr, "Assertion failed: %s, file %s, line %d\n", \
                #cond, __FILE__, __LINE__); \
        abort(); \
    } \
} while(0)
```

---

### Q7. How does dynamic memory allocation work? (`malloc`, `calloc`, `realloc`, `free`)

```c
#include <stdlib.h>

/* malloc: allocate n bytes, UNINITIALIZED — returns NULL on failure */
void *malloc(size_t n);

/* calloc: allocate n*size bytes, INITIALIZED TO ZERO */
void *calloc(size_t n, size_t size);

/* realloc: resize existing allocation */
void *realloc(void *ptr, size_t newsize);

/* free: release allocation — passing NULL is safe (does nothing) */
void free(void *ptr);
```

**Usage patterns:**
```c
/* Allocate an array */
int *arr = malloc(n * sizeof(int));
if (arr == NULL) { perror("malloc"); exit(1); }

/* Zero-initialized */
int *arr = calloc(n, sizeof(int));   /* all zeros */

/* Grow an array */
arr = realloc(arr, new_n * sizeof(int));
if (arr == NULL) { /* old allocation still valid — but we lost the pointer! */ }
/* SAFE realloc: */
int *tmp = realloc(arr, new_n * sizeof(int));
if (tmp == NULL) { free(arr); exit(1); }
arr = tmp;

/* Free */
free(arr);
arr = NULL;   /* good habit — prevents use-after-free bugs */
```

**Allocate a struct:**
```c
struct node *p = malloc(sizeof(*p));    /* sizeof(*p) = sizeof(struct node) */
/* or: malloc(sizeof(struct node)) — same thing */
if (!p) { exit(1); }
p->val = 42;
p->next = NULL;
```

---

### Q8. What is `strdup`? Why is it convenient?

`strdup` is not in standard C89 (it's POSIX), but widely available. K&R implement it:
```c
char *strdup(const char *s) {
    char *p = malloc(strlen(s) + 1);   /* +1 for '\0' */
    if (p != NULL)
        strcpy(p, s);
    return p;   /* returns NULL if malloc failed */
}
```

**Usage:** copy a string to heap memory (so it persists beyond the original's scope):
```c
char *name = strdup("Alice");   /* heap-allocated copy */
/* ... use name ... */
free(name);   /* must free when done */
```

---

### Q9. What are common memory errors and how do you detect them?

| Error | Description | Consequence |
|-------|-------------|-------------|
| **Buffer overflow** | Write past end of array | Corruption, crashes, security holes |
| **Use-after-free** | Access freed memory | Undefined behavior, usually crash |
| **Double free** | `free` same pointer twice | Heap corruption |
| **Memory leak** | Lose pointer without freeing | Memory exhaustion |
| **Null dereference** | Use pointer before checking for NULL | Segfault |
| **Uninitialized read** | Read from uninitialized memory | Garbage values |

**Detection tools:**
```bash
valgrind --leak-check=full ./myprogram   # Detects leaks, use-after-free, etc.
gcc -fsanitize=address ./myprogram       # AddressSanitizer (faster than valgrind)
gcc -fsanitize=undefined ./myprogram     # UBSanitizer
```

---

### Q10. What is the `volatile` qualifier? When to use it?

```c
volatile int device_register;   /* tells compiler: don't optimize away reads/writes */
```

`volatile` says: **"this variable may change by something the compiler doesn't know about"** — hardware interrupts, other threads, signals.

Without `volatile`, the compiler might:
- Cache the value in a register and never re-read from memory
- Optimize away seemingly redundant reads

**Use cases:**
```c
/* Memory-mapped I/O register */
volatile unsigned int *status_reg = (volatile unsigned int *)0xDEAD0000;
while (*status_reg & BUSY_BIT)   /* must re-read every iteration */
    ;

/* Flag set by signal handler */
volatile sig_atomic_t interrupted = 0;
void handler(int sig) { interrupted = 1; }
while (!interrupted) { ... }
```

---

## EXERCISES WITH SOLUTIONS

### Exercise 4-14. Macro `swap(t, x, y)`.
```c
#define swap(t, a, b)  do { t _tmp = (a); (a) = (b); (b) = _tmp; } while(0)

/* Usage */
int x = 3, y = 7;
swap(int, x, y);
/* x = 7, y = 3 */

char *s = "hello", *t = "world";
swap(char *, s, t);
/* s = "world", t = "hello" */
```
The `do { } while(0)` trick makes it a single statement that requires a `;` — behaves correctly after `if` without braces.

---

### Exercise: Implement a dynamic array (growable buffer).
```c
#include <stdlib.h>
#include <string.h>

typedef struct {
    int *data;
    int size;
    int capacity;
} IntArray;

void ia_init(IntArray *a) {
    a->data = malloc(4 * sizeof(int));
    a->size = 0;
    a->capacity = 4;
}

void ia_push(IntArray *a, int val) {
    if (a->size == a->capacity) {
        a->capacity *= 2;
        int *tmp = realloc(a->data, a->capacity * sizeof(int));
        if (!tmp) { free(a->data); exit(1); }
        a->data = tmp;
    }
    a->data[a->size++] = val;
}

void ia_free(IntArray *a) {
    free(a->data);
    a->data = NULL;
    a->size = a->capacity = 0;
}
```

---

### Exercise: Safe string copy function.
```c
/* Like strncpy but ALWAYS null-terminates, and returns number of chars copied */
int safe_strcpy(char *dst, const char *src, int dstsize) {
    if (dstsize <= 0) return 0;
    int i;
    for (i = 0; i < dstsize - 1 && src[i]; i++)
        dst[i] = src[i];
    dst[i] = '\0';
    return i;
}
```

---

### Key Interview Traps 🪤

**Trap 1: Macro vs function — argument evaluation count**
```c
int i = 0;
int x = square(i++);  /* square is a MACRO — i++ evaluated TWICE! → x=0, i=2 */
int y = square_fn(i); /* square_fn is a FUNCTION — i evaluated ONCE → y=4, i=1 */
```

**Trap 2: Missing parens in macro**
```c
#define DOUBLE(x) x+x    /* WRONG */
int y = DOUBLE(3) * 2;  /* → 3+3 * 2 = 3 + 6 = 9, not 12! */

#define DOUBLE(x) ((x)+(x))  /* CORRECT */
```

**Trap 3: `realloc` memory leak on failure**
```c
/* WRONG — if realloc fails, original ptr is lost! */
arr = realloc(arr, newsize);   

/* CORRECT */
void *tmp = realloc(arr, newsize);
if (tmp == NULL) { /* handle error, arr is still valid */ return; }
arr = tmp;
```

**Trap 4: `free(NULL)` is safe, but `free` of non-heap pointer is not**
```c
int x = 5;
int *p = &x;
free(p);         /* UNDEFINED — p doesn't point to malloc'd memory */

char *s = "hello";
free(s);         /* UNDEFINED — string literal, not heap */

int *q = malloc(4);
free(q);         /* OK */
free(q);         /* DOUBLE FREE — undefined behavior */
```

**Trap 5: Macro stringification pitfall**
```c
#define STR(x) #x
#define XSTR(x) STR(x)    /* force expansion before stringification */

#define VERSION 42
STR(VERSION)   /* → "VERSION" (not expanded!) */
XSTR(VERSION)  /* → "42" (expanded first, then stringified) */
```

**Trap 6: Include guards vs `#pragma once`**
```c
#pragma once    /* Non-standard but widely supported — simpler */
/* vs standard: */
#ifndef HEADER_H
#define HEADER_H
/* ... */
#endif
/* Both prevent multiple inclusion. Use include guards for maximum portability. */
```
