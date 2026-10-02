# Arrays (JavaScript)

> A review-friendly guide to arrays: how they work in memory, what every operation costs, the built-in JS methods, and the patterns that solve most array problems. All examples are in JavaScript.

## Table of Contents
1. [What is an Array?](#1-what-is-an-array)
2. [How Arrays Work in Memory](#2-how-arrays-work-in-memory)
3. [Arrays in JavaScript (Specifics)](#3-arrays-in-javascript-specifics)
4. [Creating and Accessing Arrays](#4-creating-and-accessing-arrays)
5. [Core Operations and Their Complexity](#5-core-operations-and-their-complexity)
6. [Iterating Over Arrays](#6-iterating-over-arrays)
7. [Essential Built-in Methods](#7-essential-built-in-methods)
8. [Sorting (and Its Gotchas)](#8-sorting-and-its-gotchas)
9. [Copying Arrays (Shallow vs Deep)](#9-copying-arrays-shallow-vs-deep)
10. [2D Arrays (Matrices)](#10-2d-arrays-matrices)
11. [Core Patterns](#11-core-patterns)
12. [Common Mistakes](#12-common-mistakes)
13. [Cheat Sheet](#13-cheat-sheet)
14. [Practice Problems](#14-practice-problems)

---

## 1. What is an Array?

An array is an **ordered collection** of elements stored in **contiguous memory**, where each element is accessed by its **index** (starting from `0`).

```
index:   0    1    2    3    4
value: [ 10 | 20 | 30 | 40 | 50 ]
```

**Strengths**
- Instant access by index: O(1)
- Fast add / remove at the end: O(1) amortized
- Cache friendly (items are next to each other)
- Simple, and the base for many other structures (stacks, queues, heaps, hash tables)

**Weaknesses**
- Insert / delete at the beginning or middle: O(n) (items must shift)
- Searching an unsorted array: O(n)

---

## 2. How Arrays Work in Memory

Because elements are contiguous and (in classic arrays) equal in size, the position of any element is computed with arithmetic:

```
address of arr[i] = base address + i × element size
```

That calculation takes the same time whether `i` is 2 or 2,000,000, which is why **index access is O(1)**.

### Why inserting in the middle is slow
```
Insert 99 at index 2 in [10, 20, 30, 40]:

Before: [10, 20, 30, 40, _ ]
Shift:  [10, 20, _ , 30, 40]    <- every element after index 2 moves right
After:  [10, 20, 99, 30, 40]
```
In the worst case (insert at index 0), all `n` elements shift: **O(n)**.

### Static vs dynamic arrays
- **Static array:** fixed size chosen up front (C, Java `int[]`).
- **Dynamic array:** can grow. When full, it allocates a bigger block (often about 2x), copies everything over, and continues. Most pushes are O(1), and the occasional resize is O(n), so `push` is **amortized O(1)**. JS arrays behave like dynamic arrays.

---

## 3. Arrays in JavaScript (Specifics)

- JS arrays are **objects** with numeric-like keys plus a `length` property, heavily optimized by the engine (V8 uses contiguous storage when it can).
- They are **dynamic** (grow / shrink) and can hold **mixed types**:
  ```js
  const mixed = [1, "two", true, null, { a: 1 }, [5, 6]];
  ```
- Keeping arrays **dense** (no holes) and **single-typed** (all numbers, for example) lets the engine optimize better.
- **Typed arrays** (`Int32Array`, `Float64Array`, ...) are fixed-size, single-type, and closer to real low-level arrays. They are useful for performance-critical numeric work.
  ```js
  const ints = new Int32Array(5);   // [0, 0, 0, 0, 0]
  ```
- `length` is always one more than the highest index and is **writable**:
  ```js
  const a = [1, 2, 3, 4];
  a.length = 2;        // [1, 2]  (truncates)
  a.length = 0;        // []      (clears the array)
  ```

---

## 4. Creating and Accessing Arrays

### Creating
```js
const a = [1, 2, 3];                       // literal (most common)
const b = new Array(3);                    // 3 EMPTY slots (holes!), avoid
const c = new Array(3).fill(0);            // [0, 0, 0]
const d = Array.from({ length: 5 }, (_, i) => i * 2);  // [0, 2, 4, 6, 8]
const e = Array.of(7);                     // [7]
const f = [..."hello"];                    // ["h", "e", "l", "l", "o"]
const g = Array.from("hello");             // same idea
const h = [...new Set([1, 1, 2, 3])];      // [1, 2, 3] (unique values)
```

### Accessing
```js
const arr = [10, 20, 30, 40, 50];

arr[0];                    // 10    first
arr[arr.length - 1];       // 50    last
arr.at(-1);                // 50    last (modern, supports negatives)
arr.at(-2);                // 40
arr[99];                   // undefined (out of range does NOT throw)
```

### Modifying
```js
arr[1] = 99;               // [10, 99, 30, 40, 50]
arr[7] = 1;                // creates holes: [10, 99, 30, 40, 50, <2 empty items>, 1]
```
> Writing far past the end creates "holes" and can make the array slower. Avoid it.

---

## 5. Core Operations and Their Complexity

| Operation | JS | Time | Notes |
|---|---|---|---|
| Access by index | `arr[i]` | O(1) | |
| Update by index | `arr[i] = x` | O(1) | |
| Add at end | `arr.push(x)` | O(1) amortized | |
| Remove from end | `arr.pop()` | O(1) | |
| Add at start | `arr.unshift(x)` | O(n) | shifts everything |
| Remove from start | `arr.shift()` | O(n) | shifts everything |
| Insert in middle | `arr.splice(i, 0, x)` | O(n) | |
| Delete in middle | `arr.splice(i, 1)` | O(n) | |
| Search (unsorted) | `indexOf`, `includes`, `find` | O(n) | |
| Search (sorted) | binary search | O(log n) | |
| Copy | `slice()`, `[...arr]` | O(n) time and space | |
| Concatenate | `concat`, spread | O(n + m) | |
| Sort | `sort()` | O(n log n) | |
| Reverse | `reverse()` | O(n) | in place |
| Length | `arr.length` | O(1) | |

**Space:** an array of `n` items uses O(n) memory.

### Examples
```js
const arr = [1, 2, 3];

arr.push(4);          // [1, 2, 3, 4]         returns new length (4)
arr.pop();            // [1, 2, 3]            returns 4
arr.unshift(0);       // [0, 1, 2, 3]         returns new length
arr.shift();          // [1, 2, 3]            returns 0

arr.splice(1, 0, 99); // insert 99 at index 1 -> [1, 99, 2, 3]
arr.splice(1, 1);     // delete 1 item at index 1 -> [1, 2, 3]
```

> **Careful with `delete arr[i]`:** it leaves a hole instead of removing the element. Use `splice` instead.
> ```js
> const x = [1, 2, 3];
> delete x[1];   // [1, <empty>, 3], length still 3
> ```

---

## 6. Iterating Over Arrays

```js
const arr = ["a", "b", "c"];

// 1. Classic for loop: full control (index, break, continue, step size)
for (let i = 0; i < arr.length; i++) {
  console.log(i, arr[i]);
}

// 2. for...of: values only, simple and readable
for (const value of arr) {
  console.log(value);
}

// 3. for...of with entries(): index and value
for (const [i, value] of arr.entries()) {
  console.log(i, value);
}

// 4. forEach: functional style (cannot break early)
arr.forEach((value, i) => console.log(i, value));

// 5. for...in: AVOID for arrays (iterates keys as strings, includes inherited props)
```
All of these are **O(n)**. Prefer `for` / `for...of` when you need `break`, `continue`, or `return` inside the loop.

---

## 7. Essential Built-in Methods

### Transform and query (do NOT mutate the original)
```js
const nums = [1, 2, 3, 4, 5];

nums.map(x => x * 2);                 // [2, 4, 6, 8, 10]
nums.filter(x => x % 2 === 0);        // [2, 4]
nums.reduce((acc, x) => acc + x, 0);  // 15

nums.find(x => x > 3);                // 4        (first match or undefined)
nums.findIndex(x => x > 3);           // 3        (or -1)
nums.some(x => x > 4);                // true     (any match?)
nums.every(x => x > 0);               // true     (all match?)
nums.includes(3);                     // true
nums.indexOf(3);                      // 2        (or -1)

nums.slice(1, 4);                     // [2, 3, 4]  (end index exclusive)
nums.concat([6, 7]);                  // [1, 2, 3, 4, 5, 6, 7]
nums.join("-");                       // "1-2-3-4-5"
[1, [2, [3]]].flat(Infinity);         // [1, 2, 3]
```

### Mutate the original array
```js
const a = [3, 1, 2];
a.push(4);       // add end
a.pop();         // remove end
a.shift();       // remove start
a.unshift(0);    // add start
a.splice(1, 1);  // remove / insert in middle
a.reverse();     // reverse in place
a.sort();        // sort in place
a.fill(0);       // fill with value
```

### `slice` vs `splice` (classic confusion)
| | `slice(start, end)` | `splice(start, deleteCount, ...items)` |
|---|---|---|
| Mutates original? | No | **Yes** |
| Returns | New sub-array | Array of removed items |
| Purpose | Copy a portion | Remove / insert / replace |

### Newer non-mutating versions (ES2023)
```js
const a = [3, 1, 2];
a.toSorted();            // [1, 2, 3]   (a unchanged)
a.toReversed();          // [2, 1, 3]
a.with(0, 99);           // [99, 1, 2]
a.toSpliced(1, 1);       // [3, 2]
```

### Complexity note for chained methods
```js
nums.filter(...).map(...).reduce(...);
// Each step is O(n) and creates a new array -> O(n) total time, O(n) extra space
```

---

## 8. Sorting (and Its Gotchas)

**`sort()` with no argument sorts as STRINGS**, which gives surprising results for numbers:
```js
[10, 9, 1, 100].sort();                  // [1, 10, 100, 9]   (wrong for numbers!)
[10, 9, 1, 100].sort((a, b) => a - b);   // [1, 9, 10, 100]   ascending
[10, 9, 1, 100].sort((a, b) => b - a);   // [100, 10, 9, 1]   descending
```

Sorting objects:
```js
const people = [
  { name: "Sam", age: 30 },
  { name: "Ali", age: 25 },
];
people.sort((p, q) => p.age - q.age);        // by age
people.sort((p, q) => p.name.localeCompare(q.name)); // by name
```

Facts to remember:
- `sort` is **in place** (mutates). Use `toSorted()` or `[...arr].sort(...)` to keep the original.
- Time **O(n log n)**. JS `sort` is **stable** in modern engines (equal elements keep their order).
- The comparator returns a negative number (a first), zero (equal), or a positive number (b first).

---

## 9. Copying Arrays (Shallow vs Deep)

```js
const a = [1, 2, 3];
const b = a;            // NOT a copy: both names point to the SAME array
b.push(4);
console.log(a);         // [1, 2, 3, 4]  (a changed too!)

const c = [...a];       // shallow copy
const d = a.slice();    // shallow copy
const e = Array.from(a);// shallow copy
```

**Shallow copy** copies only the top level. Nested arrays / objects are still shared:
```js
const grid = [[1, 2], [3, 4]];
const copy = [...grid];
copy[0][0] = 99;
console.log(grid[0][0]);   // 99  (inner array shared!)
```

**Deep copy:**
```js
const deep = structuredClone(grid);        // modern, built in
const deep2 = JSON.parse(JSON.stringify(grid)); // works for plain data only
```

Comparison gotcha:
```js
[1, 2] === [1, 2];   // false (compares references, not contents)
```

---

## 10. 2D Arrays (Matrices)

An array of arrays. Access with `grid[row][col]`.

```js
const grid = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

grid[1][2];             // 6
grid.length;            // 3 (rows)
grid[0].length;         // 3 (columns)
```

### The classic bug
```js
// WRONG: every row is the SAME array reference
const bad = new Array(3).fill(new Array(3).fill(0));
bad[0][0] = 1;
console.log(bad);       // [[1,0,0],[1,0,0],[1,0,0]]  <- all rows changed

// RIGHT: create a separate row for each
const good = Array.from({ length: 3 }, () => new Array(3).fill(0));
good[0][0] = 1;
console.log(good);      // [[1,0,0],[0,0,0],[0,0,0]]
```

### Traversal
```js
// Row by row: O(rows × cols)
for (let r = 0; r < grid.length; r++) {
  for (let c = 0; c < grid[r].length; c++) {
    console.log(grid[r][c]);
  }
}
```

### Transpose
```js
function transpose(m) {
  const rows = m.length, cols = m[0].length;
  const t = Array.from({ length: cols }, () => new Array(rows));
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      t[c][r] = m[r][c];
    }
  }
  return t;
}
// Time O(r × c), Space O(r × c)
```

### Four-direction neighbors (common in grid problems)
```js
const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];  // down, up, right, left

function neighbors(grid, r, c) {
  const result = [];
  for (const [dr, dc] of dirs) {
    const nr = r + dr, nc = c + dc;
    if (nr >= 0 && nr < grid.length && nc >= 0 && nc < grid[0].length) {
      result.push([nr, nc]);
    }
  }
  return result;
}
```

---

## 11. Core Patterns

Most array interview problems are variations of a handful of patterns.

### Pattern 1: Single pass with running variables
```js
function findMax(arr) {
  let max = -Infinity;
  for (const x of arr) if (x > max) max = x;
  return max;
}
// Time O(n), Space O(1)
```

### Pattern 2: Hash map / Set for fast lookup (trade space for time)
**Two Sum:** find two indices whose values add up to `target`.
```js
function twoSum(nums, target) {
  const seen = new Map();                     // value -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
// Time O(n), Space O(n)   (brute force would be O(n²))
```

**Contains duplicate:**
```js
function containsDuplicate(nums) {
  const seen = new Set();
  for (const x of nums) {
    if (seen.has(x)) return true;
    seen.add(x);
  }
  return false;
}
// Time O(n), Space O(n)
```

### Pattern 3: Two Pointers
Use two indices that move toward each other or in the same direction. Works great on **sorted** arrays or for **in-place** edits.

**Opposite ends: Two Sum on a sorted array**
```js
function twoSumSorted(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const sum = nums[l] + nums[r];
    if (sum === target) return [l, r];
    if (sum < target) l++;     // need a bigger sum
    else r--;                  // need a smaller sum
  }
  return [];
}
// Time O(n), Space O(1)
```

**Reverse in place**
```js
function reverseInPlace(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];   // swap
    l++; r--;
  }
  return arr;
}
// Time O(n), Space O(1)
```

**Same direction (reader / writer): remove duplicates from a sorted array in place**
```js
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let w = 1;                                 // next write position
  for (let r = 1; r < nums.length; r++) {
    if (nums[r] !== nums[w - 1]) {
      nums[w++] = nums[r];
    }
  }
  return w;                                  // new length
}
// [1,1,2,2,3] -> first 3 slots become [1,2,3], returns 3
// Time O(n), Space O(1)
```

**Move zeroes to the end (keep order)**
```js
function moveZeroes(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) {
      [nums[w], nums[r]] = [nums[r], nums[w]];
      w++;
    }
  }
  return nums;
}
// Time O(n), Space O(1)
```

### Pattern 4: Sliding Window
Maintain a window over a contiguous part of the array and slide it, updating incrementally instead of recomputing.

**Fixed size: max sum of any `k` consecutive elements**
```js
function maxSumWindow(arr, k) {
  let windowSum = 0;
  for (let i = 0; i < k; i++) windowSum += arr[i];
  let best = windowSum;

  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];        // add new, drop old
    best = Math.max(best, windowSum);
  }
  return best;
}
// Time O(n), Space O(1)   (recomputing each window would be O(n·k))
```

**Variable size: longest substring without repeating characters**
```js
function longestUnique(s) {
  const lastSeen = new Map();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    if (lastSeen.has(ch) && lastSeen.get(ch) >= left) {
      left = lastSeen.get(ch) + 1;           // shrink window past the duplicate
    }
    lastSeen.set(ch, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
// Time O(n), Space O(min(n, alphabet))
```

### Pattern 5: Prefix Sum
Precompute running totals so any range sum is O(1).
```js
function buildPrefix(arr) {
  const prefix = new Array(arr.length + 1).fill(0);
  for (let i = 0; i < arr.length; i++) {
    prefix[i + 1] = prefix[i] + arr[i];
  }
  return prefix;
}

function rangeSum(prefix, l, r) {            // sum of arr[l..r] inclusive
  return prefix[r + 1] - prefix[l];
}

const p = buildPrefix([2, 4, 6, 8]);
console.log(rangeSum(p, 1, 3));              // 4 + 6 + 8 = 18
// Build: O(n) time and space. Each query: O(1).
```

### Pattern 6: Kadane's Algorithm (maximum subarray sum)
At each element, either extend the current subarray or start fresh.
```js
function maxSubarray(nums) {
  let cur = nums[0];
  let best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    cur = Math.max(nums[i], cur + nums[i]);  // extend or restart
    best = Math.max(best, cur);
  }
  return best;
}
console.log(maxSubarray([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6  ([4, -1, 2, 1])
// Time O(n), Space O(1)
```

### Pattern 7: Binary Search (sorted arrays)
```js
function binarySearch(sorted, target) {
  let lo = 0, hi = sorted.length - 1;
  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (sorted[mid] === target) return mid;
    if (sorted[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
// Time O(log n), Space O(1)
```

### Pattern 8: Reverse trick for rotation
Rotate right by `k` in place: reverse everything, then reverse the first `k`, then the rest.
```js
function rotate(nums, k) {
  const n = nums.length;
  k %= n;

  const reverse = (l, r) => {
    while (l < r) {
      [nums[l], nums[r]] = [nums[r], nums[l]];
      l++; r--;
    }
  };

  reverse(0, n - 1);      // [5,4,3,2,1] for [1,2,3,4,5] with k = 2
  reverse(0, k - 1);      // [4,5,3,2,1]
  reverse(k, n - 1);      // [4,5,1,2,3]
  return nums;
}
// Time O(n), Space O(1)
```

### Pattern 9: Left and right passes (Product of Array Except Self)
```js
function productExceptSelf(nums) {
  const n = nums.length;
  const res = new Array(n).fill(1);

  let left = 1;
  for (let i = 0; i < n; i++) {
    res[i] = left;               // product of everything to the left
    left *= nums[i];
  }

  let right = 1;
  for (let i = n - 1; i >= 0; i--) {
    res[i] *= right;             // multiply by product of everything to the right
    right *= nums[i];
  }
  return res;
}
console.log(productExceptSelf([1, 2, 3, 4])); // [24, 12, 8, 6]
// Time O(n), Space O(1) extra (output array not counted), no division needed
```

### Pattern 10: Merge two sorted arrays
```js
function mergeSorted(a, b) {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    out.push(a[i] <= b[j] ? a[i++] : b[j++]);
  }
  while (i < a.length) out.push(a[i++]);
  while (j < b.length) out.push(b[j++]);
  return out;
}
// Time O(n + m), Space O(n + m)
```

### Which pattern when?
| Clue in the problem | Try |
|---|---|
| "Find pair / complement" in unsorted data | Hash map / Set |
| Sorted array, pair or triplet | Two pointers |
| In-place removal / partition | Reader / writer pointers |
| Contiguous subarray / substring of size k | Sliding window |
| Many range-sum queries | Prefix sum |
| Max / min sum of contiguous subarray | Kadane |
| Sorted array, find something | Binary search |
| "Product / sum of everything except me" | Left and right passes |

---

## 12. Common Mistakes

1. **Off-by-one errors.** Valid indices are `0` to `length - 1`. Use `i < arr.length`, not `<=`.
2. **Using `shift()` / `unshift()` in loops.** Each is O(n), so the loop becomes O(n²). Use an index pointer for queues.
3. **`sort()` without a comparator on numbers.** It sorts as strings.
4. **Thinking `b = a` copies an array.** It copies the reference.
5. **`new Array(n).fill([])` for 2D grids.** Every row shares one array.
6. **Mutating an array while iterating over it** (removing items in a `for` loop skips elements).
7. **Using `delete arr[i]`.** It leaves holes. Use `splice`.
8. **Comparing arrays with `===`.** It compares references, not contents.
9. **Hidden O(n) work inside loops** (`includes`, `indexOf`, `slice`, spread) turning O(n) into O(n²).
10. **Forgetting empty array edge cases.** `Math.max(...[])` is `-Infinity`, and `arr[0]` of an empty array is `undefined`.
11. **Spreading huge arrays** into function calls (`Math.max(...hugeArr)`) can exceed the argument limit. Use a loop or `reduce`.
12. **Using `forEach` when you need to `break` or `return` early.** It can't.

---

## 13. Cheat Sheet

### Operation costs
| Operation | Time |
|---|---|
| Access / update `arr[i]` | O(1) |
| `push` / `pop` | O(1) amortized / O(1) |
| `shift` / `unshift` | O(n) |
| `splice`, `slice`, `concat`, spread | O(n) |
| `indexOf`, `includes`, `find` | O(n) |
| `map`, `filter`, `forEach`, `reduce` | O(n) |
| `sort` | O(n log n) |
| Binary search (sorted) | O(log n) |

### Pattern complexity
| Pattern | Time | Space |
|---|---|---|
| Single pass | O(n) | O(1) |
| Hash map lookup | O(n) | O(n) |
| Two pointers | O(n) | O(1) |
| Sliding window | O(n) | O(1) or O(k) |
| Prefix sum (build / query) | O(n) / O(1) | O(n) |
| Kadane | O(n) | O(1) |
| Binary search | O(log n) | O(1) |
| Sort then scan | O(n log n) | O(1) to O(n) |

### Handy snippets
```js
Math.max(...arr);                         // max (small arrays only)
arr.reduce((a, b) => a + b, 0);           // sum
[...new Set(arr)];                        // unique values
arr.slice().sort((a, b) => a - b);        // sorted copy
Array.from({ length: n }, (_, i) => i);   // [0..n-1]
arr.filter(Boolean);                      // remove falsy values
const [first, ...rest] = arr;             // destructuring
[arr[i], arr[j]] = [arr[j], arr[i]];      // swap
```

---

## 14. Practice Problems

Try each one before reading the solution.

1. **Reverse an array in place.**
2. **Find the second largest element** in one pass.
3. **Two Sum** (unsorted) in O(n).
4. **Best Time to Buy and Sell Stock:** max profit from one buy then one sell.
5. **Move all zeroes to the end**, keeping the order of the others.
6. **Maximum subarray sum.**
7. **Rotate an array** right by `k`.
8. **Product of array except self** without division.

### Solutions

**1, 3, 5, 6, 7, 8:** see [Core Patterns](#11-core-patterns): `reverseInPlace`, `twoSum`, `moveZeroes`, `maxSubarray`, `rotate`, `productExceptSelf`.

**2. Second largest**
```js
function secondLargest(arr) {
  let first = -Infinity, second = -Infinity;
  for (const x of arr) {
    if (x > first) {
      second = first;
      first = x;
    } else if (x > second && x < first) {
      second = x;
    }
  }
  return second === -Infinity ? null : second;
}
// Time O(n), Space O(1)
```

**4. Best time to buy and sell stock**
```js
function maxProfit(prices) {
  let minPrice = Infinity;
  let profit = 0;
  for (const p of prices) {
    minPrice = Math.min(minPrice, p);        // cheapest day so far
    profit = Math.max(profit, p - minPrice); // best profit if selling today
  }
  return profit;
}
console.log(maxProfit([7, 1, 5, 3, 6, 4]));  // 5 (buy at 1, sell at 6)
// Time O(n), Space O(1)
```

### Next-level problems
- Container With Most Water (two pointers)
- 3Sum (sort + two pointers)
- Longest Consecutive Sequence (Set)
- Subarray Sum Equals K (prefix sum + hash map)
- Maximum Product Subarray
- Merge Intervals
- Spiral Matrix and Rotate Image (2D arrays)
- Trapping Rain Water
- Sort Colors (Dutch national flag)
- Find Minimum in Rotated Sorted Array (binary search)

---

## Study Tips
- Before coding, ask: **Is it sorted? Can I use extra space? Can I modify the input? What if it's empty?**
- If your first idea is **O(n²)** (nested loops), ask whether a **Map / Set**, **two pointers**, or **sorting** gets it to O(n) or O(n log n).
- Practice writing the complexity in a comment above every function.
- Draw small examples with indices written under the values. Most bugs are off-by-one.
- Learn the patterns above deeply. Most array problems are one of them in disguise.
