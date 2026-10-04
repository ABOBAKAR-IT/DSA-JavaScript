# Strings (Simple Notes)

> Easy notes on strings for any language. Code uses JavaScript in C++ style (loops and indexes, no built-in string methods). The ideas work in every language.

## Table of Contents
1. [What is a String?](#1-what-is-a-string)
2. [Characters Are Numbers](#2-characters-are-numbers)
3. [Encoding (ASCII, Unicode, UTF-8)](#3-encoding-ascii-unicode-utf-8)
4. [Length Can Be Confusing](#4-length-can-be-confusing)
5. [How Strings Are Stored](#5-how-strings-are-stored)
6. [Immutable vs Mutable](#6-immutable-vs-mutable)
7. [Cost of Common Operations](#7-cost-of-common-operations)
8. [Joining Strings (Concatenation)](#8-joining-strings-concatenation)
9. [Substring: Copy vs View](#9-substring-copy-vs-view)
10. [Comparing Strings](#10-comparing-strings)
11. [Developer Rules (Important)](#11-developer-rules-important)
12. [Quick Questions](#12-quick-questions)

---

## 1. What is a String?

A **string** is text: a list of characters in order.

```
"Hello"

index:   0    1    2    3    4
       +----+----+----+----+----+
       | H  | e  | l  | l  | o  |
       +----+----+----+----+----+
```

A string is like an **array of characters**. You read a character by its index, starting from 0.

```js
const s = "Hello";
console.log(s[0]);        // H
console.log(s[4]);        // o
console.log(s.length);    // 5
```

Walk through a string with a loop:

```js
for (let i = 0; i < s.length; i++) {
  console.log(s[i]);
}
```

---

## 2. Characters Are Numbers

A computer only stores numbers, so every character has a number code.

| Character | Code |
|---|---|
| `'0'` to `'9'` | 48 to 57 |
| `'A'` to `'Z'` | 65 to 90 |
| `'a'` to `'z'` | 97 to 122 |
| space | 32 |

Things to remember:
- Letters are in order: `'b'` is `'a'` + 1.
- `'a'` is 32 more than `'A'`.
- `'5'` is **not** the number 5. Its code is 53. The digit value is `code - 48`.

Because characters are numbers, comparing and sorting characters is just comparing numbers.

---

## 3. Encoding (ASCII, Unicode, UTF-8)

- **ASCII:** the old table with 128 characters (English letters, digits, symbols).
- **Unicode:** a huge table with a number for every character in every language, plus emoji.
- **UTF-8:** a way to save Unicode as bytes. It is the standard on the web and in files.

UTF-8 uses 1 to 4 bytes per character:

| Character | Bytes in UTF-8 |
|---|---|
| `A` | 1 |
| `é` | 2 |
| `€` | 3 |
| 😀 | 4 |

**Remember:** one character is **not always one byte**. Always use UTF-8, and read and write text with the same encoding.

---

## 4. Length Can Be Confusing

"Length" can mean different things:

| Meaning | Example `"😀"` |
|---|---|
| Bytes (UTF-8) | 4 |
| UTF-16 units (JS, Java, C#) | 2 |
| Code points (Python) | 1 |
| What you see on screen | 1 |

```js
console.log("😀".length);   // 2 in JavaScript, not 1
```

For plain English text, all of these are the same. With emoji or accents, they differ. **Know what `length` counts in your language.**

---

## 5. How Strings Are Stored

Strings are stored in memory like arrays: characters side by side.

**C style:** ends with a special `'\0'` character.
```
"Hi" -> [ 'H' ][ 'i' ][ '\0' ]
```
To find the length you must walk until `'\0'`, which is O(n).

```js
function strLength(chars) {         // chars ends with "\0"
  let i = 0;
  while (chars[i] !== "\0") i++;
  return i;
}
```

**Modern style:** the string object stores the length with the characters, so length is O(1).
```
[ length = 2 ] [ 'H' ][ 'i' ]
```

Most modern languages (C++ `std::string`, Java, Python, JS) work this way.

---

## 6. Immutable vs Mutable

**Immutable** means you cannot change the string after creating it. Any "change" makes a new string.

```js
let s = "cat";
s[0] = "b";        // does nothing
console.log(s);    // cat

s = "b" + "at";    // new string "bat" is created
```

| Immutable | Mutable |
|---|---|
| JavaScript, Java, Python, C#, Go | C (char array), C++ `std::string`, Rust `String` |

Why immutable? It is **safe** (nobody changes it by surprise) and strings can be **shared** easily.

The cost: changing one character copies the whole string, which is O(n).

**Trick for many edits:** use a char array, edit it, then make a string again.

```js
const chars = ["c", "a", "t"];
chars[0] = "b";                  // changes in place, O(1)
// chars is now b, a, t
```

---

## 7. Cost of Common Operations

`n` is the length of the string.

| Operation | Time |
|---|---|
| Read a character by index | O(1) |
| Get length | O(1) (O(n) in C style) |
| Go through all characters | O(n) |
| Join two strings | O(n + m) |
| Substring (copy) | O(k) |
| Compare two strings | O(n) |
| Change one character (immutable) | O(n) |
| Insert / delete in the middle | O(n) |

---

## 8. Joining Strings (Concatenation)

With immutable strings, `a + b` makes a **new string and copies both**.

Doing this in a loop is slow:

```
loop 1: copy 1 character
loop 2: copy 2 characters
loop 3: copy 3 characters
...
total = 1 + 2 + 3 + ... + n  =  O(n²)
```

**Fix:** collect the characters in a growing buffer (a dynamic array) and build the string **once at the end**. This tool is called a **StringBuilder** (Java, C#), or a list + join (Python), or `std::string` (C++).

```js
class StringBuilder {
  constructor() {
    this.capacity = 4;
    this.data = new Array(this.capacity);
    this.size = 0;
  }

  append(ch) {
    if (this.size === this.capacity) this.grow();
    this.data[this.size] = ch;
    this.size++;
  }

  grow() {
    const newData = new Array(this.capacity * 2);
    for (let i = 0; i < this.size; i++) newData[i] = this.data[i];
    this.data = newData;
    this.capacity = this.capacity * 2;
  }

  toString() {
    let s = "";
    for (let i = 0; i < this.size; i++) s += this.data[i];
    return s;
  }
}
```

Now `n` appends cost O(n) in total.

> Some languages (like JS) make `+=` fast behind the scenes, but do not depend on it. Think with the general rule.

---

## 9. Substring: Copy vs View

A **substring** is a piece of a string, like `"ell"` from `"hello"`.

- **Copy:** makes a new string. Cost O(k). Safe and independent. (Java, Python, JS, C#)
- **View:** only stores where the piece starts and how long it is. Cost O(1), no copying. But it depends on the original string staying alive. (Go, Rust `&str`, C++ `string_view`)

Taking many substrings in a loop is expensive with copies, so know which one your language uses.

---

## 10. Comparing Strings

Strings are compared **character by character using their codes**.

1. Find the first position where they differ. The smaller code is the smaller string.
2. If one is a start of the other, the shorter one is smaller.

```
"apple" vs "apply"  -> 'e' < 'y'          -> "apple" is smaller
"app"   vs "apple"  -> "app" is shorter   -> "app" is smaller
"Zoo"   vs "apple"  -> 'Z'(90) < 'a'(97)  -> "Zoo" is smaller
"10"    vs "9"      -> '1' < '9'          -> "10" is smaller (as text!)
```

```js
function compareStrings(a, b) {
  const limit = a.length < b.length ? a.length : b.length;
  for (let i = 0; i < limit; i++) {
    if (a[i] < b[i]) return -1;
    if (a[i] > b[i]) return 1;
  }
  if (a.length < b.length) return -1;
  if (a.length > b.length) return 1;
  return 0;
}
```

Remember:
- Uppercase comes before lowercase.
- Number strings sort as text, not as numbers.
- Compare **contents**, not references. (In Java use `.equals()`, not `==`.)

---

## 11. Developer Rules (Important)

1. **A string is an array of character codes.** Characters are numbers.
2. **Know what `length` counts** (bytes, UTF-16 units, or code points). Emoji and accents show the difference.
3. **Use UTF-8** for files, network and databases. Use the same encoding for reading and writing.
4. **Most strings are immutable.** For heavy editing, use a char array.
5. **Do not use `+=` in a big loop.** Use a builder.
6. **Watch hidden O(n) work** inside loops: substring, compare, concatenate.
7. **Compare contents, not references.**
8. **Clean user input before comparing:** trim spaces, handle `\r\n` vs `\n`, and watch for invisible characters.
9. **`"123"` is not `123`.** Convert it, and handle bad input.
10. **Empty string `""` is not `null`.**
11. **Never build SQL or commands by joining user input.** Use parameterized queries.
12. **In C and C++:** a C string needs one extra byte for `'\0'`. Writing past the end is a buffer overflow.
13. **Do not use a changing string as a hash key.**

---

## 12. Quick Questions

1. What is the code of `'a'` if `'A'` is 65?
2. Why is `"10"` smaller than `"9"` as strings?
3. Why is `"😀".length` equal to 2 in JS?
4. Why can `+=` in a loop be slow?
5. What is the difference between a substring copy and a view?
6. What is the difference between `""` and `null`?

### Answers
1. 97 (the difference is 32).
2. It compares character by character, and `'1'` has a smaller code than `'9'`.
3. JS counts UTF-16 units, and this emoji takes 2 of them.
4. Each `+` copies everything so far, so the total is O(n²).
5. A copy makes new memory (O(k)). A view only stores a position and length (O(1)) and needs the original to stay alive.
6. `""` is a real string with length 0. `null` means there is no string at all.

---

