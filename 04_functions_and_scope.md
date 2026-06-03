# C Interview Prep — File 4: Functions, Scope & Storage Classes

> A function is like a restaurant kitchen — you give it ingredients (arguments), it does something with them, and hands you back a dish (return value). You don't need to know HOW the kitchen works, just what to order. 🍳

---

## THEORETICAL Q&A

### Q1. What is a function in C? What's its structure?

```c
return-type function-name(parameter declarations)
{
    declarations
    statements
    return expression;
}
```

**Example:**
```c
int power(int base, int n) {
    int p;
    for (p = 1; n > 0; --n)
        p *= base;
    return p;
}
```

- Functions can appear in **any order** in source files.
- No function can be **defined inside another** function (unlike C++ lambdas).
- If return type is omitted, `int` is **assumed** (old-style — avoid this).
- `return` without an expression returns no useful value.
- "Falling off the end" of a function returns garbage — make the compiler warn with `-Wall`.

---

### Q2. What is a function prototype and why does it matter?

A **function prototype** (declaration) tells the compiler the function's return type and parameter types **before** it's used:

```c
int power(int base, int n);   /* prototype — before main */

int main() {
    printf("%d\n", power(2, 10));   /* compiler can verify argument types */
    return 0;
}

int power(int base, int n) {  /* definition — can come after */
    ...
}
```

**Without prototype:** compiler assumes function returns `int` and accepts any args — dangerous type mismatches go undetected!

**Parameter names in prototype are optional** (just for documentation):
```c
int power(int, int);   /* also valid */
```

**Prototype vs definition must agree** — mismatching types is an error.

---

### Q3. What does "call by value" mean? How do you work around it?

C passes **copies** of arguments to functions. The function can't modify the caller's original variables:

```c
void tryToDouble(int x) {
    x = x * 2;   /* modifies LOCAL copy only — caller's x unchanged */
}

int main() {
    int val = 5;
    tryToDouble(val);
    printf("%d\n", val);   /* still 5! */
}
```

**Workaround: pass a pointer** (address) instead:
```c
void actuallyDouble(int *x) {
    *x = *x * 2;   /* modifies what x POINTS TO — caller's variable! */
}

int main() {
    int val = 5;
    actuallyDouble(&val);
    printf("%d\n", val);   /* 10 ✓ */
}
```

**Exception: arrays** — when you pass an array, you pass a **pointer to its first element**, so the function CAN modify the array contents. Array name ≠ copy of array.

---

### Q4. What are the storage classes in C?

| Storage Class | Keyword | Lifetime | Scope | Default Init |
|---------------|---------|----------|-------|-------------|
| Automatic | `auto` (default inside function) | Function call duration | Block | Garbage |
| Register | `register` | Function call duration | Block | Garbage |
| Static (local) | `static` inside function | Entire program | Block | Zero |
| Static (external) | `static` outside function | Entire program | File only | Zero |
| External | `extern` | Entire program | Global | Zero |

---

### Q5. What is an external variable? When should you use it?

An **external variable** is defined outside any function — globally accessible, permanently stored:

```c
int max;            /* external — accessible to all functions */
char line[MAXLINE]; /* external array */

void getline(void) {
    /* can use max and line directly — no argument needed */
}
```

**When to use:** when multiple functions need shared state that persists between calls (e.g., a pushback buffer for `getch/ungetch`).

**When NOT to use:** don't use them just to avoid passing arguments — this creates hidden coupling between functions and makes code hard to test and debug. K&R explicitly warn: "relying too heavily on external variables is fraught with peril."

**extern keyword:** when you need to access an external variable defined in a different file:
```c
/* file1.c */
int sp = 0;   /* definition — allocates storage */

/* file2.c */
extern int sp;   /* declaration — says "sp exists somewhere else" */
```

---

### Q6. What is a static variable? What are its two uses?

**Use 1: Static local variable** — retains its value between function calls:
```c
void counter() {
    static int count = 0;   /* initialized once, persists across calls */
    count++;
    printf("Called %d times\n", count);
}
/* counter() → "Called 1 times"
   counter() → "Called 2 times"  — count remembered! */
```

**Use 2: Static external variable** — restricts visibility to the current file (internal linkage):
```c
/* In mymodule.c */
static int helper_var;        /* NOT visible to other .c files */
static void helper_func() { } /* NOT visible to other .c files */
```
This is C's way of achieving **encapsulation** — hide implementation details from other modules.

---

### Q7. What is a register variable?

```c
void f(register unsigned m, register long n) {
    register int i;
    ...
}
```

- **Hints** to the compiler to place the variable in a CPU register for speed.
- Compiler is **free to ignore** the hint.
- Cannot take the **address** of a register variable (no `&`).
- Only applies to automatic variables and function parameters.
- In modern compilers, optimizers handle register allocation better than you can — `register` is largely **obsolete**.

---

### Q8. What are scope rules in C?

**Scope** = where in the code an identifier is accessible.

- **Block scope:** declared inside `{ }` — visible from declaration to closing `}`.
- **Function scope:** labels (`goto` targets) — visible throughout the function.
- **File scope:** declared outside all functions — visible from declaration to end of file.
- **Inner scope hides outer scope:**
```c
int x = 10;   /* outer x */
void f() {
    int x = 20;   /* inner x — hides outer x inside f() */
    printf("%d\n", x);  /* 20 */
}
/* outer x still 10 */
```

**K&R advice:** avoid variable names that shadow outer scope — it's a source of subtle bugs.

---

### Q9. How does initialization work for different storage classes?

```c
/* External and static: initialized to 0 automatically */
int global_var;        /* = 0 */
static int file_var;   /* = 0 */

/* Automatic: uninitialized = GARBAGE */
void f() {
    int local;         /* could be anything! */
    int x = 5;         /* OK — initialized with expression */
    int arr[] = {1, 2, 3};  /* OK — array initializer */

    /* For automatic, initializer can be any expression: */
    int n = strlen(s) + 1;   /* valid for automatic */
}

/* For external/static, initializer must be CONSTANT expression: */
static int size = 10 * 20;   /* OK — constant expression */
static int n = strlen(s);    /* ERROR — function call not allowed */
```

---

### Q10. How does recursion work in C? What's the overhead?

Each recursive call creates a **new stack frame** with its own local variables — completely independent copies.

```c
/* Classic: print integer recursively */
void printd(int n) {
    if (n < 0) {
        putchar('-');
        n = -n;
    }
    if (n / 10)
        printd(n / 10);   /* print all but last digit first */
    putchar(n % 10 + '0');
}
/* printd(123) → calls printd(12) → calls printd(1) → prints '1'
                                                        then '2'
                                    then '3'  */
```

**Overhead:** each call uses stack memory for local vars, return address, arguments. Deep recursion → stack overflow.

**Recursion shines for:** recursive data structures (trees, linked lists), divide-and-conquer algorithms (quicksort), and problems naturally defined recursively.

**Tail recursion:** when the recursive call is the last thing in the function — smart compilers can optimize this into a loop (no stack growth). C standard doesn't guarantee this, but GCC does it with `-O2`.

---

## EXERCISES WITH SOLUTIONS

### Exercise 1-15. Rewrite temperature conversion to use a function.
```c
#include <stdio.h>

float celsius_to_fahr(float c) {
    return (9.0f / 5.0f) * c + 32.0f;
}

float fahr_to_celsius(float f) {
    return (5.0f / 9.0f) * (f - 32.0f);
}

int main() {
    float fahr;
    printf("%3s %6s\n", "F", "C");
    for (fahr = 0; fahr <= 300; fahr += 20)
        printf("%3.0f %6.1f\n", fahr, fahr_to_celsius(fahr));
    return 0;
}
```

---

### Exercise 4-1. `strindex(s, t)` — rightmost occurrence of t in s.
```c
int strindex(const char s[], const char t[]) {
    int i, j, k;
    int last = -1;

    for (i = 0; s[i] != '\0'; i++) {
        for (j = i, k = 0; t[k] != '\0' && s[j] == t[k]; j++, k++)
            ;
        if (k > 0 && t[k] == '\0')
            last = i;   /* found match at position i — keep looking for later ones */
    }
    return last;
}
```

---

### Exercise 4-2. Extend `atof` to handle scientific notation.
```c
#include <ctype.h>

double atof(const char s[]) {
    double val, power;
    int i, sign, esign, exp;

    /* skip whitespace */
    for (i = 0; isspace(s[i]); i++)
        ;

    sign = (s[i] == '-') ? -1 : 1;
    if (s[i] == '+' || s[i] == '-') i++;

    /* integer part */
    for (val = 0.0; isdigit(s[i]); i++)
        val = 10.0 * val + (s[i] - '0');

    /* fraction part */
    if (s[i] == '.') i++;
    for (power = 1.0; isdigit(s[i]); i++) {
        val = 10.0 * val + (s[i] - '0');
        power *= 10.0;
    }

    val = sign * val / power;

    /* exponent part */
    if (s[i] == 'e' || s[i] == 'E') {
        i++;
        esign = (s[i] == '-') ? -1 : 1;
        if (s[i] == '+' || s[i] == '-') i++;
        for (exp = 0; isdigit(s[i]); i++)
            exp = 10 * exp + (s[i] - '0');
        /* Apply exponent */
        while (exp-- > 0) {
            if (esign > 0) val *= 10.0;
            else           val /= 10.0;
        }
    }
    return val;
}
/* atof("1.5e3") = 1500.0, atof("-2.5e-2") = -0.025 */
```

---

### Exercise 4-10. Quicksort.
```c
void swap(int v[], int i, int j) {
    int temp = v[i];
    v[i] = v[j];
    v[j] = temp;
}

void qsort(int v[], int left, int right) {
    if (left >= right) return;

    swap(v, left, (left + right) / 2);   /* move pivot to left */
    int last = left;

    for (int i = left + 1; i <= right; i++)
        if (v[i] < v[left])
            swap(v, ++last, i);

    swap(v, left, last);   /* restore pivot */
    qsort(v, left, last - 1);
    qsort(v, last + 1, right);
}
```

---

### Exercise 4-12. Recursive `itoa`.
```c
#include <string.h>

static void _itoa_helper(int n, char s[], int *idx) {
    if (n / 10)
        _itoa_helper(n / 10, s, idx);
    s[(*idx)++] = abs(n % 10) + '0';
}

void itoa_recursive(int n, char s[]) {
    int idx = 0;
    if (n < 0) s[idx++] = '-';
    _itoa_helper(n, s, &idx);
    s[idx] = '\0';
}
```

---

### Exercise 4-13. Recursive `reverse(s)`.
```c
void reverse(char s[]) {
    int len = strlen(s);
    void rev(char s[], int l, int r);
    rev(s, 0, len - 1);
}

static void rev(char s[], int l, int r) {
    if (l >= r) return;
    char temp = s[l]; s[l] = s[r]; s[r] = temp;
    rev(s, l + 1, r - 1);
}
```

---

### Key Interview Traps 🪤

**Trap 1: Returning a pointer to a local variable**
```c
char *bad() {
    char buf[100] = "hello";   /* local — lives on stack */
    return buf;                /* DANGLING POINTER — stack frame gone after return! */
}

char *good() {
    static char buf[100] = "hello";   /* static — persists */
    return buf;   /* OK — but not thread-safe */
}
```

**Trap 2: `static` local vs `static` global**
- `static` inside a function = variable persists between calls but is local to function.
- `static` outside a function = variable/function is private to the file.
Same keyword, very different meaning based on where it appears!

**Trap 3: Forgetting that arrays decay to pointers when passed**
```c
void f(int arr[]) {        /* arr is ACTUALLY int *arr here */
    printf("%zu\n", sizeof(arr));  /* prints size of POINTER, not array! */
}
int main() {
    int a[10];
    printf("%zu\n", sizeof(a));  /* 40 (10 * 4) */
    f(a);                        /* 8 (pointer size on 64-bit) */
}
```

**Trap 4: Implicit int return type confusion**
```c
add(int a, int b) {   /* old-style: return type assumed int */
    return a + b;
}
```
This compiles in C89 but is bad practice. Always specify return type explicitly.

**Trap 5: Mutual recursion declaration order**
```c
/* Need prototypes when functions call each other */
int is_even(int n);
int is_odd(int n);

int is_even(int n) { return (n == 0) ? 1 : is_odd(n - 1); }
int is_odd(int n)  { return (n == 0) ? 0 : is_even(n - 1); }
```
