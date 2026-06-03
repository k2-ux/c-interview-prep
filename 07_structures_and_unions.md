# C Interview Prep — File 7: Structures, Unions, Typedef & Bit-fields

> A struct is like a business card 💼 — one object that bundles together different types of related information (name, phone, email). A union is like a shape-shifter 🦎 — one piece of memory that can be a different type at different times, but only one at a time.

---

## THEORETICAL Q&A

### Q1. What is a struct? How do you declare and use one?

```c
/* Declaration of structure TYPE — no storage allocated yet */
struct point {
    int x;
    int y;
};

/* Definition of variable — storage allocated here */
struct point p1;
struct point p2 = {3, 4};   /* initialized */

/* Access members with . operator */
p1.x = 10;
p1.y = 20;
printf("(%d, %d)\n", p1.x, p1.y);
```

**Key facts:**
- `struct point` is the type; `point` alone is NOT a type (unlike C++/typedef).
- The struct tag (`point`) is optional — anonymous structs are valid.
- Members can be any type, including other structs.
- Structs can be **assigned**, **passed to**, and **returned from** functions (by value — a copy is made).
- Structs **cannot be compared** with `==` — must compare member by member.

---

### Q2. How do you pass structs to functions? What's the `->` operator?

**Pass by value (copy):** safe but expensive for large structs.
```c
struct point makepoint(int x, int y) {
    struct point temp;
    temp.x = x;
    temp.y = y;
    return temp;   /* returns a copy */
}

struct point addpoint(struct point p1, struct point p2) {
    p1.x += p2.x;   /* modifying local copy — caller's p1 unchanged */
    p1.y += p2.y;
    return p1;
}
```

**Pass by pointer (efficient for large structs):**
```c
void move(struct point *pp, int dx, int dy) {
    (*pp).x += dx;   /* dereference then member — requires parens! */
    (*pp).y += dy;   /* because . has higher precedence than * */
    /* OR equivalently: */
    pp->x += dx;    /* -> is syntactic sugar for (*pp). */
    pp->y += dy;
}

struct point p = {1, 2};
move(&p, 5, 5);   /* p.x = 6, p.y = 7 */
```

**The `->` operator:** `p->member` is exactly `(*p).member`. Use `->` with pointer-to-struct, `.` with struct directly.

---

### Q3. What is a self-referential structure? How do linked lists and trees work?

A struct can contain a **pointer to itself** (but NOT an instance of itself):

```c
struct node {
    int val;
    struct node *next;   /* pointer to next node — OK */
    /* struct node self;  ERROR — infinite size! */
};
```

**Linked list:**
```c
struct node *head = NULL;

/* Insert at front */
struct node *new_node = malloc(sizeof(struct node));
new_node->val = 42;
new_node->next = head;
head = new_node;

/* Traverse */
for (struct node *p = head; p != NULL; p = p->next)
    printf("%d\n", p->val);
```

**Binary tree:**
```c
struct tnode {
    char *word;
    int count;
    struct tnode *left;
    struct tnode *right;
};

struct tnode *addtree(struct tnode *p, char *w) {
    if (p == NULL) {
        p = malloc(sizeof(struct tnode));
        p->word = strdup(w);
        p->count = 1;
        p->left = p->right = NULL;
    } else {
        int cond = strcmp(w, p->word);
        if (cond == 0)      p->count++;
        else if (cond < 0)  p->left  = addtree(p->left, w);
        else                p->right = addtree(p->right, w);
    }
    return p;
}
```

---

### Q4. What is `typedef`? When and why to use it?

`typedef` creates a **new name** (alias) for an existing type — it does NOT create a new type.

```c
typedef int Length;             /* Length is now a synonym for int */
typedef char *String;           /* String is synonym for char* */
typedef struct point Point;     /* Point is synonym for struct point */

/* Now you can write: */
Length len = 10;
String name = "Alice";
Point p = {3, 4};
```

**Common uses:**

1. **Platform portability** — abstract over type sizes:
```c
typedef unsigned int uint32_t;   /* can redefine per platform */
```

2. **Complex types** — pointers to functions:
```c
typedef int (*Comparator)(const void *, const void *);
/* Now: Comparator cmp = strcmp;  — much cleaner than: */
/*      int (*cmp)(const void*, const void*) = strcmp; */
```

3. **Self-documenting code:**
```c
typedef struct {
    double r, theta;
} Complex;

Complex z1 = {1.0, 0.0};
```

**`typedef` vs `#define` for types:**
- `typedef` is processed by the compiler — handles complex types (pointers, arrays) correctly.
- `#define` is text substitution — can break with pointer types:
```c
#define PTR int*
PTR a, b;   /* expands to: int* a, b;  — b is int, not int*! */

typedef int* Ptr;
Ptr a, b;   /* BOTH a and b are int* */
```

---

### Q5. What is a union? How does it differ from a struct?

A `union` can hold **any one** of its members at a time. All members share the same memory location.

```c
union u_tag {
    int    ival;
    float  fval;
    char  *sval;
} u;

/* Size = size of LARGEST member (+ possible padding) */
sizeof(u) == max(sizeof(int), sizeof(float), sizeof(char*))
```

**Use case: variant record (tagged union)**
```c
struct symbol {
    char *name;
    int  type;     /* TAG: INT_TYPE, FLOAT_TYPE, STRING_TYPE */
    union {
        int    ival;
        float  fval;
        char  *sval;
    } value;
};

struct symbol s;
s.type = FLOAT_TYPE;
s.value.fval = 3.14;

/* Later, when reading: */
if (s.type == FLOAT_TYPE)
    printf("%f\n", s.value.fval);
```

**You MUST track which member is active.** Reading the wrong member is undefined behavior (though often used in low-level code for type punning — be careful!).

**Struct vs Union:**
| | `struct` | `union` |
|--|--|--|
| Memory | Each member has its own storage | All members share same storage |
| Size | Sum of member sizes + padding | Largest member size + padding |
| Can hold | All members simultaneously | Only one member at a time |

---

### Q6. What are bit-fields? When are they useful?

Bit-fields pack multiple values into a single word, using only as many bits as needed:

```c
struct flags {
    unsigned int is_keyword : 1;   /* 1 bit */
    unsigned int is_extern  : 1;   /* 1 bit */
    unsigned int is_static  : 1;   /* 1 bit */
    unsigned int priority   : 3;   /* 3 bits — values 0-7 */
};

struct flags f;
f.is_keyword = 1;
f.priority   = 5;

if (f.is_extern == 0 && f.is_static == 0)
    printf("local non-extern symbol\n");
```

**When to use:**
- When you have many boolean flags and memory is tight (embedded systems).
- When you need to match an exact hardware register layout.

**Limitations:**
- Can only be `int`, `unsigned int`, or `signed int`.
- Cannot take address of a bit-field (`&f.is_keyword` is illegal).
- Ordering within a word is **implementation-defined** — not portable for hardware registers.
- Not arrays of bit-fields.

**Alternative with masks (more portable):**
```c
#define IS_KEYWORD 0x01
#define IS_EXTERN  0x02
#define IS_STATIC  0x04

unsigned int flags = 0;
flags |= IS_KEYWORD;             /* set */
flags &= ~IS_EXTERN;             /* clear */
if (flags & IS_KEYWORD) ...      /* test */
```

---

### Q7. How does `sizeof` work with structs? What is struct padding?

Structs may have **unnamed padding** between members to satisfy alignment requirements:

```c
struct example {
    char   c;   /* 1 byte */
                /* 3 bytes padding — int needs 4-byte alignment */
    int    i;   /* 4 bytes */
    char   d;   /* 1 byte */
                /* 3 bytes padding — struct must end on 4-byte boundary */
};
/* sizeof(struct example) = 12, NOT 6! */
```

**To minimize padding, order members largest to smallest:**
```c
struct packed {
    int  i;    /* 4 bytes */
    char c;    /* 1 byte */
    char d;    /* 1 byte */
             /* 2 bytes padding */
};
/* sizeof = 8 vs 12 in previous example */
```

**Always use `sizeof(struct name)` — never manually compute struct sizes!**

---

### Q8. What is `strdup` and why does it matter with structs?

When a struct contains a `char *` member, assigning the struct copies the **pointer**, not the string:

```c
struct person { char *name; int age; };

struct person p1 = {"Alice", 30};
struct person p2 = p1;    /* p2.name == p1.name — SAME pointer! */
p2.name[0] = 'B';         /* modifies p1's name too! */
```

**Deep copy with `strdup`:**
```c
struct person copy_person(struct person p) {
    struct person result = p;
    result.name = strdup(p.name);   /* malloc + strcpy — independent copy */
    return result;
}
```
Remember to `free(p.name)` when done to avoid memory leaks.

---

## EXERCISES WITH SOLUTIONS

### Exercise 6-1. Better `getword` (handles underscores, comments, strings).
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>

int getch(void);
void ungetch(int);

int getword(char *word, int lim) {
    int c;
    char *w = word;

    while (isspace(c = getch()))
        ;

    if (c != EOF) *w++ = c;

    if (c == '_' || isalpha(c)) {
        /* identifier: letters, digits, underscores */
        for (; --lim > 0; w++)
            if (!isalnum(*w = getch()) && *w != '_') {
                ungetch(*w);
                break;
            }
    } else if (c == '"') {
        /* string literal — read until closing " */
        for (; --lim > 0; w++) {
            if ((*w = getch()) == '\\') { *w++ = '\\'; --lim; *w = getch(); }
            else if (*w == '"') break;
            else if (*w == EOF) break;
        }
    } else if (c == '/' && (c = getch()) == '*') {
        /* C comment — skip until */ 
        /* simplified: skip comment, return next word */
        int prev = 0;
        while ((c = getch()) != EOF && !(prev == '*' && c == '/'))
            prev = c;
        return getword(word, lim);
    } else {
        ungetch(c);
    }

    *w = '\0';
    return word[0];
}
```

---

### Exercise 6-2. Print variable names identical in first 6 characters.
```c
/* Approach: use a trie/hash indexed by first 6 chars, collect all matching names */
/* Simplified: sort names, then compare adjacent pairs */
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAXNAMES 10000
#define MAXLEN   100
#define PREFIXLEN 6

char names[MAXNAMES][MAXLEN];
int nnames = 0;

int cmp6(const void *a, const void *b) {
    return strncmp((char*)a, (char*)b, PREFIXLEN);
}

int main() {
    /* read names into names[][] */
    /* ... */
    qsort(names, nnames, MAXLEN, cmp6);
    for (int i = 0; i < nnames - 1; i++) {
        if (strncmp(names[i], names[i+1], PREFIXLEN) == 0 &&
            strcmp(names[i], names[i+1]) != 0) {
            /* Find the whole group */
            int j = i;
            while (j < nnames && strncmp(names[i], names[j], PREFIXLEN) == 0)
                printf("%s\n", names[j++]);
            printf("---\n");
            i = j - 1;
        }
    }
    return 0;
}
```

---

### Exercise 6-5. `undef(name)` — remove from hash table.
```c
struct nlist {
    struct nlist *next;
    char *name;
    char *defn;
};

#define HASHSIZE 101
static struct nlist *hashtab[HASHSIZE];

unsigned hash(const char *s) {
    unsigned h = 0;
    for (; *s; s++) h = *s + 31 * h;
    return h % HASHSIZE;
}

void undef(const char *name) {
    unsigned h = hash(name);
    struct nlist *prev = NULL;
    struct nlist *np = hashtab[h];

    while (np != NULL) {
        if (strcmp(name, np->name) == 0) {
            if (prev == NULL)
                hashtab[h] = np->next;
            else
                prev->next = np->next;
            free(np->name);
            free(np->defn);
            free(np);
            return;
        }
        prev = np;
        np = np->next;
    }
}
```

---

### Key Interview Traps 🪤

**Trap 1: Struct comparison**
```c
struct point a = {1, 2}, b = {1, 2};
if (a == b) ...   /* COMPILE ERROR — structs can't be == compared */
/* Must compare manually: */
if (a.x == b.x && a.y == b.y) ...
```

**Trap 2: Struct copy is shallow**
```c
struct with_ptr { char *s; int n; };
struct with_ptr a = {strdup("hello"), 5};
struct with_ptr b = a;   /* b.s and a.s point to SAME memory */
free(a.s);
printf("%s\n", b.s);   /* USE AFTER FREE — b.s is now dangling! */
```

**Trap 3: `->` vs `.`**
```c
struct point p = {1, 2};
struct point *pp = &p;

p.x   /* direct member access */
pp->x /* member via pointer — same as (*pp).x */
(*pp).x /* verbose but correct */
pp.x  /* WRONG — pp is a pointer, not a struct */
*pp.x /* WRONG — same as *(pp.x) — tries to dereference an int */
```

**Trap 4: Sizeof struct ≠ sum of member sizes**
Always measure:
```c
printf("%zu\n", sizeof(struct mystruct));   /* use this in allocations */
/* never hardcode: 1 + 4 = 5 bytes — WRONG due to padding */
```

**Trap 5: Union type punning (non-portable)**
```c
union { float f; unsigned int i; } pun;
pun.f = 1.0f;
printf("0x%x\n", pun.i);   /* reading different member — technically UB in C99 but common in C */
```
