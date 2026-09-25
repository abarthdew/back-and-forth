# Closures vs. Callback Functions (Q&A notes)

## Contents
- [What is a Closure?](#what-is-a-closure)
- [What is a Callback Function?](#what-is-a-callback-function)
- [Comparison](#comparison)
- [How they combine](#how-they-combine)
- [Worked example: async callback](#worked-example-async-callback)

---

## What is a Closure?

A closure is a function that "remembers" variables from its outer scope even after that scope has exited.

```javascript
function outerFunction() {
  let outerVariable = "I am from the outer scope";
  function innerFunction() {
    console.log(outerVariable);
  }
  return innerFunction;
}
const closureFunction = outerFunction();
closureFunction(); // "I am from the outer scope"
```

## What is a Callback Function?

A callback function is a function passed as an argument to another function and executed later, often after an asynchronous operation completes.

```javascript
function fetchData(callback) {
  setTimeout(() => {
    console.log("Data fetched");
    callback("Data");
  }, 1000);
}
fetchData((data) => console.log("Callback executed with:", data));
```

## Comparison

| Aspect | Closure | Callback Function |
|---|---|---|
| Definition | Remembers variables from its outer scope | Passed as an argument to another function |
| Purpose | Preserve state / access variables after the outer function has executed | Define a task to run later, often asynchronously |
| Timing | Any time a nested function needs outer variables | Typically after an event or async operation completes |
| Example use case | Private counters, private variables | Handling data after an API call |

## How they combine

A callback can itself be (or rely on) a closure:

```javascript
function createCounter() {
  let count = 0;
  return function incrementAndCallback(callback) {
    count++;
    callback(count);
  };
}
const counter = createCounter();
counter((count) => console.log("Current count:", count)); // 1
counter((count) => console.log("Current count:", count)); // 2
```

- **Closure**: `incrementAndCallback` keeps access to `count` even though `createCounter` already returned.
- **Callback**: the arrow function passed to `counter(...)` is the callback that receives the result.

## Worked example: async callback

```javascript
function findUserAndCallBack(id, cb) {
  const user = { id, name: "User" + id, email: id + "@test.com" };
  cb(user);
}
findUserAndCallBack(1, (user) => console.log("user:", user));
```

Why this matters with `setTimeout`:

```javascript
function findUser(id) {
  let user;
  setTimeout(() => {
    user = { id, name: "User" + id, email: id + "@test.com" };
  }, 100);
  return user;         // returns before setTimeout runs
}
console.log(findUser(1)); // undefined — the async work hasn't completed yet
```

```javascript
function findUserAndCallBack(id, cb) {
  setTimeout(() => {
    const user = { id, name: "User" + id, email: id + "@test.com" };
    cb(user);           // fires only once the async work is done
  }, 100);
}
```

Regular functions return immediately even if the result isn't ready; callbacks are invoked only when the result actually is ready, which is why they (and later, Promises/async-await) are the correct tool for async code.

# 색인과 출처
- 개인 Q&A 학습 노트 — 클로저와 콜백 함수의 관계, setTimeout 비동기 콜백 예제
