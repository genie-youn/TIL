# Node와 Process

### 들어가며
Node 환경에서 개발을 하다 보면  `process`란 객체를 쓸 일이 왕왕 생긴다. 특히 `process.NODE_ENV` 같은 아이들..
런타임 혹은 빌드타임에 무언과 실행환경과 관련된 값이 필요하면 보통 이 객체 안에 있으니 가져다 쓰겠는데, 이 아이는 왜 있는건지 언제부터 있던건지 누가 생성해주는것인지 궁금해져 정리하려 한다.

> 레퍼런스는 [다음](https://nodejs.org/api/process.html) 을 참고한다.

### Node에서의 Process
`process` 객체는 현재 Node.js 프로세스의 정보와 제어를 가능하게 하는 글로벌 객체이다. 모든 Node.js 어플리케이션이라면 접근 가능하다. (r명시적으로 `require`를 통해 접근할 지는 선택의 자유이다. 즉 없이도 접근 가능하다.)

한가지 신기한건 이 `process` 객체가 Node의 `EventEmitter` 타입의 인스턴스라는것.

다음과 같은 것들을 할 수 있다.

#### 라이프 사이클
프로세스의 라이프사이클을 감지하여 특정한 훅을 받아 실행할 수 있다.
```javascript
process.on('{hook}}', (code) => {
  console.log('do something: ', code);
});
```

감지할 수 있는 훅 (이벤트)은 다음과 같다.

- beforeExit: Node의 이벤트루프가 비어있고 더 이상 추가할 작업이 없을 때
  + `process.exit()`이나 처리되지 않은 예외로 인한 종료등 명시적인 종료에 의해 애플리케이션이 종료될 때는 **호출되지 않음** 에 유의한다.
  + `beforeExit`는 특정한 작업을 추가하려는 의도가 아니고, 단순히 `exit` 대신 사용하면 안된다.
  > 왜지

- disconnect: `Child Process`나 `Cluster Mode`로 동작시 생성된 IPC 채널이 종료되었을 때
- exit: 다음과 같은 이유로 Node의 프로세스가 종료될 때 호출됨
  + `process.exit()`가 명시적으로 호출되었을 때
  + Node의 이벤트 루프가 비어있고 더 이상 추가할 작업이 없을 때
  + 이 시점에서 이벤트 루프가 종료되는 것을 막을 방법은 없으며, 모든 `exit` 이벤트에 대한 리스너가 실행되고나면 Node의 프로세스는 종료된다.
  + `exit`의 이벤트 리스너가 모두 실행되고 나면 이벤트루프에 다른 태스크가 등록되어도 실행하지 않고 Node의 프로세스가 종료되기 때문에 리스너의 콜백함수는 **무조건** 동기적으로 실행되어야 함.
> 그럼 process.exit()를 하면 beforeExit은 호출안되고 exit만 호출?

- message: `Child Process`나 `Cluster Mode` 로 동작시 부모 프로세스로 부터 메세지를 받았을 때
- multipleResolves: `Promise` 가 다음중 하나의 일을 했을 때 마다
  + 한번이상 Resolved 되었을 때
  + 한번이상 Rejected 되었을 때
  + Resolved 된 이후 Rejected 되었을 때
  + Rejected 된 이후 Resolved 되었을 때
  + 잠재적인 버그를 찾는데 도움이 되지만, 감지된다고 무조건 버그라고 할 수는 없다. 예를들어 `Promise.race()` 는 `multipleResolves` 이벤트를 발생시킴
