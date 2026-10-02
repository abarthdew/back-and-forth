# 8. jQuery 소스 코드 분석
## jQuery 1.0 소스 코드 구조
### 1. jQuery 함수 객체
### 2. 변수 \$를 jQuery() 함수로 매핑
### 3. jQuery.prototype 객체 변경
### 4. 객체 확장 - extend() 메서드
### 5. jQuery 소스 코드의 기본 구성 요소
## jQuery의 id 셀렉터 동작 분석
### 1. \$(”#myDib”) 살펴보기
### 1-1. jQuery.find(a, c) 살펴보기
### 1-2. this.get() 메서드 살펴보기
### 1-3. 다시 jQuery() 함수 코드로, \$(”myDiv”) 결과값
### 2. \$(”#myDiv”).text() 살펴보기
### jQuery 이벤트 핸들러 분석
### 1. jQuery 이벤트 처리 예제
### 2. .click() 메서드 정의
### 3. \$(”#clickDiv”).click() 호출 코드 분석
### 4. \$(”#clickDiv”).bind() 메서드 분석
### 4-1. .bind() 메서드 정의
### 4-2. \$(”#clickDiv”).bind() 호출
### 4-3. \$(”clickDiv”).each() 호출
### 4-4. jQuery.event.add(this, type, fn) 메서드 호출 분석
### 5. Click 이벤트 핸들러 실행 과정
### 6. jQuery 이벤트 핸들러 특징
