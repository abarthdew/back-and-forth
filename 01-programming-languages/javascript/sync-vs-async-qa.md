# Sync vs. Async, and Where Arrow Functions Fit (Q&A notes)

## Contents
- [Synchronous file read](#synchronous-file-read)
- [Asynchronous file read](#asynchronous-file-read)
- [Comparison](#comparison)
- [Execution order example](#execution-order-example)
- [When to use which](#when-to-use-which)
- [Arrow functions are not inherently async](#arrow-functions-are-not-inherently-async)

---

## Synchronous file read

```javascript
const fs = require('fs');
console.log(fs.readFileSync('./text_1.txt').toString());
```

- The program **blocks** until the file is fully read; nothing else runs during that time.
- Pros: simple to reason about. Cons: freezes the program during long operations — a poor fit for servers handling multiple requests.

## Asynchronous file read

```javascript
fs.readFile('./text_1.txt', (err, data) => console.log(data.toString()));
```

- `fs.readFile` starts the read and returns immediately; the callback fires only once the read completes.
- Pros: non-blocking, other work continues in the meantime — ideal for responsive servers. Cons: requires callbacks/Promises/`async-await`, which can be harder to read and debug.

## Comparison

| Aspect | Synchronous | Asynchronous |
|---|---|---|
| Execution | Blocks until done | Doesn't block; continues while waiting |
| Complexity | Simple | Needs callbacks, Promises, or async/await |
| Responsiveness | Freezes during long ops | Stays responsive |
| Use case | CLI tools, simple scripts | Web servers, real-time apps, I/O-heavy tasks |

## Execution order example

```javascript
const fs = require('fs');
console.log('Sync:', fs.readFileSync('./text_1.txt').toString());
fs.readFile('./text_1.txt', (err, data) => console.log('Async:', data.toString()));
console.log('Done!');
```
Output order:
```
Sync: Hello, World!
Done!
Async: Hello, World!
```
The sync call blocks and finishes first; the async call is only *scheduled* before `Done!` runs, so its callback fires after.

## When to use which

- **Sync**: quick scripts, CLI tools, reading a config file at startup — where blocking briefly is acceptable.
- **Async**: production servers, database queries, network requests — always prefer this where responsiveness under concurrent load matters.

## Arrow functions are not inherently async

Arrow functions are just a shorthand syntax; sync/async behavior depends entirely on how a function is written and used, not on the `=>` syntax itself.

```javascript
const add = (a, b) => a + b;     // synchronous — returns immediately
console.log(add(2, 3));           // 5
```

A function becomes asynchronous when it:
1. Uses `async`/`await`, or
2. Deals with async operations (Promises, `setTimeout`, I/O).

```javascript
// Returns a Promise -> async in effect, though not marked `async`
const fetchData = () => new Promise(resolve => setTimeout(() => resolve("Data fetched"), 1000));
fetchData().then(data => console.log(data));

// Explicitly async with the `async` keyword
const fetchDataAsync = async () => {
  const data = await new Promise(resolve => setTimeout(() => resolve("Data fetched"), 1000));
  console.log(data);
};
```

**Key takeaway**: arrow functions themselves are neither sync nor async — that depends on whether the function uses `async`/`await` or interacts with Promises/async APIs.

# 색인과 출처
- 개인 Q&A 학습 노트 — Node.js sync/async 파일 읽기 비교, 화살표 함수와 비동기성의 관계
