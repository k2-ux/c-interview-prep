# File 12: Transitioning from C to C++

> C++ is not "C with classes." It's a completely different philosophy that happens to be almost backwards-compatible with C. Think of C as a sharp knife 🔪 — precise, dangerous, powerful. C++ is a Swiss Army knife 🪖 — more tools, more safety features, but you still need to understand the knife underneath.

---

## THE BIG PICTURE: HOW C++ THINKS DIFFERENTLY

| C Thinking | C++ Thinking |
|-----------|-------------|
| "Give me a block of memory" | "Give me an object that manages its own memory" |
| "I'll call `free` when I'm done" | "The destructor will clean up automatically" |
| "Pass a pointer to modify" | "Pass by reference — cleaner, same effect" |
| "Use `void *` for generic code" | "Use templates — type-safe generics" |
| "Struct is just data" | "Class = data + behaviour + invariants" |
| "Error codes from functions" | "Throw an exception" |
| "Manual everything" | "RAII — resources tied to object lifetime" |

The mental shift: **In C, you control memory. In C++, objects control their own memory.**

---

## PART 1: THINGS THAT ARE JUST BETTER SYNTAX

### References — the better pointer 📌
```cpp
// C way: use a pointer to modify caller's variable
void double_c(int *x) { *x *= 2; }
int n = 5; double_c(&n);  // n = 10

// C++ way: use a reference — no * or & noise at call site
void double_cpp(int &x) { x *= 2; }
int n = 5; double_cpp(n);  // n = 10 — looks like pass-by-value but isn't!
```

**References vs Pointers:**
| | Pointer `int *p` | Reference `int &r` |
|--|--|--|
| Can be null | ✅ Yes | ❌ No — must bind to real object |
| Can be rebound | ✅ Yes | ❌ No — fixed after initialisation |
| Requires `*` to dereference | ✅ Yes | ❌ No — used like a normal variable |
| Can do arithmetic | ✅ Yes | ❌ No |

**Pass by const reference** — the C++ idiom for "read-only access to large object":
```cpp
void print_name(const std::string &name) {  // no copy, no modification
    std::cout << name;
}
```

### `new` and `delete` — C++ allocation
```cpp
// C way
int *p = (int*)malloc(sizeof(int));
*p = 42;
free(p);

// C++ way
int *p = new int(42);   // allocates AND initialises
delete p;               // releases memory AND calls destructor

// Arrays:
int *arr = new int[10];
delete[] arr;   // note the [] — MUST match new[]!
```

**But in modern C++, you rarely use `new`/`delete` directly** — use smart pointers instead (covered below).

### `nullptr` instead of `NULL`
```cpp
// C: NULL is ((void*)0) or 0 — ambiguous
int *p = NULL;

// C++: nullptr is a proper keyword with type nullptr_t
int *p = nullptr;
void *q = nullptr;
// nullptr won't accidentally convert to int (unlike NULL)
```

### `bool` is a proper type (no header needed)
```cpp
bool found = false;
bool done = true;
// No #include <stdbool.h> needed in C++
```

---

## PART 2: THE BIGGEST NEW CONCEPTS

### Classes — Structs with superpowers 🦸
```cpp
// C struct — just data
struct Point {
    int x, y;
};

// C++ class — data + behaviour + access control
class Point {
public:          // accessible from outside
    Point(int x, int y) : x_(x), y_(y) {}   // constructor
    
    int x() const { return x_; }   // getter — const means "won't modify object"
    int y() const { return y_; }
    
    double distance_from_origin() const {
        return sqrt(x_*x_ + y_*y_);
    }
    
    Point operator+(const Point &other) const {   // operator overloading!
        return Point(x_ + other.x_, y_ + other.y_);
    }

private:         // hidden from outside
    int x_, y_;  // convention: trailing underscore for private members
};

// Usage
Point p(3, 4);
std::cout << p.distance_from_origin();  // 5.0
Point q = p + Point(1, 1);
```

**Key concept — `struct` vs `class` in C++:** Identical except `struct` members default to `public`, `class` members default to `private`. C++ `struct` can have constructors, methods, inheritance — everything a class has.

### Constructors and Destructors — automatic setup/cleanup 🏗️💥
```cpp
class FileHandle {
public:
    FileHandle(const char *path) {
        fp_ = fopen(path, "r");   // constructor: opens file
        if (!fp_) throw std::runtime_error("can't open file");
    }
    
    ~FileHandle() {
        if (fp_) fclose(fp_);    // destructor: AUTOMATICALLY called when object goes out of scope
    }
    
    int read_char() { return fgetc(fp_); }

private:
    FILE *fp_;
};

// Usage — no manual fclose needed!
void process_file(const char *path) {
    FileHandle f(path);    // constructor called — file opened
    int c;
    while ((c = f.read_char()) != EOF)
        putchar(c);
}   // f goes out of scope — destructor called — file closed AUTOMATICALLY
```

This is **RAII** (Resource Acquisition Is Initialisation) — the most important C++ pattern. Resources (memory, files, sockets, locks) are tied to object lifetime, so they CANNOT be leaked.

### RAII vs C manual management
```c
// C — manual, error-prone
void process(const char *path) {
    FILE *fp = fopen(path, "r");
    if (!fp) return;
    
    char *buf = malloc(1024);
    if (!buf) {
        fclose(fp);   // must remember to clean up!
        return;
    }
    
    // ... do work ...
    
    free(buf);
    fclose(fp);   // easy to forget, especially in error paths
}
```

```cpp
// C++ with RAII — exception-safe, leak-proof
void process(const std::string &path) {
    FileHandle f(path);           // RAII for file
    std::vector<char> buf(1024);  // RAII for memory (vector manages itself)
    
    // ... do work ...
    
}   // Both f and buf destroyed automatically, even if an exception is thrown!
```

---

## PART 3: THE STANDARD LIBRARY (STL)

The STL replaces most of what you do manually in C.

### `std::string` — Proper string class 🎻
```cpp
#include <string>

// C strings are painful
char *greet_c(const char *name) {
    char *buf = malloc(strlen("Hello, ") + strlen(name) + 2);
    strcpy(buf, "Hello, ");
    strcat(buf, name);
    return buf;  // caller must free! often forgotten
}

// C++ strings — value semantics, automatic memory
std::string greet(const std::string &name) {
    return "Hello, " + name + "!";  // just works
}

std::string s = "hello";
s += " world";              // append
s.length();                 // 11
s.substr(0, 5);             // "hello"
s.find("world");            // 6
s[0] = 'H';                 // mutable
s.c_str();                  // get const char* for C APIs

// Comparison works naturally
if (s == "Hello world") { }  // not strcmp!
```

### `std::vector` — Dynamic array (replaces malloc arrays) 📦
```cpp
#include <vector>

// C: manual dynamic array
int *arr = malloc(n * sizeof(int));
arr[0] = 1;
free(arr);  // must remember

// C++: vector manages itself
std::vector<int> v;
v.push_back(1);   // add element — resizes automatically
v.push_back(2);
v.push_back(3);

v[0];            // 1 — array access, no bounds check
v.at(0);         // 1 — with bounds check (throws if out of range)
v.size();        // 3
v.front();       // 1 (first element)
v.back();        // 3 (last element)

// Initialise with values
std::vector<int> nums = {10, 20, 30, 40};

// Iterate (C++ range-based for loop — like Python for)
for (int x : nums)
    std::cout << x << "\n";

// C-style iteration still works
for (int i = 0; i < nums.size(); i++)
    std::cout << nums[i] << "\n";

// Pointer to underlying array (for C APIs)
int *raw = nums.data();
```

### Other Key STL Containers
```cpp
#include <map>
#include <unordered_map>
#include <set>
#include <list>

std::map<std::string, int> wordcount;    // sorted dictionary (BST)
wordcount["hello"]++;

std::unordered_map<std::string, int> fast_map;  // hash map (O(1) avg)
fast_map["key"] = 42;

std::set<int> unique_vals = {3, 1, 4, 1, 5, 9};  // {1, 3, 4, 5, 9} — sorted, unique

std::list<int> linked;  // doubly-linked list
linked.push_front(1);
linked.push_back(2);
```

### `std::cout` and `std::cin` — I/O
```cpp
#include <iostream>

// Output — use << (insertion operator)
std::cout << "Hello, world!" << std::endl;
std::cout << "x = " << x << ", y = " << y << "\n";

// Input — use >> (extraction operator)
int n;
std::cin >> n;

std::string name;
std::cin >> name;   // reads one word
std::getline(std::cin, name);  // reads whole line

// printf still works in C++ — many C++ devs prefer it for formatting
printf("x = %d\n", x);
```

---

## PART 4: TEMPLATES — TYPE-SAFE GENERICS 🧬

In C, generic code uses `void *` (loses type info) or macros (ugly, dangerous).  
In C++, **templates** generate type-specific code at compile time:

```cpp
// Generic max function — works for ANY type that supports >
template <typename T>
T max_val(T a, T b) {
    return a > b ? a : b;
}

max_val(3, 5);          // T=int  → 5
max_val(3.14, 2.71);    // T=double → 3.14
max_val('a', 'z');      // T=char → 'z'
// No void* casts, no macros, full type safety!

// Generic swap
template <typename T>
void swap(T &a, T &b) {
    T temp = a; a = b; b = temp;
}
```

**STL algorithms use templates** — one implementation works for all types:
```cpp
#include <algorithm>
#include <vector>

std::vector<int> v = {3, 1, 4, 1, 5, 9};
std::sort(v.begin(), v.end());        // sorted: {1,1,3,4,5,9}
std::reverse(v.begin(), v.end());     // reversed
int s = std::accumulate(v.begin(), v.end(), 0);  // sum

auto it = std::find(v.begin(), v.end(), 4);  // find 4
if (it != v.end())
    std::cout << "Found: " << *it;
```

---

## PART 5: SMART POINTERS — AUTOMATIC MEMORY MANAGEMENT 🧠

The #1 source of C bugs is manual `malloc`/`free`. C++ smart pointers fix this:

### `std::unique_ptr` — Single owner (no shared, no copy)
```cpp
#include <memory>

// C way
struct Node *node = malloc(sizeof(struct Node));
node->val = 42;
// ... use node ...
free(node);  // must not forget!

// C++ way
auto node = std::make_unique<Node>(42);  // allocated
// ... use node ...
// AUTOMATICALLY freed when node goes out of scope — even if exception thrown!

// unique_ptr cannot be copied — only moved
auto p1 = std::make_unique<int>(5);
auto p2 = std::move(p1);  // ownership transferred — p1 is now null
```

### `std::shared_ptr` — Shared ownership (reference counted)
```cpp
auto p1 = std::make_shared<Node>(42);  // ref count = 1
auto p2 = p1;   // shared — ref count = 2
// When both p1 and p2 go out of scope, ref count hits 0 → freed automatically
```

### Rule: Prefer smart pointers to raw `new`/`delete`
```cpp
// Modern C++ guideline:
// - unique_ptr when one owner
// - shared_ptr when multiple owners
// - raw pointer (no ownership) when just observing/borrowing
// - NEVER call delete manually in user code
```

---

## PART 6: NAMESPACES — AVOIDING NAME COLLISIONS 📛

```cpp
// In C, everything is global — naming conflicts are common:
// math.h has `pow` — what if you want your own `pow`?

// C++ solution: namespaces
namespace mylib {
    int pow(int base, int exp) { /* ... */ }
    struct Point { int x, y; };
}

namespace std {  // the standard library lives here
    // string, vector, cout, etc.
}

// Usage:
mylib::pow(2, 10);   // your pow
std::cout << "hi";   // standard library cout

// Shortcut for common namespaces:
using std::cout;
using std::endl;
cout << "no std:: prefix needed now" << endl;

// OR (common but often discouraged in headers):
using namespace std;   // brings ALL of std into scope
cout << "hello";
```

---

## PART 7: INHERITANCE AND POLYMORPHISM 🧬

```cpp
// Base class
class Shape {
public:
    virtual double area() const = 0;   // pure virtual — MUST be overridden
    virtual void print() const {
        std::cout << "Area: " << area() << "\n";
    }
    virtual ~Shape() {}   // ALWAYS virtual destructor in base class!
};

class Circle : public Shape {
public:
    Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
private:
    double radius_;
};

class Rectangle : public Shape {
public:
    Rectangle(double w, double h) : width_(w), height_(h) {}
    double area() const override { return width_ * height_; }
private:
    double width_, height_;
};

// Polymorphism — same code works for any Shape
void print_area(const Shape &s) {
    s.print();   // calls the right version at runtime!
}

Circle c(5.0);
Rectangle r(3.0, 4.0);
print_area(c);  // "Area: 78.5..."
print_area(r);  // "Area: 12"
```

---

## PART 8: HABITS TO BREAK WHEN MOVING TO C++ 🚫

| C Habit | C++ Replacement | Why |
|---------|----------------|-----|
| `malloc`/`free` | `make_unique`, `make_shared`, or container classes | Automatic memory management |
| `char *` strings | `std::string` | Safe, value semantics, no buffer overflows |
| `printf`/`scanf` | `std::cout`/`std::cin` (or keep printf for formatting) | Type-safe, no format string bugs |
| `void *` for generics | Templates | Type-safe, no casts |
| `struct` with function pointers | Class with virtual methods | Cleaner polymorphism |
| Global variables for shared state | Class members or parameters | Encapsulation |
| `#define` for constants | `constexpr` or `const` | Typed and scoped |
| `#define` for macros | Inline functions or templates | Type-checked, debuggable |
| Manual resource tracking | RAII (constructors/destructors) | Exception-safe, leak-proof |
| `int` boolean | `bool` | Clarity |

---

## PART 9: WHAT STAYS THE SAME ✅

Good news — everything from C still works in C++!
- All operators (`*`, `&`, `++`, bitwise, etc.)
- All control flow (`if`, `for`, `while`, `switch`)
- All C standard library (`<stdio.h>` etc.) — use `<cstdio>` in C++ (same functions, `std::` namespace)
- Pointers (still exist, still work the same way)
- `sizeof`, `typedef`, `struct`, `union`
- Bitwise operations, pointer arithmetic
- `extern "C"` to call C code from C++:

```cpp
// In C++ code, to use a C library:
extern "C" {
    #include "my_c_library.h"   // tells C++ not to mangle these function names
}
```

---

## PART 10: THE LEARNING ROADMAP 🗺️

If you know C well, here's the order to learn C++ topics:

1. **References** — immediately useful, replaces most pointer-passing patterns
2. **Classes and constructors** — structs with methods
3. **`std::string` and `std::vector`** — replace char* and malloc arrays
4. **RAII and destructors** — the philosophy shift
5. **`std::unique_ptr` / `std::shared_ptr`** — replace manual memory management
6. **Templates basics** — generic functions
7. **STL algorithms** (`sort`, `find`, `transform`, etc.)
8. **Inheritance and virtual functions** — OOP
9. **Move semantics (`&&`, `std::move`)** — C++11, performance critical
10. **Lambdas** — anonymous functions: `[](int x) { return x * 2; }`
11. **Exception handling** (`try`/`catch`/`throw`)
12. **Advanced templates** and template metaprogramming (optional rabbit hole 🐰)

---

## QUICK C++ CHEAT SHEET FOR C PROGRAMMERS

```cpp
// Include standard headers (C++ versions of C headers)
#include <iostream>   // cout, cin
#include <string>     // std::string
#include <vector>     // std::vector
#include <memory>     // unique_ptr, shared_ptr
#include <algorithm>  // sort, find, etc.
#include <cstdio>     // printf, scanf (C's stdio.h)
#include <cstring>    // strlen, strcpy (C's string.h)

// Basic I/O
std::cout << "output" << "\n";
std::cin  >> variable;

// String
std::string s = "hello";
s += " world";
s.length(); s.c_str(); s.substr(0,3); s.find("ell");

// Vector
std::vector<int> v = {1,2,3};
v.push_back(4); v.size(); v[0]; v.front(); v.back();
for (int x : v) { }   // range-for

// Auto — let compiler deduce type
auto x = 42;
auto it = v.begin();

// Lambda — anonymous function
auto square = [](int x) { return x * x; };
square(5);  // 25

// Smart pointer
auto p = std::make_unique<MyClass>(args);
p->method();   // use like a pointer
// auto-deleted when p goes out of scope

// Nullptr
if (p != nullptr) { }

// Range-based for
for (const auto &item : container) { }

// Structured binding (C++17) — like Python tuple unpacking
auto [key, value] = *map.begin();
```
