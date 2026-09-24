---
title: JAVA Method Reference
date: 2021-03-16
categories: [Back, Java]
tags: [Back, Java]
---

# 메서드 레퍼런스

## Overview of Method References

| 종류 | Syntax | Method Reference | Lambda Expression |
| --- | --- | --- | --- |
| 1. Reference to a static method  | ContainingClass::staticMethodName | String::valueOf | s → String.valueOf(s) |
| 2. Reference to an instance method<br>of a particular object | ContainingObject::instanceMethodName | s::toString | s → s.toString() |
| 3. Reference to instance method<br>of an arbitrary object of a given type | ContainingTypoe::methodName | String::toString |  s → s.toString() |
| 4. Reference to a constructor | ClassName::new | String::new | () → new String() |

- A method reference is a compact, easy-to-read handle for a method that already has a name.

## 1) 개념

- `람다 표현식`이란? : 익명 클래스를 반복적으로 작성하면 코드가 지저분해지므로, 이를 줄이는 것.
- `메서드 레퍼런스`란? : 메서드의 레퍼런스를 전달한다는 의미. 람다 표현식에서 메서드 호출 1회로 코드가 끝나는 경우, 메서드 레퍼런스를 이용해 이미 줄인 코드를 더 줄일 수 있음.

```java
Function<String, Integer> f = str -> Integer.parseInt(str);
```

- Function 인터페이스는 하나의 인자를 받아 다른 타입의 리턴타입을 갖는 apply() 메서드를 가지고 있는 함수형 인터페이스.
- 메서드 구현부분이 Integer.parseInt() 메서드 호출 한번으로 끝남.

```java
Function<String, Integer> f = Integer::parseInt
Integer result = f.apply("123");
```

- Integer 클래스의 정적 메서드 parseInt()를 메서드 레퍼런스로 전달 → 사용가능한 지역변수는 String 타입의 str 밖에 없음 → 리턴타입은 제네릭을 이용해 Integer임을 추론.
- 때문에, 메서드만 전달해주면 컴파일러가 알아서 호출. 선언적 코딩을 할 수 있도록 함.

```java
Function<String, Boolean> f = String::isEmpty;
Boolean result = f.apply("123");
```

- isEmpty()는 정적 메서드가 아니지만, 넘어오는 인자 타입이 String이기 때문에 해당 타입의 메서드를 호출.
- 메서드 레퍼런스 예 마다 사용법, 추론법이 약간씩 다름.

## 2) 정적 메서드 레퍼런스

- `정적 클래스` :: `메서드명`

```java
Function<String, Integer> f = Integer.parseInt;
Integer result = f.apply("123");
```

## 3) 인자 인스턴스 메서드 레퍼런스

- `타입 클래스` :: `메서드명`

```java
Function<String, Boolean> f = String::isEmpty;
Boolean result = f.apply("123");
```

## 4) 생성자 메서드 레퍼런스

- 생성자도 메서드 레퍼런스로 표기가 가능.
- Supperlier는 인자 없이 리턴만 하는 메서드를 가짐. 호출 시 new 로 생성된 String 객체 리턴(기본 생성자 호출).

```java
Supplier<String> s = String::new;
```

## 5) 바깥 인스턴스 메서드 레퍼런스

- 람다 캡쳐링을 이용, 람다 표현식 바깥에 있는 인스턴스의 메서드를 호출할 때 사용.
- 인자 인스턴스와 다르니 주의.

```java
String str = "hello";
Predicate<String> p = str::equals; // 이런 형태로는 작성할 수 없음.
p.test("world");
```

- equal() 메서드는 호출하는 객체와 인자로 넘기는 객체. 총 2개의 객체가 필요한 메서드. Predicate를 수행할 때 인자로 넘어오는 객체는 1개 뿐이기 때문에 이런 형태로는 작성할 수 없음.
- 바깥에 있는 str의 equals를 호출해야 인자로 넘어오는 String 객체를 equals() 메서드의 인자로 보내겠다는 의도를 추론할 수 있다.

```java
// 다른 경우
Comparator<String> c = String::compareTo;
```

- compareTo() 메서드를 호출하기 위해 호출하는 개체와 인자로 보내는 객체, 객체 2개가 필요함. compareTo() 메서드에서 2개의 인자가 넘어옴. 첫번째 인자로 호출하고 두번째 인자를 인자로 보냄.

# 색인과 출처

- https://multifrontgarden.tistory.com/126
