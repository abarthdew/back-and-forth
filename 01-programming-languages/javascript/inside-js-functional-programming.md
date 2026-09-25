# 인사이드 자바스크립트 — 함수형 프로그래밍

## 목차
- [함수형 프로그래밍의 개념](#함수형-프로그래밍의-개념)
- [자바스크립트에서의 함수형 프로그래밍](#자바스크립트에서의-함수형-프로그래밍)
- [reduce로 일반화하기](#reduce로-일반화하기)
- [메모이제이션](#메모이제이션)
- [함수 적용과 커링](#함수-적용과-커링)
- [bind](#bind)
- [래퍼(Wrapper)](#래퍼wrapper)
- [반복 함수: each / map / reduce](#반복-함수-each--map--reduce)

---

## 함수형 프로그래밍의 개념

- 함수의 조합으로 작업을 수행하며, 작업이 이뤄지는 동안 **데이터와 상태는 변하지 않는다** — 변하는 것은 오직 함수(로직)뿐이다.
- 외부에 영향을 주지 않고 입력을 받아 결과만 돌려주는 함수를 **순수 함수**라 하고, 함수를 인자나 반환값으로 다루는 함수를 **고계 함수(higher-order function)**라 한다.
- 반대 개념은 **명령형 프로그래밍**이다 — 특정 작업의 순차적 명령을 기술하는 데 중점을 두며, 여기서 쓰이는 함수는 순수 함수와 목적이 다른 **프로시저**(예: `printf`)인 경우가 많다.

## 자바스크립트에서의 함수형 프로그래밍

일급 객체 함수와 클로저 덕분에 함수형 스타일이 가능하다.
```javascript
var get_encrypted = function(func) {
  var str = 'zoe';
  return function() {           // 클로저 — str은 외부에서 접근 불가
    return func.call(null, str);
  };
};
var result = get_encrypted(f1)();
```

**명령형 vs 함수형** 비교 예시(배열 합/곱):
```javascript
// 명령형 — 연산마다 새 함수를 만들어야 함
function sum(arr) { /* ... */ }
function multiply(arr) { /* ... */ }
```

## reduce로 일반화하기

```javascript
function reduce(func, arr, memo) {
  var accum = memo;
  for (var i = 0; i < arr.length; i++) accum = func(accum, arr[i]);
  return accum;
}

var sum = (x, y) => x + y;
var multiply = (x, y) => x * y;

reduce(sum, [1,2,3,4], 0);       // 10
reduce(multiply, [1,2,3,4], 1);  // 24
```
- 연산 로직(`sum`, `multiply`)을 함수로 분리해 `reduce`에 꽂아 넣기만 하면 되므로, 명령형 방식보다 한 단계 높은 모듈화가 가능하다.

## 메모이제이션

이전 계산 결과를 캐시에 저장해 재계산을 피하는 기법.
```javascript
var fact = (function() {
  var cache = {'0': 1};
  var func = function(n) {
    if (typeof cache[n] === 'number') return cache[n];
    return cache[n] = n * func(n - 1);
  };
  return func;
})();
```
- 범용화하려면 `Function.prototype.memoization`을 정의해 어떤 함수에서든 재사용할 수 있다.
- 피보나치 수열, 팩토리얼 등 재귀 연산에서 특히 유용하다. jQuery의 `data()`/`cleanData()`도 같은 메모이제이션 패턴을 응용한 것이다.

## 함수 적용과 커링

- **함수 적용(Applying functions)**: `func.apply(obj, args)`처럼 함수를 특정 데이터에 "적용"시키는 것. 자바스크립트의 `apply` 메서드 이름이 여기서 유래했다.
- **커링(Currying)**: 함수의 인자 일부를 고정하고, 나머지를 받는 새 함수를 만드는 것.
```javascript
function curry(func) {
  var args = Array.prototype.slice.call(arguments, 1);
  return function() {
    return func.apply(null, args.concat(Array.prototype.slice.call(arguments)));
  };
}
function calculate(a, b, c) { return a * b + c; }
var new_func1 = curry(calculate, 1);
new_func1(2, 3);   // 5 (1*2+3)
```
- 자바스크립트는 커링을 기본 제공하지 않지만 `Function.prototype.curry`로 확장해 쓸 수 있다. 중간 인자만 고정하려면(`curry2`처럼) `undefined`를 자리표시자로 쓰는 변형이 필요하다. 이렇게 일부 인자만 채워 새 함수를 만드는 것을 **함수의 부분 적용**이라 한다.

## bind

```javascript
Function.prototype.bind = function() {
  var fn = this, args = Array.prototype.slice.call(arguments, 1);
  return function() {
    return fn.apply(this, args.concat(Array.prototype.slice.call(arguments)));
  };
};
```
- 커링과 유사하지만, **함수 호출 시 `this`에 바인딩할 객체까지 지정**할 수 있다는 점이 다르다.
- ES5의 `Function.prototype.bind`는 반환된 함수가 원본 함수의 프로토타입까지 상속하도록 구현되어 있어, `new`로 호출해도 정상 동작한다.

## 래퍼(Wrapper)

기존 함수를 자신의 로직으로 덮어쓰되, 원래 기능은 유지하는 패턴(객체지향의 오버라이드와 유사).
```javascript
function wrap(object, method, wrapper) {
  var fn = object[method];
  return object[method] = function() {
    return wrapper.apply(this, [fn].concat(Array.prototype.slice.call(arguments)));
  };
}
```
- 사용자는 래핑 함수의 첫 번째 인자로 원래 함수의 참조를 받아, 그 함수를 실행한 뒤 자신의 로직을 추가로 수행할 수 있다.
- 기존 라이브러리 함수에 로직을 추가하거나, 특정 플랫폼의 버그를 우회할 때 유용하다.

## 반복 함수: each / map / reduce

```javascript
// each: 각 요소를 꺼내 함수에 적용 (jQuery 1.0 스타일)
function each(obj, fn, args) {
  if (obj.length === undefined) {
    for (var i in obj) fn.apply(obj[i], args || [i, obj[i]]);
  } else {
    for (var i = 0; i < obj.length; i++) fn.apply(obj[i], args || [i, obj[i]]);
  }
  return obj;
}

// map: 각 요소를 변환해 새 배열 생성
Array.prototype.map = function(callback) {
  var result = [];
  for (var i = 0; i < this.length; i++) result[i] = callback(this[i]);
  return result;
};
[1, 2, 3].map(v => v * v);   // [1, 4, 9]

// reduce: 각 요소를 누적
Array.prototype.reduce = function(callback, memo) {
  var acc = memo;
  for (var i = 0; i < this.length; i++) acc = callback(acc, this[i]);
  return acc;
};
[1, 2, 3].reduce((a, b) => a + b * b, 0);  // 14
```
- `each`에 넘기는 콜백은 대개 부수 효과(로그 출력 등)를 위한 **프로시저**이지, 순수 함수형 예제는 아니다.
- 라이브러리마다 `each` 콜백에 넘기는 인자 순서(인덱스 먼저 vs 값 먼저)가 다를 수 있으니 문서를 확인해야 한다.

# 색인과 출처
- 『인사이드 자바스크립트』 7장 학습 노트 — 함수형 프로그래밍
