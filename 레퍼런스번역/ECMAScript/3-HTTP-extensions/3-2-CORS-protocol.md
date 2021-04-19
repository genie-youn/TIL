# 3.2 CORS protocol

서로 다른 출처 (origin) 의 응답을 공유하거나 HTML의 form 요소에서 가능한 것 보다 더 다양한 `fetchs` 를 활용하기 위해 CORS protocol 은 존재한다. HTTP 위에 계층화 되어 있으며 응답이 다른 origins 에 공유될 수 있음을 선언한다.

> 그럼 form 에서 발생하는 요청은 CORS를 안탄다는건가

> 방화벽 뒤로부터 (인트라넷) 오는 응답의 데이터가 유출되는것을 막기 위해선 opt-in 메커니즘이 필요하다. 또한 credentials 를 포함하는 requests 를 위해 잠재적으로 민감한 데이터의 유출을 막기 위해서도 opt-in 메커니즘이 필요하다. opt-in 메커니즘?

### 3.2.1 General
CORS protocol 은 응답이 서로 다른 출처에서 공유되어질 수 있는지를 나타내는 헤더들의 집합으로 구성된다.
