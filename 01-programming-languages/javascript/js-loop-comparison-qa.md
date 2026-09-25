# for vs forEach vs for...in vs for...of (Q&A notes)

## Contents
- [for](#for)
- [forEach](#foreach)
- [for...in](#forin)
- [for...of](#forof)
- [Summary table](#summary-table)
- [When to use which](#when-to-use-which)

---

## for

```javascript
const array = [10, 20, 30];
for (let i = 0; i < array.length; i++) {
  console.log(array[i]);
}
```
Full manual control over the index/condition/increment — supports `break`/`continue`. Verbose, but the only option when you need fine-grained control over iteration.

## forEach

```javascript
const array = [10, 20, 30];
array.forEach(value => console.log(value));
```
Array method, simplest syntax for "just do something with every value." **No `break`/`continue` support** — you can't exit early.

## for...in

Iterates over an object's **keys** (or an array's indices as strings).
```javascript
const obj = { a: 1, b: 2, c: 3 };
for (let key in obj) console.log(key, obj[key]);
// a 1 / b 2 / c 3

const array = [10, 20, 30];
for (let index in array) console.log(index, array[index]);
// 0 10 / 1 20 / 2 30
```
Best for plain objects. Using it on arrays is discouraged — it also iterates over any enumerable custom properties added to the array, not just numeric indices.

## for...of

Iterates over the **values** of iterables (arrays, strings, Maps, Sets, ...).
```javascript
const array = [10, 20, 30];
for (let value of array) console.log(value);
// 10 20 30

for (let char of "abc") console.log(char);
// a b c
```
Not usable on plain (non-iterable) objects.

## Summary table

| Loop | Best for | Gives | break/continue | Typical use |
|---|---|---|---|---|
| `for` | Arrays, counting | Index & value | Yes | Full manual control |
| `forEach` | Arrays | Value | No | Simple pass over array values |
| `for...in` | Objects (and arrays, less common) | Key (index) | Yes | Iterating object properties |
| `for...of` | Iterables | Value | Yes | Working directly with array/string values |

## When to use which

- **`for`**: need fine control — skipping with `continue`, exiting with `break`.
- **`forEach`**: simplest array loop where you only need values and won't break early.
- **`for...in`**: iterating an object's own/inherited enumerable keys.
- **`for...of`**: the default choice for arrays/strings/iterables when you just want each value.

# 색인과 출처
- 개인 Q&A 학습 노트 — JavaScript 반복문(for/forEach/for-in/for-of) 비교
