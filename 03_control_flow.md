# C Interview Prep — File 3: Control Flow

> Think of control flow as traffic signals for your code. Without them, every car (instruction) would just barrel straight through — chaos! 🚦

---

## THEORETICAL Q&A

### Q1. What is the if-else statement? What's the dangling-else problem?

```c
if (expression)
    statement1
else
    statement2
```

The `else` is **optional**. The expression is evaluated; if non-zero → `statement1` runs, else → `statement2`.

**The dangling-else trap** 🪤 — which `if` does this `else` belong to?
```c
if (n > 0)
    if (a > b)
        z = a;
    else        /* This else belongs to the INNER if, not the outer! */
        z = b;
```
In C, an `else` always matches the **closest previous else-less if**. To change this:
```c
if (n > 0) {
    if (a > b)
        z = a;
}
else        /* Now this else belongs to the outer if */
    z = b;
```
**Rule of thumb:** always use braces with nested ifs. Future-you will thank you. 🙏

---

### Q2. What is the else-if chain? When should you use it over switch?

```c
if (condition1)
    ...
else if (condition2)
    ...
else if (condition3)
    ...
else
    /* default case — handles "none of the above" */
    ...
```

Use **else-if** when:
- Conditions involve ranges or complex expressions: `if (x > 10 && x < 20)`
- Conditions test different variables
- Conditions are not equality checks on a single integer

Use **switch** when:
- You're testing **one integer or character** against multiple **constant** values
- Many branches — switch can be faster (jump table vs sequential tests)

---

### Q3. How does switch work? What is fall-through?

```c
switch (expression) {      /* expression must be integral */
    case const1:
        statements;
        break;             /* EXIT the switch — without this: FALL-THROUGH */
    case const2:
        statements;
        break;
    default:
        statements;
        break;             /* good practice even on last case */
}
```

**Fall-through** 🏊 — without `break`, execution flows from one case into the next:
```c
switch (c) {
    case '0': case '1': case '2': case '3': case '4':
    case '5': case '6': case '7': case '8': case '9':
        ndigit[c - '0']++;   /* one action for all digit cases */
        break;
    case ' ': case '\n': case '\t':
        nwhite++;
        break;
    default:
        nother++;
        break;
}
```
Intentional fall-through (multiple labels for one action) is fine. Accidental fall-through is a **sneaky bug** 🐛.

**Key rules:**
- `case` values must be **integer constant expressions**.
- `default` is optional but handle it — don't leave a case unhandled silently.
- Cases can appear in **any order**; `default` doesn't have to be last.

---

### Q4. Explain while, for, and do-while loops. When to use each?

**while — test at top:**
```c
while (condition)
    body;
```
Use when you don't know how many iterations upfront, or may need zero iterations.
```c
while ((c = getchar()) != EOF)
    putchar(c);
```

**for — all loop control in one place:**
```c
for (init; condition; update)
    body;
```
Equivalent to:
```c
init;
while (condition) {
    body;
    update;
}
```
Use when you have a clear init, condition, and step — especially array/range iteration.
```c
for (i = 0; i < n; i++)
    /* process arr[i] */
```
Any of the three parts can be **omitted** (but semicolons stay):
```c
for (;;) { }   /* infinite loop — break out with break/return */
```

**do-while — test at bottom, always executes at least once:**
```c
do {
    body;
} while (condition);
```
Use when you need at least one iteration — like generating digits of a number:
```c
/* itoa: generates digits in reverse, then reverses */
do {
    s[i++] = n % 10 + '0';
} while ((n /= 10) > 0);
```

**Memory trick:** while = "check first then act", do-while = "act first then check". 🎯

---

### Q5. What is the difference between break and continue?

**`break`** — **escapes** the innermost `for`, `while`, `do`, or `switch` immediately.
```c
/* Find first negative in array */
for (i = 0; i < n; i++) {
    if (arr[i] < 0)
        break;   /* jumps to after the for loop */
}
/* i now holds the index of first negative (or n if none found) */
```

**`continue`** — **skips** the rest of the current iteration and goes to the next.
```c
/* Process only non-negative elements */
for (i = 0; i < n; i++) {
    if (arr[i] < 0)
        continue;   /* skip the rest of loop body for this i */
    process(arr[i]);
}
```

For `while` and `do-while`, `continue` goes to the **test**. For `for`, `continue` goes to the **update expression** (the third part), then the test.

`continue` does NOT apply to `switch` — only to loops. A `continue` inside a `switch` inside a `loop` affects the **loop**, not the switch.

---

### Q6. What is goto and when (if ever) should you use it?

```c
label:
    statement;

goto label;   /* jumps to label — must be in the same function */
```

The **only legitimate use**: breaking out of deeply nested loops (which `break` can't do, since break only exits one level).

```c
for (...) {
    for (...) {
        for (...) {
            if (disaster)
                goto cleanup;
        }
    }
}
cleanup:
    /* handle error */
```

**Without goto:**
```c
int found = 0;
for (i = 0; i < n && !found; i++)
    for (j = 0; j < m && !found; j++)
        if (a[i] == b[j])
            found = 1;
```

**Rule:** `goto` should be **rare**. Code with `goto` is harder to understand and maintain. K&R themselves say "goto statements should be used rarely, if at all." For 99% of situations, restructure with flags, functions, or break/continue.

---

### Q7. What makes a good loop in C?

**The at-the-top test advantage:** `while` and `for` test before each iteration. If the input has zero elements, the loop body never executes — no special case needed:
```c
while (getchar() != EOF)
    ++nc;
/* If input is empty, nc stays 0 — correct! */
```

**The null statement:** a lone `;` is a valid empty statement, useful when all work is in the loop control:
```c
/* Skip whitespace — all work done in the condition */
while ((c = getchar()) == ' ' || c == '\n' || c == '\t')
    ;   /* null statement — body intentionally empty */

/* Count chars — all work in for expression */
for (nc = 0; getchar() != EOF; ++nc)
    ;
```
Put the `;` on its own line to make it visually obvious the empty body is intentional.

---

## EXERCISES WITH SOLUTIONS

### Exercise 3-1. Binary search with only one comparison inside the loop.
**Original has two comparisons** (`<` and `>`). Here's one-comparison version:
```c
int binsearch_one(int x, int v[], int n) {
    int low = 0;
    int high = n;

    while (low < high) {
        int mid = low + (high - low) / 2;
        if (x <= v[mid])
            high = mid;
        else
            low = mid + 1;
    }
    /* After loop, low == high. Check if found. */
    return (low < n && v[low] == x) ? low : -1;
}
```
**Trade-off:** Fewer comparisons per iteration, but always runs to completion — no early exit on exact match. For large arrays, original's early exit is often faster in practice.

---

### Exercise 3-2. `escape(s, t)` — convert special chars to escape sequences.
```c
void escape(char s[], const char t[]) {
    int i, j;
    for (i = j = 0; t[i] != '\0'; i++) {
        switch (t[i]) {
            case '\n':
                s[j++] = '\\';
                s[j++] = 'n';
                break;
            case '\t':
                s[j++] = '\\';
                s[j++] = 't';
                break;
            case '\\':
                s[j++] = '\\';
                s[j++] = '\\';
                break;
            default:
                s[j++] = t[i];
                break;
        }
    }
    s[j] = '\0';
}

/* Reverse: convert escape sequences back to real characters */
void unescape(char s[], const char t[]) {
    int i, j;
    for (i = j = 0; t[i] != '\0'; i++) {
        if (t[i] == '\\') {
            i++;
            switch (t[i]) {
                case 'n':  s[j++] = '\n'; break;
                case 't':  s[j++] = '\t'; break;
                case '\\': s[j++] = '\\'; break;
                default:
                    s[j++] = '\\';
                    s[j++] = t[i];
                    break;
            }
        } else {
            s[j++] = t[i];
        }
    }
    s[j] = '\0';
}
```

---

### Exercise 3-3. `expand(s1, s2)` — expand shorthand like `a-z` to full list.
```c
void expand(const char s1[], char s2[]) {
    int i = 0, j = 0;

    while (s1[i] != '\0') {
        /* Check for pattern: char - char */
        if (s1[i+1] == '-' && s1[i+2] != '\0' && s1[i+2] > s1[i]) {
            /* Expand the range */
            char c;
            for (c = s1[i]; c <= s1[i+2]; c++)
                s2[j++] = c;
            i += 3;
        } else {
            s2[j++] = s1[i++];   /* literal character */
        }
    }
    s2[j] = '\0';
}

/* Test:
   expand("a-z", buf) → "abcdefghijklmnopqrstuvwxyz"
   expand("0-9", buf) → "0123456789"
   expand("a-zA-Z0-9", buf) → all alphanumerics
   expand("-a", buf)  → "-a"  (leading dash is literal)
*/
```

---

### Exercise 3-4. `itoa` that handles the most negative int.
The problem: `-INT_MIN` overflows (since INT_MIN = -2147483648 and INT_MAX = 2147483647).

```c
#include <string.h>

void itoa(int n, char s[]) {
    int i = 0, sign;

    /* Work with long to handle INT_MIN safely */
    long ln = n;
    if ((sign = ln) < 0)
        ln = -ln;

    do {
        s[i++] = ln % 10 + '0';
    } while ((ln /= 10) > 0);

    if (sign < 0)
        s[i++] = '-';
    s[i] = '\0';

    /* Reverse the string */
    int j;
    char temp;
    for (i--, j = 0; j < i; j++, i--) {
        temp = s[j]; s[j] = s[i]; s[i] = temp;
    }
}
```

---

### Exercise 3-5. `itob(n, s, b)` — convert n to base-b representation.
```c
void itob(int n, char s[], int b) {
    static const char digits[] = "0123456789abcdefghijklmnopqrstuvwxyz";
    int i = 0, sign;
    long ln = n;

    if ((sign = ln) < 0) ln = -ln;

    do {
        s[i++] = digits[ln % b];
    } while ((ln /= b) > 0);

    if (sign < 0) s[i++] = '-';
    s[i] = '\0';

    /* reverse */
    int j; char temp;
    for (i--, j = 0; j < i; j++, i--) {
        temp = s[j]; s[j] = s[i]; s[i] = temp;
    }
}
/* itob(255, buf, 16) → "ff"
   itob(8,   buf, 2)  → "1000" */
```

---

### Exercise 3-6. `itoa` with minimum field width (pad with blanks).
```c
void itoa_width(int n, char s[], int width) {
    char temp[100];
    int i = 0, sign;
    long ln = n;

    if ((sign = ln) < 0) ln = -ln;
    do { temp[i++] = ln % 10 + '0'; } while ((ln /= 10) > 0);
    if (sign < 0) temp[i++] = '-';

    /* Pad with leading blanks if needed */
    int j = 0;
    while (i < width) temp[i++] = ' ';
    temp[i] = '\0';

    /* Reverse */
    for (i--, j = 0; j < i; j++, i--) {
        char t = temp[j]; temp[j] = temp[i]; temp[i] = t;
    }
    strcpy(s, temp);
}
```

---

### Key Interview Traps 🪤

**Trap 1: Missing break in switch = silent fall-through**
```c
switch (grade) {
    case 'A': pts = 4;   /* falls through to B! */
    case 'B': pts = 3;   /* falls through to C! */
    case 'C': pts = 2; break;
}
/* grade='A' gives pts=2, not 4 */
```

**Trap 2: Semicolon after for/while accidentally makes empty loop**
```c
for (i = 0; i < n; i++);   /* the semicolon IS the body — empty loop */
    printf("i = %d\n", i); /* this runs ONCE, after the loop */
```

**Trap 3: Infinite loops from wrong update**
```c
for (i = 1; i != 0; i += 2)   /* i goes 1, 3, 5, ... never hits 0 if int overflows */
    ...
```

**Trap 4: continue in switch vs loop**
```c
for (i = 0; i < n; i++) {
    switch (arr[i]) {
        case 0: continue;   /* continues THE FOR LOOP, not the switch! */
        default: process(arr[i]);
    }
}
```
