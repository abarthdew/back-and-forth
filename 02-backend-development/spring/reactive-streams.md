# 1. Reactive Stream

---

## 1) 리액티브 스트림의 목적

- 논블로킹 백프레셔로 비동기 스트림 처리를 위한 표준을 제공하는 것.

## 2) 최신 릴리즈

- Maven Central에서 사용 가능 :

```xml
<dependency>
  <groupId>org.reactivestreams</groupId>
  <artifactId>reactive-streams</artifactId>
  <version>1.0.3</version>
</dependency>
<dependency>
  <groupId>org.reactivestreams</groupId>
  <artifactId>reactive-streams-tck</artifactId>
  <version>1.0.3</version>
  <scope>test</scope>
</dependency>
```

# 2. 목표, 설계 및 범위

---

## 1) 필요성

- 데이터 `Stream`(특히 볼륨이 미리 결정되지 않은 "실시간" 데이터)을 처리하려면 비동기 시스템에서 각별한 주의가 필요.
- 가장 두드러진 문제는 빠른 데이터 소스가 스트림 대상을 압도하지 않도록 리소스 소비를 세심하게 제어할 필요가 있음.
- 단일 시스템 내에서 네트워크 호스트 또는 여러 CPU 코어를 협업할 때 컴퓨팅 리소스를 병렬로 사용하려면 비동기화가 필요.

## 2) 목표, 설계 및 범위

- `Reactive Stream`의 주요 목표는 수신 측에서 임의 양의 데이터를 버퍼링하지 않도록 하면서 비동기 경계(다른 스레드 또는 스레드 풀로 요소를 전달하는 방식)에서 스트림 데이터 교환을 제어하는 것.
- 다시 말해서, `Backpressure`는 스레드 사이를 중재하는 대기열을 경계로 만들기 위해 이 모델의 필수적인 부분.
- `Backpressure`가 동기적이라면 비동기적 처리의 이점은 부정될 수 있으므로, 리액티브 스트림 구현의 모든 측면에 대해 완전 비차단적 및 비동기적 동작을 의무화하는 데 주의를 기울임.
- 이 규격은 규칙을 준수함으로써 스트림 애플리케이션의 전체 처리 그래프에서 전술한 유익성과 특성을 보존하면서 원활하게 상호 운용될 수 있는 많은 적합한 구현을 만들 수 있도록 하기 위함.
- 스트림 조작의 정확한 특성(변환, 분할, 병합 등)은 해당 문서에서 다루지 않음.
- 개발 과정에서 스트림을 결합하는 모든 기본 방법이 표현될 수 있도록 주의하며, 리액티브 스트림은 서로 다른 API 구성 요소 간 데이터 스트림을 조장하는 데만 쓰임.

## 3) 요약

> 💡 **리액티브 스트림은 JVM을 위한 스트림 지향 라이브러리의 표준 및 규격이며, 다음과 같은 기능을 제공함.**
>
> ① 무한한 수의 요소를 처리,
>
> ② 순서대로,
>
> ③ 구성 요소 간 요소를 비동기적으로 전달함,
>
> ④ 필수적인 논블로킹 백프레셔를 사용해서!

# 3. 반응형 스트림 사양의 구성

---

- API는 반응형 스트림을 구현하고 서로 다른 구현 간 상호 운용성을 달성하는 유형 지정.
- TCK(Technology Compatibility Kit)는 구현 적합성 테스트를 위한 표준 테스트 제품군.
- API 요구 사항을 준수하고 TCK에서 테스트를 통과하기만 하면 규격에서 다루지 않는 추가 기능을 자유롭게 구현 가능.

# 4. API 구성 요소

---

> 💡 **API는 리액티브 스트림 구현에서 제공하는 데 필요한 다음과 같은 구성 요소로 구성됨.**
>
> ① Publisher
>
> ② Subscriber
>
> ③ Subscription
>
> ④ Processor

- Publisher는 잠재적으로 제한되지 않는 수의 시퀀스 요소를 제공하는 공급자며, Subscriber로부터 수신한 요구에 따라 게시(publishing)함.
- `Publisher.subscribe(Subscriber)` 해당 호출에 대한 응답으로, **`Subscriber`**에 대한 메서드 중 호출 가능한 시퀀스는 다음 프로토콜에 의해 제공됨.

```javascript
onSubscribe onNext* (onError | onComplete)?
```

- 즉, `Subscription`이 취소되지 않는 한, 항상 `onSubscribe`로 작업 처리
- `onNext` (Subscriber가 요청한) : 무한 작업 처리
- `onError` : 장애가 발생한 경우
- `onComplete` : 또는 더 이상의 요소를 사용할 수 없는 경우, 작업 종료

# 5. 용어 사전

---

> 💡 **Difference between `Calling` and `Invoking`**
>
> Function calling is when you call a function yourself in a program. While function invoking is when it gets called automatically.

| 용어 | 설명 |
|---|---|
| Signal | 명사 : `onSubscribe`, `onNext`, `onComplete`, `onError`, `request(n)` , `cancel` 중 하나를 가리킴. 동사 : 신호 호출/실행. |
| Demand | 명사 : Publisher가 아직 전달(완료)하지 않은 Subscriber가 요청한 총 요소 수. 동사 : 더 많은 요소를 요구하는 행위. |
| Synchronous(ly, 동기) | calling Thread에서 실행 |
| Return normally | 선언된 유형의 값만 호출자에게 반환. Subscriber에게 실패를 알리는 적법한 방법은 `onError` 메서드를 사용하는 것이 유일. |
| Responsivity(응답성) | 호출에 대한 대응 준비와 기량. 본 문서에서는 각각 다른 구성요소가 반응에 대한 서로의 기량을 손상시키지 않아야 한다는 것을 나타냄. |
| Non-obstructing(방해 방지) | 최대한 빨리 실행되는 호출 스레드에 대한 품질 설명. 즉, 예를 들어, 호출자의 실행 스레드를 지연시킬 수 있는 과도한 계산과 다른 것들을 피하는 것을 의미. |
| Terminal state(터미널 상태) | Publisher에 대한 의미 : `onComplete`나 `onError` 신호가 발생했을 때. Subscriber에 대한 의미 : `onComplete`나 `onError` 신호를 받았을(수신) 때. |
| NOP | 호출 스레드에 감지할 수 있는 영향이 없는 실행은 여러 번 안전하게 호출할 수 있음. |
| Serial(ly) | 신호 체계에서 겹치지 않음. JVM의 맥랙에서, 객체에 있는 메서드들에 대한 호출은 해당 호출들 사이에 발생 전 관계가 있는 경우에만 연속됨(호출이 겹치지 않는다는 의미도 포함). 호출이 비동기적으로 수행될 때, 발생 전 관계를 설정하기 위한 조정은 atomics, monitors, lock과 같은 기술을 사용하여 구현되어야 함. |
| Thread-safe | 프로그램의 정확성을 보안하기 위해 외부 동기화를 요구하지 않고 동기식 또는 비동기식으로 안전하게 호출할 수 있음. |

# 6. 사양

---

## 1) Publisher

```javascript
package org.reactivestreams;

public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

### *규칙 :

- Publisher가 Subscriber에게 보낸 `onNext`의 총 수는 Subscriber의 Subscription에서 항상 요청한 총 요소 수보다 항상 작거나 같아야 함.
- Publisher는 요청받은 것보다 적은 `onNext` 신호를 보낼 있으며, `onComplete`나 `onError` 를 호출해서 구독을 종료할 수 있다.
- Subscriber에게 보내진 `onSubscribe`, `onNext`, `onError`, `onComplete`는 반드시 연속적으로 신호를 보내야 한다. (각 신호 사이 관계 설정 전에 발생되는 경우, 여러 스레드로부터 신호의 전달을 허용함)
- Publisher가 작업이 실패했을 경우 `onError`신호를 보냄.
- Publisher가 작업을 성공적으로 종료했을 때 `onComplete` 신호를 보냄.
- Publisher가 Subscriber에 `onError`나 `onComplete` 신호를 보냈을 때, Subscriber의 Subscription은 취소한 것으로 간주함.
- 일단, 터미널 상태의 신호가 처리되면(`onError`, `onComplete`) 추가 신호가 발생하지 않아야 함.
- Subscription이 취소되면 Subscriber는 신호 수신이 중지됨.

## 2) Subscriber

```javascript
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

## 3) Subscription

```javascript
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

## 4) Processor

```javascript
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

# 7. 비동기 vs 동기 작업

# 8. Subscriber 제어 큐

## 색인과 출처

<details>
<summary>Reactive Streams</summary>

- [https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification)

</details>
