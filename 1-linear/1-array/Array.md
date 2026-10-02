# Arrays: Core Concepts (Language-Independent)

> Goal: understand arrays the way a systems programmer does, so the knowledge transfers to **C, C++, Java, Python, Go, Rust, JS, or any language**. This note covers **concepts only**. Algorithms (searching, sorting, two pointers, sliding window, etc.) are studied separately.
>
> Code snippets use JavaScript written in a C++ style (manual indexes, manual size, no built-in array methods). They are only there to make the ideas concrete. The ideas matter, not the syntax.

## Table of Contents
1. [What is an Array?](#1-what-is-an-array)
2. [How Memory Works (The Foundation)](#2-how-memory-works-the-foundation)
3. [The Address Formula](#3-the-address-formula)
4. [Why Indexing Starts at 0](#4-why-indexing-starts-at-0)
5. [Static vs Dynamic Arrays](#5-static-vs-dynamic-arrays)
6. [Size vs Capacity](#6-size-vs-capacity)
7. [Core Operations: Mechanics and Cost](#7-core-operations-mechanics-and-cost)
8. [How Dynamic Arrays Grow](#8-how-dynamic-arrays-grow)
9. [Cache Locality: Why Arrays Are Fast in Practice](#9-cache-locality-why-arrays-are-fast-in-practice)
10. [Multidimensional Arrays](#10-multidimensional-arrays)
11. [Bounds, Out-of-Range Access, and Safety](#11-bounds-out-of-range-access-and-safety)
12. [Value vs Reference: Copying Arrays](#12-value-vs-reference-copying-arrays)
13. [Arrays Across Languages](#13-arrays-across-languages)
14. [Special Array Concepts](#14-special-array-concepts)
15. [Arrays vs Other Structures](#15-arrays-vs-other-structures)
16. [Strengths, Weaknesses, When to Use](#16-strengths-weaknesses-when-to-use)
17. [Common Misconceptions and Mistakes](#17-common-misconceptions-and-mistakes)
18. [Concept Check Questions](#18-concept-check-questions)

---

## 1. What is an Array?

An **array** is a data structure that stores a fixed number of elements of the **same type** in **one continuous block of memory**, where each element is identified by an **index** (its position number).

```
index:    0     1     2     3     4
        +-----+-----+-----+-----+-----+
value:  | 10  | 20  | 30  | 40  | 50  |
        +-----+-----+-----+-----+-----+
```

Four defining properties:

| Property | Meaning |
|---|---|
| **Indexed** | Every element has a position; you reach it by that number |
| **Contiguous** | Elements sit directly next to each other in memory, with no gaps |
| **Homogeneous** | (Classic arrays) all elements are the same type, so each takes the same number of bytes |
| **Fixed size** | (Classic static arrays) the size is decided at creation and cannot change |

> **The one-sentence summary:** an array trades flexibility for speed. Because everything is evenly spaced and side by side, the computer can jump to any element instantly.

---

## 2. How Memory Works (The Foundation)

To really understand arrays, picture memory (RAM) as a very long street of numbered mailboxes:

```
address:  1000  1001  1002  1003  1004  1005  1006  1007 ...
          [ 1 byte each ]
```

- Memory is a huge list of **bytes**, each with a unique **address** (a number).
- Different types need different numbers of bytes:

| Type (typical) | Size |
|---|---|
| `char` / `int8` | 1 byte |
| `short` / `int16` | 2 bytes |
| `int` / `int32` / `float` | 4 bytes |
| `long` / `int64` / `double` | 8 bytes |
| pointer / reference (64-bit system) | 8 bytes |

When you create an `int` array of 5 elements, the system reserves **5 × 4 = 20 continuous bytes**:

```
address: 1000    1004    1008    1012    1016
        +-------+-------+-------+-------+-------+
        |  10   |  20   |  30   |  40   |  50   |
        +-------+-------+-------+-------+-------+
index:     0       1       2       3       4
```

Notice: addresses go up by **4** each step, because each `int` is 4 bytes.

### Stack vs Heap (where arrays live)
| | Stack | Heap |
|---|---|---|
| Typical use | Small, fixed-size local arrays | Large or dynamically sized arrays |
| Allocation | Automatic, very fast | Manual or by runtime, slower |
| Lifetime | Freed when the function returns | Lives until freed / garbage collected |
| Size limit | Small (a few MB) | Large (limited by RAM) |

In C/C++: `int a[5];` is usually on the stack. `new int[n]` / `malloc` is on the heap. In managed languages (Java, Python, JS, Go), arrays are generally allocated on the heap by the runtime.

---

## 3. The Address Formula

This single formula is the **heart of arrays**:

```
address of arr[i] = base_address + (i × element_size)
```

- `base_address`: where the array starts (address of `arr[0]`)
- `i`: the index
- `element_size`: bytes per element

**Example:** `int` array, base address = `1000`, element size = 4

```
arr[0] -> 1000 + 0×4 = 1000
arr[3] -> 1000 + 3×4 = 1012
arr[9] -> 1000 + 9×4 = 1036
```

### What this formula tells you
1. **Accessing any element costs the same**, whether it's `arr[0]` or `arr[1000000]`: one multiply, one add. That's **O(1)**, called **random access**.
2. The computer never "searches" for an element by index. It **calculates** where it is.
3. This only works because all elements have the **same size** and are **contiguous**.

```js
// Illustration of the formula in code (conceptual)
function addressOf(base, index, elementSize) {
  return base + index * elementSize;
}
console.log(addressOf(1000, 3, 4)); // 1012
```

> This is also why most languages (including C++ and Java) require array elements to be the same type: the compiler needs a fixed `element_size` for the formula to work.

---

## 4. Why Indexing Starts at 0

The index is really an **offset**: how many elements to skip from the start.

```
arr[0] -> skip 0 elements -> address = base + 0×size = base
arr[1] -> skip 1 element  -> address = base + 1×size
```

If indexing started at 1, the formula would need an extra subtraction every time (`base + (i - 1) × size`). Zero-based indexing makes the first element sit exactly at the base address and keeps the math minimal. Almost every major language is 0-based (a few, like MATLAB, Lua, and Fortran, use 1-based indexing by default).

**Useful consequences**
- Valid indexes for `n` elements: `0` to `n - 1`
- Last element: `arr[n - 1]`
- Number of elements between index `a` and `b` inclusive: `b - a + 1`

---

## 5. Static vs Dynamic Arrays

### Static array
- Size is **fixed at creation** (known at compile time in C/C++).
- Memory is reserved once and never moves.
- Cannot grow or shrink.

```cpp
int arr[5];               // C/C++ static array of 5 ints
```

### Dynamic array
- Size can **change at runtime**.
- Internally it still uses a **static array** underneath. When it fills up, it allocates a **bigger** one and copies the data over (see section 8).
- Examples: C++ `std::vector`, Java `ArrayList`, Python `list`, JS `Array`, Go slices, C# `List<T>`, Rust `Vec<T>`.

| | Static | Dynamic |
|---|---|---|
| Size | Fixed | Grows / shrinks |
| Speed | Fastest, no resizing | Fast, occasional resize cost |
| Memory | Exactly what you asked for | May hold spare capacity |
| Flexibility | Low | High |
| Use when | Size is known and fixed | Size is unknown or changes |

---

## 6. Size vs Capacity

Two different numbers describe an array that can hold fewer items than it has room for:

- **Capacity:** how many elements the memory block can hold (total slots reserved).
- **Size** (also *length* or *count*): how many slots are **actually in use**.

```
capacity = 8, size = 5

index:    0    1    2    3    4    5    6    7
        +----+----+----+----+----+----+----+----+
        | 10 | 20 | 30 | 40 | 50 |  ? |  ? |  ? |
        +----+----+----+----+----+----+----+----+
          ^---- used (size = 5) ----^   ^-- spare room
```

- The slots after `size` hold **garbage / leftover values**. They are not part of the array's logical content.
- The next free slot is always at index `size`.
- The array is **full** when `size == capacity`.
- The array is **empty** when `size == 0`.

```js
// C++-style manual tracking
const capacity = 8;
const data = new Array(capacity);   // reserve 8 slots
let size = 0;                       // nothing used yet

data[size] = 10; size = size + 1;   // append 10
data[size] = 20; size = size + 1;   // append 20
```

> In static arrays, you track `size` yourself. In dynamic arrays (`vector`, `ArrayList`), the structure tracks `size` and `capacity` for you.

---

## 7. Core Operations: Mechanics and Cost

For each operation, understand **what physically happens in memory**. That is what determines the cost.

### 7.1 Access (read) by index: O(1)
Compute the address with the formula and read it.
```js
const value = data[3];     // direct jump
```

### 7.2 Update (write) by index: O(1)
Compute the address and overwrite the value.
```js
data[3] = 99;
```

### 7.3 Traversal: O(n)
Visit every element once, in order.
```js
for (let i = 0; i < size; i++) {
  // use data[i]
}
```

### 7.4 Insert at the end: O(1) (if there is room)
Write into slot `size`, then increase `size`. No other element moves.
```js
data[size] = value;
size++;
```

### 7.5 Insert at an index (beginning / middle): O(n)
Elements must stay **contiguous with no gaps**, so you must **make room** by shifting every element from that index onward **one slot to the right**. Shift starting from the **last** element, or you will overwrite data.

```
Insert 25 at index 2 in [10, 20, 30, 40, 50 | _ _ _]

step 1: shift 50 right   [10, 20, 30, 40, _, 50]
step 2: shift 40 right   [10, 20, 30, _, 40, 50]
step 3: shift 30 right   [10, 20, _, 30, 40, 50]
step 4: place 25         [10, 20, 25, 30, 40, 50]
```
```js
for (let i = size; i > index; i--) {
  data[i] = data[i - 1];     // shift right, back to front
}
data[index] = value;
size++;
```
Cost: up to `n` elements move. Inserting at index `0` is the worst case (everything shifts). Inserting at the end is the best case (nothing shifts).

### 7.6 Delete at an index: O(n)
To avoid leaving a **hole**, shift every element after the deleted one **one slot to the left**, going **front to back**, then decrease `size`.

```
Delete index 1 in [10, 20, 30, 40, 50]

step 1: shift 30 left    [10, 30, 30, 40, 50]
step 2: shift 40 left    [10, 30, 40, 40, 50]
step 3: shift 50 left    [10, 30, 40, 50, 50]
step 4: size--           [10, 30, 40, 50] + one stale slot ignored
```
```js
for (let i = index; i < size - 1; i++) {
  data[i] = data[i + 1];     // shift left, front to back
}
size--;
```
Deleting the **last** element is O(1): just decrease `size`.

### 7.7 Search for a value (unsorted): O(n)
There is no address formula for "where is the value 40?". The array has no idea; you must check element by element. (Faster searching on **sorted** data is an algorithm topic, studied separately.)

### Summary table

| Operation | Time | Why |
|---|---|---|
| Access / update by index | O(1) | Address formula |
| Insert / delete at end | O(1) | Nothing shifts |
| Insert / delete at start or middle | O(n) | Elements shift |
| Search by value (unsorted) | O(n) | Must check each element |
| Traverse | O(n) | Visit each element |

> **Mental model:** arrays are fast at "give me element number i" and slow at "make room in the middle" or "find this value."

---

## 8. How Dynamic Arrays Grow

A dynamic array hides a **static array** inside. When `size == capacity` and you want to add one more:

1. Allocate a **new, larger block** (commonly **2×** the old capacity; some use 1.5×).
2. **Copy** all existing elements into the new block.
3. **Free** the old block.
4. Add the new element.

```
capacity 4, full:   [1][2][3][4]       -> add 5

new block (cap 8):  [1][2][3][4][5][ ][ ][ ]
```

### Why can't it just extend in place?
The memory right after the array might already belong to something else. Memory blocks can't be stretched safely, so the array has to **move**.

> **Side effect to know:** because the array moves, any old **pointer or reference to an element** becomes invalid after a resize. In C++, this is called **iterator/pointer invalidation**.

### Amortized O(1) append
A single resize costs O(n), but resizes are rare. With doubling:

```
capacity: 1 -> 2 -> 4 -> 8 -> 16 -> ...
copies:   1    2    4    8    16   ...  (total across n appends < 2n)
```
So `n` appends cost about `2n` copy operations in total, which averages to **O(1) per append**. This is called **amortized constant time**: individual appends occasionally cost O(n), but the average over many is O(1).

### Growth factor trade-off
| Factor | Effect |
|---|---|
| Bigger (2×) | Fewer resizes, more wasted spare memory |
| Smaller (1.5×) | More resizes, less waste, friendlier to memory reuse |
| Fixed increment (+10) | Resizing stays frequent -> total cost O(n²). **Avoid.** |

### Shrinking
Some implementations shrink when usage drops low (e.g., size < capacity / 4) to release memory. Many never shrink automatically (C++ `vector` keeps capacity until you call `shrink_to_fit`).

---

## 9. Cache Locality: Why Arrays Are Fast in Practice

Modern CPUs are much faster than RAM. To hide the delay, the CPU keeps a small fast **cache**, and it loads memory in **chunks** called **cache lines** (typically 64 bytes), not one byte at a time.

- Reading `arr[0]` loads neighbors `arr[1]`, `arr[2]`, ... into the cache **for free**.
- The next accesses are then served from the fast cache, not slow RAM.

This is **spatial locality**. Because arrays are contiguous, **sequential traversal is extremely cache-friendly**.

```
Array:        [a][b][c][d][e][f][g][h]   <- one cache line loads many at once
Linked list:  [a] ... [b] ........ [c]   <- nodes scattered, many cache misses
```

**Practical lesson:** even when two structures have the same Big-O, the array is usually **much faster in real life** because of cache behavior. That's why arrays (or vectors) are the default choice in most real programs.

---

## 10. Multidimensional Arrays

### 10.1 Logical view
A 2D array is a table with rows and columns.
```
          col 0  col 1  col 2
row 0  [    1      2      3  ]
row 1  [    4      5      6  ]
row 2  [    7      8      9  ]
```
`m[row][col]`: `m[1][2]` is `6`.

### 10.2 Physical view: memory is 1-dimensional
Memory has no rows or columns. A 2D array must be **flattened** into one line. There are two conventions:

**Row-major order** (C, C++, Java, Python/NumPy default, Rust, JS typed arrays by convention)
Rows are stored one after another.
```
memory: [1][2][3] [4][5][6] [7][8][9]
          row 0     row 1     row 2

address(i, j) = base + (i × cols + j) × element_size
```

**Column-major order** (Fortran, MATLAB, R, Julia)
Columns are stored one after another.
```
memory: [1][4][7] [2][5][8] [3][6][9]
          col 0     col 1     col 2

address(i, j) = base + (j × rows + i) × element_size
```

**Example (row-major):** 3×3 `int` matrix, base 1000. Address of `m[1][2]` = `1000 + (1×3 + 2)×4` = `1020`.

### 10.3 Why order matters: traversal performance
Walking along the direction memory is laid out hits the cache. Walking against it jumps around.

```
Row-major array:
  loop rows outer, columns inner  -> sequential memory, FAST
  loop columns outer, rows inner  -> big strides, many cache misses, SLOWER
```
Same result, same Big-O, very different real speed on large matrices.

```js
// Flat storage with row-major indexing (how C/C++ lays out 2D arrays)
const rows = 3, cols = 4;
const flat = new Array(rows * cols);
// element (i, j) lives at flat[i * cols + j]
```

### 10.4 Array of arrays vs true 2D block
| | True 2D block (C `int a[3][4]`) | Array of arrays (Java `int[][]`, JS, Python lists) |
|---|---|---|
| Memory | One contiguous block | Outer array of references to separate row arrays |
| Cache behavior | Excellent | Rows contiguous; rows themselves may be scattered |
| Row lengths | All equal | Can differ (**jagged arrays**) |
| Extra memory | None | One reference per row |

### 10.4.1 Jagged arrays
Rows with different lengths (possible with array-of-arrays):
```
row 0: [1, 2, 3]
row 1: [4]
row 2: [5, 6]
```

### 10.5 3D and higher
The same idea extends:
```
address(i, j, k) = base + ((i × dim2 + j) × dim3 + k) × element_size    (row-major)
```

---

## 11. Bounds, Out-of-Range Access, and Safety

Valid indexes for an array of `n` elements are `0` to `n - 1`. What happens outside that range **depends on the language**:

| Language | Out-of-bounds read/write |
|---|---|
| **C / C++** (raw arrays) | **Undefined behavior**: garbage values, silent memory corruption, crashes, or security vulnerabilities (buffer overflow). No automatic check. |
| **C++ `vector::at()`** | Throws an exception. (`operator[]` does **not** check.) |
| **Java** | Throws `ArrayIndexOutOfBoundsException` |
| **Python** | Raises `IndexError` |
| **Rust** | Panics (safe, checked) |
| **Go** | Runtime panic |
| **JavaScript** | Reading gives `undefined` silently. Writing past the end grows the array, possibly leaving "holes." |

**Why C/C++ doesn't check:** checking costs a comparison on every access. Those languages prioritize raw speed and trust the programmer.

> **Buffer overflow** (writing past the end of an array) is one of the most famous classes of security bugs in history. Always know your bounds.

### Defensive habit
Before using an index `i`, you should be able to state: **`0 <= i < size`**.

```js
function safeGet(data, size, i) {
  if (i < 0 || i >= size) {
    throw new Error("Index out of range");
  }
  return data[i];
}
```

### Uninitialized memory
In C/C++, a freshly created local array contains **garbage** (leftover bytes). Always initialize: `int a[5] = {0};`. Managed languages typically zero-initialize numeric arrays or fill with null/undefined.

---

## 12. Value vs Reference: Copying Arrays

### Two meanings of "copy"
- **Copy the reference (alias):** two variables point to the **same** array. Changing one changes both.
- **Copy the data:** create a **new** block and copy each element.

```js
const a = [1, 2, 3];
const b = a;          // alias: SAME array in memory
b[0] = 99;
console.log(a[0]);    // 99  (surprise!)
```

### Behavior by language
| Language | `b = a` for arrays |
|---|---|
| C++ `std::vector` | **Copies the data** (value semantics) |
| C++ raw array | Can't assign directly; pointer assignment copies the address only |
| Java, Python, JS, C# | Copies the **reference** (both names point to one array) |
| Go slices | Copy a small descriptor (pointer, length, capacity); the **underlying array is shared** |
| Rust | Moves ownership (the original can no longer be used) or clones explicitly |

### Shallow vs deep copy
When elements themselves are references to other objects (e.g., an array of arrays):
- **Shallow copy:** copies the outer array; the inner objects are still **shared**.
- **Deep copy:** copies everything, recursively.

```
Shallow copy of [[1,2],[3,4]]:
  new outer array -> points to the SAME inner arrays
  modify inner -> the change shows through both copies
```

### Passing arrays to functions
- C/C++: a raw array **decays to a pointer** (the size is lost; pass the size separately). `std::vector` passes by value (a copy!) unless passed by reference (`&`).
- Java/Python/JS: the reference is passed, so the function **can modify the original**.

---

## 13. Arrays Across Languages

The concept is universal. Only the packaging differs.

| Language | Fixed-size array | Dynamic array | Notes |
|---|---|---|---|
| **C** | `int a[5]` | `malloc` + `realloc` manually | You manage everything |
| **C++** | `int a[5]`, `std::array<int,5>` | `std::vector<int>` | Value semantics, templates |
| **Java** | `int[] a = new int[5]` | `ArrayList<Integer>` | Fixed arrays can't resize; boxed generics |
| **Python** | `array` module / NumPy `ndarray` | `list` | `list` stores references to objects, not packed values |
| **JavaScript** | Typed arrays (`Int32Array`) | `Array` | Plain arrays are flexible objects |
| **Go** | `[5]int` | slice `[]int` | Slices view an underlying array |
| **Rust** | `[i32; 5]` | `Vec<i32>` | Checked bounds, ownership rules |
| **C#** | `int[]` | `List<int>` | |

### Important idea: "packed" vs "array of references"
- **Packed arrays** (C `int[]`, Java `int[]`, NumPy, typed arrays): the actual values sit contiguously. Compact and cache-friendly.
- **Arrays of references** (Python `list`, Java `Integer[]`, JS arrays of objects): the array holds **pointers**, and the real values live elsewhere on the heap. More flexible (mixed types possible) but uses more memory and has worse cache behavior.

This is why a Python `list` of a million numbers uses far more memory than a C array of a million ints.

---

## 14. Special Array Concepts

### 14.1 Array of structs / objects
Each element can itself be a composite record. In languages with value types (C, C++, Rust, Go), the whole record is stored inline:
```
element_size = size of the entire struct
[ id | age | score ][ id | age | score ][ id | age | score ] ...
```
(Alignment/padding may add extra bytes inside each struct.)

### 14.2 Sparse arrays
An array where **most elements are zero/empty**. A normal array wastes memory storing all those zeros. Alternatives: store only non-zero entries (e.g., index-value pairs, a hash map, or special sparse formats).

### 14.3 Circular array (ring buffer)
A fixed-size array treated as if its end connects back to its start, using modular arithmetic.
```
physical index = (head + logicalIndex) % capacity
```
```
capacity 5:    [d][e][ ][ ][c]     head = 4 (points at c)
logical order:   c, d, e   (wraps around)
```
It lets you add/remove at both ends in O(1) without shifting. This is the foundation of **queues, deques, and buffers** (audio, networking, logging).

### 14.4 Typed / fixed-width arrays
Arrays with a declared element type and width (e.g., `Int32Array`, C `uint8_t[]`). They give predictable memory size and are used for binary data, graphics, and performance-critical code.

### 14.5 Bit arrays (bitsets)
Store one **bit** per element instead of one byte. A boolean array of 1,000,000 flags can fit in about 125 KB instead of 1 MB. Access uses bit math:
```
byte index = i / 8
bit position = i % 8
```

### 14.6 Associative "arrays"
Despite the name, **hash maps / dictionaries / associative arrays** are a different structure (indexed by arbitrary keys, not contiguous positions). Don't confuse them with true arrays.

### 14.7 Memory alignment (brief)
CPUs read data fastest when it sits at addresses that are multiples of its size (4-byte `int` at an address divisible by 4). Compilers add **padding** to keep things aligned, which can make `sizeof(struct)` bigger than the sum of its fields.

---

## 15. Arrays vs Other Structures

| Feature | Array | Linked List | Hash Map |
|---|---|---|---|
| Memory layout | Contiguous | Scattered nodes with pointers | Table of buckets |
| Access by index | **O(1)** | O(n) | n/a (access by key) |
| Search by value | O(n) | O(n) | **O(1)** average |
| Insert / delete at start | O(n) | **O(1)** | n/a |
| Insert / delete at end | O(1) amortized | O(1) with tail pointer | O(1) average |
| Insert / delete in middle (position known) | O(n) shift | **O(1)** relink | n/a |
| Extra memory per element | None | 1-2 pointers | Overhead for buckets / hashing |
| Cache friendliness | **Excellent** | Poor | Moderate |
| Resizing | Costly occasionally | Never (grows one node at a time) | Rehash occasionally |
| Ordered by position | Yes | Yes | No (generally) |

**Key trade-off:** arrays win on random access and cache speed. Linked lists win on cheap insert/delete **when you already hold the position**, but finding that position costs O(n), and cache misses usually make them slower than arrays in practice.

---

## 16. Strengths, Weaknesses, When to Use

### Strengths
- **O(1) random access** by index
- **Cache-friendly**, fast iteration in practice
- **Low memory overhead** (no per-element pointers)
- **Simple**, predictable memory layout
- Foundation for many other structures (stacks, queues, heaps, hash tables, matrices, strings)

### Weaknesses
- **Insert/delete in the middle or front is O(n)** (shifting)
- **Fixed size** (static) or **occasional costly resize** (dynamic)
- **Needs contiguous memory**: a very large array can fail to allocate even when total free memory is enough, if it's fragmented
- **Searching by value is O(n)** if unordered
- **Wasted space** from spare capacity

### Use an array when
- You need **fast access by position**
- The size is **known** or changes mostly by appending
- You will **iterate** over elements often
- Data is naturally **sequential or tabular** (a list of scores, pixels in an image, a grid)

### Consider something else when
- You frequently **insert/delete at the front or middle** -> linked list or deque
- You need fast **lookup by key** -> hash map
- You need **sorted order with fast inserts** -> balanced tree
- Memory is **very fragmented / huge** -> chunked structures

---

## 17. Common Misconceptions and Mistakes

1. **"Arrays store values in the order I inserted them."** Only in the sense that positions are fixed. Insert in the middle shifts everything.
2. **"Access is O(1), so everything is fast."** Insert/delete in the middle and value search are O(n).
3. **"`arr.length` means how many items I stored."** It may be the **capacity** (static arrays) or the **size** (dynamic arrays). Know which one your language gives you.
4. **Off-by-one errors.** Valid indexes are `0` to `n - 1`. The last index is `n - 1`, not `n`.
5. **Assuming out-of-bounds always errors.** In C/C++ it's silent undefined behavior; in JS it returns `undefined`.
6. **Assuming `b = a` copies an array.** In many languages it only copies the reference.
7. **Forgetting that resizing moves the array**, which invalidates old pointers/references to its elements.
8. **Believing dynamic arrays are "free" to grow.** Append is amortized O(1), but individual resizes cost O(n) and temporarily need memory for both old and new blocks.
9. **Confusing size and capacity.**
10. **Ignoring memory layout in 2D arrays**, such as iterating column-first over a row-major array on big data.
11. **Confusing arrays with hash maps** because some languages call dictionaries "associative arrays."
12. **Reading uninitialized memory** in C/C++.

---

## 18. Concept Check Questions

Answer these without looking, then check the hints.

1. Why can an array access `arr[i]` be done in constant time?
2. An `int` array (4 bytes per element) starts at address 2000. What is the address of `arr[7]`?
3. Why must insertion at index 0 shift all elements, and in which direction does the shifting loop run (and why)?
4. What is the difference between **size** and **capacity**?
5. Why does a dynamic array double its capacity instead of adding a fixed amount each time?
6. Why is traversing an array usually faster than traversing a linked list of the same length?
7. In a row-major 4×5 `int` matrix starting at address 5000, what is the address of `m[2][3]`?
8. Why do arrays require elements of the same size (or references of the same size)?
9. What can go wrong when reading `arr[n]` in C++? In Java? In JS?
10. After `b = a` with arrays in Java, you modify `b[0]`. Does `a[0]` change? Why?
11. Why can a pointer to an element of a dynamic array become invalid after an append?
12. When would you pick a linked list or hash map over an array?

### Hints / Answers

1. Address formula: `base + i × size`, one multiply and one add, regardless of `n`.
2. `2000 + 7 × 4 = 2028`.
3. To keep elements contiguous, everything from that index moves one slot right. Shift from the **last element backward**, otherwise you overwrite elements before copying them.
4. Capacity = total slots reserved; size = slots currently in use.
5. Doubling makes total copy work O(n) over `n` appends (amortized O(1) each). A fixed increment makes resizing frequent and total cost O(n²).
6. Contiguous memory -> cache lines load several elements at once (spatial locality). Linked list nodes are scattered, causing cache misses.
7. `5000 + (2 × 5 + 3) × 4 = 5052`.
8. The address formula needs a fixed `element_size` to jump directly to any index.
9. C++: undefined behavior (garbage, corruption, crash). Java: `ArrayIndexOutOfBoundsException`. JS: `undefined`.
10. Yes. `b = a` copies the reference, so both names point to the same array.
11. A resize allocates a new block and moves the data, so the old address no longer holds the array.
12. Linked list: frequent insert/delete at known positions or ends. Hash map: fast lookup by key rather than by position.

---

## Study Tips
- **Draw memory.** For every concept, sketch the boxes, indexes, and addresses. Visualizing beats memorizing.
- **Always ask:** *what does this operation do to memory?* The cost follows directly.
- Practice the **address formula** until it is automatic, including the 2D version.
- Re-implement a **dynamic array** from scratch in two different languages. If you can, you understand arrays.
- Remember the core trade-off: **arrays buy fast access by giving up flexible insertion**.
