## 3.1. 컴포넌트란 무엇인가?
- vue.js는 UI 컴포넌트를 잘 다룰 수 있는 메커니즘인 컴포넌트 시스템 제공
### 3.1.1. 모든 것은 UI 컴포넌트로
- 웹 애플리케이션의 UI는 이를 구성하는 여러 부품의 조합
- 이러한 부품을 UI 컴포넌트라고 함
- 웹 사이트 규모 상관없이 컴포넌트가 트리 구조를 이룬 컴포넌트 트리 형태로 구성됨
- 한 페이지 안에서 기능이나 외관이 비슷한 부분, 서로 다른 페이지끼리 재사용 가능
### 3.1.2. 컴포넌트의 장점과 주의할 점
- 재사용 향상 → 개발 효율성
- 품질 보장
- 적절히 분할한 컴포넌트가 느슨하게 결합 → 유지 보수성 향상
- 캡슐화를 통해 개발 작업에서 신경 써야 할 부분 최소화
### 3.1.3. vue.js의 컴포넌트 시스템
- vue.js 컴포넌트: 재사용 가능한 Vue 인스턴스
	```javascript
	// html
	<ul>
	 <list-item/>
	</ul>

	// script
	Vue.component('list-item', {
	template: '<li>foo</li>'
	})

	// 최상위 Vue 인스턴스 생성
	new Vue({ el: '#example' })
	```
## 3.2. Vue 컴포넌트 정의하기
- Vue 컴포넌트는 전역/지역 컴포넌트로 나뉨
- 정의하는 방법:
	1. 커스텀 태그 방식: Vue.component()
	2. 하위 생성자: Vue.extend()
### 3.2.1. 전역 컴포넌트 정의하기
- 커스텀 태그 방식
	```javascript
	Vue.component(tagName, opitons)
	```

- 두번째 인자 options: 컴포넌트 설정 정보 객체

	⇒ Vue 인스턴스의 설정 옵션 사용 가능(Vue 인스턴스 옵션, template, props, 생애주기 훅 포함)

| data | UI 상태 및 데이터 |
|---|---|
| filters | 데이터를 문자열로 포매팅 |
| methods | 이벤트가 발생했을 때의 동작 |
| computed | 데이터에서 파생된 값 |
| template | 컴포넌트 템플릿 |
| props | 부모 컴포넌트로부터 받은 데이터 |
| created 외 | 생애주기 훅(생성 지점) |

- 컴포넌트를 정의할 때 사용하는 것: template, props
- 컴포넌트 구현의 간단한 예제
	```javascript
	<div id="fruits-list">
	<fruits-list-title/>
	</div>

	<script>
	Vue.component('fruits-list-title', {
	template: '<h1>과일 목록</h1>'
	})

	new Vue({
	el: '#fruits-list'
	})
	</script>
	```

- 자식 컴포넌트와 부모 컴포넌트: 생략
### 3.2.2. 생성자를 사용해 컴포넌트 정의하기
- 전역 API Vue.extend()를 사용, Vue 생성자를 상속받는 하위 생성자 생성 가능
- 하위 생성자를 사용하는 방법으로 컴포넌트 생성 가능
- Vue.extend()를 사용한 컴포넌트 정의(정의한 컴포넌트를 #mont 함수를 사용해 특정 요소에 바로 마운트)
	```javascript
	var FruitsListTitle = Vue.extend({
	template: '<h1>과일 목록</h1>'
	})

	new FruitsListTitle().$mount('#fruits-list')
	```

- 이전 방법은 Vue.component() 의 두 번째 인자로 옵션 객체 바로 전달
- 해당 방법은 하위 생성자를 전달해 컴포넌트 등록 → 커스텀 요소 사용 가능
	```javascript
	Vue.component('fruits-list-title', FruitsListTitle)
	```
### 3.2.3. 지역 컴포넌트 정의하기
- 이전까지는 전역 vue.js 인스턴스에 컴포넌트 정의
- 지역 컴포넌트: 어떤 특정한 Vue 인스턴스에만 등록 가능한 컴포넌트
- 부모 Vue 인스턴스 혹은 컴포넌트 옵션에 components 객체 정의, 여기에 컴포넌트 등록
	```javascript
	<div id="fruits-list">
	<fruits-list-title/>
	</div>

	<script>
	new Vue({
	el: '#fruits-list',
	components: {
		'fruits-list-title': {
			template: '<h1>과일 목록</h1>'
		}
	}
	})
	</script>
	```
### 3.2.4. 템플릿을 만드는 그 외의 방법
- text/x-template
- render 함수
- 단일 파일 컴포넌트
- 인라인 템플릿
- JSX
### 3.2.5. 컴포넌트 명명 규칙
- 케밥 케이스 사용 권장: 웹 컴포넌트의 custom element 규격의 드래프트가 케밥 케이스를 기준
	```javascript
	<pascal-fruits-list/>
	```
### 3.2.6. 컴포넌트 생애주기
- 각 컴포넌트는 저마다의 생애주기를 가짐
- Vue 인스턴스와 마찬가지로 생애주기 훅이 있음
- 컴포넌트는 각 생애주기 시점마다 이에 해당하는 이벤트를 발생시킴
- Vue 인스턴스처럼 이벤트에 맞춰 실행되는 훅 함수 정의 가능
### 3.2.7. 컴포넌트 데이터
- Vue 인스턴스의 data 속성은 객체 형태로 정의 → 컴포넌트의 data 속성을 객체 형태로 정의하면 모든 인스턴스가 이 data 객체를 공유

	⇒ 인스턴스 간 서로 다른 데이터를 가지기 위해 객체를 반환하는 함수 정의 → 이 함수를 data 속성의 값으로 지정

	```javascript
	// data를 return 문으로 변환 
	Vue.component('single-counter', {
	template: '<h1>과일 목록</h1>',
	data: function() {
		return {
			fruits: [1, 2]
		}
	}
	})
	```

- data 속성값에 객체를 지정하면? ⇒ vue.js가 이 사실을 경고로 알려 줌
	- 컴포넌트의 data 속성에 함수가 아닌 객체를 값으로 지정하면, 모든 컴포넌트 인스턴스가 같은 객체를 참조함
	- data 속성 외 el 속성도 모든 컴포넌트가 같은 대상을 참조하므로 함수 형태로 선언해야 함
## 3.3. 컴포넌트 간 통신
- vue.js의 컴포넌트는 각 독립된 유효범위를 가짐

	![](images/vue03-01.png)

### 3.3.1. 부모 컴포넌트에서 자식 컴포넌트로 데이터 전달하기
- props
- [https://jsfiddle.net/flourscent/vqsh81wy](https://jsfiddle.net/flourscent/vqsh81wy)
### 3.3.2. 자식 컴포넌트에서 부모 컴포넌트로 데이터 전달하기
- 자식 컴포넌트 → 부모 컴포넌트: 커스텀 이벤트로 정보 전달

| 용도 | 인터페이스 |
|---|---|
| 이벤트 리스닝 | $on(eventName) |
| 이벤트 트리거 | $emit(EventName) |

- [https://jsfiddle.net/flourscent/teo94h6j](https://jsfiddle.net/flourscent/teo94h6j)
- props와 이벤트 없이 부모/자식 컴포넌트 간 통신하기
	- $parent, $children 사용(권장하지 않음)
	```javascript
	<script src="https://unpkg.com/vue@2.5.17"></script>

	<div id='test-container'>
	<fruits-name/>
	</div>

	<script>
	Vue.compnent('fruits-name', {
	template: '<p>{{ this.$parent.fruits[0].name }}</p>'
	})

	new Vue({
	el: '$test-container',
	data: {
		fruits: [
			{name: '배'},
			{name: '딸기'},
		]
	}
	})
	</script>
	```

	- 자식 컴포넌트를 직접 참조하려면 ref 사용 가능
	- 다음과 같은 방법으로 부모 → 자식 컴포넌트 참조 가능
	```javascript
	<div id='test-container'>
	<fruits-name ref="counter" />
	</div>

	<script>
	var parent = new Vue({ el: '#test-container' })
	var child = parent.$refs.counter
	</script>
	```

- 부모 자식 관계가 아닌 컴포넌트끼리 데이터 주고받기
	- 형제 관계에 있는 여러 컴포넌트가 같은 값을 공유해야 하는 경우
	- 컴포넌트 상태, 이를 관리하는 함수를 여러 컴포넌트끼리 공유해야 하는 경우

	⇒ 스토어라는 객체에 상태를 저장해서 관리하는 방법이 효과적

	- 상태 관리만을 목적으로 하는 별도의 대상을 만들고 여기서 상태 관리
	- vuex 참조
- **자식 컴포넌트가 부모 컴포넌트에서 발생하는 네이티브 DOM 이벤트의 정보를 전달받아야 할 경우 - .native 수정자**
	- 부모 컴포넌트의 요소에서 발생한 네이티브 이벤트(click 등)를 트리거로 삼아 자식 컴포넌트에서 메서드를 실행하고 싶을 때
		```javascript
		<my-component @click.native="someMethod"/>
		```

		⇒ 부모 요소의 DOM 이벤트 모니터링 가능

- **props 값을 양방향으로 바인딩해야 하는 경우 - .sync 수정자**
	- vue.js에서 컴포넌트 간 통신: 부모 → 자식 컴포넌트로 단방향성 전달
	- props에 전달한 값이 수정되면 수정된값이 부모 → 자식 컴포넌트로 전달되는 형태(반대는 불가능)
	- .sync 사용: 자식 컴포넌트의 이벤트 구독 → 이 이벤트가 발생한 시점에 부모 컴포넌트의 값을 수정
## 3.4. 컴포넌트 설계
### 3.4.1. 컴포넌트를 분할하는 원칙
- 네비게이션 바
- 사이드 바
- 메인 콘텐츠
### 3.4.2. 컴포넌트 설계하기
- 부모 컴포넌트가 무엇이든 느슨한 결합을 갖도록 인터페이스 설계
- 아토믹 디자인: 컴포넌트를 분할하는 원칙으로, 2013 브래드 프로스트가 제창
	- 원자/분자/유기체/템플릿/페이지 5 단계로 컴포넌트 설계
	1. 원자: 버튼, 레이블, 컬러 팔레트, 폰트 등 최소 구성 요소
	2. 분자: 하나 이상의 원자로, 레이블이 붙은 폼 등
	3. 유기체: 분자보다 복잡하며, 로그인 폼, 댓글창, 네비게이션 바
	4. 템플릿: 유기체의 조합으로, 디자인 와이어프레임이며, 실제 데이터를 표시하지는 않지만 페이지 구성을 설명할 수 있는 단계
	5. 페이지: 템플릿에 실제 데이터를 담은 것으로, 완성된 페이지
### 3.4.3. 슬롯 콘텐츠를 살린 헤더 컴포넌트 구현하기
- 슬롯 콘텐츠: 부모 컴포넌트 별로 자식 컴포넌트의 내용을 바꿀 수 있는 메커니즘
- 컴포넌트 안에서 부모 컴포넌트가 쉽게 수정할 수 있는 부분을 만드는 메커니즘
- 헤더 컴포넌트 안에 slot이라는 요소를 포함시키며, 이 부분이 부모 컴포넌트에 의해 내용이 변경될 부분
	- 콘텐츠를 삽입했을 때
	```javascript

	<div id="fruits-list">
	  <page-header class="header">
		<!--부모 컴포넌트에서 자식 컴포넌트의 slot 요소에 name 속성값을 지정하면 자식 컴포넌트의 콘텐츠를 커스터마이징 가능-->
	    <h1 slot="header">
	      겨울 과일
	    </h1>
	  </page-header>
	  <page-content class="content">
	    <ul slot="content">
	      <li>사과</li>
	      <li>딸기</li>
	    </ul>
	  </page-content>
	</div>
	```

	- 콘텐츠를 삽입하지 않았을 때
	```javascript
	<div id="fruits-list">
	  <page-header class="header"></page-header>
	  <page-content class="content"></page-content>
	</div>
	```

	- 공통 script
	```javascript
	<script>
	var headerTemplate = `
	  <div>
	    <slot name="header"><h1>No title</h1></slot>
	  </div>
	`

	var contentTemplate = `
	  <div>
	    <slot name="content"><li>No contents(부모 컴포넌트로부터 전달받은 것이 없으면 이 문장을 표시)</li></slot>
	  </div>
	`

	Vue.component('page-header', {
	  template: headerTemplate
	})
	Vue.component('page-content', {
	  template: contentTemplate
	})

	new Vue({
	  el: "#fruits-list"
	})
	</script>
	```

- 자주 사용하는 레이아웃을 slot 요소를 적용한 컴포넌트로 만들어 두고, 안의 콘텐츠만 바꿔 서로 다른 UI 컴포넌트 구성 가능
### 3.4.4. 로그인폼 컴포넌트 구현하기
- 생략
### 3.4.5. 컴포넌트의 단위 테스트
- 컴포넌트는 재사용을 통해 위력을 발휘함
- 컴포넌트에 결함이 있으면, 애플리케이션 전체에 그 영향이 미칠 가능성이 큼
- 컴포넌트 같은 비교적 작은 단위를 테스트로 검증하는 것은 애플리케이션 전체의 품질을 확보하는 데 중요함
- karma를 테스트 러너로, mocha를 테스트 프레임워크를 사용해 Vue 컴포넌트를 테스트하는 방법
	- karma 설치, 설정 → 패키지 초기화, package.json 파일 작성 후 karma, mocha 설치
	```javascript
	$ mkdir vue-components && cd vue-components
	$ npm init -y
	$ npm install -g karma
	$ npm install --save-dev mocha
	```

	- mocha 초기화 → 테스트 프레임워크 mocha 선택, 테스트 대상 파일과 테스트 파일의 위치를 각각 component/*.js, test/*.js로 설정
	```javascript
	$ karma init
	> mocha
	> components/*.js
	> test/*.js
	```

	- karma start 명령: 서버가 문제없이 시작되는지 확인
	- webpack을 함께 사용해 브라우저에서도 require가 동작하는지 확인
	- 테스트 위치에 테스트 대상 컴포넌트 배치
	- 예제) 로그인 폼 컴포넌트를 앞서 지정한 components 디렉토리 안에 components/loginForm.js라는 이름으로 저장
	```javascript
	var Vue = require('vue')
	var auth = {
	login: function() {
		return ({
			userid: id,
			password: pass,
		})
	}
	}

	module.exports = Vue.extend({
	template: '#login-template',
	data: function() {
		return {
			userid: '',
			return {
				userid: '',
				password: '',
			}
		},
		mothods: {
			login: function() {
				return auth.login(this.userid, this.password);
			}
		}
	}
	})
	```

	- 컴포넌트 테스트 케이스 작성
	- 다음 내용을 test/test.js 파일에 저장
	```javascript
	var assert = require('assert') // webpack을 사용해 모듈 간의 의존관계를 해소
	var loginForm = require('../components/loginForm')

	describe('login()', function() {
	var vm
	beforeEach(function() {
		vm = new loginForm().$mount()
	})
	// userid, password의 초기값을 확인
	it('check initial values', function() {
		assert.equal(vm.userid, '')
		assert.equal(vm.password, '')
	})
	// login() 메서드의 반환값 테스트
	it('check returned value - login()', function() {
		vm.userid = 'testuser'
		vm.password = 'password'
		var result = vm.login()
		assert.deepEqual(result, {
			userid: 'testuser',
			password: 'password'
		})
	})
	})
	```

	- userid와 password의 초기값을 테스트
	- 두 번째 테스트 케이스는 login() 메서드를 테스트
	- karma를 사용해 테스트 케이스를 실행, 테스트 통과 확인
	- 위와 같은 방법으로 컴포넌트의 데이터와 메서드 테스트
