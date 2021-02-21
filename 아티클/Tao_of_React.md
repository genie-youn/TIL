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
- Use Reducers
  + 
