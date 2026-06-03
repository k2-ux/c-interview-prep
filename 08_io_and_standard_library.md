# C Interview Prep — File 8: Input/Output & Standard Library

> C's I/O is not part of the language itself — it's all in the standard library. This is one of C's strengths: the core language stays lean, and I/O is replaceable. printf is basically a function someone wrote, just a really, REALLY good one. 🖨️

---

## THEORETICAL Q&A

### Q1. What is the standard I/O model in C?

Everything in C I/O is based on **streams** — sequences of characters. There are two types:
- **Text stream:** sequence of lines, each ending with `'\n'`. The library handles OS-specific newline conversions.
- **Binary stream:** raw bytes — what you write is what you read back.

Three streams are open automatically when a program starts:
| Name | File Pointer | Default Connection |
|------|-------------|-------------------|
| `stdin` | standard input | keyboard |
| `stdout` | standard output | screen |
| `stderr` | standard error | screen (unbuffered) |

All are of type `FILE *` (defined in `<stdio.h>`).

**Redirection (shell-level, transparent to program):**
```bash
prog < infile           # stdin reads from infile
prog > outfile          # stdout writes to outfile
prog >> outfile         # stdout appends to outfile
prog1 | prog2           # stdout of prog1 → stdin of prog2
prog 2> errfile         # stderr writes to errfile
```

---

### Q2. What are `getchar`, `putchar`, and their FILE-based equivalents?

**Character-at-a-time I/O:**
```c
int c = getchar();          /* read one char from stdin — returns int! */
putchar(c);                 /* write one char to stdout */

int getc(FILE *fp);         /* getchar is getc(stdin) */
int putc(int c, FILE *fp);  /* putchar(c) is putc(c, stdout) */
int fgetc(FILE *fp);        /* function version of getc (macro-safe) */
int fputc(int c, FILE *fp); /* function version of putc */
```

**Why does `getchar` return `int` and not `char`?** 🤔
Because it needs to return **both** all possible `char` values (0..255) AND the special value `EOF` (typically −1). If it returned `char`, `EOF` might look like a valid character on machines where `char` is unsigned!

```c
/* CORRECT */
int c;
while ((c = getchar()) != EOF) { ... }

/* WRONG — if char is unsigned, c can never be -1 (EOF) */
char c;
while ((c = getchar()) != EOF) { ... }  /* potential infinite loop! */
```

---

### Q3. What do `printf` and `scanf` format specifiers mean?

**`printf` format:**
```
%[flags][width][.precision][length]conversion
```

| Conversion | Prints as |
|-----------|-----------|
| `d`, `i` | Signed decimal integer |
| `u` | Unsigned decimal integer |
| `o` | Unsigned octal |
| `x`, `X` | Unsigned hex (lower/upper) |
| `f` | Float: `[-]ddd.ddd` |
| `e`, `E` | Float: `[-]d.ddde±xx` |
| `g`, `G` | Shorter of `%f` and `%e` |
| `c` | Single character |
| `s` | String (until `'\0'` or precision) |
| `p` | Pointer (address) |
| `%%` | Literal `%` |

**Flags:** `-` (left-align), `+` (always show sign), `0` (zero-pad), `#` (alternate form: `0x` for hex)

**Length modifiers:** `h` (short), `l` (long), `L` (long double)

```c
printf("%5d",   42);    /*    42  — right-aligned in field of 5 */
printf("%-5d",  42);    /* 42     — left-aligned */
printf("%05d",  42);    /* 00042  — zero-padded */
printf("%.3f",  3.14159); /* 3.142 — 3 decimal places */
printf("%10.3f", 3.14);  /*      3.140 — width 10, 3 decimal places */
printf("%s",   "hi");   /* hi */
printf("%.3s", "hello");/* hel — max 3 chars from string */
printf("%p",   &x);     /* 0x7fff... — pointer value */
printf("%ld",  1234L);  /* long */
printf("%zu",  sizeof(int)); /* size_t */
```

**`scanf` format:**
```c
int n;
float f;
char str[100];

scanf("%d", &n);         /* read int — MUST pass address! */
scanf("%f", &f);         /* read float */
scanf("%s", str);        /* read whitespace-delimited token into str */
scanf("%d %d", &a, &b);  /* read two ints */
scanf("%lf", &d);        /* read double — use %lf for double in scanf! */
```

**Common scanf pitfall:** `scanf` for `double` requires `%lf` (not `%f`). For `printf`, `%f` works for both `float` and `double` (due to default argument promotion), but `scanf` needs `%lf` for `double`.

---

### Q4. How do you do formatted input/output with `sprintf` and `sscanf`?

```c
/* sprintf: printf to a string buffer */
char buf[100];
sprintf(buf, "%d-%02d-%02d", year, month, day);
/* buf = "2024-03-15" */

/* sscanf: scanf from a string */
char date[] = "2024-03-15";
int y, m, d;
sscanf(date, "%d-%d-%d", &y, &m, &d);
/* y=2024, m=3, d=15 */
```

**⚠️ Warning:** `sprintf` has no bounds checking! Use `snprintf` instead:
```c
snprintf(buf, sizeof(buf), "%.50s", long_string);   /* safe — won't overflow */
```

---

### Q5. How does file I/O work with `fopen`, `fclose`, `fread`, `fwrite`?

```c
#include <stdio.h>

FILE *fp = fopen("filename.txt", "r");  /* open for reading */
if (fp == NULL) {
    perror("fopen");  /* prints system error message */
    exit(1);
}

/* Read/write */
int c;
while ((c = fgetc(fp)) != EOF)
    putchar(c);

/* OR for formatted: */
int n;
fscanf(fp, "%d", &n);
fprintf(fp, "%d\n", n);

/* OR for lines: */
char line[1000];
while (fgets(line, sizeof(line), fp) != NULL)
    fputs(line, stdout);

fclose(fp);  /* ALWAYS close when done */
```

**File modes:**
| Mode | Meaning |
|------|---------|
| `"r"` | Read only — file must exist |
| `"w"` | Write only — creates/truncates |
| `"a"` | Append — creates if needed |
| `"r+"` | Read+write — file must exist |
| `"w+"` | Read+write — creates/truncates |
| `"a+"` | Read+append — creates if needed |
| `"rb"`, `"wb"`, etc. | Binary mode |

---

### Q6. What is `stderr` and `exit`? How should errors be handled?

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    FILE *fp;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s filename\n", argv[0]);  /* to stderr! */
        exit(1);   /* non-zero = error */
    }

    if ((fp = fopen(argv[1], "r")) == NULL) {
        fprintf(stderr, "%s: can't open %s\n", argv[0], argv[1]);
        exit(1);
    }

    /* ... process file ... */

    if (ferror(fp)) {
        fprintf(stderr, "%s: error reading %s\n", argv[0], argv[1]);
        exit(2);
    }

    fclose(fp);
    exit(0);   /* 0 = success */
}
```

**Why `stderr`?** Because `stdout` might be redirected to a file or pipe. Error messages to `stderr` always appear on the terminal so the user sees them.

**`exit` vs `return` from `main`:** They're equivalent for `main`. `exit` can be called from anywhere; `return` only from `main`. Both call `atexit` functions and flush buffers.

---

### Q7. What are the key functions from other standard headers?

**`<string.h>`:**
```c
strlen, strcpy, strncpy, strcat, strncat, strcmp, strncmp
strchr, strrchr, strstr, strtok, memcpy, memmove, memset, memcmp
```

**`<ctype.h>`:**
```c
isalpha(c), isdigit(c), isalnum(c), isspace(c)
isupper(c), islower(c)
toupper(c), tolower(c)
```

**`<stdlib.h>`:**
```c
malloc(n)          /* allocate n bytes, uninitialized */
calloc(n, size)    /* allocate n*size bytes, initialized to 0 */
realloc(p, size)   /* resize allocation at p */
free(p)            /* release allocation */
atoi(s)            /* string to int */
atof(s)            /* string to double */
strtol(s, end, base) /* string to long, robust version */
rand()             /* pseudo-random int 0..RAND_MAX */
srand(seed)        /* set random seed */
qsort(arr, n, size, cmp)  /* sort */
bsearch(key, arr, n, size, cmp) /* binary search */
abs(n), labs(n)    /* absolute value */
exit(status)       /* terminate program */
system(cmd)        /* execute shell command */
getenv(name)       /* get environment variable */
```

**`<math.h>`:**
```c
sin, cos, tan, asin, acos, atan2
exp, log, log10, pow, sqrt, fabs
ceil, floor, fmod
```
Remember to link with `-lm` on Unix: `gcc prog.c -lm`

---

### Q8. What is `fgets` vs `gets`? What about `fgets` vs `scanf("%s")`?

**`gets` — NEVER USE IT** 🚨
```c
char buf[100];
gets(buf);   /* reads until '\n', NO LENGTH LIMIT — classic buffer overflow! */
/* gets is removed from C11 standard */
```

**`fgets` — safe alternative:**
```c
char buf[100];
fgets(buf, sizeof(buf), stdin);  /* reads at most sizeof(buf)-1 chars */
/* INCLUDES the '\n' if it fits — remove manually if unwanted: */
buf[strcspn(buf, "\n")] = '\0';
```

**`scanf("%s")` — also dangerous:**
```c
char word[10];
scanf("%s", word);   /* reads until whitespace — no length limit! */
scanf("%9s", word);  /* SAFER — limit to 9 chars + '\0' */
```

**`fgets` vs `scanf("%s")`:**
- `fgets` reads a whole line including spaces.
- `scanf("%s")` stops at whitespace — reads one word.

---

### Q9. What is `ungetc`? What's it used for?

```c
int ungetc(int c, FILE *fp);
/* Pushes character c back onto the input stream */
/* Next read of fp will return c first */
/* Only ONE character of pushback is guaranteed */
```

Used when you've read one character too many (lookahead):
```c
/* Collect an integer — stop when non-digit seen, push it back */
int read_int(FILE *fp) {
    int c, val = 0;
    while (isdigit(c = fgetc(fp)))
        val = val * 10 + (c - '0');
    if (c != EOF) ungetc(c, fp);   /* push back the non-digit */
    return val;
}
```

---

## EXERCISES WITH SOLUTIONS

### Exercise 7-1. Convert case based on program name.
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>

int main(int argc, char *argv[]) {
    int (*convert)(int) = NULL;

    if (strstr(argv[0], "upper"))
        convert = toupper;
    else if (strstr(argv[0], "lower"))
        convert = tolower;
    else {
        fprintf(stderr, "Name program 'upper' or 'lower'\n");
        return 1;
    }

    int c;
    while ((c = getchar()) != EOF)
        putchar(convert(c));
    return 0;
}
```

---

### Exercise 7-2. Print non-graphic chars in hex/octal.
```c
#include <stdio.h>
#include <ctype.h>

#define LINEWIDTH 70

int main() {
    int c, col = 0;

    while ((c = getchar()) != EOF) {
        if (isprint(c) && c != '\\') {
            putchar(c);
            col++;
        } else {
            int n = printf("\\x%02x", (unsigned char)c);
            col += n;
        }

        if (col >= LINEWIDTH) {
            putchar('\n');
            col = 0;
        }
    }
    if (col > 0) putchar('\n');
    return 0;
}
```

---

### Exercise 7-6. Compare two files, print first differing line.
```c
#include <stdio.h>
#include <string.h>

#define MAXLINE 1000

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Usage: %s file1 file2\n", argv[0]);
        return 1;
    }

    FILE *f1 = fopen(argv[1], "r");
    FILE *f2 = fopen(argv[2], "r");

    if (!f1 || !f2) { perror("fopen"); return 1; }

    char line1[MAXLINE], line2[MAXLINE];
    int lineno = 0;

    while (fgets(line1, MAXLINE, f1) && fgets(line2, MAXLINE, f2)) {
        lineno++;
        if (strcmp(line1, line2) != 0) {
            printf("Line %d differs:\n< %s> %s", lineno, line1, line2);
            fclose(f1); fclose(f2);
            return 1;
        }
    }

    if (!feof(f1) || !feof(f2))
        printf("Files have different numbers of lines\n");
    else
        printf("Files are identical\n");

    fclose(f1); fclose(f2);
    return 0;
}
```

---

### Key Interview Traps 🪤

**Trap 1: Forgetting `&` in `scanf`**
```c
int n;
scanf("%d", n);    /* WRONG — n is passed by value, not address! */
scanf("%d", &n);   /* CORRECT */
```
This is so common it's practically a C rite of passage. The compiler may not warn. The program will crash or silently produce wrong output.

**Trap 2: `printf(s)` vs `printf("%s", s)`**
```c
char s[] = "Hello %d World";
printf(s);           /* CRASH — %d with no argument! */
printf("%s", s);     /* SAFE */
```

**Trap 3: `fclose` and return value**
```c
fclose(fp);   /* Most people ignore the return value */
/* fclose returns EOF on error (e.g., write failure for buffered output) */
/* For correctness: if (fclose(fp) == EOF) { handle error } */
```

**Trap 4: `fgets` keeps the `'\n'`**
```c
char buf[100];
fgets(buf, sizeof(buf), stdin);
printf("You entered: [%s]\n", buf);
/* Output: [hello\n] — the newline is there! */
```

**Trap 5: `scanf` skips whitespace for most formats, but not for `%c`**
```c
int n; char c;
scanf("%d %c", &n, &c);   /* the space in format skips whitespace before %c */
scanf("%d%c", &n, &c);    /* NO space — %c reads the '\n' from after the number! */
```

**Trap 6: `printf` with `%d` for `long`**
```c
long x = 1234567890123L;
printf("%d\n", x);    /* WRONG — %d expects int, but x is long — undefined behavior */
printf("%ld\n", x);   /* CORRECT */
```
