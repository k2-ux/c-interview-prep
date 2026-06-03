# C Interview Prep — File 6: Arrays & Strings

> Arrays in C are like a row of numbered mailboxes 📬. Strings are arrays of `char` with a special `'\0'` postman at the end saying "Stop reading here." Every C string question is secretly a pointer question. Spoiler: everything in C is secretly a pointer question.

---

## THEORETICAL Q&A

### Q1. How are arrays declared, initialized, and accessed?

**Declaration:**
```c
int arr[10];             /* array of 10 ints — indices 0..9 */
float matrix[3][4];      /* 3x4 matrix — 3 rows, 4 columns */
```

**Initialization:**
```c
int days[] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
/* size inferred = 12 */

int zeroed[10] = {0};    /* first element 0, rest also 0 */
int partial[5] = {1, 2}; /* partial init: {1, 2, 0, 0, 0} */

/* For external/static arrays: uninitialized elements = 0 */
/* For automatic arrays: uninitialized elements = GARBAGE */
```

**Access:**
```c
arr[i]       /* i-th element (0-indexed) */
*(arr + i)   /* IDENTICAL — pointer arithmetic */
```

**No bounds checking!** `arr[15]` on a 10-element array? C lets you do it. It reads/writes garbage memory. You're on your own. 🤠

---

### Q2. What is the relationship between strings and char arrays?

In C, a **string** is just a `char` array terminated by a **null character** `'\0'` (ASCII value 0).

```c
char s[] = "hello";
/* Stored as: {'h', 'e', 'l', 'l', 'o', '\0'} — 6 bytes, not 5! */

/* Equivalent initialization: */
char s[] = {'h', 'e', 'l', 'l', 'o', '\0'};
```

**String length ≠ array size:**
```c
char s[] = "hello";
strlen(s) == 5    /* doesn't count '\0' */
sizeof(s) == 6    /* counts '\0' */
```

**How to traverse a string:**
```c
/* Option 1: index until '\0' */
for (int i = 0; s[i] != '\0'; i++) ...

/* Option 2: pointer, check for '\0' which is falsy */
for (char *p = s; *p; p++) ...   /* *p is false when *p == '\0' */

/* Option 3: use strlen */
for (int i = 0, n = strlen(s); i < n; i++) ...
```

---

### Q3. What's the difference between `char arr[]` and `char *ptr` for strings?

```c
char arr[] = "hello";    /* array — 6 bytes on stack, MUTABLE */
char *ptr = "hello";     /* pointer — points to READ-ONLY string literal */
```

| | `char arr[]` | `char *ptr` |
|---|---|---|
| Modifiable? | ✅ Yes | ❌ No (undefined behavior) |
| `sizeof()` | 6 (size of array) | 8 (pointer size on 64-bit) |
| Can reassign? | ❌ No (`arr = ...` illegal) | ✅ Yes (`ptr = "world"` OK) |
| Storage | Stack (if local) | Stack for ptr var; literal in read-only segment |

**Which to use?** Use `char arr[]` when you need to modify the string. Use `char *ptr` when pointing to string literals you'll only read.

---

### Q4. What are the key string functions from `<string.h>`?

```c
strlen(s)           /* length of s, not counting '\0' */
strcpy(dst, src)    /* copy src to dst — dst must be big enough! */
strncpy(dst, src, n)/* copy at most n chars — pads with '\0' if src shorter */
strcat(dst, src)    /* append src to end of dst */
strncat(dst, src, n)/* append at most n chars of src */
strcmp(s, t)        /* <0 if s<t, 0 if equal, >0 if s>t */
strncmp(s, t, n)    /* compare at most n chars */
strchr(s, c)        /* pointer to first occurrence of c in s, or NULL */
strrchr(s, c)       /* pointer to last occurrence of c in s, or NULL */
strstr(s, t)        /* pointer to first occurrence of string t in s, or NULL */
```

**Deadly mistake:**
```c
char *s = "hello";
strcpy(s, "world");   /* UNDEFINED BEHAVIOR — s points to read-only literal! */

char s[10] = "hello";
strcpy(s, "world");   /* OK — s is a mutable array with room */
```

**Why `strcmp` returns 0 for equal:**
```c
if (!strcmp(s, "yes"))  /* idiom: ! converts 0 to 1 (true) for equal case */
    printf("confirmed\n");
```

---

### Q5. How do multi-dimensional arrays work?

```c
int matrix[3][4];  /* 3 rows, 4 columns — stored row-major (C order) */
matrix[2][3]       /* row 2, column 3 */
```

**Memory layout (row-major):**
```
matrix[0][0] matrix[0][1] matrix[0][2] matrix[0][3]
matrix[1][0] matrix[1][1] ...
matrix[2][0] ...                       matrix[2][3]
```
Elements are contiguous — `&matrix[0][0] + 1 == &matrix[0][1]`.

**Passing to functions:** must specify all dimensions except first:
```c
void f(int matrix[][4], int nrows)   /* columns must be specified */
/* OR: */
void f(int (*matrix)[4], int nrows)  /* pointer to array of 4 ints */
```

**2D vs array of pointers:**
```c
int a[10][20];    /* true 2D array — 200 ints contiguous in memory */
int *b[10];       /* array of 10 pointers — each can point to different-length rows */
```
`b` is more flexible (jagged arrays) but requires manual allocation per row.

---

### Q6. What are pointer arrays? The classic `char *name[]` pattern.

```c
char *month_names[] = {
    "Illegal month", "January", "February", "March",
    "April", "May", "June", "July", "August",
    "September", "October", "November", "December"
};
/* month_names[1] == "January" — pointer to string literal */
```

This is how `main`'s `argv` works — it's an array of pointers to strings!

```c
int main(int argc, char *argv[]) {
    /* argv[0] = program name
       argv[1] = first argument
       ...
       argv[argc-1] = last argument
       argv[argc]  = NULL (guaranteed) */
}
```

**Sort strings using pointer array** (efficient — swap pointers, not strings):
```c
char *lines[MAXLINES];
/* ... fill lines[] with pointers to strings ... */
/* Sorting: swap pointers in lines[], strings themselves don't move */
```

---

### Q7. How do command-line arguments work?

```c
/* prog -x -n pattern */
int main(int argc, char *argv[]) {
    /* argc = 4, argv = {"prog", "-x", "-n", "pattern", NULL} */

    while (--argc > 0 && (*++argv)[0] == '-') {
        /* (*++argv)[0] = first char of next arg */
        /* **++argv = same thing — ** dereferences twice */
        char c;
        while ((c = *++argv[0])) {   /* walk through chars of current arg */
            switch (c) {
                case 'x': except = 1; break;
                case 'n': number = 1; break;
            }
        }
    }
    /* argv now points to first non-option arg (the pattern) */
}
```

---

### Q8. How do you implement common string operations yourself?

These come up constantly in interviews! Know them cold. 🧊

**strlen:**
```c
int my_strlen(const char *s) {
    const char *p = s;
    while (*p) p++;
    return p - s;
}
```

**strcpy:**
```c
char *my_strcpy(char *dst, const char *src) {
    char *start = dst;
    while ((*dst++ = *src++))  /* copies including '\0', then loop exits */
        ;
    return start;
}
```

**strcmp:**
```c
int my_strcmp(const char *s, const char *t) {
    for (; *s == *t; s++, t++)
        if (*s == '\0') return 0;   /* equal strings */
    return (unsigned char)*s - (unsigned char)*t;
}
/* Cast to unsigned char — important for characters > 127! */
```

**strcat:**
```c
char *my_strcat(char *s, const char *t) {
    char *start = s;
    while (*s) s++;         /* find end of s */
    while ((*s++ = *t++))   /* append t */
        ;
    return start;
}
```

---

## EXERCISES WITH SOLUTIONS

### Exercise 1-9. Replace multiple blanks with single blank.
```c
#include <stdio.h>

int main() {
    int c, lastc = 0;
    while ((c = getchar()) != EOF) {
        if (c == ' ' && lastc == ' ')
            continue;   /* skip extra blanks */
        putchar(c);
        lastc = c;
    }
    return 0;
}
```

---

### Exercise 1-16. Print length of arbitrarily long input lines.
```c
#include <stdio.h>
#define MAXLINE 1000

int getline_full(char *line, int lim) {
    int c, i = 0;
    while ((c = getchar()) != EOF && c != '\n') {
        if (i < lim - 1)
            line[i++] = c;
        /* count continues even if we stop storing */
    }
    if (c == '\n') {
        if (i < lim - 1) line[i++] = c;
    }
    line[i < lim ? i : lim-1] = '\0';
    return i;  /* actual length, may exceed lim-1 */
}
```

---

### Exercise 1-19. `reverse(s)` — reverse a string in place.
```c
#include <string.h>

void reverse(char s[]) {
    int i, j;
    char temp;
    for (i = 0, j = strlen(s) - 1; i < j; i++, j--) {
        temp = s[i];
        s[i] = s[j];
        s[j] = temp;
    }
}
```

---

### Exercise 5-8. Add bounds checking to `day_of_year` and `month_day`.
```c
static int daytab[2][13] = {
    {0,31,28,31,30,31,30,31,31,30,31,30,31},
    {0,31,29,31,30,31,30,31,31,30,31,30,31}
};

int day_of_year(int year, int month, int day) {
    if (year < 1 || month < 1 || month > 12) return -1;
    int leap = (year%4==0 && year%100!=0) || year%400==0;
    if (day < 1 || day > daytab[leap][month]) return -1;

    int i;
    for (i = 1; i < month; i++)
        day += daytab[leap][i];
    return day;
}
```

---

### Exercise 5-9. `day_of_year` and `month_day` with pointers.
```c
int day_of_year_ptr(int year, int month, int day) {
    int leap = (year%4==0 && year%100!=0) || year%400==0;
    int *p = daytab[leap];   /* pointer to the right row */
    for (int i = 1; i < month; i++)
        day += *(p + i);
    return day;
}
```

---

### Exercise 5-10. Reverse Polish expression evaluator (command-line).
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

double stack[100];
int sp = 0;

void push(double v) { stack[sp++] = v; }
double pop(void)    { return stack[--sp]; }

int main(int argc, char *argv[]) {
    for (int i = 1; i < argc; i++) {
        if (!strcmp(argv[i], "+"))
            push(pop() + pop());
        else if (!strcmp(argv[i], "*"))
            push(pop() * pop());
        else if (!strcmp(argv[i], "-")) {
            double b = pop();
            push(pop() - b);
        } else if (!strcmp(argv[i], "/")) {
            double b = pop();
            push(pop() / b);
        } else
            push(atof(argv[i]));
    }
    printf("%.8g\n", pop());
    return 0;
}
/* Usage: ./rpn 2 3 4 + *  →  14 */
```

---

### Key Interview Traps 🪤

**Trap 1: Off-by-one — no room for `'\0'`**
```c
char s[5];
strcpy(s, "hello");   /* "hello" is 6 chars with '\0' — buffer overflow! */
char s[6];            /* needs to be at least 6 */
```

**Trap 2: `strcpy` vs `strncpy` — `strncpy` doesn't guarantee null termination**
```c
char dst[5];
strncpy(dst, "hello world", 5);
/* dst = "hello" — NO '\0' at end if src longer than n! */
dst[4] = '\0';   /* manual termination required */
/* Safe alternative: */
snprintf(dst, sizeof(dst), "%s", src);  /* always null-terminates */
```

**Trap 3: Modifying `argv`**
```c
argv[0][0] = 'X';   /* technically undefined — string literals are read-only */
```

**Trap 4: Comparing strings with `==`**
```c
char *a = "hello";
char *b = "hello";
if (a == b) ...   /* compares ADDRESSES (may or may not be same) not content! */
if (strcmp(a, b) == 0) ...  /* CORRECT way to compare strings */
```

**Trap 5: Multi-dimensional array parameter decay**
```c
void f(int a[][3]) { }   /* OK */
void g(int **a) { }      /* NOT interchangeable with int a[][3]! */
```
A 2D array and an array of pointers are **different memory layouts**. Don't mix them up.

**Trap 6: `sizeof` on a string literal**
```c
sizeof("hello")   /* = 6 (includes '\0') */
strlen("hello")   /* = 5 (excludes '\0') */
```
