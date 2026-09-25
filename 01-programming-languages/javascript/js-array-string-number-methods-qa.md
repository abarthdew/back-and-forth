# Non-DOM JavaScript Methods Reference (Q&A notes)

Array, String, Number, and Math methods that don't touch the DOM — the methods you reach for in everyday data processing.

## Contents
- [Array methods](#array-methods)
- [String methods](#string-methods)
- [Number methods](#number-methods)
- [Math methods](#math-methods)
- [Object methods](#object-methods)

---

## Array methods

```javascript
[1, 2, 3].map(num => num * 2);                        // [2, 4, 6]
["red", "green", "blue"].forEach(c => console.log(c)); // red green blue (no return value)
[1, 2, 3, 4].filter(num => num % 2 === 0);             // [2, 4]
[1, 2, 3, 4].reduce((acc, curr) => acc + curr, 0);     // 10
[1, 2, 3].some(num => num % 2 === 0);                  // true
[2, 4, 6].every(num => num % 2 === 0);                 // true
[1, 3, 4, 5].find(num => num % 2 === 0);               // 4
[1, 3, 4, 5].findIndex(num => num % 2 === 0);          // 2
[1, [2, [3, 4]], 5].flat(2);                            // [1, 2, 3, 4, 5]
[1, 2, 3].flatMap(num => [num, num * 2]);               // [1, 2, 2, 4, 3, 6]
```

| Method | Description |
|---|---|
| `map` | New array from calling a function on every element |
| `forEach` | Runs a function per element, returns nothing |
| `filter` | New array of elements passing a test |
| `reduce` | Single accumulated value from a reducer function |
| `some` | `true` if at least one element passes the test |
| `every` | `true` if all elements pass the test |
| `find` | First element passing the test |
| `findIndex` | Index of the first element passing the test |
| `flat(depth)` | Flattens nested sub-arrays up to `depth` |
| `flatMap` | `map` then flattens one level |

## String methods

```javascript
"Hello, world!".includes("world");    // true
"Hello, world!".startsWith("Hello");  // true
"Hello, world!".endsWith("!");        // true
"hi".repeat(3);                       // "hihihi"
"Hello, world!".split(" ");           // ["Hello,", "world!"]
"  Hello, world!  ".trim();           // "Hello, world!"
"hello".toUpperCase();                // "HELLO"
"HELLO".toLowerCase();                // "hello"
```

## Number methods

```javascript
Number.isInteger(4);        // true
Number.isInteger(4.5);      // false
Number.parseFloat("3.14");  // 3.14
Number.parseInt("42px");    // 42
```

## Math methods

```javascript
Math.max(1, 3, 2);   // 3
Math.min(1, 3, 2);   // 1
Math.floor(4.7);     // 4
Math.ceil(4.3);      // 5
Math.round(4.5);     // 5
Math.random();       // random number in [0, 1)
Math.sqrt(9);        // 3
```

## Object methods

```javascript
const obj = { a: 1, b: 2, c: 3 };
Object.keys(obj);     // ["a", "b", "c"]
Object.values(obj);   // [1, 2, 3]
```

# 색인과 출처
- 개인 Q&A 학습 노트 — 배열/문자열/숫자/Math/Object 내장 메서드 레퍼런스
