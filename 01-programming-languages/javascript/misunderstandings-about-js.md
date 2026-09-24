# this
- this는 호출 방식에 의해 결정됨
```js
const obj = {
  hello: function() {
    console.log(this)
  }
}
obj.hello() // this === obj
```
- 여기서, this는 obj.hello()로 호출됐기 때문에 obj가 됨

- this를 강제로 지정하는 메서드: apply, call, bind

```js
function greet() {
  console.log(this.name)
}
const user = {name: "Nana"}

// call, apply
greet.call(user) // 인자 하나씩 전달
greet.apply(user, []) // 인자 배열로 전달

// bind: 즉시 실행 X, this 고정을 위해 새로운 함수 생성 필요
const bound = greet.bind(user)
bound()

// 모든 경우, this === user
```

- call: 즉시 실행 + this 지정
- apply: 즉시 실행 + 배열 인자
- bind: 즉시 실행 X, this가 고정된 새로운 함수를 반환

- js의 this 법칙
  1. new -> 새 객체
  2. 객체.메서드() -> 점 앞 객체
  3. 그냥 함수() -> (브라우저) window / (strict mode) undefine
  4. call/apply/bind -> 강제 지정
  5. 화살표 함수 -> 상위 this 고정

# prototype
```js
function Person(name) {
  this.name = name
}

Person.prototype.say = function() {
  console.log(this.name)
}

const p = new Person("Nana")
p.say() // Person.prototype 에서 찾음
p.toString() // Object.prototype 에서 찾음
```

- say는 prototype에 존재
- this는 p
- this가 p인 이유는 p.say()로 호출했기 때문
- 내부 구조:
  ```lua
  p
   └── [[Prototype]] → Person.prototype
                         └── [[Prototype]] → Object.prototype
                                                └── [[Prototype]] → null
  ```
- JS는 속성을 찾을 때:
  1. 인스턴스 자신에서 찾고
  2. 없으면 [[Prototype]] 체인을 따라 올라간다.

# arguments
- 유사 배열 객체: Array는 아니고 관련 메서드도 없지만, length 존재, index 접근 가능, 화살표 함수에는 없음
- 요즘은 rest parameter 사용
  ```js
  function test(...args) {
    console.log(args)
  }
  ```

# 런타임 언어인가?
- 컴파일해서 실행 파일 만들지 않음 -> 인터프리터/vm(v8 등)이 바로 실행
- 이벤트 루프(콜 스택이 비면, 큐에서 콜백 하나 꺼내스 스택에 올리는 무한 루프) 전체 구조
  ```mathematica
  ┌──────── JS Engine ────────┐
  │  Call Stack               │
  └─────────▲─────────────────┘
            │
            │
       Event Loop
            │
  ┌─────────┴─────────┐
  │ Task Queue         │  (setTimeout, I/O 등)
  │ Microtask Queue    │  (Promise.then, await)
  └────────────────────┘
  
  ┌──────── Browser / Node ────────┐
  │  Timer / Network / I/O         │
  └────────────────────────────────┘
  ```
  
- 이벤트(트리거): 나중에 처리해달라고 큐에 들어오는 작업 (클릭, 타이머 완료, 네트워크 응답 도착, 파일 읽기 완료)
- 함수: 실행되는 코드 단위
  ```js
  button.addEventListener("click", () => {
    console.log("clicked");
  });
  // 이벤트: click, 실행되는 것: 콜백 함수
  ```

# 비동기 언어인가? -> 아님.
- JS: 단일 스레드 + 동기실행
  ```js
  console.log(1);
  console.log(2);
  console.log(3);
  // 무조건 1, 2, 3 순서로 출력
  ```
- 비동기처럼 보이는 이유: 단일 스레드 + 이벤트 루프 + 비동기 api
- JS 엔진은 한 번에 한 줄만 실행 + 타이머 / 네트워크 / 파일 io는 브라우저, Node가 백그라운드에서 처리
  ```js
  console.log("a");
  setTimeout(() => {
  console.log("b");
  }, 0);
  console.log("c");
  // 출력: a -> c -> b 순서
  // a 출력 -> setTImeout 등록 -> c 출력 -> b는 이벤트 큐에 들어갔다가 나중에 실행됨
  ```
- async / await: 비동기를 "동기 코드처럼 보이게" 쓰는 문법 설탕 (멀티 스레드 아님)
- 스레드를 제어하는 기능이 아니라, Promise 기반 비동기 흐름의 실행 순서를 기술하는 문법
  ```js
  async function f() {
    const res = await fetch(url); // await: "여기서 이 함수만 잠깐 멈추기" -> 메인 스레드는 안 멈춤
    console.log(res);
  }
  ```
- 비동기 실행 흐름 실제 예시
  ```js
  console.log("a");
  setTimeout(() => console.log("b"), 0);
  Promise.resolve().then(() => console.log("c"));
  console.log("d");
  // 출력: a -> d -> c -> b
  ```
  1. a (스택)
  2. setTimeout 등록 (브라우저에 위임)
  3. Promise.then 등록 (microtask queue)
  4. d (스택)
  5. 스택 비면: c (microtask 먼저) -> b (task queue)

  ```js
  async function f() {
    console.log("a");
    await fetch(url);
    console.log("b");
  }
  console.log("c");
  f();
  console.log("d");
  // 출력: c -> a -> d -> b
  ```
  1. f() 호출
  2. "a" 출력
  3. await fetch()를 만나서: f() 함수 실행 중단, 나머지는 Promise.then으로 등록, 그래도 메인 스레드는 계속됨
  4. "d" 출력
  5. fetch 끝나고 -> "b" 출력

- 정리
  ```js
    async function f() {
      A;
      await P; // Promise를 멈춰서 기다리는 것이 아니라, resolve될 때까지 함수 실행을 일시 중단
      B;
    }
  
    // 이 문법과 같음
    function f() {
      A; // A 실행 (동기)
      return P.then(() => { // P.then(...) 실행
         B; // P가 resolve되면 B 실행
      }); 
    } // f는 Promise를 반환
  ```

# 호이스팅
- 실행 컨텍스트 생성 단계에서 선언부를 미리 등록
  ```js
  console.log(x);
  var x = 10;

  // 엔진 내부
  var x; // 선언 호이스팅
  console.log(x);
  x = 10;
  ```



