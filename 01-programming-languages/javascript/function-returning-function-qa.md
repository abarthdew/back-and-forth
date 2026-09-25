# Functions That Return Functions — Invocation Patterns (Q&A notes)

## Contents
- [The example](#the-example)
- [Returning a value vs. returning a function](#returning-a-value-vs-returning-a-function)
- [Calling both outer and inner function](#calling-both-outer-and-inner-function)
- [Calling just the inner function](#calling-just-the-inner-function)
- [Key takeaway](#key-takeaway)

---

## The example

```javascript
var self = function() {
  console.log('a');
  return function () {
    console.log('b');
  }
}
self()();          // a, b
self = self();     // a  (self now points to the inner function)
self();            // b
```

Full output across all three statements: `a b a b`.

## Returning a value vs. returning a function

When a function **returns a value** directly, you use the result immediately — no extra parentheses needed:
```javascript
var add = function(x, y) { return x + y; };
console.log(add(2, 3));  // 5
```

When a function **returns another function**, you must explicitly invoke the returned function with `()` if you want it to execute:
```javascript
var self = function() {
  console.log('a');
  return function () { console.log('b'); };
};
```

## Calling both outer and inner function

```javascript
self()();
```
1. `self()` executes the outer function → logs `'a'` → returns the inner function.
2. The trailing `()` immediately invokes that returned inner function → logs `'b'`.

Output: `a` then `b`.

## Calling just the inner function

```javascript
var func = self();  // calls outer, logs 'a', stores the returned inner function
func();              // invokes it later, logs 'b'
```

Same output (`a`, then `b`), but split across two statements — useful when you want to defer calling the inner function.

## Key takeaway

- `self()();` calls the outer and the returned inner function in one statement.
- `var func = self(); func();` calls the outer function, stores the returned inner function, and invokes it later.
- Whenever a function returns another function, you need an explicit `()` to execute the returned one. If it returns a plain value, you just use the value directly.

# 색인과 출처
- 개인 Q&A 학습 노트 — 함수를 반환하는 함수의 호출 표현식 분석
