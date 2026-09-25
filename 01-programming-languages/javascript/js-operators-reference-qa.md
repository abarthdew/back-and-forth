# JavaScript Operators Reference (Q&A notes)

## Contents
- [Equality: === vs ==](#equality---vs-)
- [Ternary operator](#ternary-operator)
- [Nullish coalescing (??)](#nullish-coalescing-)
- [Logical OR / AND / NOT](#logical-or--and--not)
- [Nullish assignment (??=)](#nullish-assignment-)
- [Optional chaining (?.)](#optional-chaining-)
- [Arithmetic, comparison, assignment, bitwise](#arithmetic-comparison-assignment-bitwise)
- [typeof / delete](#typeof--delete)
- [Summary table](#summary-table)

---

## Equality: === vs ==

- `===` (strict equality): compares value **and** type, no coercion. `5 === '5'` → `false`.
- `==` (loose equality): coerces types before comparing. `5 == '5'` → `true`; `null == undefined` → `true`.
- Default to `===`; reserve `==` for intentional `null`/`undefined` checks in legacy code.

## Ternary operator

```javascript
const canVote = (age >= 18) ? 'Yes' : 'No';
```
Shorthand for `if...else` — good for short conditional expressions, not for complex branching.

## Nullish coalescing (??)

```javascript
const userName = name ?? "Guest";   // only falls back on null/undefined
```
Unlike `||`, `??` does **not** treat `0`, `""`, or `false` as "missing" — only `null`/`undefined` trigger the fallback.

## Logical OR / AND / NOT

```javascript
const displayName = name || "Guest";     // || treats ALL falsy values as missing
const username = user && user.name;      // && short-circuits on the first falsy value
console.log(!isLoggedIn);                 // negate a boolean
console.log(!!value);                     // double-NOT: coerce any value to boolean
```

## Nullish assignment (??=)

```javascript
let userColor = null;
userColor ??= "blue";   // assigns only if null/undefined (vs ||= which assigns on any falsy value)
```

## Optional chaining (?.)

```javascript
const user = { profile: { name: "Alice" } };
user.profile?.name;     // "Alice"
user.profile?.age;      // undefined
user.address?.street;   // undefined, no error
```
Avoids manual chained existence checks when accessing deeply nested properties that may not exist.

## Arithmetic, comparison, assignment, bitwise

```javascript
2 ** 3;                 // 8 — exponentiation, concise alternative to Math.pow(2, 3)
10 + 5; 10 - 5; 10 * 5; 10 / 5; 10 % 3;   // basic arithmetic
5 > 3; 5 < 3; 5 >= 5; 5 <= 3;              // comparison

let x = 5;
x += 3;  // 8
x *= 2;  // 16

5 & 1;  // bitwise AND
5 | 1;  // bitwise OR

let y = (1, 2, 3);  // comma operator — evaluates all, keeps the last: y === 3
```

## typeof / delete

```javascript
typeof "hello";   // "string"
typeof 42;        // "number"
typeof true;      // "boolean"

const obj = { a: 1, b: 2 };
delete obj.a;      // { b: 2 }
```

## Summary table

| Operator | Description | Use case |
|---|---|---|
| `===`, `==` | Strict vs loose equality | Comparisons |
| `\|\|`, `&&`, `!` | Logical OR / AND / NOT | Defaults, short-circuiting, negation |
| `??` | Nullish coalescing | Defaults that must not trigger on `0`/`""`/`false` |
| `?.` | Optional chaining | Safe access to deeply nested properties |
| `? :` | Ternary | Short `if...else` expressions |
| `+ - * / % **` | Arithmetic & exponentiation | Basic math |
| `> < >= <=` | Comparison | Loops, conditions |
| `typeof`, `delete` | Type checking, property deletion | Dynamic data handling |
| `+= -= *=` etc. | Compound assignment | In-place updates |

# 색인과 출처
- 개인 Q&A 학습 노트 — JavaScript 비교/논리/할당/비트 연산자 정리
