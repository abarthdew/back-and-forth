# DevTools Network 탭으로 HTTP 요청 읽기

Chrome 개발자 도구(DevTools) Network 탭에서 요청 하나를 클릭하면 보이는 항목을 key별로 정리한다.
예시는 실제로 캡처한 요청 2건이다.

- **A**: Jira에서 이슈 본문을 저장할 때 나가는 GraphQL 요청 (`POST`)
- **B**: GitHub 이슈에 이미지를 첨부할 때 나가는 업로드 확정 요청 (`PUT`)

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
| Referrer policy | `strict-origin-when-cross-origin` | 이 요청에 `referer` 헤더를 얼마나 담을지 정한 정책. 같은 출처면 전체 URL, 다른 출처면 origin만, HTTPS→HTTP면 보내지 않는다 |

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

## 참고

- [MDN: HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [MDN: Fetch metadata request header](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_metadata_request_header)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN: Server-Timing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)
- [Chrome DevTools: Network features reference](https://developer.chrome.com/docs/devtools/network/reference)
