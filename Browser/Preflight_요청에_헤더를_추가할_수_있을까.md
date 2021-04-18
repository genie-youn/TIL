# Preflight 요청에 헤더를 추가할 수 있을까?

서버에서 특정 헤더가 존재하지 않으면 에러 응답을 주는 스펙이 추가 되었고,
이미 해당 헤더를 포함하여 요청하고 있었기 때문에 서비스에 별다른 영향이 없어야함에도 불구하고 모든 요청이 실패하는 일이 있었다.

확인해보니 CORS를 위한 Preflight 요청은 해당 헤더가 포함되지 않은채 전송되고 있었고 이 요청에 대한 응답이 에러로 내려오면서 CORS에 위배되어 이후 요청들이 실패하게 된 것이다.

해당 헤더는 Axios의 인터셉터를 통해 추가되도록 되어 있기 때문에 모든 요청에 포함되어 전달될것이라 기대했지만, Preflight 요청에는 추가되지 않았고 Preflight 요청은 헤더를 추가하는등의 제어를 할 수 없나? 하는 의구심이 들었다.

결론부터 얘기하자면 CORS-preflight fetch 는 온전히 브라우저가 제어하는 영역이므로 사용자가 별도의 헤더를 추가하는등의 행동을 할 수 없다.

https://fetch.spec.whatwg.org/#cors-preflight-fetch

*request* request가 주어지면, 다음과 같은 단계로 **CORS-preflight fetch** 가 이루어진다.

1. *preflight* 는
```markdown
{
  method: `OPTIONS`,
  URL: *request* 의 current URL,
  initiator: *request* 의 initiator,
  destination: *request* 의 destination,
  origin: *request* 의 origin,
  referrer: *request* 의 referrer,
  referrer policy: *request* 의 referrer policy,
  mode: "cors",
  tainted origin flag: *request* 의 tainted origin flag,
  response tainting: "cors"
}
````
로 이루어진 새로운 request가 된다.

> 이 알리고리즘은 HTTP fetch가 아닌 HTTP-network-or-cache fetch 를 사용하므로 service-workers mode의 preflight는 문제가 되지 않는다??

2. *preflight* 의 header list에 `Accept` 헤더의 값으로 `*/*` 를 추가한다.
3. *preflight* 의 header list에 `Access-Control-Request-Mehtod` 헤더의 값으로 *request* 의 method 를 추가한다.
4. *headers* 는 *request* 의 header list 를 **CORS-unsafe request-header names** 로 변환한 값이 된다.
5. 만약 *headers* 가 비어있지 않다면,
  - *value* 는 *headers* 를 `,` 로 나눈 각각의 요소가 된다.
  - *preflight* 의 header list에 `Access-Control-Request-Headers` 헤더의 값으로 *value* 를 추가한다.
6. *response* 는 *preflight* 를 request 로 한 HTTP-network-or-cache fetch를 실행시킨 결과가 된다.
7. 만약 *request* 와 *response* 에 대한 CORS check 가 성공하고 *response* 의 status 가 ok status 라면,

> preflight 가 정확한 credentials mode 를 사용하는지보다 CORS check가 먼저 이루어진다.

  1. *methods* 는 `Access-Control-Allow-Methods` 의 extracting header list values 와 *response* 의 header list의 결과가 된다. ??
  2. *headerNames* 는 `Access-Control-Allow-Methods` 의 extracting header list values 와 *response* 의 header list의 결과가 된다. ??
  3. *methods* 나 *headerNames* 가 실패한다면 network error 를 반환한다.
  4. *methods* 가 null 이고 *request* 의 use-CORS-preflight-flag 가 설정되어 있으면 *methods* 를 *request* 의 method 를 포함하는 새로운 리스트로 설정한다.
  5. 만약 *request* 의 method 가 *methods* 에 존재하지 않으면 *request* 의 method 가 CORS-safelisted mehtod가 아니고 *request* 의 credentials mode가 "include" 또는 *methods* 가 `*` 를 포함하고 있지 않다면 network error를 반환한다... ?
  6. 만약 request의 header list의 이름들 중 하나가 CORS non-wildcard request-header name 이고 *headerNames* 의 항목과 byte-case-insensitive match 하지 않으면 network error를 반환한다.
  8. *max-age* 는 *response* 의 헤더 리스트 중 `Access-Control-Max-Age` 의 extracting header list values 의 결과가 된다.
  9. 만약 *max-age* 가 실패 혹은 null 이라면 max-age를 5로 설정한다.
  10. 만약 *max-age* 가 max-age 에 부여된 값보다 큰 경우 `max-age` 를 부여된 값으로 설정한다.
  11. 만약 user agent 가 cache 를 제공하지 않으면 *response* 를 반환
  12. *methods* 의 각각의 *method*
