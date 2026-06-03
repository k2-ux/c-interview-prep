# C Interview Prep — File 11: Modern C (C99 → C23)

> K&R's book is from 1988. C has had FOUR major revisions since then. Writing C89 in 2024 is like driving a car without seatbelts — it works, but why? Let's catch up. 🚀

---

## THE TIMELINE AT A GLANCE

```
C89/C90  — The K&R standard. What the book teaches.
C99      — Big update. Most important for day-to-day coding.
C11      — Threading, atomics, generics. Modernised the language.
C17/C18  — Bugfix release. Nothing exciting, but fixes matter.
C23      — Latest. Borrows good ideas from C++. Ships in compilers now.
```

**Check what standard your compiler uses:**
```bash
gcc --std=c99 file.c     # compile as C99
gcc --std=c11 file.c     # compile as C11
gcc --std=c17 file.c     # compile as C17
gcc --std=c23 file.c     # compile as C23 (GCC 13+)
gcc --std=gnu17 file.c   # C17 + GNU extensions (common default)
```

---

## C99 — THE BIG ONE 🎉

### 1. `//` Single-line comments
```c
int x = 5;  // This is now legal in C! (Was C++ only before C99)
/* This still works too */
```

### 2. `<stdint.h>` — Guaranteed-size integer types
No more guessing if `int` is 16 or 32 bits. Use these in any code that cares about exact sizes:

```c
#include <stdint.h>

int8_t    a;   /* exactly  8 bits, signed  */
uint8_t   b;   /* exactly  8 bits, unsigned */
int16_t   c;   /* exactly 16 bits, signed  */
uint16_t  d;   /* exactly 16 bits, unsigned */
int32_t   e;   /* exactly 32 bits, signed  */
uint32_t  f;   /* exactly 32 bits, unsigned */
int64_t   g;   /* exactly 64 bits, signed  */
uint64_t  h;   /* exactly 64 bits, unsigned */

/* Fastest types (at least N bits, whatever's fastest on this CPU): */
int_fast32_t  fast;
uint_fast64_t fast_u;

/* Pointer-sized integer (holds any pointer): */
intptr_t  ptr_val = (intptr_t)some_pointer;
uintptr_t ptr_u   = (uintptr_t)some_pointer;

/* Size and pointer difference: */
size_t    n = sizeof(int);    /* also in <stddef.h> */
ptrdiff_t d = p2 - p1;       /* difference between pointers */
```

**Interview rule:** If you're writing portable code or talking to hardware, always use `uint32_t` etc. instead of `unsigned int`. Shows you know what you're doing. 🎯

### 3. `<stdbool.h>` — Boolean type
```c
#include <stdbool.h>

bool flag = true;
bool done = false;

if (flag) {
    done = true;
}

/* Internally: bool = _Bool, true = 1, false = 0 */
/* _Bool is the actual C keyword; bool/true/false are macros from the header */
```
In C23, `bool`, `true`, `false` become proper keywords — no header needed!

### 4. `long long` type
```c
long long x = 9223372036854775807LL;    /* max int64 */
unsigned long long y = 18446744073709551615ULL;

printf("%lld\n",  x);   /* %lld for long long */
printf("%llu\n",  y);   /* %llu for unsigned long long */
```

### 5. Variable-Length Arrays (VLAs) 🌊
Arrays whose size is determined at runtime — on the stack:
```c
void process(int n) {
    int arr[n];   /* size known at runtime — C99! */
    /* ... */
}   /* arr freed automatically when function returns */
```

**Pros:** No malloc/free, simpler code.  
**Cons:** Stack overflow risk for large n. Not in C++. Made **optional** in C11 (implementations don't have to support them).  
**Rule:** Use VLAs only for small, known-bounded sizes. For anything large, use `malloc`.

### 6. Designated Initializers 🎯
Initialize specific members by name — order doesn't matter:
```c
struct point {
    int x, y, z;
};

/* Old C89 way — order must match struct */
struct point p = {1, 2, 3};

/* C99 designated initializer — clear and order-independent */
struct point p = { .y = 2, .x = 1, .z = 3 };
struct point q = { .x = 5 };   /* y and z default to 0 */

/* Also works for arrays! */
int arr[6] = { [0] = 1, [3] = 10, [5] = 100 };
/* arr = {1, 0, 0, 10, 0, 100} */

/* Combine both for nested structs */
struct rect r = {
    .pt1 = { .x = 0, .y = 0 },
    .pt2 = { .x = 10, .y = 10 }
};
```

### 7. Compound Literals
Create temporary objects inline — think of them as anonymous variables:
```c
struct point p = (struct point){.x = 3, .y = 4};

/* Pass struct directly to a function without a named variable */
draw_point((struct point){.x = 10, .y = 20});

/* Temporary array */
int sum = add_array((int[]){1, 2, 3, 4, 5}, 5);
```

### 8. Mixed Declarations and Code
In C89, all declarations had to come before any statements in a block. C99 removes this restriction:
```c
/* C89 — declarations must come first */
{
    int i, j;
    /* ... only statements after this ... */
    i = 0;
}

/* C99 — declare anywhere */
{
    for (int i = 0; i < n; i++) {   /* i declared inside for! */
        int temp = arr[i] * 2;      /* declared mid-block */
        /* ... */
    }
    /* i is not accessible here */
}
```
Declaring variables close to first use = better readability. This is why modern C looks much more like C++.

### 9. `snprintf` — The safe printf 🛡️
```c
char buf[64];

/* Old dangerous way */
sprintf(buf, "Value: %d", x);   /* BUFFER OVERFLOW if result > 63 chars */

/* C99 safe way */
snprintf(buf, sizeof(buf), "Value: %d", x);   /* ALWAYS null-terminates, never overflows */
```
`snprintf` returns the number of characters that WOULD have been written (excluding `\0`). Compare to `sizeof(buf)` to detect truncation.

### 10. `restrict` — Pointer aliasing hint
```c
void add_arrays(int * restrict a, int * restrict b, int * restrict c, int n) {
    for (int i = 0; i < n; i++)
        c[i] = a[i] + b[i];
}
```
`restrict` promises the compiler that pointers don't alias (point to overlapping memory). This lets the compiler optimise aggressively. Violating the promise = undefined behaviour. Used in standard library: `memcpy(void * restrict dst, const void * restrict src, size_t n)`.

### 11. `inline` functions
```c
static inline int max(int a, int b) {
    return a > b ? a : b;
}
```
Hints the compiler to expand the function body at call sites (no function call overhead). Better than macros because:
- Type-checked ✅
- No double-evaluation of arguments ✅
- Can be debugged ✅
- Still just a hint — compiler may ignore it

### 12. `__func__` — Current function name
```c
void my_function(void) {
    printf("Currently in: %s\n", __func__);  /* prints "my_function" */
}

/* Great for debug logging: */
#define LOG(msg) printf("[%s:%d] %s\n", __func__, __LINE__, msg)
```

### 13. Flexible Array Members
A struct can have an array of unspecified size as its LAST member:
```c
struct packet {
    int   length;
    char  data[];   /* flexible array member — no size! */
};

/* Allocate with enough space for the data */
int n = 100;
struct packet *pkt = malloc(sizeof(struct packet) + n * sizeof(char));
pkt->length = n;
pkt->data[0] = 'H';   /* valid — we allocated the space */
```

---

## C11 — THREADING AND GENERICS 🧵

### 1. `_Static_assert` — Compile-time assertions
Catch bugs at compile time, not runtime:
```c
_Static_assert(sizeof(int) == 4, "int must be 4 bytes on this platform");
_Static_assert(sizeof(long) >= 8, "long must be at least 8 bytes");

/* If the condition is false, compilation FAILS with the message */
```
In C23 you can just write `static_assert(...)` without the underscore.

### 2. `_Generic` — Type-generic expressions
C's way of "overloading" by type — evaluated at compile time:
```c
#define print_value(x) _Generic((x),    \
    int:    printf("%d\n", x),          \
    float:  printf("%f\n", x),          \
    double: printf("%lf\n", x),         \
    char*:  printf("%s\n", x),          \
    default: printf("unknown type\n")   \
)

print_value(42);       /* selects int branch */
print_value(3.14);     /* selects double branch */
print_value("hello");  /* selects char* branch */
```

### 3. Anonymous Structs and Unions
```c
struct Token {
    int type;
    union {              /* anonymous union — no name needed */
        int   ival;
        float fval;
        char *sval;
    };                   /* members accessed directly: token.ival, not token.u.ival */
};

struct Token t = { .type = 1, .ival = 42 };
printf("%d\n", t.ival);   /* no .u. needed! */
```

### 4. `<stdatomic.h>` — Atomic operations
Thread-safe operations without explicit locks:
```c
#include <stdatomic.h>

atomic_int counter = 0;

/* These are thread-safe — no race condition */
atomic_fetch_add(&counter, 1);
int val = atomic_load(&counter);
atomic_store(&counter, 0);

/* Atomic compare-and-swap */
int expected = 5;
atomic_compare_exchange_strong(&counter, &expected, 10);
/* If counter == 5, sets it to 10 and returns true */
```

### 5. `<threads.h>` — Standard threading (C11)
```c
#include <threads.h>

int worker(void *arg) {
    printf("Thread running!\n");
    return 0;
}

int main() {
    thrd_t t;
    thrd_create(&t, worker, NULL);
    thrd_join(t, NULL);
    return 0;
}
```
Note: In practice, most code still uses `pthreads` (POSIX) on Linux/Mac or Windows threads — platform libraries are more mature and featureful.

### 6. `_Alignas` and `_Alignof`
```c
#include <stdalign.h>   /* provides alignas, alignof macros */

/* Force 16-byte alignment (for SIMD instructions) */
alignas(16) float vector[4];

/* Query alignment requirement */
printf("int alignment: %zu\n", alignof(int));   /* typically 4 */
printf("double alignment: %zu\n", alignof(double)); /* typically 8 */
```

### 7. `_Thread_local` — Per-thread variables
```c
#include <threads.h>

_Thread_local int thread_id = 0;   /* each thread gets its own copy */

/* In C23: thread_local (without underscore) */
```

### 8. `_Noreturn` — Functions that never return
```c
#include <stdnoreturn.h>   /* provides noreturn macro */

noreturn void fatal_error(const char *msg) {
    fprintf(stderr, "FATAL: %s\n", msg);
    exit(1);
    /* never reaches here */
}
/* Lets compiler know it doesn't need to worry about return paths */
```

---

## C17/C18 — THE BUGFIX RELEASE 🔧

No new features — just defect reports and clarifications from C11. The most useful thing C17 did was confirm that `__STDC_VERSION__` is `201710L`.

```c
#if __STDC_VERSION__ >= 201710L
    /* C17 or later */
#elif __STDC_VERSION__ >= 201112L
    /* C11 or later */
#elif __STDC_VERSION__ >= 199901L
    /* C99 or later */
#else
    /* C89 */
#endif
```

---

## C23 — THE FUTURE IS NOW ✨

C23 is the newest standard, shipping in GCC 13+, Clang 16+.

### 1. `nullptr` — A proper null pointer constant
```c
/* C23: nullptr has type nullptr_t */
int *p = nullptr;   /* cleaner than NULL or 0 */

/* NULL was always a bit awkward — it's typically ((void*)0) */
/* nullptr is unambiguously a pointer null, not integer 0 */
```

### 2. `bool`, `true`, `false` as keywords (no header!)
```c
/* C23: these are keywords — no #include <stdbool.h> needed */
bool flag = true;
bool done = false;
```

### 3. `typeof()` — Get the type of an expression
```c
int x = 5;
typeof(x) y = 10;   /* y is also int */

/* Safe generic max macro — no double evaluation! */
#define SAFE_MAX(a, b) ({           \
    typeof(a) _a = (a);             \
    typeof(b) _b = (b);             \
    _a > _b ? _a : _b;              \
})
```

### 4. `auto` — Type inference
```c
auto x = 42;          /* int */
auto f = 3.14;        /* double */
auto p = malloc(10);  /* void* */

/* Mainly useful for long type names */
auto it = some_complex_function();
```
Note: C's `auto` is simpler than C++'s. It just infers the type from the initializer.

### 5. `constexpr` — Compile-time constants
```c
constexpr int MAX = 100;       /* evaluated at compile time */
constexpr double PI = 3.14159;

/* Better than #define — it's typed and scoped */
/* Better than const — guaranteed compile-time evaluation */
```

### 6. `_BitInt(N)` — Bit-precise integers
```c
_BitInt(7)  small = 63;    /* 7-bit signed integer */
_BitInt(128) big  = 1;     /* 128-bit integer! */
unsigned _BitInt(256) huge;  /* 256-bit unsigned */
```

### 7. `[[]] ` Attributes (C23 standardises them)
```c
[[nodiscard]] int important_result(void);   /* warn if caller ignores return */
[[deprecated("use new_func instead")]] void old_func(void);
[[noreturn]] void crash(void);
[[maybe_unused]] int x;   /* suppress unused variable warning */
[[fallthrough]];           /* in switch — marks intentional fall-through */
```

---

## QUICK REFERENCE: WHAT TO USE WHEN

| Task | Before C99 | C99+ (Modern) |
|------|-----------|--------------|
| Boolean | `int flag = 0/1` | `bool flag = true/false` |
| Exact int sizes | `unsigned long` (guessing) | `uint32_t`, `int64_t` |
| Safe string format | `sprintf` (unsafe) | `snprintf` |
| Loop variable | Declare before loop | `for (int i = 0; ...)` |
| Struct init | `{0, 1, 2}` (positional) | `{.x=0, .y=1}` (named) |
| Compile check | `assert()` (runtime) | `_Static_assert()` (compile) |
| Null pointer | `NULL` or `0` | `nullptr` (C23) |
| Type query | Not possible | `typeof()` (C23) |

---

## INTERVIEW QUESTIONS ON MODERN C

**Q: What's the difference between `const int *p` and `int * const p`?**
A: `const int *p` — the pointed-to value is const (can't do `*p = 5`). `int * const p` — the pointer itself is const (can't do `p = &other`). C89 had both, but `restrict` (C99) adds a third qualifier meaning "this pointer doesn't alias."

**Q: Why prefer `int32_t` over `int` for portable code?**
A: `int` can be 16, 32, or 64 bits depending on the platform/compiler. `int32_t` from `<stdint.h>` is *always* exactly 32 bits. Critical for network protocols, file formats, hardware registers.

**Q: What's wrong with VLAs for large arrays?**
A: VLAs allocate on the **stack**. Stack is typically 1–8 MB. `int arr[1000000]` on the stack = **stack overflow**. Use `malloc` for large or runtime-sized arrays. Also, VLAs are optional in C11 — embedded compilers may not support them.

**Q: What does `_Static_assert(sizeof(struct Foo) == 8, "wrong size")` do?**
A: Causes a **compile error** with the message "wrong size" if the struct's size isn't 8 bytes. Zero runtime cost. Essential for ABI-sensitive code (e.g., serialisation, hardware structs).

**Q: What's `_Generic` and why is it useful?**
A: It's compile-time type dispatch — lets you write a macro that does different things for different types, like a type-safe overloaded function. Avoids the cast-to-void-pointer trick in qsort-style code.

---

## KEY TRAPS WITH MODERN C 🪤

**Trap 1: VLA size isn't checked**
```c
void f(int n) {
    int arr[n];   /* n=10000000? Stack overflow. No warning. No mercy. */
}
```

**Trap 2: `bool` arithmetic is surprising**
```c
bool b = 256;   /* b = true (1) — any non-zero truncates to 1 */
int x = b + b;  /* x = 2, not 512 */
```

**Trap 3: `snprintf` truncation detection**
```c
char buf[10];
int written = snprintf(buf, sizeof(buf), "Hello, %s!", "world");
if (written >= (int)sizeof(buf))
    printf("Output was truncated!\n");   /* Must check this! */
```

**Trap 4: `_Static_assert` doesn't work in all contexts**
```c
/* OK at file scope or inside a function */
_Static_assert(sizeof(int) == 4, "need 32-bit int");

/* NOT OK inside a struct (use offsetof tricks instead) */
```

**Trap 5: `typeof` and side effects (C23)**
```c
/* typeof evaluates the type but NOT the expression — no side effects */
int i = 0;
typeof(i++) j = 5;   /* i is still 0 — i++ not executed! */
```
