# Recursion (JavaScript)

> A review-friendly guide to recursion: how it works, how to think about it, and the patterns you'll see again and again. All examples are in JavaScript.

## Table of Contents
1. [What is Recursion?](#1-what-is-recursion)
2. [The Two Required Parts](#2-the-two-required-parts)
3. [The Call Stack (How It Really Works)](#3-the-call-stack-how-it-really-works)
4. [How to Think Recursively](#4-how-to-think-recursively)
5. [Classic Examples](#5-classic-examples)
6. [Types of Recursion](#6-types-of-recursion)
7. [Recursion on Arrays and Strings](#7-recursion-on-arrays-and-strings)
8. [Divide and Conquer](#8-divide-and-conquer)
9. [Recursion on Trees](#9-recursion-on-trees)
10. [Backtracking](#10-backtracking)
11. [Memoization (Fixing Slow Recursion)](#11-memoization-fixing-slow-recursion)
12. [Time & Space Complexity of Recursion](#12-time--space-complexity-of-recursion)
13. [Stack Overflow and Tail Calls](#13-stack-overflow-and-tail-calls)
14. [Recursion vs Iteration](#14-recursion-vs-iteration)
15. [Common Mistakes](#15-common-mistakes)
16. [Cheat Sheet](#16-cheat-sheet)
17. [Practice Problems](#17-practice-problems)

---

## 1. What is Recursion?

Recursion is a technique where a function **calls itself** to solve a smaller version of the same problem, until it reaches a case simple enough to answer directly.

**Real-life analogies**
- **Nesting dolls:** open a doll; if another doll is inside, repeat. If it's empty, stop.
- **Looking up a word in a dictionary where the definition uses another word:** keep looking up until you reach a word you already know.
- **Standing in a line and asking "how many people are in front of me?":** each person asks the one in front. The first person says "0". Each person then adds 1 to the answer they get and passes it back.

```js
function countDown(n) {
  if (n === 0) {              // base case: stop here
    console.log("Done!");
    return;
  }
  console.log(n);
  countDown(n - 1);           // recursive case: smaller problem
}

countDown(3);
// 3
// 2
// 1
// Done!
```

---

## 2. The Two Required Parts

Every correct recursive function needs both:

| Part | Purpose | If missing |
|---|---|---|
| **Base case** | Stops the recursion; returns an answer directly | Infinite recursion -> stack overflow |
| **Recursive case** | Calls itself with a **smaller / closer-to-base** input | Never reaches the base case |

```js
function factorial(n) {
  if (n <= 1) return 1;          // 1. base case
  return n * factorial(n - 1);   // 2. recursive case (n shrinks toward 1)
}
```

**Checklist for any recursive function**
1. What is the **smallest** input I can answer directly? (base case)
2. How do I make the input **smaller** each call? (progress)
3. Given the answer to the smaller problem, how do I **build** the answer for this one? (combine)

---

## 3. The Call Stack (How It Really Works)

Every function call gets a **stack frame** holding its parameters and local variables. Frames are stacked. A call can only finish when the call it made finishes.

```js
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
factorial(4);
```

**Going down (calls pile up):**
```
factorial(4)  -> waits for factorial(3)
  factorial(3)  -> waits for factorial(2)
    factorial(2)  -> waits for factorial(1)
      factorial(1)  -> returns 1   (base case!)
```

**Coming back up (calls resolve):**
```
      factorial(1) returns 1
    factorial(2) returns 2 * 1 = 2
  factorial(3) returns 3 * 2 = 6
factorial(4) returns 4 * 6 = 24
```

Key points:
- The stack grows by one frame per call. Depth of recursion = **space used**.
- Work can happen **before** the recursive call (on the way down) or **after** it (on the way back up). The order matters:

```js
function demo(n) {
  if (n === 0) return;
  console.log("down", n);   // runs on the way down
  demo(n - 1);
  console.log("up", n);     // runs on the way back up
}
demo(3);
// down 3
// down 2
// down 1
// up 1
// up 2
// up 3
```

---

## 4. How to Think Recursively

### The "Leap of Faith"
Don't try to follow every call in your head. Instead:

1. **Write down the function's contract:** "`sum(arr)` returns the sum of all numbers in `arr`."
2. **Trust** that the contract holds for smaller inputs.
3. Ask: *"If `sum(rest)` already works, how do I get `sum(arr)`?"* -> `arr[0] + sum(rest)`.
4. Add the base case: the sum of an empty array is `0`.

### Handy question: "What does one step look like?"
Reduce the problem by one piece (or half), handle that piece yourself, and delegate the rest.

```
sum([1, 2, 3, 4])  =  1 + sum([2, 3, 4])
                   =  1 + (2 + sum([3, 4]))
                   =  1 + (2 + (3 + sum([4])))
                   =  1 + (2 + (3 + (4 + sum([]))))
                   =  1 + 2 + 3 + 4 + 0 = 10
```

---

## 5. Classic Examples

### Sum of 1 to n
```js
function sumTo(n) {
  if (n === 0) return 0;
  return n + sumTo(n - 1);
}
// Time O(n), Space O(n)
```

### Power
```js
function power(base, exp) {
  if (exp === 0) return 1;
  return base * power(base, exp - 1);
}
// Time O(n), Space O(n)

// Faster: halve the exponent each call
function fastPower(base, exp) {
  if (exp === 0) return 1;
  const half = fastPower(base, Math.floor(exp / 2));
  return exp % 2 === 0 ? half * half : half * half * base;
}
// Time O(log n), Space O(log n)
```

### Fibonacci
```js
function fib(n) {
  if (n <= 1) return n;                // base cases: fib(0)=0, fib(1)=1
  return fib(n - 1) + fib(n - 2);     // two recursive calls
}
// Time O(2ⁿ) (slow!), Space O(n). See memoization below.
```

### Greatest Common Divisor (Euclid)
```js
function gcd(a, b) {
  if (b === 0) return a;
  return gcd(b, a % b);
}
// Time O(log(min(a, b)))
```

### Sum of digits
```js
function sumDigits(n) {
  if (n < 10) return n;
  return (n % 10) + sumDigits(Math.floor(n / 10));
}
// sumDigits(1234) -> 4 + 3 + 2 + 1 = 10
```

### Tower of Hanoi
Move `n` disks from one peg to another, using a third, never putting a larger disk on a smaller one.
```js
function hanoi(n, from, to, via, moves = []) {
  if (n === 0) return moves;
  hanoi(n - 1, from, via, to, moves);   // move n-1 disks out of the way
  moves.push(`${from} -> ${to}`);       // move the biggest disk
  hanoi(n - 1, via, to, from, moves);   // move n-1 disks on top of it
  return moves;
}
console.log(hanoi(3, "A", "C", "B").length); // 7  (always 2ⁿ - 1 moves)
```

---

## 6. Types of Recursion

### Linear recursion
One recursive call per invocation (`factorial`, `sumTo`). Forms a single chain.

### Binary (tree) recursion
Two recursive calls per invocation (`fib`, tree traversal, merge sort). Forms a tree of calls.

### Multiple recursion
Several calls per invocation (permutations, N-Queens).

### Tail recursion
The recursive call is the **last** thing the function does (nothing left to do after it returns). Usually uses an accumulator:
```js
// Not tail recursive: still has "n *" to do after the call returns
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Tail recursive: result is carried along in `acc`
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, acc * n);
}
```
> Note: in theory tail calls can reuse the same stack frame, but **most JS engines (V8 / Node / Chrome) do not optimize tail calls**. So in practice, tail recursion in JS still uses O(n) stack.

### Mutual recursion
Two functions call each other.
```js
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1);
}
function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1);
}
```

### Nested recursion
The argument itself is a recursive call (rare, e.g. the Ackermann function).

---

## 7. Recursion on Arrays and Strings

### Reverse a string
```js
function reverse(str) {
  if (str === "") return "";
  return reverse(str.slice(1)) + str[0];
}
// reverse("abc") -> reverse("bc") + "a" -> (reverse("c") + "b") + "a" -> "cba"
// Time O(n²) because of slice, Space O(n²) in the worst case (each slice is a new string)
```

### Palindrome check (index-based, avoids slicing)
```js
function isPalindrome(s, left = 0, right = s.length - 1) {
  if (left >= right) return true;
  if (s[left] !== s[right]) return false;
  return isPalindrome(s, left + 1, right - 1);
}
// Time O(n), Space O(n) (stack depth n/2)
```

### Sum of an array
```js
// Version 1: slice (clean but copies, O(n²))
function sumArr(arr) {
  if (arr.length === 0) return 0;
  return arr[0] + sumArr(arr.slice(1));
}

// Version 2: pass an index (O(n) time)
function sumArrIdx(arr, i = 0) {
  if (i === arr.length) return 0;
  return arr[i] + sumArrIdx(arr, i + 1);
}
```
> **Tip:** pass an **index** instead of slicing to avoid hidden O(n) copies in every call.

### Flatten a nested array
```js
function flatten(arr) {
  const result = [];
  for (const item of arr) {
    if (Array.isArray(item)) result.push(...flatten(item)); // recurse on nested
    else result.push(item);
  }
  return result;
}
console.log(flatten([1, [2, [3, [4]], 5]])); // [1, 2, 3, 4, 5]
```

### The "helper function" pattern
When you need to collect results, use an inner recursive helper so the outer function stays clean:
```js
function collectOdds(arr) {
  const result = [];

  function helper(i) {
    if (i === arr.length) return;
    if (arr[i] % 2 === 1) result.push(arr[i]);
    helper(i + 1);
  }

  helper(0);
  return result;
}
```

---

## 8. Divide and Conquer

Split the problem into smaller sub-problems, solve each recursively, then **combine**.

### Binary search (recursive)
```js
function binarySearch(sorted, target, lo = 0, hi = sorted.length - 1) {
  if (lo > hi) return -1;                          // base case: not found
  const mid = Math.floor((lo + hi) / 2);
  if (sorted[mid] === target) return mid;
  if (sorted[mid] < target) return binarySearch(sorted, target, mid + 1, hi);
  return binarySearch(sorted, target, lo, mid - 1);
}
// Time O(log n), Space O(log n)
```

### Merge sort
```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;                 // base case

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));       // divide
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);                       // combine
}

function merge(a, b) {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    out.push(a[i] <= b[j] ? a[i++] : b[j++]);
  }
  return out.concat(a.slice(i), b.slice(j));
}
// Time O(n log n), Space O(n)
```

### Recurrence relations (reading complexity)
| Recurrence | Meaning | Result |
|---|---|---|
| `T(n) = T(n-1) + O(1)` | one call, shrink by 1, constant work | O(n) |
| `T(n) = T(n/2) + O(1)` | one call, halve (binary search) | O(log n) |
| `T(n) = 2T(n/2) + O(n)` | two halves + linear merge (merge sort) | O(n log n) |
| `T(n) = 2T(n-1) + O(1)` | two calls, shrink by 1 (naive fib) | O(2ⁿ) |

---

## 9. Recursion on Trees

Trees are naturally recursive: **a tree is a node plus smaller trees (its children).** So tree problems are the perfect place to practice recursion.

```js
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

//        1
//       / \
//      2   3
//     / \
//    4   5
const root = new TreeNode(1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3)
);
```

### Max depth
```js
function maxDepth(node) {
  if (node === null) return 0;                     // base case: empty tree
  return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}
console.log(maxDepth(root)); // 3
```

### Sum of all nodes
```js
function treeSum(node) {
  if (node === null) return 0;
  return node.val + treeSum(node.left) + treeSum(node.right);
}
```

### Traversals (DFS)
```js
function preorder(node, out = []) {   // root, left, right
  if (!node) return out;
  out.push(node.val);
  preorder(node.left, out);
  preorder(node.right, out);
  return out;
}

function inorder(node, out = []) {    // left, root, right
  if (!node) return out;
  inorder(node.left, out);
  out.push(node.val);
  inorder(node.right, out);
  return out;
}

function postorder(node, out = []) {  // left, right, root
  if (!node) return out;
  postorder(node.left, out);
  postorder(node.right, out);
  out.push(node.val);
  return out;
}

console.log(preorder(root));   // [1, 2, 4, 5, 3]
console.log(inorder(root));    // [4, 2, 5, 1, 3]
console.log(postorder(root));  // [4, 5, 2, 3, 1]
```
**Complexity:** Time O(n) (every node visited once). Space O(h), where `h` is the tree height (O(log n) if balanced, O(n) if it's a straight line).

---

## 10. Backtracking

Backtracking = recursion that **tries a choice, explores, then undoes the choice** (so it can try the next one). It's used for subsets, permutations, combinations, puzzles (Sudoku, N-Queens), and path finding.

**Template**
```
function backtrack(state):
    if state is a complete solution: save it; return
    for each choice available:
        make the choice          (choose)
        backtrack(new state)     (explore)
        undo the choice          (un-choose)
```

### Subsets (power set)
```js
function subsets(nums) {
  const result = [];

  function backtrack(start, path) {
    result.push([...path]);                    // every path is a valid subset
    for (let i = start; i < nums.length; i++) {
      path.push(nums[i]);                      // choose
      backtrack(i + 1, path);                  // explore
      path.pop();                              // un-choose
    }
  }

  backtrack(0, []);
  return result;
}
console.log(subsets([1, 2, 3]));
// [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
// Time O(n · 2ⁿ), Space O(n) extra (excluding output)
```

### Permutations
```js
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);

  function backtrack(path) {
    if (path.length === nums.length) {
      result.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      used[i] = true;                          // choose
      path.push(nums[i]);
      backtrack(path);                         // explore
      path.pop();                              // un-choose
      used[i] = false;
    }
  }

  backtrack([]);
  return result;
}
console.log(permute([1, 2, 3]).length); // 6
// Time O(n · n!), Space O(n) extra
```

### Generate valid parentheses
```js
function generateParens(n) {
  const result = [];

  function go(open, close, cur) {
    if (cur.length === 2 * n) {
      result.push(cur);
      return;
    }
    if (open < n) go(open + 1, close, cur + "(");
    if (close < open) go(open, close + 1, cur + ")");
  }

  go(0, 0, "");
  return result;
}
console.log(generateParens(3));
// ["((()))", "(()())", "(())()", "()(())", "()()()"]
```

---

## 11. Memoization (Fixing Slow Recursion)

Naive `fib(5)` recomputes the same values many times:
```
                    fib(5)
                 /          \
            fib(4)          fib(3)
           /      \         /     \
       fib(3)   fib(2)   fib(2)  fib(1)
       ...
```
`fib(3)` and `fib(2)` are computed repeatedly. These are **overlapping subproblems**.

**Memoization** = cache results the first time and reuse them.
```js
function fibMemo(n, memo = new Map()) {
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n);             // reuse cached result
  const value = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, value);
  return value;
}
// Time O(n), Space O(n)
```

### A reusable memoize wrapper
```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n) => n * n;
const fastSquare = memoize(slowSquare);
```

### Example: climbing stairs
You can climb 1 or 2 steps at a time. How many ways to reach step `n`?
```js
function climbStairs(n, memo = {}) {
  if (n <= 2) return n;
  if (memo[n]) return memo[n];
  memo[n] = climbStairs(n - 1, memo) + climbStairs(n - 2, memo);
  return memo[n];
}
// Time O(n), Space O(n)
```

> Memoization is the bridge from recursion to **dynamic programming**.

---

## 12. Time & Space Complexity of Recursion

Two questions:
1. **How many calls** are made in total? (-> time)
2. **How deep** does the stack get at its deepest? (-> space)

| Function | Calls | Time | Max depth (Space) |
|---|---|---|---|
| `factorial(n)` | n | O(n) | O(n) |
| `fastPower(b, n)` | log n | O(log n) | O(log n) |
| `fib(n)` naive | ~2ⁿ | O(2ⁿ) | O(n) |
| `fibMemo(n)` | n | O(n) | O(n) |
| `binarySearch` | log n | O(log n) | O(log n) |
| `mergeSort` | 2n | O(n log n) | O(log n) stack + O(n) arrays |
| `subsets` | 2ⁿ | O(n · 2ⁿ) | O(n) |
| `permute` | n! | O(n · n!) | O(n) |
| tree traversal | n | O(n) | O(h) |

**Important:** space is the **deepest** chain of active calls, not the total number of calls. In `fib(n)`, there are ~2ⁿ calls but only about `n` frames exist at any moment.

Also remember to count work done *per call* (e.g., `slice` costs O(n) each time).

---

## 13. Stack Overflow and Tail Calls

Each call uses stack memory, which is limited. If recursion goes too deep:
```js
function infinite(n) {
  return infinite(n + 1);        // no base case!
}
infinite(0);
// RangeError: Maximum call stack size exceeded
```

It can also happen with a valid function on huge input:
```js
function sumTo(n) {
  if (n === 0) return 0;
  return n + sumTo(n - 1);
}
sumTo(1_000_000); // likely RangeError in Node/Chrome (limit is roughly 10,000 to 15,000 frames; varies)
```

**Ways to avoid it**
- Make sure the base case is reachable.
- Convert to a loop (iteration).
- Use an explicit stack (array) to simulate recursion.
- Reduce depth (e.g., divide and conquer halves the problem each time).
- Use memoization to avoid redundant deep chains.

```js
// Iterative version, no stack limit problem
function sumToIter(n) {
  let total = 0;
  for (let i = 1; i <= n; i++) total += i;
  return total;
}
```

---

## 14. Recursion vs Iteration

| | Recursion | Iteration |
|---|---|---|
| Readability | Often cleaner for trees, graphs, backtracking | Cleaner for simple repetition |
| Space | Uses call stack (O(depth)) | Usually O(1) |
| Risk | Stack overflow | Off-by-one errors |
| Speed | Function call overhead | Typically a bit faster |
| Best for | Hierarchical / branching problems | Linear loops |

Any recursion can be converted to iteration (often with an explicit stack):

```js
// Recursive DFS preorder
function preorderRec(node, out = []) {
  if (!node) return out;
  out.push(node.val);
  preorderRec(node.left, out);
  preorderRec(node.right, out);
  return out;
}

// Iterative equivalent with explicit stack
function preorderIter(root) {
  const out = [];
  const stack = root ? [root] : [];
  while (stack.length) {
    const node = stack.pop();
    out.push(node.val);
    if (node.right) stack.push(node.right);  // push right first so left is processed first
    if (node.left) stack.push(node.left);
  }
  return out;
}
```

---

## 15. Common Mistakes

1. **Missing base case** (or a base case that is never reached) -> stack overflow.
2. **Not making progress** toward the base case: `f(n)` calling `f(n)`, or `f(n + 1)` when the base is `n === 0`.
3. **Forgetting to `return` the recursive call.**
   ```js
   function sum(n) {
     if (n === 0) return 0;
     n + sum(n - 1);          // BUG: result is thrown away -> returns undefined
   }
   ```
4. **Mutating shared state without undoing it** in backtracking (forgetting `path.pop()`).
5. **Pushing the same array reference** instead of a copy: `result.push(path)` instead of `result.push([...path])`.
6. **Hidden O(n) work per call** (`slice`, spread, string concatenation) making the algorithm slower than expected.
7. **Recomputing overlapping subproblems** (naive Fibonacci) instead of memoizing.
8. **Tracing every call mentally.** Trust the contract instead.
9. **Using default mutable parameters carelessly.** `function f(arr = [])` is fine in JS (a fresh array per call), but shared accumulators passed across calls will persist, so be deliberate.

---

## 16. Cheat Sheet

### Problem-solving template
```js
function solve(input) {
  // 1. Base case(s): smallest input -> direct answer
  if (isSmallest(input)) return directAnswer;

  // 2. Shrink: break into smaller piece(s)
  const smaller = reduce(input);

  // 3. Recurse: trust that solve(smaller) works
  const subResult = solve(smaller);

  // 4. Combine: build this answer from the sub-answer
  return combine(input, subResult);
}
```

### Pattern recognition
| If you see... | Think... |
|---|---|
| "Sum / count / product of a sequence" | Linear recursion, base case on empty / zero |
| "Halve the input" | Binary search / fast power, O(log n) |
| "Sort", "merge", "split in half" | Divide and conquer |
| "Tree" or "nested structure" | Recurse on children, base case `null` |
| "All subsets / permutations / combinations" | Backtracking (choose, explore, un-choose) |
| "Count ways", "min / max cost" with repeated subproblems | Recursion + memoization (DP) |
| "Try all possibilities with constraints" | Backtracking with pruning |

### Complexity quick reference
| Recurrence | Complexity |
|---|---|
| `T(n) = T(n-1) + O(1)` | O(n) |
| `T(n) = T(n-1) + O(n)` | O(n²) |
| `T(n) = T(n/2) + O(1)` | O(log n) |
| `T(n) = 2T(n/2) + O(n)` | O(n log n) |
| `T(n) = 2T(n-1) + O(1)` | O(2ⁿ) |
| `T(n) = n · T(n-1)` | O(n!) |

---

## 17. Practice Problems

Try each one yourself before looking at the solutions.

1. **`countdown(n)`**: print `n` down to `1`.
2. **`sumArray(arr)`**: return the sum of an array recursively.
3. **`reverseString(s)`**: reverse a string recursively.
4. **`isPalindrome(s)`**: check if a string reads the same both ways.
5. **`fibonacci(n)`**: with and without memoization. Compare speed for `n = 40`.
6. **`power(x, n)`**: in O(log n).
7. **`subsets(nums)`**: return all subsets.
8. **`maxDepth(root)`**: height of a binary tree.

### Solutions

**1. countdown**
```js
function countdown(n) {
  if (n <= 0) return;
  console.log(n);
  countdown(n - 1);
}
```

**2. sumArray**
```js
function sumArray(arr, i = 0) {
  if (i === arr.length) return 0;
  return arr[i] + sumArray(arr, i + 1);
}
// Time O(n), Space O(n)
```

**3. reverseString**
```js
function reverseString(s) {
  if (s.length <= 1) return s;
  return reverseString(s.slice(1)) + s[0];
}
// Time O(n²) due to slice, Space O(n) stack
```

**4. isPalindrome**
```js
function isPalindrome(s, l = 0, r = s.length - 1) {
  if (l >= r) return true;
  if (s[l] !== s[r]) return false;
  return isPalindrome(s, l + 1, r - 1);
}
// Time O(n), Space O(n)
```

**5. fibonacci**
```js
function fib(n) {                       // O(2ⁿ): fib(40) is noticeably slow
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

function fibMemo(n, memo = new Map()) { // O(n): instant
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n);
  const v = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, v);
  return v;
}
```

**6. power in O(log n)**
```js
function power(x, n) {
  if (n === 0) return 1;
  if (n < 0) return 1 / power(x, -n);
  const half = power(x, Math.floor(n / 2));
  return n % 2 === 0 ? half * half : half * half * x;
}
```

**7. subsets**: see [Backtracking](#10-backtracking).

**8. maxDepth**: see [Recursion on Trees](#9-recursion-on-trees).

### Next-level problems
- Climbing stairs (memoization)
- Generate parentheses
- Permutations / combinations
- N-Queens
- Sudoku solver
- Word search in a grid
- Flood fill
- Merge two sorted linked lists (recursive)
- Validate a binary search tree
- Lowest common ancestor in a binary tree

---

## Study Tips
- **Draw the call tree** on paper for a small input (like `n = 4`). Do this a few times, then stop tracing and trust the contract.
- **Always write the base case first.**
- Ask: *"What's the smallest version of this problem?"* and *"If I had the answer for a smaller one, what's the one step to finish?"*
- If it's slow, look for **repeated subproblems** and add memoization.
- If it's deep, consider an **iterative** solution.
- Tie it back to Big-O: count calls for time, depth for space.
