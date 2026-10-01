# Big-O Notation: Time & Space Complexity (JavaScript)

> A review-friendly guide to understanding how algorithms scale. All examples are in JavaScript.

## Table of Contents
1. [What is Big-O?](#1-what-is-big-o)
2. [The 3 Simplification Rules](#2-the-3-simplification-rules)
3. [Common Time Complexities](#3-common-time-complexities)
4. [Space Complexity](#4-space-complexity)
5. [Best, Average, Worst Case](#5-best-average-worst-case)
6. [How to Analyze Code (Step by Step)](#6-how-to-analyze-code-step-by-step)
7. [Recursion Complexity](#7-recursion-complexity)
8. [Amortized Analysis](#8-amortized-analysis)
9. [Cost of Built-in JS Operations](#9-cost-of-built-in-js-operations)
10. [Common Mistakes](#10-common-mistakes)
11. [Cheat Sheet](#11-cheat-sheet)
12. [Practice Problems](#12-practice-problems)

---

## 1. What is Big-O?

Big-O describes how the **cost** (time or memory) of an algorithm **grows** as the input size `n` grows. It is not a stopwatch measurement. It describes the *shape* of growth, and it usually describes the **worst case** (an upper bound).

Why not just measure seconds? Because speed depends on your machine, language, and load. Big-O gives a hardware-independent way to compare algorithms.

**Example:** with `n = 1,000,000` items:

| Complexity | Approx. operations |
|---|---|
| O(1) | 1 |
| O(log n) | ~20 |
| O(n) | 1,000,000 |
| O(n log n) | ~20,000,000 |
| O(n²) | 1,000,000,000,000 |

An O(n²) algorithm that is fine for 100 items can be unusable for a million.

---

## 2. The 3 Simplification Rules

### Rule 1: Drop constants
```js
function printTwice(arr) {
  for (const x of arr) console.log(x); // n
  for (const x of arr) console.log(x); // n
}
// 2n  -> O(n)
```

### Rule 2: Drop non-dominant terms
```js
function example(arr) {
  for (const a of arr) {          // n
    for (const b of arr) {}       // n * n = n²
  }
  for (const a of arr) {}         // n
}
// n² + n -> O(n²)
```

### Rule 3: Different inputs use different variables
```js
function twoArrays(a, b) {
  for (const x of a) {}  // O(a)
  for (const y of b) {}  // O(b)
}
// O(a + b), NOT O(n)

function nested(a, b) {
  for (const x of a) {
    for (const y of b) {}
  }
}
// O(a * b)
```

**Rule of thumb:** sequential steps **add**, nested steps **multiply**.

---

## 3. Common Time Complexities

### O(1): Constant
Cost doesn't depend on input size.
```js
function getFirst(arr) {
  return arr[0];
}
```

### O(log n): Logarithmic
The problem size is **halved** each step. Binary search is the classic example (the array must be sorted).
```js
function binarySearch(sorted, target) {
  let lo = 0;
  let hi = sorted.length - 1;

  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (sorted[mid] === target) return mid;
    if (sorted[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
// Time: O(log n)   Space: O(1)
```
> Intuition: log₂(n) = "how many times can I halve n until I reach 1?" For n = 1,000,000 that's about 20.

### O(n): Linear
Touch each element a constant number of times.
```js
function findMax(arr) {
  let max = -Infinity;
  for (const x of arr) {
    if (x > max) max = x;
  }
  return max;
}
```

### O(n log n): Linearithmic
Typical of efficient sorting (merge sort, heap sort, and the average case of quick sort).
```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    result.push(left[i] <= right[j] ? left[i++] : right[j++]);
  }
  return result.concat(left.slice(i), right.slice(j));
}
// Time: O(n log n)   Space: O(n)
```
> Why? The array is split log n times (depth), and each level does O(n) work merging.

### O(n²): Quadratic
Nested loops over the same input.
```js
function hasDuplicate(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) return true;
    }
  }
  return false;
}
// Time: O(n²)   Space: O(1)
```
Bubble sort, insertion sort, and selection sort are all O(n²).

> Note: `j` starts at `i + 1`, so there are about n²/2 comparisons. Dropping the constant gives O(n²).

### O(2ⁿ): Exponential
Each step branches into two more. Typical of naive recursion.
```js
function fibNaive(n) {
  if (n <= 1) return n;
  return fibNaive(n - 1) + fibNaive(n - 2);
}
// Time: O(2ⁿ)   Space: O(n) (call stack depth)
```

### O(n!): Factorial
Generating all permutations.
```js
function permutations(arr) {
  if (arr.length <= 1) return [arr];
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    const rest = [...arr.slice(0, i), ...arr.slice(i + 1)];
    for (const p of permutations(rest)) {
      result.push([arr[i], ...p]);
    }
  }
  return result;
}
// Time: O(n!)
```

### Growth Order
```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)
```

---

## 4. Space Complexity

Space complexity measures the **extra memory** an algorithm needs as `n` grows. We usually count **auxiliary space** (extra space), not the input itself.

### O(1) space
```js
function sum(arr) {
  let total = 0;            // one variable, regardless of n
  for (const x of arr) total += x;
  return total;
}
```

### O(n) space
```js
function double(arr) {
  const result = [];        // grows with n
  for (const x of arr) result.push(x * 2);
  return result;
}
```

### O(n²) space
```js
function makeGrid(n) {
  const grid = [];
  for (let i = 0; i < n; i++) {
    grid.push(new Array(n).fill(0));  // n rows x n columns
  }
  return grid;
}
```

### Time vs Space Trade-off
You can often trade memory for speed. Example: finding two numbers that sum to a target.

```js
// Brute force: Time O(n²), Space O(1)
function twoSumBrute(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) return [i, j];
    }
  }
  return [];
}

// Hash map: Time O(n), Space O(n)
function twoSumFast(nums, target) {
  const seen = new Map();             // value -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
```
We spent O(n) extra memory to cut time from O(n²) to O(n).

---

## 5. Best, Average, Worst Case

Same algorithm, different inputs:

```js
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}
```

| Case | Situation | Complexity |
|---|---|---|
| Best | Target is the first element | O(1) |
| Average | Target is somewhere in the middle | O(n) |
| Worst | Target is last or missing | O(n) |

Related notations:
- **Big-O (O):** upper bound ("at most this slow")
- **Big-Omega (Ω):** lower bound ("at least this much")
- **Big-Theta (Θ):** tight bound (both upper and lower)

In interviews and day-to-day use, "Big-O" almost always means the **worst case**.

---

## 6. How to Analyze Code (Step by Step)

1. Identify the input(s) and name them (`n`, `m`, ...).
2. Count loops: a single loop over the input is O(n), and nested loops multiply.
3. Check whether a loop **halves or doubles** its variable. If so, it's O(log n).
4. Check hidden costs inside loops (e.g., `slice`, `includes`, `indexOf`, string concatenation).
5. Add sequential blocks, multiply nested ones.
6. Apply the simplification rules.
7. Separately, count extra memory: new arrays, maps, objects, and recursion depth.

### Log loop example
```js
for (let i = 1; i < n; i *= 2) {
  // i = 1, 2, 4, 8, ... -> runs log₂(n) times
}
// O(log n)
```

### Hidden cost example
```js
function bad(arr) {
  for (const x of arr) {          // n iterations
    if (arr.includes(x + 1)) {}   // includes is O(n)
  }
}
// O(n) * O(n) = O(n²), not O(n)!
```

Fix with a Set:
```js
function good(arr) {
  const set = new Set(arr);       // O(n)
  for (const x of arr) {          // n iterations
    if (set.has(x + 1)) {}        // O(1) average
  }
}
// O(n) time, O(n) space
```

---

## 7. Recursion Complexity

Two things to figure out: **how many calls** and **how much work per call**.

### Linear recursion: O(n) time, O(n) space
```js
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
// n calls, each O(1). Call stack depth = n -> O(n) space.
```

### Halving recursion: O(log n)
```js
function power(base, exp) {
  if (exp === 0) return 1;
  const half = power(base, Math.floor(exp / 2));
  return exp % 2 === 0 ? half * half : half * half * base;
}
// O(log n) time, O(log n) space
```

### Two branches: O(2ⁿ)
`fibNaive` above. Each call spawns two more, forming a binary tree of calls about `n` levels deep.

### Fixing it with memoization
```js
function fibMemo(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n] !== undefined) return memo[n];
  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  return memo[n];
}
// Time: O(n)   Space: O(n)
```

### Iterative version: best of both
```js
function fibIter(n) {
  if (n <= 1) return n;
  let prev = 0, curr = 1;
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  return curr;
}
// Time: O(n)   Space: O(1)
```

> **Key idea:** recursion depth = space. Every pending call lives on the call stack. Very deep recursion can throw `RangeError: Maximum call stack size exceeded` in JS.

---

## 8. Amortized Analysis

Some operations are usually cheap but occasionally expensive. **Amortized** cost is the average cost per operation over a long sequence.

`Array.prototype.push` is the classic example. The engine over-allocates capacity. Most pushes are O(1), but occasionally the array must grow and copy all elements (O(n)). Averaged out, `push` is **amortized O(1)**.

```js
const arr = [];
for (let i = 0; i < 1_000_000; i++) {
  arr.push(i);   // amortized O(1) each, O(n) total
}
```

---

## 9. Cost of Built-in JS Operations

Knowing these prevents accidental slowdowns. (Values are typical for V8; exact internals can vary.)

### Arrays
| Operation | Time |
|---|---|
| `arr[i]` read / write | O(1) |
| `push()` / `pop()` | O(1) amortized |
| `shift()` / `unshift()` | O(n), shifts every element |
| `splice()` | O(n) |
| `slice()` | O(n) |
| `indexOf()` / `includes()` / `find()` | O(n) |
| `concat()` | O(n) |
| `forEach` / `map` / `filter` / `reduce` | O(n) |
| `sort()` | O(n log n) |
| `[...arr]` spread copy | O(n) time and space |

### Objects / Map / Set
| Operation | Time (average) |
|---|---|
| `obj[key]`, `map.get/set/has/delete` | O(1) |
| `set.add/has/delete` | O(1) |
| `Object.keys/values/entries` | O(n) |

Hash tables can degrade to O(n) in the worst case (many collisions), but average O(1) is what you rely on.

### Strings
Strings are immutable. Every change creates a new string.
```js
let s = "";
for (let i = 0; i < n; i++) {
  s += "a";        // may copy -> can be O(n²) overall in a naive model
}

// Safer pattern:
const parts = [];
for (let i = 0; i < n; i++) parts.push("a");
const result = parts.join("");   // O(n)
```
(Modern engines optimize `+=` with ropes, but don't rely on it when reasoning about complexity.)

### Common pitfall: `shift()` in a loop
```js
// O(n²) total, because shift() is O(n) each time
while (queue.length) {
  const item = queue.shift();
}
```
Use an index pointer instead:
```js
let head = 0;
while (head < queue.length) {
  const item = queue[head++];   // O(1)
}
```

---

## 10. Common Mistakes

1. **Forgetting hidden loops.** `includes`, `indexOf`, `slice`, `concat`, spread, and `sort` all cost time.
2. **Counting only loops, not recursion stack space.**
3. **Using `n` for everything.** Two different inputs need two variables (`a`, `b`).
4. **Thinking O(2n) is "twice as bad" in Big-O terms.** It's still O(n). Constants matter in real life but not in Big-O.
5. **Assuming a nested loop is always O(n²).** If the inner loop runs a fixed number of times, it's O(n).
6. **Ignoring input size.** For tiny `n`, an O(n²) algorithm can be faster than an O(n log n) one in practice.
7. **Forgetting that sorting costs O(n log n)** when you "sort first and then scan."

---

## 11. Cheat Sheet

### Time complexity at a glance
| Big-O | Name | Typical example |
|---|---|---|
| O(1) | Constant | Array index, Map lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop, linear search |
| O(n log n) | Linearithmic | Merge sort, `Array.sort` |
| O(n²) | Quadratic | Nested loops, bubble sort |
| O(2ⁿ) | Exponential | Naive Fibonacci, subsets |
| O(n!) | Factorial | Permutations |

### Sorting algorithms
| Algorithm | Best | Average | Worst | Space |
|---|---|---|---|---|
| Bubble sort | O(n) | O(n²) | O(n²) | O(1) |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) |
| Selection sort | O(n²) | O(n²) | O(n²) | O(1) |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Quick sort | O(n log n) | O(n log n) | O(n²) | O(log n) |
| Heap sort | O(n log n) | O(n log n) | O(n log n) | O(1) |

### Data structures
| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Linked list | O(n) | O(n) | O(1)* | O(1)* |
| Hash table (Map/Set) | n/a | O(1) avg | O(1) avg | O(1) avg |
| Stack | O(n) | O(n) | O(1) | O(1) |
| Queue (linked / pointer) | O(n) | O(n) | O(1) | O(1) |
| Binary search tree (balanced) | O(log n) | O(log n) | O(log n) | O(log n) |

\* once you already hold the node reference.

### Quick feel for input limits (rough guide)
| Input size `n` | Acceptable complexity |
|---|---|
| ≤ 10 | O(n!) |
| ≤ 20 | O(2ⁿ) |
| ≤ 5,000 | O(n²) |
| ≤ 1,000,000 | O(n log n) |
| ≤ 100,000,000 | O(n) |
| Any | O(log n), O(1) |

---

## 12. Practice Problems

Analyze the time and space complexity of each. Answers are at the bottom.

**Q1**
```js
function q1(arr) {
  return arr[arr.length - 1];
}
```

**Q2**
```js
function q2(n) {
  let count = 0;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < 10; j++) count++;
  }
  return count;
}
```

**Q3**
```js
function q3(n) {
  let count = 0;
  for (let i = 1; i < n; i *= 2) count++;
  return count;
}
```

**Q4**
```js
function q4(arr) {
  const sorted = [...arr].sort((a, b) => a - b);
  return sorted[0];
}
```

**Q5**
```js
function q5(n) {
  if (n === 0) return 0;
  return 1 + q5(n - 1);
}
```

**Q6**
```js
function q6(a, b) {
  for (const x of a) {
    for (const y of b) console.log(x, y);
  }
}
```

**Q7**
```js
function q7(n) {
  for (let i = 0; i < n; i++) {
    for (let j = i; j < n; j++) {}
  }
}
```

**Q8**
```js
function q8(arr) {
  const seen = new Set();
  for (const x of arr) seen.add(x);
  return seen.size;
}
```

### Answers
1. **Time O(1), Space O(1).** Direct index access.
2. **Time O(n), Space O(1).** Inner loop runs a constant 10 times.
3. **Time O(log n), Space O(1).** `i` doubles each step.
4. **Time O(n log n), Space O(n).** The copy takes O(n) memory and the sort costs O(n log n). (Finding the minimum with a loop would be O(n) time, O(1) space.)
5. **Time O(n), Space O(n).** n calls stacked on the call stack.
6. **Time O(a · b), Space O(1).**
7. **Time O(n²), Space O(1).** About n²/2 iterations.
8. **Time O(n), Space O(n).** The Set can hold up to n items.

---

## Study Tips
- Always ask two questions: **"how many steps?"** and **"how much extra memory?"**
- Practice by writing the complexity in a comment above every function you write.
- Look for loops, recursion, and hidden built-in costs.
- When you see O(n²), ask: *can a Map or Set bring this to O(n)?*
- When you see a sorted array, ask: *can binary search make this O(log n)?*
