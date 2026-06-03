# C Interview Prep — File 1: Basics, Data Types & Constants

> C is the language that every other language secretly wishes it was. It's so close to the hardware that it practically whispers sweet nothings to the CPU 🖥️. Understand C, and you understand computing. Let's start from the ground up!

---

## THEORETICAL Q&A

### Q1. What is the basic structure of a C program? 🏗️
A C program consists of **functions** and **variables**. Every program must have a `main()` function — execution begins there. A minimal program:
```c
#include <stdio.h>

int main() {
    printf("hello, world\n");
    return 0;
}
```
- `#include <stdio.h>` pulls in the standard I/O library declarations.
- `main` returns `int`; 0 means success, non-zero means error.
- Statements end with `;`. Blocks are enclosed in `{ }`.

---

### Q2. What are the basic data types in C? (Choose your data type like choosing a box size 📦)

| Type | Meaning | Typical Size |
|------|---------|-------------|
| `char` | Single byte, holds one character | 1 byte |
| `short int` | Short integer | 2 bytes |
| `int` | Natural integer for the machine | 2 or 4 bytes |
| `long int` | Long integer | 4 or 8 bytes |
| `float` | Single-precision floating point | 4 bytes |
| `double` | Double-precision floating point | 8 bytes |
| `long double` | Extended precision floating point | 10–16 bytes |

**Key rule:** `short` ≤ `int` ≤ `long`. Guaranteed minimums: `short` and `int` ≥ 16 bits, `long` ≥ 32 bits.

---

### Q3. What is the difference between `signed` and `unsigned`?
- `signed`: can hold negative, zero, or positive values. An 8-bit `signed char` ranges from −128 to 127.
- `unsigned`: holds only non-negative values. An 8-bit `unsigned char` ranges from 0 to 255.
- Whether plain `char` is signed or unsigned is **implementation-defined**.
- Printable characters are always non-negative.

---

### Q4. What are the different kinds of constants in C?

**Integer constants:**
```c
1234        // decimal int
01234       // octal (leading 0)
0x1A2F      // hexadecimal (leading 0x)
1234L       // long
1234U       // unsigned
1234UL      // unsigned long
```

**Floating-point constants:**
```c
123.4       // double
1e-2        // double (scientific notation)
123.4F      // float
123.4L      // long double
```

**Character constants:**
```c
'x'         // integer equal to ASCII value of 'x' (120)
'\n'        // newline (10)
'\t'        // tab (9)
'\0'        // null character (0)
'\013'      // octal escape
'\x1b'      // hexadecimal escape
```

**String constants:**
```c
"hello"     // array of chars: {'h','e','l','l','o','\0'}
"" ""       // adjacent strings are concatenated at compile time
```

**Enumeration constants:**
```c
enum boolean { NO, YES };         // NO=0, YES=1
enum months { JAN=1, FEB, MAR };  // JAN=1, FEB=2, MAR=3
```

---

### Q5. What is the difference between a character constant and a string literal?
- `'x'` is a **character constant**: it is an `int` with the numeric value of `x` in the machine's character set.
- `"x"` is a **string literal**: it is an array of characters `{'x', '\0'}`, i.e., two bytes.
- They are **not interchangeable**.

---

### Q6. What escape sequences does C provide?

| Sequence | Meaning |
|----------|---------|
| `\n` | newline |
| `\t` | horizontal tab |
| `\v` | vertical tab |
| `\b` | backspace |
| `\r` | carriage return |
| `\f` | form feed |
| `\a` | alert (bell) |
| `\\` | backslash |
| `\'` | single quote |
| `\"` | double quote |
| `\?` | question mark |
| `\ooo` | octal value |
| `\xhh` | hexadecimal value |
| `\0` | null character (value 0) |

---

### Q7. What is a symbolic constant and why use them?
A symbolic constant is defined with `#define`:
```c
#define MAXLINE 1000
#define PI 3.14159
```
- The preprocessor replaces every occurrence of `MAXLINE` with `1000` before compilation.
- No storage is allocated; it is a **text substitution**.
- Advantages: single point of change, meaningful names instead of magic numbers, no type issues.
- Convention: symbolic constants are written in **ALL_CAPS**.
- No semicolon at end of `#define` line.

---

### Q8. How are variables declared and initialized in C?
```c
int i = 0;
char c = 'A';
float f = 3.14f;
double d = 2.71828;

// Multiple variables of same type:
int lower, upper, step;

// const qualifier — value cannot be changed:
const double e = 2.71828182845905;
const char msg[] = "warning: ";
```
- `extern` and `static` variables are **automatically initialized to zero** if not explicitly initialized.
- Automatic (local) variables have **undefined (garbage) values** if not initialized.
- Initializer for external/static must be a **constant expression**.
- Initializer for automatic may be any expression.

---

### Q9. What is `sizeof` and what does it return?
`sizeof` is a compile-time unary operator that returns the size in bytes of its operand.
```c
sizeof(int)        // typically 4
sizeof(char)       // always 1
sizeof(double)     // typically 8
sizeof(arr)        // total bytes of array arr
```
- Return type is `size_t` (defined in `<stddef.h>`), which is an unsigned integer type.
- Cannot be used in `#if` directives (preprocessor doesn't evaluate it).
- Common idiom: `sizeof keytab / sizeof(struct key)` counts array elements.

---

### Q10. What are the rules for variable names (identifiers)?
- Made of **letters and digits**; first character must be a **letter**.
- Underscore `_` counts as a letter (but avoid leading underscore — library uses those).
- Upper and lower case are **distinct**: `x` and `X` are different.
- At least the first **31 characters** are significant for internal names.
- External names: implementations may treat only **6 characters** as significant (legacy).
- **Keywords** (`if`, `else`, `int`, `for`, etc.) cannot be used as variable names.
- Convention: lowercase for variables, UPPERCASE for constants.

---

## EXERCISES WITH SOLUTIONS

### Exercise 1-1. Run "hello, world" and experiment with errors.
```c
#include <stdio.h>

int main() {
    printf("hello, world\n");
    return 0;
}
```
**What happens if you omit `\n`?** Output appears without a newline; cursor stays on same line.  
**What if you omit `#include <stdio.h>`?** Compiler warns that `printf` is undeclared (implicit declaration assumed to return `int`).  
**What if you omit `return 0;`?** Undefined return value from `main`; most compilers warn.

---

### Exercise 1-2. What happens when `\c` is in a string (c not a valid escape)?
```c
printf("test\q");   // behavior is undefined — do not rely on it
printf("test\n");   // \n is valid — newline printed
printf("test\\");   // \\ prints a literal backslash
```
**Answer:** The C standard says the behavior of an unrecognized escape sequence (like `\q`) is **undefined**. Some compilers treat it as the character itself, others warn or error.

---

### Exercise 1-3. Modify temperature table to print a heading.
```c
#include <stdio.h>

int main() {
    float fahr, celsius;
    float lower = 0, upper = 300, step = 20;

    printf("%3s %6s\n", "F", "C");  // heading
    printf("--- ------\n");

    for (fahr = lower; fahr <= upper; fahr += step) {
        celsius = (5.0 / 9.0) * (fahr - 32.0);
        printf("%3.0f %6.1f\n", fahr, celsius);
    }
    return 0;
}
```

---

### Exercise 1-4. Print Celsius-to-Fahrenheit table.
```c
#include <stdio.h>

int main() {
    float celsius, fahr;
    float lower = -20, upper = 100, step = 10;

    printf("%6s %6s\n", "C", "F");
    for (celsius = lower; celsius <= upper; celsius += step) {
        fahr = (9.0 / 5.0) * celsius + 32.0;
        printf("%6.1f %6.1f\n", celsius, fahr);
    }
    return 0;
}
```

---

### Exercise 2-1. Determine ranges of char, short, int, long.
```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main() {
    printf("char:   %d to %d\n", CHAR_MIN, CHAR_MAX);
    printf("uchar:  0 to %u\n", UCHAR_MAX);
    printf("short:  %d to %d\n", SHRT_MIN, SHRT_MAX);
    printf("int:    %d to %d\n", INT_MIN, INT_MAX);
    printf("long:   %ld to %ld\n", LONG_MIN, LONG_MAX);
    printf("ushort: 0 to %u\n", USHRT_MAX);
    printf("uint:   0 to %u\n", UINT_MAX);
    printf("ulong:  0 to %lu\n", ULONG_MAX);

    /* floating-point ranges */
    printf("float:  %g to %g, epsilon %g\n", FLT_MIN, FLT_MAX, FLT_EPSILON);
    printf("double: %g to %g, epsilon %g\n", DBL_MIN, DBL_MAX, DBL_EPSILON);
    return 0;
}
```
**Computing directly without headers (char example):**
```c
/* Unsigned char max = 2^CHAR_BIT - 1 */
unsigned char uc = ~0;  // all bits set = UCHAR_MAX
printf("UCHAR_MAX = %u\n", (unsigned)uc);

/* Signed char: sign bit is MSB */
signed char sc_max = ~(1 << (sizeof(char)*8 - 1));  // 01111111 = 127
signed char sc_min = 1 << (sizeof(char)*8 - 1);     // 10000000 = -128
```

---

### Key Interview Traps

**Trap 1:** `char c = 255;` — if `char` is signed, this is implementation-defined (overflow). Use `unsigned char` explicitly.

**Trap 2:** `int x = 010;` — this is **8 in decimal** (octal literal). `010` ≠ `10`.

**Trap 3:** `sizeof('a')` — in C, character constants have type `int`, so this is `sizeof(int)`, typically 4. (Different from C++ where it's 1.)

**Trap 4:** String literals are read-only. `char *p = "hello"; p[0] = 'H';` is **undefined behavior**. Use `char p[] = "hello";` for a modifiable copy.

**Trap 5:** `long` is not guaranteed to be 64-bit. Use `long long` (C99) or `int64_t` from `<stdint.h>` for guaranteed 64-bit.
