# Preflight 요청에 헤더를 추가할 수 있을까?

서버에서 특정 헤더가 존재하지 않으면 에러 응답을 주는 스펙이 추가 되었고,
이미 해당 헤더를 포함하여 요청하고 있었기 때문에 서비스에 별다른 영향이 없어야함에도 불구하고 모든 요청이 실패하는 일이 있었다.

확인해보니 CORS를 위한 Preflight 요청은 해당 헤더가 포함되지 않은채 전송되고 있었고 이 요청에 대한 응답이 에러로 내려오면서 CORS에 위배되어 이후 요청들이 실패하게 된 것이다.

해당 헤더는 Axios의 인터셉터를 통해 추가되도록 되어 있기 때문에 모든 요청에 포함되어 전달될것이라 기대했지만, Preflight 요청에는 추가되지 않았고 Preflight 요청은 헤더를 추가하는등의 제어를 할 수 없나? 하는 의구심이 들었다.

결론부터 얘기하자면 CORS-preflight fetch 는 온전히 브라우저가 제어하는 영역이므로 사용자가 별도의 헤더를 추가하는등의 행동을 할 수 없다.

https://fetch.spec.whatwg.org/#cors-preflight-fetch

*request* request가 주어지면, 다음과 같은 단계로 **CORS-preflight fetch** 가 이루어진다.

1. *preflight* 는
{
  method: `OPTIONS`,
  URL: *request* 의 current URL,
  initiator: *request* 의 initiator,
  destination: *request* 의 destination
}
