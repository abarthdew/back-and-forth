- vue.js 그 자체는 단순한 뷰 계층 라이브러리
- 공식 플러그인인 Vue Router를 사용해 단일 페이지 애플리케이션을 비롯한 URL 이동이 필요한 동작 구현 가능
## 4.1. Vue Router를 이용한 단일 페이지 애플리케이션
- SPA: 최초에 HTML 페이지 하나를 로드 → 이후 사용자 인터랙션에 따라 AJAX로 정보를 받아 와 동적 페이지 업데이트하는 웹 애플리케이션
- `일반적 웹 애플리케이션`: 페이지 이동 시, 대상 URL을 서버에 요청해 전체 페이지에 해당하는 HTML 콘텐츠를 받아 옴

	↔ `SPA`: 페이지 이동을 클라이언트에서 처리

- 페이지 이동 시 AJAX를 사용해 적시에 필요한 데이터를 받아와 뷰를 화면에 표시 → 애플리케이션 속도 향상, 매끄러운 사용자 경험 제공
- SPA 구현 고려 사항
	1. 클라이언트 사이드에서 히스토리를 관리하는 페이지 이동
	2. 비동기로 데이터 받아오기
	3. 뷰 렌더링
	4. 모듈화된 코드 관리
- 라우팅 설정에 따라 URL 별로 특정 컴포넌트를 선택적으로 표시하는 방법으로 페이지 이동 구현
### 4.1.1. Vue Router란 무엇인가
- vue.js 공식 플러그인으로 제공되는 라우팅 라이브러리, SPA에 사용
- 라우팅(페이지 이동 등)관리 담당
- 라우트 정의
	```javascript
	new VueRouter({
	routes: [
		{
			path: '/home',  // /home에 접근 -----> Home 컴포넌트 렌더링
			component: Home
		},
		{
			path: '/about', // /about에 접근 -----> About 컴포넌트 렌더링
			component: About
		},
	]
	})
	```

- Vue Router는 기본적 페이지 이동 기능 외 고급 기능도 제공
	1. 중첩 라우팅
	2. 리다이렉션과 앨리어싱
	3. HTML5 History API와 URL 해시를 이용한 히스토리 관리(IE9에서는 자동으로 폴백)
	4. 자동으로 CSS 클래스가 활성화되는 링크 기능
	5. vue.js 트랜지션 기능을 이용한 페이지 이동 트랜지션
	6. 커스터마이즈된 스크롤링
## 4.2. 기초 라우팅
### 4.2.1. 라우터 설치하기
```javascript
<script src="https://unpkg.com/vue@2.5.17"/>
<script src="https://unpkg.com/vue-router@3.0.1"/>
```
### 4.2.2. 라우팅 설정
- 라우트와 라우터 생성자를 사용
- 라우터: URL과 뷰의 정보를 저장한 레코드
	- 어떤 URL에 대해 어떤 페이지를 표시해야 하는지에 대한 정보
	- 애플리케이션을 구성하는 페이지마다 라우트를 정의, 사용할 라우트를 지정

		→ 해당 라우트가 연결된 페이지로 이동

			(SPA의 경우, 노출 여부 수정)

- Vue Router의 라우트는 vue.js의 컴포넌트를 특정 URL에 대응시킨 객체 형태를 가짐
- 이 객체를 라우터 생성자를 사용해 라우터를 초기화할 때 routes 옵션으로 설정
- 라우트 정의와 라우터 생성자의 예 

	→ 다음과 같이 라우트 정의를 작성, 라우터 생성자에 이 정의를 전달

	→ Vue 인스턴스를 생성할 때 라우팅 설정이 반영

		(어떤 URL에 접근할 때 어떤 컴포넌트를 렌더링해야 하는지 지정됨)

	```javascript
	// 라우트 정의
	{
	path: '/someurl', // url 지정 - 파일명 #/someurl로 접근
	component: {
		template: '...' // 컴포넌트 문법 또는 생성자 기반 컴포넌트 사용
	}
	}

	// 라우터 생성자, new Vue()에 인자로 전달
	new Router({
	routes: [] // 배열로 라우트 정의를 전달
	})
	```

- 전체 예제
	```javascript
	<div id="app">
	  <!--'to' 프로퍼티에 링크 대상을 지정-->
	  <!--<router-link>는 기본적으로 '<a>' 태그로 렌더링됨-->
	  <router-link to="/top">최상위 페이지</router-link>
	  <router-link to="/users">사용자 목록 페이지</router-link>
	  <router-view></router-view>
	</div>

	// vue.js와 Vue Router 로딩
	<script>
	// 라우트 옵션을 지정해 라우터 인스턴스를 생성
	var router = new VueRouter({
	// 컴포넌트를 매핑한 라우트 정의를 배열 형태로 전달
	// 각 라우트에 컴포넌트를 매핑
	// 컴포넌트는 생성자로 만들든지 옵션 객체를 전달해 만들든지 상관없음
	routes: [
	  	{
	    	path: '/top',
	      component: {
	      	template: '<div>최상위 페이지</div>'
	      }
	    },
	    {
	    	path: '/users',
	      component: {
	      	template: '<div>사용자 목록 페이지</div>'
	      }
	    },
	  ]
	})
	// 라우터 인스턴스를 루트 Vue 인스턴스에 전달
	var app = new Vue({
	router: router
	}).$mount('#app')
	</script>
	```
## 4.3. 실용적인 라우팅을 구현하기 위한 기능
### 4.3.1. URL 파라미터를 처리하는 방법과 패턴 매칭
- SPA는 접근 대상 URL의 패턴 매칭을 통해 파라미터를 전달하는 경우가 많음
- 예) 사용자의 상세 페이지를 /user/:userId와 같은 URL로 전달받아, userId를 따라 페이지 구성
- 이 경우, URL 경로에 :을 붙여 패턴 작성
- URL에서 이 패턴과 일치하는 파라미터는 컴포넌트에서 `$route.params`의 속성 중 패턴에 쓰인 파라미터 이름과 같은 속성명을 접근 가능
	```javascript
	var router = new VueRouter({
	routes: [
	  	// 패턴 매칭에 사용되는 패턴은 콜론으로 시작
	  	{
	    	path: '/user/:userId',
	      component: {
	      	template: '<div>사용자 id는 {{ $route.params.userId }} 입니다.</div>'
	      }
	    },
	  ]
	})
	```
### 4.3.2. 이름을 가진 라우트
- Vue Router는 라우트에 이름을 붙여 정의하고 html에서 이 이름을 사용해(<router-link>) 페이지 이동을 수행할 수 있음
- /user/:userId 경로에 user라는 이름을 붙여 라우트를 정의한 예
	```javascript
	var router = new VueRouter({
	routes: [
	  	{
	    	path: '/user/:userId',
		name: 'user', // 이름 붙임
	      component: {
	      	template: '<div>사용자 id는 {{ $route.params.userId }} 입니다.</div>'
	      }
	    },
	  ]
	})
	```

- 위에서 정의한 이름을 가진 라우트를 호출하려면 to 파라미터 지정/URL 패턴 파라미터도 함께 전달
	```javascript
	<router-link :to="{name: 'user', params: {userId: 123}}">
	 사용자 상세 정보 페이지
	</router-link>
	```
### 4.3.4. router.push를 사용한 페이지 이동
- 이전까지의 <router-link>는 선언적 방식
- router.push를 사용해 프로그램적 방식으로 페이지 이동 가능
	```javascript
	router.push({
	name: 'user',
	params: {userId:123}	
	})
	```
### 4.3.5. 훅 함수
- Vue Router는 페이지 이동 전후 시점에 추가 처리를 삽입할 수 있는 훅 함수 제공
- 리다이렉트나 페이지 이동 전 사용자 확인 등을 구현하는 데 사용
- 전역 훅 함수, 라우트 단위 훅 함수, 컴포넌트 내 훅 함수 등 패턴이 있음
	1. 전역 훅 함수
		- 모든 페이지 이동에 설정 가능한 함수
		- router.beforeEach 함수에 훅을 설정하면 페이지 이동 직전 해당 함수 실행
		- 인자: 
			- to: 현재 페이지
			- from: 이동 대상 페이지

			⇒ 2가지 인자에 담긴 라우트는 패턴이 일치한 라우트의 경로나 컴포넌트의 정보를 가짐

			```javascript
			router.beforeEach(function(to, from, next) {
				// 예제: 사용자 목록 페이지로 접속하면 /top으로 리다이렉트
				if (to.path === '/users') {
					next('/top')
				} else {
				// 인자 없이 next를 호출하면 일반적인 페이지 이동(일반적인 라우팅)
					next()
				}
			})
			```

		- 이 훅 함수에서 next를 호출하지 않으면 페이지 이동이 영원히 반복되므로 주의
	2. 라우트 단위 훅 함수
		- 특정 라우트만을 대상으로(per-route) 훅을 추가하려면 Vue Router를 초기화할 때 라우트 정의에서 개별적으로 설정
		- 라우트 정의에 beforeEnter를 작성하면 페이지 이동 전에 실행되는 훅을 추가
			```javascript
			var router = new VueRouter({
				routes: [
			  	{
			    	path: '/users',
			      component: UserList,
			      beforeEnter: function(to, from, next) {
			      	// /user?redirect=true로 접근할 때만 top으로 리다이렉트하는 훅 함수 추가
			        if (to.query.redirect === 'true') {
			        	next('/top')
			        } else {
			        	next()
			        }
			      }
			    },
			  ]
			})
			```

	3. 컴포넌트 내 훅 함수
		- 라우트 뿐만 아니라 컴포넌트에서도(in-component) 훅 함수 정의 가능
		- 컴포넌트 옵션에서 beforeRouteEnter를 정의, 이를 사용해서 데이터를 받아오는 예
			```javascript
			var UserList = {
				template: '#user-list',
			  data: function() {
			  	return {
			    	users: function() {
			      	return []
			      },
			      error: null
			    }
			  },
			  // 페이지는 이동되었으나 컴포넌트가 초기화되기 전에 호출
			  beforeRouteEnter: function(to, from, next) {
			  	getUsers((functon(err, users) {
			    	if (err) {
			      	this.error = err.toString()
			      } else {
			      // next로 전달되는 콜백함수로 자기 자신에 접근한다
			      	next(function(vm) {
			        	vm.users = users
			        })
			      }
			    }).bind(this))
			  }
			}
			```

			- 컴포넌트가 화면에 표시되는 시점에 실행되는 훅인 beforeRouteEnter를 사용
			- 그 외, 다음 페이지 이동으로 인해 컴포넌트가 사라지는 시점에 실행되는 훅인 beforeRouteLeave를 사용할 수도 있음
			- beforeRouteLeave는 저장하지 않은 수정사항 등의 유실을 사용자에게 확인받기 위한 기능을 구현하는 데도 사용
## 4.4 예제 애플리케이션 구현
### 4.4.1. 리스트 페이지 구현하기
- 생략
### 4.4.2. API와 통신하기
- SPA는 페이지를 이동할 때마다 API를 통해 받아온 데이터를 UI에 표시하는 처리를 자주 수행
- Vue Router를 사용해 AJAX로 데이터를 받아오는 경우, vue.js 컴포넌트의 created와 watch 속성을 사용해 구현하는 것이 일반적
- watch: 계산 프로퍼티를 일반적인 용도로 사용할 수 있게 만든 vue 컴포넌트 옵션 속성
- 예제) $route 모니터링, 라우트에 변경이 생길 때마다 watch로 변경 여부 탐지
	```javascript
	<script type="text/x-template" id="user-list">
	<div>
	  	<div class="loading" v-if="loading">로딩 중..</div>
	    <div v-if="error" class="error">
	    	{{ error }}
	    </div>
	    <!--users의 로딩이 끝나면 각 사용자의 이름을 표시-->
	    <div v-for="user in users" :key="user.id">
	    	<h2>{{ user.name }}</h2>
	    </div>
	  </div>
	</script>

	<script>
	// json을 반환하는 함수
	// 이 함수를 사용해 가상의 web api를 통해 정보를 받아옴
	var getUsers = function(callback) {
	setTimeout(function() {
	  	callback(null, [
	    	{
	      	id: 1,
	        name: 'Takuya Tejima'
	      },
	    	{
	      	id: 2,
	        name: 'yohei'
	      },
	    ])
	  }, 1000)
	}

	// UserList를 수정
	var UserList = {
	// html에 있는 script 태그의 id를 지정
	  template: '#user-list',
	  data: function() {
	  	return {
	    	loading: false,
	      users: function() { return [] }, // 초기값은 빈 배열
	      error: null
	    }
	  },

	  // 초기화할 때 데이터를 받아 옴
	  created: function() {
	  	this.fetchData()
	  },

	  // $route의 변경을 모니터링하며 라우팅이 수정되면 데이터를 다시 받아옴
	  watch: {
	  	'$route': 'fetchData'
	  },

	  methods: {
	  	fetchData: function() {
	    	this.loading = true
	      // 받아 온 데이터를 users에 저장
	      // Function.prototype.bind는 this의 유효범위를 전달하기 위해 사용
	      getUsers((function(err, users) {
	      	this.loading = false
	        if (err) {
	        	this.error = err.toString()
	        } else {
	        	this.users = users
	        }
	      }).bind(this))
	    }
	  }
	}
	</script>
	```
### 4.4.3. 상세 정보 페이지 구현하기
- 생략
### 4.4.4. 사용자 등록 페이지 구현하기
- 생략
### 4.4.5. 로그인/로그아웃 구현하기
- 생략
### 4.4.6. 예제 애플리케이션 전체 코드
- [https://jsfiddle.net/flourscent/wp2h5arj](https://jsfiddle.net/flourscent/wp2h5arj)
## 4.5. Vue Router의 고급 기능
### 4.5.1. Router 인스턴스와 Route 객체
- 페이지 이동이나 모니터링에 사용했던 $router와 $route는 전혀 다름
- this.$router.push: 컴포넌트에서 Router 인스턴스에 접근하는 코드
	- $router는 Router 인스턴스를 가리킴
	- Router 인스턴스: 웹 애플리케이션 전체에서 딱 하나만 존재하는 것/전반적 라우터 기능을 관리
	- 애플리케이션 전체에서 히스토리를 어떻게 관리할지에 대한 설정, 
	- router-link 요소 없이 프로그램적인 방법으로 페이지를 이동할 때 사용
	- Router 객체의 주요 프로퍼티와 메서드

| 프로퍼티/메서드명 | 설명 |
|---|---|
| app | 라우터를 사용하는 루트 Vue 인스턴스 |
| mode | 라우터 모드(히스토리 관리와 함께 설명함) |
| currentRoute | 현재 라우트에 대한 Route 객체 |
| push(location, onCOmplete?, onAbort?) | 페이지 이동 실행. 히스토리에 새 엔트리를 추가하고 브라우저에서 뒤로가기 버튼이 눌리면 앞의 URL로 돌아감. |
| replace(location, onCompete?, onAbort?) | 페이지 이동 실행. 히스토리에 새 엔트리 추가하지 않음. |
| go(n) | 히스토리 단계에서 n단계 이동. window.history.go(n)과 비슷함. |
| back() | 히스토리에서 한 단계 돌아감. history.back()과 같음. |
| forward() | 히스토리에서 한 단계 앞으로 나아감. |
| addRoutes(routes) | 라우터에 동적으로 라우트를 추가. |

- this.$route.params 등의 코드에 나오는 $route는 Route 객체
	- 페이지 이동 등으로 라우팅이 발생할 때마다 생성됨
	- 활성화된 라우트의 상태를 저장한 객체로, 현재 경로 및 URL 파라미터 등의 정보를 이 객체에서 받음
	- 컴포넌트 내부에 구현된 Router 훅 함수 등을 통해서도 참조 가능
	- watch에서 모니터링하기도 함
	- Route 객체의 주요 프로퍼티

| 프로퍼티 | 설명 |
|---|---|
| path | 현재 라우트의 경로를 나타내는 문자열. |
| params | 정의된 URL 패턴과 일치하는 파라미터의 키-값 쌍을 담고 있는 객체. 파라미터가 없다면 빈 객체. |
| query | 쿼리 문자열의 키-값 쌍을 담고 있는 객체. 쿼리가 없다면 빈 객체. 경로가 /foo?user=1이면 $route.query.user === 1이 됨. |
| hash | 현재 URL에 URL 해시가 있을 경우 라우트의 해시값을 갖는다. 해시가 없다면 빈 객체. |
| fullPath | 쿼리 및 해시를 포함하는 전체 URL. |
| name | 이름을 가진 라우트의 경우 라우트의 이름. |

### 4.5.2. 중첩 라우팅
- 애플리케이션이 복잡해짐에 따라 중첩 라우팅이 필요해지는 경우가 있음
- Vue Router의 중첩 라우팅은 어떤 컴포넌트 안에 든 컴포넌트에 대한 라우트 정의를 말함
- 예) 페이지 내용은 `/user/userId`를 기본으로 하되, `/user/userId/posts`면 포스트 정보를 노출하고 `/user/userId/profile`이면 프로필 정보를 노출하는 정의
	```javascript
	<script type="text/x-template" id="user-test">
	<div class="user">
	  	<h2>사용자 id는 {{ $route.params.userId }} 입니다.</h2>
	    <router-link :to=`${ '/user' + $router.params.userId + '/profile' }`>사용자 프로필 보기</router-link>
	    <router-link :to=`${ '/profile' + $router.params.userId + '/posts' }`>사용자 포스트 보기</router-link>
	    <router-view></router-view>
	  </div>
	</script>

	<script type="text/x-template" id="user-profile">
	<div class="user-profile">
	  	<h3>사용자 {{ $route.params.userId }} 의 프로필 페이지입니다.</h3>
	  </div>
	</script>

	<script type="text/x-template" id="user-posts">
	<div class="user-posts">
	  	<h3>사용자 {{ $route.params.userId }} 의 포스트 페이지입니다.</h3>
	  </div>
	</script>

	<script>
	// 사용자 상세 정보 페이지 컴포넌트 정의
	var User = {
	template: '#user-test'
	}

	// 사용자 상제 정보 페이지에 부분적으로 나오는 사용자 프로필 페이지
	var UserProfile = {
	template: '#user-profile'
	}

	// 사용자 상제 정보 페이지에 부분적으로 나오는 사용자 글 모음 페이지
	var UserPosts = {
	template: '#user-posts'
	}

	var router = new VueRouter({
	routes: [
	  	{
	    	path: '/user/:userId',
	      name: 'user',
	      component: User,
	      children: [
	      	{
	        	// /user/:userId/profile와 일치한 경우
	          // UserProfile 컴포넌트는 User 컴포넌트의 <router-view> 안에서 렌더링됨
	          path: 'profile',
	          component: UserProfile,
	        },
	        {
	        	// /user/:userId/posts와 일치한 경우
	          // UserPosts 컴포넌트는 User 컴포넌트의 <router-view> 안에서 렌더링됨
	          path: 'posts',
	          component: UserPosts
	        }
	      ]
	    }
	  ]
	})
	</script>
	```
### 4.5.3. 리다이렉션과 앨리어싱
- SPA에서도 일반적인 웹 애플리케이션과 마찬가지로 리다이렉션 기능을 사용해야 하는 경우가 있음
- Vue Router는 URL을 바꿔주는 리다이렉션과 URL 수정 없이 라우팅 처리만 해 주는 앨리어싱 기능을 제공
1. 리다이렉션
	- /a에 접근하면 /b를 연결
	- * 사용 시 현재 정의된 모든 라우트와 일치하지 않았을 때, 리다이렉션 대상 지정 가능
	- Not Found 페이지 만들 시 유용
		```javascript
		var router = new VueRouter({
			routes: [
		{ path: '/a', redirect: '/b' },
		{ path: '/b', component: B },
		{ path: '/notfound', component: NotFound },
		// 현재 URL이 어떤 라우트와도 일치하지 않으면 /notfound로 이동
		{ path: '*', redirect: '/notfound' },
			]
		})
		```

2. 앨리어싱
	- 접근한 URL은 그대로 두고 다른 라우트에서 정의한 페이지로 이동
	- 예) /b에 접근했을 때 URL은 /b에 그대로 두고, 컴포넌트는 A를 렌더링해서 마치 /a에 접근한 것처럼 보이게 함
	- 앨리어싱은 하나 이상 지정 가능
		```javascript
		var router = new VueRouter({
			routes: [
		  	{ path: '/a', component: A, alias: '/b' },
		  	{ path: '/c', component: C, alias: ['/d', '/e'] },
		  ]
		})
		```
### 4.5.4. 히스토리 관리
- SPA는 서버사이드의 라우팅을 거치지 않기 때문에 브라우저 히스토리 역시 클라이언트에서 관리
	1. URL 해시를 사용하는 방법
		- URL 끝에 #를 붙여 라우팅 경로를 관리
		- Vue Router는 기본적으로 URL 해시로 동작
		- 클라이언트 쪽에서 URL이 변화하기 때문에 브라우저 히스토리에는 URL이 각각 추가
		- 브라우저에서 뒤로 가기/앞으로 가기 버튼을 누르면 내부적으로는 hashchange 이벤트로 라우팅 변경 시와 같은 처리가 일어남
		- 사용자가 직접 브라우저에 URL을 입력해 접근해도 따로 특별한 처리를 하지 않음
	2. HTML5 History API를 사용하는 방법
		- HTML5 History API로 히스토리 스택 조작
		- #/을 붙이지 않아도 일반적인 서버사이드 라우팅과 같은 방식의 URL 사용 가능
		- 사용자가 브라우저에 직접 URL을 입력해 접근한 경우 적절한 SPA 페이지를 반환하도록 하는 처리 필요
		- Vue Router 인스턴스를 만들 때 옵션 객체의 mode 속성을 'history'로 설정하면 사용 가능
- Vue Router를 사용한 대규모 애플리케이션 구현
	- 여러 컴포넌트 간 데이터 관리가 복잡해지는 것을 방지하기 위해 Vuex 사용 권장
	- Vue Router 기반 애플리케이션에서 vuex-router-sync 사용 시 vuex 연동 가능
	- 현재 라우팅 정보를 vuex에서 관리, 컴포넌트와 라우팅 상태를 한 곳에서 관리 가능
	- 애플리케이션 특성 및 설계 정책에 따라 Vue Router와 Vuex 중 하나/혹은 둘 다 적용하는 등 적절한 판단 필요
	- 애플리케이션 특징에 따른 플러그인 조합 적합성 판단 기준
		1. Vue Router와 Vuex 모두 사용할 필요 없는 애플리케이션의 예
			- 기존 전자상거래 사이트
			- 라우팅은 서버 사이드에서 수행, 클라이언트 사이드도 컴포넌트로 구성되지 않은 애플리케이션
			- 서비스 소개 사이트 등 일부 컴포넌트가 동적으로 동작하는 랜딩 페이지
		2. Vue Router만 적용하면 되는 애플리케이션의 예
			- SPA 기반 관리 화면 등 각 페이지에서 간단한 기능을 제공하는 애플리케이션
			- 네이티브 애플리케이션처럼 경쾌하게 동작하는 클라이언트 페이지 이동을 제공하는 애플리케이션 혹은 게임
		3. Vuex만 적용하면 되는 애플리케이션의 예
			- 대시보드, 채팅 애플리케이션, 사진 가공 애플리케이션 등 한 페이지 안에서 여러 컴포넌트 간의 데이터 연동이 필요한 단일 도구 형태의 애플리케이션
		4. Vue Router와 Vuex 모두 적용해야 하는 애플리케이션의 예
			- 메일 클라이언트나 캘린더 클라이언트 등 여러 페이지에서 복잡한 컴포넌트 구성이 예상되는 대규모 SPA
- Vue Router와 React Router
	- 프론트엔드 라이브러리 React에도 React Router라는 라우팅 라이브러리 존재
