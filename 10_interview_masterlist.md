# C Interview Prep — File 10: The Ultimate Interview Master List 🏆

> This is your cheat sheet, your last-minute cram session, your "read this on the train before the interview" file. 60+ questions, code snippets with expected output, and the sneakiest traps interviewers love to throw. Let's go! 🚀

---

## PART 1: RAPID-FIRE CONCEPTUAL Q&A

### Q1. What's the size of `int` in C?
**A:** Implementation-defined. Typically 4 bytes on modern 32/64-bit systems, but only guaranteed to be at least 16 bits. Use `sizeof(int)` to check, or `<stdint.h>` types like `int32_t` for guaranteed sizes.

### Q2. What happens when you declare `int arr[5]` globally vs locally?
**A:** Global: initialized to all zeros. Local (automatic): contains garbage values.

### Q3. Is `char` signed or unsigned?
**A:** **Implementation-defined!** It varies by platform and compiler flags. Use `signed char` or `unsigned char` explicitly when you need a specific behavior.

### Q4. What is the difference between `++i` and `i++`?
**A:** `++i` increments i and **returns the new value**. `i++` **returns the old value** and then increments. In a standalone statement (`i++;`), they're identical. The difference matters in expressions: `int x = i++` gives x the old value; `int x = ++i` gives x the new value.

### Q5. Can you increment a pointer? Can you increment an array?
**A:** You can increment a **pointer variable** (`p++` where `int *p`). You **cannot** increment an array name — it's not a lvalue (`arr++` is illegal). Think of array name as a `const` pointer.

### Q6. What is `void *` and why is it special?
**A:** It's a generic pointer — can hold any pointer type without a cast (in C). Cannot be dereferenced directly — must cast to a concrete type first. Used by `malloc`, `qsort`, `memcpy`, etc.

### Q7. What does `static` mean in different contexts?
**A:** 
- Inside a function: variable persists between calls (lifetime = program duration)
- Outside all functions (file scope): variable/function is private to the file (internal linkage)
- Both cases: initialized to zero if no explicit initializer

### Q8. What's the difference between `extern` declaration and definition?
**A:** A **definition** allocates storage (e.g., `int x = 5;` at file scope). A **declaration** just announces the type (e.g., `extern int x;`). There must be exactly **one definition** across the whole program, but multiple declarations (in different files) are fine.

### Q9. What is undefined behavior? Give 3 examples.
**A:** Code whose result is not specified by the C standard — the compiler can do *anything* including nasal demons.
1. Integer overflow for signed integers
2. Dereferencing a NULL or dangling pointer
3. Reading uninitialized variables
4. `arr[10]` when arr has 10 elements (index 10 is out of bounds)
5. Signed integer left shift by negative amount

### Q10. What's the difference between `a[i]` and `*(a+i)`?
**A:** They are **exactly identical** by definition. The C standard defines `a[i]` as `*(a+i)`. This means `i[a]` is also valid (same as `*(i+a)` = `*(a+i)`)! 🤯

### Q11. Can a function return an array?
**A:** **No.** Functions cannot return arrays in C. They can return a pointer to an array's first element, or return a struct that contains an array.

### Q12. What is a function prototype? Why is it important?
**A:** A declaration that specifies a function's return type and parameter types before the function is called. Without it, the compiler assumes the function returns `int` and can't check argument types — silent bugs result.

### Q13. What does `sizeof("hello")` return?
**A:** **6** — `sizeof` on a string literal counts the null terminator `'\0'`. `strlen("hello")` returns 5.

### Q14. Is it legal to compare pointers to different arrays?
**A:** Technically **undefined behavior** in standard C. Only pointer comparisons within the same array (and one-past-the-end) are defined. In practice, most platforms allow it (virtual memory is flat), but don't rely on it.

### Q15. What is the difference between `malloc(0)` and passing zero to `calloc`?
**A:** Implementation-defined — `malloc(0)` may return NULL or a non-NULL pointer that can't be dereferenced. Always check and handle null returns from allocation functions.

---

## PART 2: OUTPUT PREDICTION QUESTIONS

### Q16. What does this print?
```c
int i = 5;
printf("%d %d\n", i++, ++i);
```
**A:** **Undefined behavior** — the order of evaluation of function arguments is unspecified. Don't write code like this! Depending on the compiler: could print `5 7`, `6 7`, or something else entirely.

---

### Q17. What does this print?
```c
int a = 10;
int *p = &a;
*p = 20;
printf("%d\n", a);
```
**A:** `20` — `*p = 20` modifies `a` through the pointer.

---

### Q18. What does this print?
```c
int arr[] = {10, 20, 30, 40};
int *p = arr + 2;
printf("%d %d\n", *p, *(p-1));
```
**A:** `30 20` — `p` points to `arr[2]`(=30), `p-1` points to `arr[1]`(=20).

---

### Q19. What does this print?
```c
char s[] = "hello";
printf("%d\n", sizeof(s));
printf("%d\n", strlen(s));
```
**A:** `6` and `5` — `sizeof` includes `'\0'`, `strlen` does not.

---

### Q20. What does this print?
```c
int x = 5;
printf("%d\n", x == 5);
printf("%d\n", x = 10);
printf("%d\n", x);
```
**A:** `1`, `10`, `10` — the comparison `x==5` yields 1 (true). `x=10` is an assignment expression with value 10. `x` is now 10.

---

### Q21. What does this print?
```c
int arr[3] = {1, 2, 3};
int *p = arr;
printf("%d\n", *(p++));
printf("%d\n", *p);
```
**A:** `1` then `2` — `*(p++)` uses current `*p`(=1) then advances p. Now `*p` = `arr[1]` = 2.

---

### Q22. What does this print?
```c
static int count = 0;
void f() { printf("%d\n", ++count); }
int main() { f(); f(); f(); return 0; }
```
**A:** `1`, `2`, `3` — `static` local variable persists between calls.

---

### Q23. What does this print?
```c
int x = 7;
int y = x >> 1;
int z = x << 2;
printf("%d %d\n", y, z);
```
**A:** `3` `28` — right shift by 1 divides by 2 (truncating), left shift by 2 multiplies by 4.

---

### Q24. What does this print?
```c
char *names[] = {"Alice", "Bob", "Charlie"};
printf("%c\n", names[1][0]);
printf("%s\n", names[2] + 3);
```
**A:** `B` and `rlie` — `names[1][0]` is first char of "Bob". `names[2]+3` is pointer to 4th char of "Charlie" = "rlie".

---

### Q25. What does this print?
```c
int a = 5, b = 3;
printf("%d\n", a & b);
printf("%d\n", a | b);
printf("%d\n", a ^ b);
```
**A:** `1`, `7`, `6`  
a=101₂, b=011₂:  
AND=001₂=1, OR=111₂=7, XOR=110₂=6

---

## PART 3: TRICKY CODE SNIPPETS

### Q26. What's wrong with this code?
```c
int *makeArray(int n) {
    int arr[n];
    for (int i = 0; i < n; i++) arr[i] = i;
    return arr;   /* BUG! */
}
```
**A:** Returns a pointer to a **local (stack) array** — that memory is freed when function returns. **Dangling pointer**. Fix: use `malloc(n * sizeof(int))` and return that.

---

### Q27. What's wrong here?
```c
char *greeting = "Hello";
greeting[0] = 'J';   /* crash? */
```
**A:** String literals are **read-only** in practice (though C standard says it's "undefined behavior"). This typically causes a segfault. Fix: `char greeting[] = "Hello";` — this creates a mutable copy on the stack.

---

### Q28. Is this loop correct?
```c
unsigned int i;
for (i = 10; i >= 0; i--)
    printf("%u\n", i);
```
**A:** **Infinite loop!** `unsigned int` can never be negative. When `i` reaches 0 and decrements, it wraps around to `UINT_MAX` (e.g., 4294967295) — still ≥ 0. Use `signed int` or loop differently.

---

### Q29. What happens here?
```c
int i = INT_MAX;
i++;
printf("%d\n", i);
```
**A:** **Undefined behavior** — signed integer overflow. On most machines you'll see `INT_MIN` (−2147483648), but the standard does not guarantee this. `unsigned` overflow IS defined (wraps around).

---

### Q30. What's the output?
```c
int a = 1, b = 2, c = 3;
printf("%d\n", a < b ? b : c);     /* 2 */
printf("%d\n", a > b ? b : c);     /* 3 */
printf("%d\n", a < b < c);         /* ??? */
```
**A:** `2`, `3`, and `1`.  
The last one: `a < b` = 1 (true), then `1 < c` = `1 < 3` = 1. NOT a 3-way comparison! Use `a < b && b < c` for that.

---

## PART 4: MUST-KNOW IMPLEMENTATIONS

### Implement `strlen` (two ways):
```c
/* Loop version */
int my_strlen(const char *s) {
    int n = 0;
    while (*s++) n++;
    return n;
}

/* Pointer difference version */
int my_strlen2(const char *s) {
    const char *p = s;
    while (*p) p++;
    return p - s;
}
```

### Implement `strcpy`:
```c
char *my_strcpy(char *d, const char *s) {
    char *start = d;
    while ((*d++ = *s++));  /* copies including '\0' */
    return start;
}
```

### Implement `strcmp`:
```c
int my_strcmp(const char *s, const char *t) {
    for (; *s == *t; s++, t++)
        if (*s == '\0') return 0;
    return (unsigned char)*s - (unsigned char)*t;
}
```

### Implement `memcpy`:
```c
void *my_memcpy(void *dst, const void *src, size_t n) {
    char *d = dst;
    const char *s = src;
    while (n--) *d++ = *s++;
    return dst;
}
/* Note: undefined behavior if regions overlap — use memmove for overlap */
```

### Implement `memmove` (handles overlap):
```c
void *my_memmove(void *dst, const void *src, size_t n) {
    char *d = dst;
    const char *s = src;
    if (d < s)
        while (n--) *d++ = *s++;          /* copy forward */
    else {
        d += n; s += n;
        while (n--) *--d = *--s;          /* copy backward */
    }
    return dst;
}
```

### Implement `atoi`:
```c
int my_atoi(const char *s) {
    int val = 0, sign = 1;
    while (isspace(*s)) s++;
    if (*s == '-') { sign = -1; s++; }
    else if (*s == '+') s++;
    while (isdigit(*s)) val = val * 10 + (*s++ - '0');
    return sign * val;
}
```

### Implement `itoa`:
```c
char *my_itoa(int n, char *s) {
    char *start = s;
    int sign = n < 0;
    long ln = sign ? -(long)n : n;
    do { *s++ = ln % 10 + '0'; } while ((ln /= 10) > 0);
    if (sign) *s++ = '-';
    *s = '\0';
    /* reverse */
    char *l = start, *r = s - 1;
    while (l < r) { char t = *l; *l++ = *r; *r-- = t; }
    return start;
}
```

### Implement `reverse` (string):
```c
void reverse(char *s) {
    char *e = s + strlen(s) - 1;
    while (s < e) { char t = *s; *s++ = *e; *e-- = t; }
}
```

---

## PART 5: KEY CONCEPTUAL SUMMARIES (Quick Reference Cards)

### The Big Pointer Equations:
```
arr[i]     == *(arr + i)
&arr[i]    == arr + i
sizeof(arr) != sizeof(pointer)   /* in calling function */
arr[i][j]  == *(*(arr + i) + j)
```

### Storage Class Summary:
```
auto     — default inside functions; garbage initial value; stack lifetime
static   — in function: persists; outside function: file-private
extern   — shared across files; zero-initialized
register — hint for CPU register; no & allowed; largely obsolete
```

### printf Format Quick Reference:
```
%d  — int (decimal)         %ld  — long
%u  — unsigned int          %lu  — unsigned long
%f  — float/double          %lf  — double (scanf only!)
%e  — scientific notation   %g   — shorter of %f/%e
%c  — char                  %s   — string
%p  — pointer               %x   — hex (lowercase)
%o  — octal                 %zu  — size_t
%%  — literal %
Width: %5d (right-aligned), %-5d (left-aligned), %05d (zero-padded)
Precision: %.3f (3 decimal places), %.10s (max 10 chars of string)
```

### Operator Precedence — The 4 Most Important Levels:
```
()  []  ->  .              highest — postfix, member access
!  ~  *  &  (cast)  sizeof  high — unary
*  /  %  +  -              arithmetic
<<  >>  <  <=  >  >=       relational
==  !=  &  ^  |  &&  ||    bitwise and logical (decreasing)
?:  =  +=  etc.            assignment and ternary
,                          lowest
```

### The Most Common C Interview Gotchas 🪤:
1. `=` vs `==` in conditions
2. Missing `&` in `scanf`
3. Returning pointer to local variable
4. `sizeof` on array parameter (gives pointer size, not array size)
5. Signed integer overflow is UB; unsigned wraps
6. `char` signedness is implementation-defined
7. String literals are read-only (`char *s = "hi"; s[0]='H'` = UB)
8. `free(p); p = NULL;` — always null out after free
9. `realloc` — save return value in temp pointer
10. `++i` vs `i++` in complex expressions — multiple evaluation

---

## PART 6: ALGORITHM PATTERNS IN C

### Binary Search:
```c
int bsearch_c(int x, int v[], int n) {
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;  /* avoid overflow vs (lo+hi)/2 */
        if (x < v[mid])      hi = mid - 1;
        else if (x > v[mid]) lo = mid + 1;
        else return mid;
    }
    return -1;
}
```

### qsort with custom comparator:
```c
#include <stdlib.h>
int int_cmp(const void *a, const void *b) {
    return (*(int*)a) - (*(int*)b);   /* ascending */
}
int arr[] = {3, 1, 4, 1, 5, 9, 2, 6};
qsort(arr, 8, sizeof(int), int_cmp);
```

### Generic max/min (type-safe with inline function, not macro):
```c
static inline int imax(int a, int b) { return a > b ? a : b; }
static inline int imin(int a, int b) { return a < b ? a : b; }
/* Or use macros for any type (with double-evaluation risk) */
#define MAX(a, b) ((a) > (b) ? (a) : (b))
```

### Bit manipulation cheat sheet:
```c
x & (x-1)       /* clear rightmost 1-bit */
x | (x+1)       /* set rightmost 0-bit */
x & (-x)        /* isolate rightmost 1-bit */
!(x & (x-1))    /* true if x is a power of 2 (and x > 0) */
x ^ y           /* swap without temp: x^=y; y^=x; x^=y; (but use temp!) */
(x >> n) & 1    /* check if bit n is set */
x | (1 << n)    /* set bit n */
x & ~(1 << n)   /* clear bit n */
x ^ (1 << n)    /* toggle bit n */
```

---

## PART 7: 10 THINGS EVERY C INTERVIEW TESTS

1. **Memory model:** stack vs heap vs data segment vs text segment
2. **Pointer arithmetic:** what `p+1` really means for different types
3. **String as char array:** null terminator, strlen vs sizeof
4. **`malloc`/`free` pattern:** always check return, always free, never double-free
5. **Struct vs union:** simultaneous storage vs shared storage
6. **Function pointers:** declaration syntax and usage in callbacks
7. **`static` keyword:** the dual meaning (lifetime vs linkage)
8. **Type coercion:** implicit conversions, signed/unsigned mixing
9. **Preprocessor macros:** pitfalls with side effects and missing parens
10. **Undefined behavior:** recognizing and avoiding it

---

*Good luck! Remember: C is one of those languages where the best way to learn it is to also know all the ways it can silently destroy you. Now you do. You've got this. 💪*
