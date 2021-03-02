> https://alexkondov.com/tao-of-react/

컴포넌트
- 가능한 Class Components 보다는 Functional Components 를
  + 불필요한 보일러 플레이트 코드도 없어지고 가독성이 좋아짐
- 일관된 컴포넌트 작성
  + 컴포넌트 작성 컨벤션을 유지할것. 동일한 곳에 헬퍼 함수를 사용하고 동일한 방식으로 export 하고 이관적인 네이밍 규칙을 따르고 등등..
- 가능한 Name Components
  + 디버깅에 용이함
- 헬퍼 함수들 구성하기
  + 컴포넌트의 클로저를 가지고 있을 필요가 없는 함수는 컴포넌트 정의로 부터 헬퍼함수로 분리해서 컴포넌트 앞에 위치하여 위에서 아래로 읽히도록 하기. 컴포넌트내 로직에 집중할 수 있도록 하기
- 마크업 하드코딩 하지 않기
- 컴포넌트 길이
  + 너무 많은 기능을 하여 복잡하다면 적절히 추출하여 적절한 길이를 유지하도록 하기. 가독성
- JSX 안에서 주석 활용하기
- Error Boundaries 사용
  + 가능한 컴포넌트 내에서 적절히 에러를 처리해 특정 컴포넌트가 에러가 발생해도 전체 페이지가 사용이 불가능하도록 만들지 말라는..
- Destructure Props
  + 보통의 경우엔 Destructure Props 를 사용하는게 이점이 있음
- Number of Props
  + 보통 프로퍼티가 5개 넘으면 악취, 컴포넌트 분리를 고려할것
  + 컴포넌트가 많은 프로퍼티를 받을 수록 다시 렌더링 될 여지가 더 많이 생긴다는걸 명심
- Pass Objects Instead of Primitives
  + 개별의 기본 타입 값들을 프로퍼티로 전달하는 대신 객체로 전달
- Move Lists in a Separate Component
  + 리스트를 디스플레이하는 책임만을 갖는 컴포넌트를 별도로 분리하자
  + 정제된 리스트를 프로퍼티로 주입받아 순수하게 리스트를 렌더링하는 책임만 가지도록 해 읽기 쉬운 코드를 만듬
- Assign Default Props When Destructuring
  + 프로퍼티를 비구조할당할때 기본값을 부여할 것

상태관리
- Reducer 사용
- 고차함수나 렌더 프로퍼티보다 훅을 우선 고려
- 데이터 fetch 라이브러리 사용
- 상태 관리 라이브러리 사용

컴포넌트 기본 모델
- Container & Presentational
  + 컨테이너는 비즈니스로직과 데이터 fetching, 상태 관리를 갖고
  + 컴포넌트는 props 로 주입받은걸 순수하게 렌더링하기만 함
  + 그러나 현대의 UI 어플리케이션에서는 이러한 패턴이 한계를 가짐
  + 몇 가지 컴포넌트에서 로직을 끌어올리면
  + 너무 많은 책임을 지게 되고 관리하기 어려워짐
  + 앱이 성장함에 따라 몇 군데 집중된 장소에 복잡성을 두는건 유지관리성 측면에서 좋지 않음
- Stateless & Stateful
  + 상태를 갖는게 어떤 컴포넌트의 책임일지 신중히 생각하고 결정하라는 뭐 그런

애플리케이션 설계
- Group by Route/Module
  + 디렉토리를 container와 component로 나누는건 보기 어렵다.
  + 모듈과 도메인으로 분류하여 패키징할 것
- Create a Common Module
  + 범용적으로 사용될 수 있는 버튼, 카드, 인풋등과 같은 컴포넌트를 별도의 공통 모듈로 분리하여 관리
  + 아토믹 컴포넌트.. 중복을 막는 첫번째 걸음이다.
- Use Absolute Paths
  + 상대경로보단 절대경로를 사용하는 것이 추후 변경에 유연하게 대응하는데 용이하다.
- Wrap External Components
  + 3rd party 라이브러리를 직접 import 하지 말것
  + 어답터를 만들어 관리하면 외부 라이브러리의 변경을 한 지점에서 대응할 수 있다.
  + 나도 참 열심히 미는 패턴
- Move Components in Folders

성능
- Don't Optimize Prematurely
- Watch The Bundle Size
- Rerenders - Callbacks, Arrays and Objects

테스팅
