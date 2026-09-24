# 쿠키, 캐시, 세션
- 쿠키: 브라우저에 저장되는 작은 데이터
  - 저장 위치: 브라우저
  - "브라우저가 들고 다니는 메모지"
  - 자동으로 매 요청마다 서버에 딸려감 / 만료시간 설정 가능
  - 용도: 로그인 토큰, 설정값(다크모드, 언어), 트래킹 id
  ```js
  // server
  Set-Cookie: sessionId=abc123
  // browser
  요청 보낼 때마다 Cookie: sessionId=abc123
  ```
  
- 캐시: 다시 안 받아오려고 임시 저장해두는 것
  - 저장 위치: 브라우저, 서버, cdn, 메모리 등
  - "다시 계산/다운로드 안 하려고 저장"
  - 이미지, js, css 다시 안 받기 / api 응답 다시 계산 안 하기 / db 조회 결과 다시 안 가져오기
  - 용도: 성능 최적화용, 사용자 상태랑 직접 관계 없음

- 세션: '로그인한 사용자 상태'를 서버가 기억하는 것
  - 저장 위치: 서버
  - "서버가 사용자마다 만들어주는 개인 서랍"
  - 진짜 데이터는 서버에 존재 / 브라우저는 세션 id만 쿠키로 들고 다님
  - 서버 재시작하면 휘발(보통)
  - 용도: 로그인 구현(전통적 방식)
  ```js
  // browser
  Cookie = JSESSIONID=xyz
  // server
  Map<"xyz", UserSessionObject>
  ```
  
- 관계:
  ```
  [브라우저]
  ├─ Cookie (세션ID, 토큰, 설정값)
  ├─ Cache (이미지, JS, CSS)
  ↓ 요청
  [서버]
  ├─ Session (로그인 정보, 장바구니)
  ├─ Cache (쿼리 결과, 계산 결과)

  // “로그인은 세션 기반이고, 세션 키는 쿠키로 들고 다닌다. 캐시는 성능 때문에 여기저기 쓴다.”
  브라우저 (쿠키) ──[sessionId=abc]──▶ 서버
  서버 (세션) ──[abc → { userId=1, role=ADMIN }] 
  ```
  
- 로그인 예시
  - 세션
    1. 로그인 성공
    2. 서버: 세션 생성 -> sessionId = "abc:
    3. 서버 -> 브라우저: Set-Cookie: JSESSIONID=abc
    4. 이후 요청:
      ```js
        브라우저 → Cookie: JSESSIONID=abc
        서버 → 세션 조회 → "아 이 사용자구나"
      ```
  - jwt
    1. 로그인 성공
    2. 서버: jwt 발급
    3. 브라우저: cookie에 저장 or localStorage에 저장
    4. 이후 요청: Authorization: Bearer jwt...

# SSL, TLS (HTTPS), CSRF, XSS, CORS
- ssl, tls: 통신 암호화
  - 역할: 중간에서 패킷 훔쳐봐도 내용 못 봄, 인증서로 서버 신원 확인
- csrf: 로그인된 사용자를 속여서 요청 보내게 하는 공격
  - 예시: 로그인 상태에서 악성 사이트 접속 -> 몰래 송금 api 호출
  - 방어: csrf 토큰, SameSite 쿠키, Spring Security의 기본 방어 기능 사용
- XSS: 스크립트 주입 공격
  - 방어: html escape, csp
- CORS: 브라우저가 막는 출처 다른 요청
  - 에러: Blocked by CORS policy
 
# session / cookie, JWT, Reverse Proxy, Load Balancer, Timeout
- Cookie / Session
  - Cookie: 브라우저 저장 / Session: 서버 저장(또는 Redis)
- JWT: 세션 없는 인증 토큰
  - 장점: 서버 상태 안 가짐 / 단점: 로그아웃 어려움, 토큰 탈취 시 위험
- Reverse Proxy: nginx / apache 역할
  - 하는 일: ssl 처리, 정적 파일 처리, was 앞단 보호, 로드밸런싱
- Load Balancer: 트래픽 분산기
  - L4 / L7, Sticky session, 헬스체크
- Timeout: 운영 장애 원인 TOP3
  - 종류: Client timeout, nginx timeout, tomcat timeout, DB timeout

# Connection Pool, Thread Pool, CDN, SSE / WebSocket, HTTP 상태코드
- Connection Pool: DB 커넥션 재사용 풀 (max pool size, idle, timeout)
- Thread Pool: Tomcat 워커 스레드 풀
- Cache: Redis / Local cache: DB 부하 감소, 응답속도 개선
- CDN: 정적 파일 배달 전용 서버
- SSE / WebSocket: 실시간 통신
- HTTP 상태코드
  - 200 (OK), 301/302 (Redirect), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 500 (Internal Server Error), 502/504 (프록시/was 문제)

# SSR, CSR, SPA
### 어디서 랜더링 하나?
- SSR: 서버가 HTML 완성해서 보내줌 (예전 JSP, Thymeleaf, Freemarker) -> SSR 기반 MPA 에서는 서버에서 HTML 받아올 때 화면이 비기 때문에, 다시 그려지며 페이지 깜빡거림 현상 있음
- CSR: 서버는 JSON(데이터)만, 화면은 브라우저가 그림

### 페이지 구조가 어떤가?
- SPA: 페이지 이동 없이 CSR로 화면 갈아끼우는 앱 (페이지 하나짜리 웹앱: React, Vue, Angular) -> 대부분 csr 기반
- MPA: Multi Page Application

### 조합
- `SPA + (CSR | SSR) / MPA + (CSR | SSR)`











