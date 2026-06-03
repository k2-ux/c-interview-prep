# C Interview Prep — File 2: Operators & Expressions

> Operators are the verbs of C — they DO things to values. Know them cold, because interviewers love asking "what does this expression evaluate to?" and then watching you sweat over operator precedence. We'll make you sweat-proof! 💪

---

## THEORETICAL Q&A

### Q1. What are the arithmetic operators in C?

| Operator | Meaning | Notes |
|----------|---------|-------|
| `+` | Addition | |
| `-` | Subtraction | |
| `*` | Multiplication | |
| `/` | Division | Integer division **truncates** toward zero |
| `%` | Modulus (remainder) | Operands must be integral; sign of result matches dividend in C99+ |
| `+x` | Unary plus | No-op but triggers integral promotion |
| `-x` | Unary minus | Negation |

**Critical:** `5/9` is `0` (integer division truncates). `5.0/9.0` is `0.555...`.  
**Critical:** `%` cannot be applied to `float` or `double`.

---

### Q2. What are the relational and logical operators?

**Relational (yield 1 if true, 0 if false):**
```
>   >=   <   <=   ==   !=
```
- Precedence: `<`, `<=`, `>`, `>=` are higher than `==`, `!=`.
- Both groups are lower than arithmetic operators.

**Logical:**
```
&&   (AND)     ||   (OR)     !   (NOT)
```
- `&&` has higher precedence than `||`.
- Both are lower than relational operators.
- **Short-circuit evaluation**: `&&` stops if left side is false; `||` stops if left side is true.
- `!x` converts non-zero to 0, zero to 1.

**Example:**
```c
// Short-circuit prevents out-of-bounds access:
if (i < n && arr[i] != 0) { ... }  // arr[i] only evaluated if i < n
```

---

### Q3. Explain the bitwise operators.

| Operator | Name | Effect |
|----------|------|--------|
| `&` | Bitwise AND | 1 only if both bits are 1 |
| `\|` | Bitwise OR | 1 if either bit is 1 |
| `^` | Bitwise XOR | 1 if bits differ |
| `~` | One's complement | Flips all bits |
| `<<` | Left shift | Shifts left, fills with 0 |
| `>>` | Right shift | Fills with 0 for unsigned; implementation-defined for signed |

**Apply only to integral types** (`char`, `short`, `int`, `long`).

```c
n = n & 0177;       // zero all but low-order 7 bits (masking)
x = x | SET_ON;     // turn on specific bits
x = x ^ MASK;       // toggle bits
x = x & ~077;       // clear low 6 bits (portable — no word-size assumption)
x = x << 2;         // multiply by 4
x = x >> 1;         // divide by 2 (unsigned only — safe)
```

**Critical distinction:** `&` (bitwise AND) vs `&&` (logical AND). If `x=1, y=2`: `x & y = 0`, `x && y = 1`.

---

### Q4. What are the increment and decrement operators? When does prefix vs postfix matter?

- `++n` (prefix): increments n, **then** uses the new value.
- `n++` (postfix): uses the current value of n, **then** increments.
- `--n` / `n--`: same pattern for decrement.

```c
int n = 5;
int x = n++;  // x = 5,  n = 6  (use then increment)
int y = ++n;  // y = 7,  n = 7  (increment then use)
```

**When it matters** (in expressions):
```c
/* Postfix: copy character then advance pointer */
*p++ = val;       // stores val at *p, then advances p

/* Prefix: advance pointer then use */
x = *++p;         // advance p first, then fetch *p
```

**Cannot apply to non-lvalues:** `(i+j)++` is **illegal**.

---

### Q5. What are the assignment operators?

```c
i += 2;    // i = i + 2
x -= 1;    // x = x - 1
x *= y+1;  // x = x * (y + 1)  — NOTE: right side treated as whole expression
```

Available: `+=`, `-=`, `*=`, `/=`, `%=`, `<<=`, `>>=`, `&=`, `^=`, `|=`

**An assignment is an expression with a value** (the value stored):
```c
while ((c = getchar()) != EOF) { ... }  // assignment inside condition
n1 = n2 = n3 = 0;   // right-associative chaining
```

---

### Q6. What is the conditional (ternary) operator?

```c
expr1 ? expr2 : expr3
```
- If `expr1` is non-zero, the result is `expr2`; otherwise `expr3`.
- Only one of `expr2` or `expr3` is evaluated.
- Type of result: usual arithmetic conversions applied to `expr2` and `expr3`.

```c
z = (a > b) ? a : b;                    // max of a and b
printf("You have %d item%s.\n", n, n==1 ? "" : "s");
```

---

### Q7. What are the implicit type conversion rules in C?

**Integral promotion:** `char` and `short` are converted to `int` (or `unsigned int`) before any arithmetic.

**Usual arithmetic conversions (applied to binary operators):**
1. If either operand is `long double` → convert other to `long double`
2. Else if either is `double` → convert other to `double`
3. Else if either is `float` → convert other to `float`
4. Else: integral promotions performed, then:
   - If either is `unsigned long` → both become `unsigned long`
   - Else if one is `long` and other is `unsigned int`:
     - If `long` can represent all `unsigned int` values → `unsigned int` → `long`
     - Else both → `unsigned long`
   - Else if either is `long` → other → `long`
   - Else if either is `unsigned int` → other → `unsigned int`
   - Else both are `int`

**Key consequence:** `float` operands may do single-precision arithmetic (unlike the old rule of always using `double`).

**Assignment conversions:** right side converted to type of left side. Narrowing may lose information.

---

### Q8. What is an explicit type cast?

```c
(type) expression
```

```c
sqrt((double) n)   // convert int n to double before passing
(int) 3.7          // truncates to 3
(unsigned char) -1 // 255 on most machines
```

- Cast has the same high precedence as other unary operators.
- Does **not** modify the original variable.
- If function prototype is in scope, arguments are automatically coerced — cast often not needed:
  ```c
  double sqrt(double);
  root2 = sqrt(2);   // 2 automatically converted to 2.0
  ```

---

### Q9. What is the precedence and associativity of C operators?

From highest to lowest:

| Operators | Associativity |
|-----------|--------------|
| `()` `[]` `->` `.` | Left → Right |
| `!` `~` `++` `--` `+` `-` `*` `(type)` `sizeof` | Right → Left (unary) |
| `*` `/` `%` | Left → Right |
| `+` `-` | Left → Right |
| `<<` `>>` | Left → Right |
| `<` `<=` `>` `>=` | Left → Right |
| `==` `!=` | Left → Right |
| `&` | Left → Right |
| `^` | Left → Right |
| `\|` | Left → Right |
| `&&` | Left → Right |
| `\|\|` | Left → Right |
| `?:` | Right → Left |
| `=` `+=` `-=` etc. | Right → Left |
| `,` | Left → Right |

**Critical pitfalls:**
```c
if ((x & MASK) == 0)    // CORRECT: & has lower precedence than ==, need parens
if (x & MASK == 0)      // WRONG: evaluated as x & (MASK == 0)

// Bitwise operators &, ^, | all have lower precedence than ==
```

---

### Q10. What is the comma operator and when is it used?

```c
expr1, expr2
```
- Both expressions evaluated left to right.
- Value and type of result = value and type of **right** operand.
- `expr1` is evaluated for its **side effects**.

```c
// Most common use: for loop with multiple variables
for (i = 0, j = strlen(s)-1; i < j; i++, j--) { ... }
```

**Note:** commas in function argument lists and declarations are **not** comma operators.

---

### Q11. What is the order of evaluation problem?

C does **not** specify the order in which operands of most operators are evaluated. Only `&&`, `||`, `?:`, and `,` guarantee left-to-right evaluation.

```c
/* WRONG — undefined behavior */
printf("%d %d\n", ++n, power(2, n));   // n may be incremented before or after

/* WRONG — a[i] subscript vs i++ */
a[i] = i++;   // undefined: is old or new value of i used as subscript?

/* Fix: separate the side effects */
++n;
printf("%d %d\n", n, power(2, n));
```

---

## EXERCISES WITH SOLUTIONS

### Exercise 2-2. Write a `for` loop equivalent without `&&` or `||`.
**Original:**
```c
for (i = 0; i < lim-1 && (c = getchar()) != '\n' && c != EOF; ++i)
    s[i] = c;
```
**Equivalent without `&&`:**
```c
i = 0;
while (1) {
    if (i >= lim - 1) break;
    c = getchar();
    if (c == '\n') break;
    if (c == EOF) break;
    s[i++] = c;
}
```

---

### Exercise 2-3. Write `htoi(s)` — hex string to integer.
```c
#include <stdio.h>
#include <ctype.h>

int htoi(const char s[]) {
    int val = 0;
    int i = 0;

    /* skip optional 0x or 0X prefix */
    if (s[0] == '0' && (s[1] == 'x' || s[1] == 'X'))
        i = 2;

    for (; s[i] != '\0'; i++) {
        int digit;
        if (isdigit(s[i]))
            digit = s[i] - '0';
        else if (s[i] >= 'a' && s[i] <= 'f')
            digit = s[i] - 'a' + 10;
        else if (s[i] >= 'A' && s[i] <= 'F')
            digit = s[i] - 'A' + 10;
        else
            break;   /* invalid character */
        val = 16 * val + digit;
    }
    return val;
}

int main() {
    printf("%d\n", htoi("0xFF"));   /* 255 */
    printf("%d\n", htoi("1A"));     /* 26  */
    printf("%d\n", htoi("0x1a2b")); /* 6699 */
    return 0;
}
```

---

### Exercise 2-4. `squeeze(s1, s2)` — delete from s1 any chars in s2.
```c
void squeeze(char s1[], const char s2[]) {
    int i, j, k;
    for (i = j = 0; s1[i] != '\0'; i++) {
        /* check if s1[i] is in s2 */
        for (k = 0; s2[k] != '\0' && s2[k] != s1[i]; k++)
            ;
        if (s2[k] == '\0')          /* s1[i] not found in s2 */
            s1[j++] = s1[i];
    }
    s1[j] = '\0';
}
```

---

### Exercise 2-5. `any(s1, s2)` — return first location in s1 of any char from s2.
```c
int any(const char s1[], const char s2[]) {
    for (int i = 0; s1[i] != '\0'; i++)
        for (int j = 0; s2[j] != '\0'; j++)
            if (s1[i] == s2[j])
                return i;
    return -1;
}
```

---

### Exercise 2-6. `setbits(x, p, n, y)` — set n bits of x at position p from y.
```c
/* Returns x with the n bits beginning at position p set to
   the rightmost n bits of y */
unsigned setbits(unsigned x, int p, int n, unsigned y) {
    /* Create a mask of n ones at position p */
    unsigned mask = ~(~0u << n) << (p + 1 - n);
    /* Clear those bits in x, then OR in the bits from y */
    return (x & ~mask) | ((y << (p + 1 - n)) & mask);
}
```

---

### Exercise 2-7. `invert(x, p, n)` — invert n bits of x starting at position p.
```c
unsigned invert(unsigned x, int p, int n) {
    return x ^ (~(~0u << n) << (p + 1 - n));
}
```

---

### Exercise 2-8. `rightrot(x, n)` — rotate x right by n positions.
```c
unsigned rightrot(unsigned x, int n) {
    int bits = sizeof(unsigned) * 8;
    n = n % bits;
    return (x >> n) | (x << (bits - n));
}
```

---

### Exercise 2-9. Faster `bitcount` using `x &= (x-1)`.
**Explanation:** `x & (x-1)` deletes the rightmost 1-bit.  
*Why?* `x-1` flips the rightmost 1 to 0 and all trailing 0s to 1s. ANDing with `x` clears that bit and the trailing 0s remain 0s.

Example: `x = 1010 0000`, `x-1 = 1001 1111`, `x & (x-1) = 1000 0000` (rightmost 1 removed).

```c
int bitcount(unsigned x) {
    int b;
    for (b = 0; x != 0; x &= x-1)
        b++;
    return b;
}
```

---

### Exercise 2-10. `lower` with conditional expression.
```c
int lower(int c) {
    return (c >= 'A' && c <= 'Z') ? c + 'a' - 'A' : c;
}
```

---

### Key Interview Traps

**Trap 1:** `=` vs `==` in conditions.
```c
if (x = 5) { }   // ALWAYS true — assigns 5 to x, then tests 5 (non-zero)
if (x == 5) { }  // CORRECT — comparison
```

**Trap 2:** Integer division truncation.
```c
int result = 7 / 2;   // result is 3, NOT 3.5
```

**Trap 3:** Right shift of negative signed integers is implementation-defined. For bit manipulation, always use `unsigned`.

**Trap 4:** `~0` is all-bits-set for any integer width — portable way to get a mask of all 1s. Better than `-1` for bit operations.

**Trap 5:** Operator precedence with bitwise operators. Always parenthesize: `(x & MASK) == 0`, never `x & MASK == 0`.

**Trap 6:** Comma operator in macro — looks like multiple arguments to a function but isn't:
```c
#define SWAP(x, y) (temp=x, x=y, y=temp)  // comma operator, single expression
```
