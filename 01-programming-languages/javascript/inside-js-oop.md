# 인사이드 자바스크립트 — 객체지향 프로그래밍

## 목차
- [클래스 기반 vs 프로토타입 기반](#클래스-기반-vs-프로토타입-기반)
- [클래스, 생성자, 메서드](#클래스-생성자-메서드)
- [프로토타입을 이용한 상속](#프로토타입을-이용한-상속)
- [클래스 기반의 상속](#클래스-기반의-상속)
- [캡슐화](#캡슐화)
- [subClass 헬퍼 함수](#subclass-헬퍼-함수)

---

## 클래스 기반 vs 프로토타입 기반

- **클래스 기반**(Java, C++ 등): 클래스가 객체의 형태·기능을 정의하고, 생성자로 인스턴스를 만든다. 모든 인스턴스가 클래스 정의를 따르며 보통 런타임에 구조를 바꿀 수 없다.
- **프로토타입 기반**(JavaScript): 객체의 자료구조·메서드를 동적으로 변경할 수 있다. 정확성·예측성 측면에서는 클래스 기반보다 불리하지만, 유연성이 높다.
- 프로토타입은 자바스크립트에서 객체지향을 구현하는 필수 요소다.

## 클래스, 생성자, 메서드

자바스크립트에는 `class` 키워드가 원래 없었으므로(ES6 이전), 함수로 클래스·생성자·메서드를 모두 구현한다.

```javascript
function Person(arg) {
  this.name = arg;
  this.getName = function() { return this.name; };  // (비효율적인 방식)
}
var me = new Person("zzoon");
```

- 위 방식의 문제: `Person`으로 인스턴스를 여러 개 만들면, 인스턴스마다 `getName` 함수 객체를 **새로** 생성해 메모리를 낭비한다.
- **개선**: 메서드를 `Person.prototype`에 한 번만 정의하면, 모든 인스턴스가 프로토타입 체인을 통해 공유해서 쓴다.

```javascript
function Person(arg) { this.name = arg; }
Person.prototype.getName = function() { return this.name; };
Person.prototype.setName = function(value) { this.name = value; };

var me = new Person("me");
var you = new Person("you");
```

- 더글라스 크락포드는 함수를 생성자로 쓰는 것(즉 `new`로 호출하는 패턴) 자체를 권장하지 않았다 — `new` 없이 직접 호출될 수도 있어 `this` 바인딩이 호출 방식에 따라 달라지는 문제가 있기 때문. 최소한 생성자로 쓸 함수는 첫 글자를 대문자로 표기하는 관례를 권고했다.

## 프로토타입을 이용한 상속

더글라스 크락포드가 소개한 패턴(ES5의 `Object.create()`와 동일한 개념):
```javascript
function create_object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

var person = { name: "zzoon", getName() { return this.name; } };
var student = create_object(person);   // person을 상속하는 student
```

- 클래스의 인스턴스를 따로 만드는 게 아니라, 부모 객체와 그 객체를 프로토타입 체인으로 참조하는 자식 객체를 만들어 상속을 구현한다.
- **확장(extend)**: jQuery류 라이브러리처럼 객체에 다른 객체의 프로퍼티를 복사해 넣는 `extend()` 함수로 자식을 확장한다. 단, `obj[i] = prop[i]`는 **얕은 복사**라서 복사 대상이 객체/배열이면 참조만 공유된다 — 깊은 복사가 필요하면 재귀적으로 `extend`를 호출해야 한다(jQuery의 `deep` 플래그 방식).

## 클래스 기반의 상속

```javascript
function Person(arg) { this.name = arg; }
Person.prototype.setName = function(value) { this.name = value; };
Person.prototype.getName = function() { return this.name; };

function Student(arg) {
  Person.apply(this, arguments);   // 부모 생성자 호출 (인스턴스 초기화를 위해 필수)
}

function F() {}
F.prototype = Person.prototype;
Student.prototype = new F();               // 빈 함수를 중개자로 두어 Person/Student 인스턴스를 독립시킴
Student.prototype.constructor = Student;

var me = new Student("zoe");
```

- `Student.prototype = new Person()`처럼 직접 연결하면, `Person`과 `Student`의 인스턴스가 프로토타입 체인을 공유하게 되어 서로 영향을 줄 수 있다 — 그래서 빈 함수 `F()`를 중개자로 넣어 두 인스턴스 체인을 분리한다.
- `new Student(...)`만으로는 부모 생성자(`Person`)가 자동으로 호출되지 않는다 — 자식 생성자 안에서 `Person.apply(this, arguments)`로 명시적으로 호출해야 인스턴스가 제대로 초기화된다.

## 캡슐화

C++/Java의 `public`/`private` 키워드가 없어도, 클로저로 정보 은닉이 가능하다.

```javascript
var Person = function(arg) {
  var name = arg ? arg : "zoe";     // private
  this.getName = function() { return name; };  // public
  this.setName = function(v) { name = v; };
};
var me = new Person();
console.log(me.name);   // undefined — name은 클로저 안에 감춰짐
```

- `this`에 붙인 프로퍼티는 외부에서 접근 가능(public), `var`로 선언한 변수는 클로저를 통해서만 접근 가능(private)하다.
- 더 깔끔한 형태로는 객체를 반환하는 패턴도 쓰이지만, 이 경우 반환된 객체는 `Person.prototype`에 접근할 수 없어 상속 구현이 까다로워진다 — 그래서 **객체 대신 함수를 반환**하고 그 함수의 `prototype`에 공개 메서드를 정의하는 절충안도 쓰인다.
- **주의**: private 멤버가 객체/배열이면 얕은 복사로 참조만 반환되어 사용자가 외부에서 내용을 바꿀 수 있다 — 객체를 꼭 반환해야 한다면 깊은 복사로 복사본을 내보내야 한다.

## subClass 헬퍼 함수

프로토타입 체이닝 + `extend` + 생성자 자동 호출을 하나로 묶은 범용 상속 헬퍼(스토얀 스테파노프 방식 응용):

```javascript
var subClass = (function() {
  var F = function() {};  // 클로저로 한 번만 생성 (매 호출마다 재생성 방지)

  return function subClass(obj) {
    var parent = (this === (typeof window !== 'undefined' ? window : global)) ? Function : this;
    var child = function() {
      var _parent = child.parent;
      if (_parent && _parent !== Function) {
        _parent.apply(this, arguments);           // 부모 생성자 재귀 호출
      }
      if (child.prototype.hasOwnProperty("_init")) {
        child.prototype._init.apply(this, arguments);
      }
    };

    F.prototype = parent.prototype;
    child.prototype = new F();
    child.prototype.constructor = child;
    child.parent = parent;
    child.subClass = subClass;

    for (var i in obj) {
      if (obj.hasOwnProperty(i)) child.prototype[i] = obj[i];
    }
    return child;
  };
})();
```

핵심 설계 포인트:
- 부모 참조를 `child.parent_constructor`(또는 `child.parent`)에 저장해두면, 손자·증손자까지 이어지는 다단계 상속에서도 최상위까지 생성자를 재귀적으로 호출할 수 있다.
- `_init`을 `hasOwnProperty`로 검사하는 이유: 그냥 `if (parent._init)`처럼 검사하면 프로토타입 체인을 타고 올라가 상위 클래스의 `_init`을 중복 호출할 위험이 있다.
- 최상위 클래스는 `Function`을 상속받는 것으로 취급해, 재귀 호출이 `Function`에 닿으면 멈추게 한다.

```javascript
var Person = subClass({ _init() { console.log("person init"); }, getName() { return this._name; }, setName(n) { this._name = n; } });
var Student = Person.subClass({ _init() { console.log("student init"); }, getName() { return "Student: " + this._name; } });

var student = new Student();   // person init, student init 순서로 출력
```

# 색인과 출처
- 『인사이드 자바스크립트』 6장 학습 노트 — 객체지향 프로그래밍
